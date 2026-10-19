## XPC and mach-ports

XPC is Apple's library for sending messages between two processes on the same machine. The usual case is an app asking a daemon for something. Here an app asks expd (the experimentation daemon) which variant it's in: it sends the dictionary `{"experiment": "new_onboarding"}` and gets back `{"variant": "B"}`. It does the job a UDS does in the [sockets note](Sockets%20and%20kqueue.md#unix-domain-sockets), but only on Apple platforms.

What XPC does for you that you'd write yourself with a UDS:

1. Find: the client names the server with a string like `com.example.expd` instead of a socket path.
2. Start: launchd starts expd when the first message arrives, and again after a crash.
3. Encode: messages are typed dictionaries, so there's no framing or parsing.
4. Reply: each request's reply goes to that request's own handler, so there are no request IDs to match.
5. Report: if the other side dies, you get an error event instead of a hang.
6. Check: expd can require callers to have an entitlement before it hears from them.

The note follows one request from start to end (How it works), then shows the code for each side (Using XPC in code), then covers errors, security, and how XPC compares with a UDS.

## How it works

```
  client app                          kernel + launchd                       expd

  xpc_connection_send_message_with_reply()
        │  XPC encodes the dictionary into a Mach message
        ▼
  mach_msg() ──────────────▶ message waits on expd's mach-port
                                     │  expd not running? launchd sees
                                     │  the message and starts it
                                     ▼
                             libdispatch in expd wakes ──────────▶ your handler gets the dictionary
                                                                     │  builds a reply, sends it
  your reply handler runs ◀── kernel: message on the reply port ◀────┘
```

The sections below follow the diagram in order: the mach-port the message travels on, launchd finding and starting expd, the reply coming back, and libdispatch running the handler on each side.

### Mach-ports: what the message travels on

A mach-port is a queue of messages inside the kernel. It's the channel underneath XPC, the way a socket is the channel underneath a UDS connection. Two things differ from a stream UDS:

- Whole messages: each send puts one message on the queue and each receive takes one whole message off, so there's no length prefix to add.
- One way: a mach-port has exactly one receiver, and everyone else can only send to it. So requests go to expd's mach-port, and replies need a second mach-port that the client receives on.

A process never holds a mach-port itself. It holds a right to one and refers to it by a small number, like `0x1103`, the way it refers to a socket by fd `3`. The kernel keeps each process's rights in a table, like the fd table, so the same mach-port has a different number in each process. There are three kinds of right:

- Receive right: take messages off the queue. Exactly one process holds it, and that process owns the mach-port. expd holds the receive right for its mach-port.
- Send right: put messages on the queue. Any number of processes can hold one. Every app talking to expd holds a send right to expd's mach-port.
- Send-once right: a send right that's used up by one message. Replies use it.

Holding the right is the permission. There's no separate check.

A right can also travel inside a message. The kernel takes it out of the sender's table and adds it to the receiver's under a new number, like the client's `0x1103` showing up as `0x2207` in expd. A raw number means nothing in another process, so this is the only way to hand over a right. It's how the client gives expd a way to reply, and how XPC sends an open fd.

When the process with the receive right exits, the kernel destroys the mach-port and tells everyone holding a send right to it. XPC turns that into an error event (see When things go wrong).

The syscall underneath is `mach_msg()`, which sends or receives one message. XPC calls it for you. App code almost never uses raw mach-ports: you'd lay out every message's bytes by hand, and you'd have to give back every right by hand. A forgotten right shows up as a hang, not an error.

### launchd: finding expd and starting it

A mach-port has no path, so a client can't find one in the filesystem. It gets its first send right from launchd, which keeps a table of service names and their mach-ports. Every process starts out holding a send right to launchd, called its bootstrap port, so it can always ask.

1. Register: expd's plist lists `com.example.expd` under `MachServices` (see The plist). At boot, launchd creates a mach-port for it, keeps the receive right, and records `"com.example.expd" → that mach-port` in its table. expd isn't running yet.
2. Look up: the client asks launchd for `com.example.expd` and gets back a send right to that mach-port. launchd checks the client's sandbox first, which can forbid the name. XPC does this step with `bootstrap_look_up()`.
3. Send: the client's message waits in the mach-port's queue, because nobody is receiving yet.
4. Launch: launchd watches the mach-port with `EVFILT_MACHPORT` in its kqueue (see [kqueue and kevent](Sockets%20and%20kqueue.md#kqueue-and-kevent)), sees the waiting message, and starts expd.
5. Check in: expd asks launchd for the receive right and gets it, with the message still queued on it. XPC does this step with `bootstrap_check_in()`. expd then receives the message as if it had been running all along.

If expd later crashes, the receive right goes back to launchd instead of being destroyed. The clients' send rights stay valid, and the next message starts expd again.

launchd keeps a separate table, called a domain, for the whole system and for each logged-in user:

- System domain: daemons in `/Library/LaunchDaemons`, one per machine, running as root. expd is one.
- User domain: agents in `~/Library/LaunchAgents`, plus your apps.

A lookup searches the caller's own domain, then the system domain. A client of a system daemon passes `XPC_CONNECTION_MACH_SERVICE_PRIVILEGED` to search only the system domain, so an agent in the user's domain can't register `com.example.expd` and receive the client's messages.

### Replies: one mach-port per request

1. Make a port: when the client sends a request that expects a reply, XPC makes a new mach-port just for it. The client holds the receive right.
2. Attach: XPC puts a send-once right to that mach-port in the request.
3. Move: delivering the request moves the send-once right into expd's table.
4. Reply: expd's reply goes out on that send-once right, which uses it up.
5. Deliver: the reply lands on the per-request mach-port. XPC knows which request it belongs to, runs that request's reply handler, and destroys the mach-port.

Each request has its own mach-port, so replies can't get mixed up and there are no request IDs.

### Receiving: libdispatch runs your handlers

Neither side writes a receive loop. XPC hands its mach-ports to libdispatch (see [libdispatch](libdispatch.md)), which watches them with `EVFILT_MACHPORT`:

1. Wait: no thread is running. The kernel watches the mach-port, and the process uses no CPU.
2. Wake: a message arrives, and the kernel wakes a thread from libdispatch's pool.
3. Decode: XPC calls `mach_msg()` to take the message off and turns it into a dictionary.
4. Run: libdispatch runs your handler with the dictionary, on the dispatch queue you gave the connection.

A handler is a block, Apple's C closure, like `^(xpc_object_t msg) { ... }`. On a serial queue, like expd's, handlers run one at a time, so expd's state needs no lock.

## Using XPC in code

Your code holds three kinds of XPC object and never sees a mach-port:

- Object: `xpc_object_t`, a value XPC can send, usually a dictionary like `{"experiment": "new_onboarding"}`. Strings, `int64`s, bools, bytes and arrays are objects too. Objects are reference counted, and freeing one gives back any rights inside it.
- Connection: `xpc_connection_t`, one end of a two-way channel to one other process. The client holds one to expd.
- Listener: a connection on the server that owns the receive right for `com.example.expd`. It never carries messages. Each new client shows up on it as a new peer connection, like `accept()` returning a `conn_fd`, and from then on that client and expd talk over the peer.

The calls each side makes, in order (`xpc_connection_` dropped):

```
      expd (server)                                    client app

  create_mach_service(LISTENER)                   create_mach_service(PRIVILEGED)
     │  checks in: gets the receive right            │  nothing sent yet
     ▼                                               ▼
  set_event_handler(listener)                     set_event_handler(conn)    errors only
     ▼                                               ▼
  activate(listener)                              activate(conn)
     ▼                                               ▼
  dispatch_main(): waits for events               send_message_with_reply(msg)
     │                                               │  encode dict, make reply port,
     │                                               │  mach_msg()
     │ ◀─────────── first message ────────────────── │
     ▼                                               │
  listener handler(peer)          like accept()      │
     ▼                                               │
  set_event_handler(peer), activate(peer)            │
     ▼                                               │
  peer handler(msg)                                  │
     │  create_reply(msg), send_message(peer, reply) │
     │ ─────────── reply ──────────────────────────▶ ▼
     │                                            reply handler(reply)
     ▼                                               ▼
  peer handler(XPC_ERROR_CONNECTION_INVALID) ◀──  client exits or cancels
```

If expd isn't running when the first message is sent, the message waits while launchd starts expd. expd then runs its column from the top and finds the message already waiting.

### The plist

expd is a system daemon, so its plist goes in `/Library/LaunchDaemons/com.example.expd.plist`:

```xml
<key>Label</key>
<string>com.example.expd</string>
<key>ProgramArguments</key>
<array>
    <string>/usr/local/libexec/expd</string>
</array>
<key>MachServices</key>
<dict>
    <key>com.example.expd</key>
    <true/>
</dict>
```

- `MachServices`: each key inside it, here `com.example.expd`, is a service name clients look up.
- `ProgramArguments`: the program launchd starts, and the only program allowed to check in for that name.

A LaunchAgent in `~/Library/LaunchAgents` looks the same but runs once per logged-in user.

### Server

Calls, in the order expd uses them:

1. Make a queue: `dispatch_queue_create(label, attr)` makes the dispatch queue the handlers run on, like `dispatch_queue_create("com.example.expd.xpc", DISPATCH_QUEUE_SERIAL)`. The label shows up in crash logs.
2. Make the listener: `xpc_connection_create_mach_service(name, queue, flags)` makes a connection for a launchd service name. With `XPC_CONNECTION_MACH_SERVICE_LISTENER` as `flags`, it's a listener, and XPC checks in with launchd for the receive right.
3. Set the handler: `xpc_connection_set_event_handler(conn, handler)` sets the block that runs for each event on `conn`. On the listener, each event is a new peer connection. On a peer, each event is a message or an error. One connection's handler runs one event at a time, in the order they were sent.
4. Check the type: `xpc_get_type(obj)` says what an event is: `XPC_TYPE_CONNECTION`, `XPC_TYPE_DICTIONARY` or `XPC_TYPE_ERROR`. Errors arrive through the same handler as everything else, so every handler checks the type first.
5. Start: `xpc_connection_activate(conn)` starts delivering events. Connections start paused so you can set the handler before anything arrives. Older code calls `xpc_connection_resume()` for the same thing.
6. Read the request: `xpc_dictionary_get_string(dict, key)` returns one value, like `xpc_dictionary_get_string(msg, "experiment")` returning `"new_onboarding"`, or `NULL` if the key is missing. Each type has its own getter, like `xpc_dictionary_get_int64()` and `xpc_dictionary_get_bool()`.
7. Make the reply: `xpc_dictionary_create_reply(msg)` makes an empty dictionary that's already tied to that request's reply port. Fill it with a setter: `xpc_dictionary_set_string(dict, key, value)`, like `xpc_dictionary_set_string(reply, "variant", "B")`.
8. Send: `xpc_connection_send_message(conn, msg)` sends a dictionary. It doesn't block: XPC sends it in the background.
9. Release: `xpc_release(obj)` drops your reference to an object you made. In plain C you release what you create. Objects handed to your handler belong to XPC.
10. Park: `dispatch_main()` parks the main thread forever, and libdispatch runs the handlers.

`variant_for()` is expd's own lookup code.

```c
#include <xpc/xpc.h>   // also includes dispatch/dispatch.h

int main(void) {
    dispatch_queue_t queue = dispatch_queue_create("com.example.expd.xpc", DISPATCH_QUEUE_SERIAL);
    xpc_connection_t listener = xpc_connection_create_mach_service(
        "com.example.expd", queue, XPC_CONNECTION_MACH_SERVICE_LISTENER);

    xpc_connection_set_event_handler(listener, ^(xpc_object_t peer) {   // a new client
        if (xpc_get_type(peer) != XPC_TYPE_CONNECTION) return;          // an error on the listener itself

        xpc_connection_set_event_handler(peer, ^(xpc_object_t msg) {    // a message from that client
            if (xpc_get_type(msg) == XPC_TYPE_ERROR) return;            // the client went away

            const char *exp = xpc_dictionary_get_string(msg, "experiment");   // "new_onboarding"
            xpc_object_t reply = xpc_dictionary_create_reply(msg);
            xpc_dictionary_set_string(reply, "variant", variant_for(exp));    // "B"
            xpc_connection_send_message(peer, reply);
            xpc_release(reply);
        });
        xpc_connection_activate(peer);
    });

    xpc_connection_activate(listener);
    dispatch_main();   // never returns
}
```

### Client

The client uses the same three connection calls, then builds and sends a request:

1. Connect: `xpc_connection_create_mach_service("com.example.expd", queue, XPC_CONNECTION_MACH_SERVICE_PRIVILEGED)`, the same call without the listener flag, so you get a send right instead of the receive right. Nothing is sent yet.
2. Set the handler and start: `xpc_connection_set_event_handler()` and `xpc_connection_activate()`, as on the server. A client that only sends requests gets only errors in this handler.
3. Build the request: `xpc_dictionary_create_empty()` makes an empty dictionary. Fill it with setters, like `xpc_dictionary_set_string(msg, "experiment", "new_onboarding")`.
4. Send and get the reply: `xpc_connection_send_message_with_reply(conn, msg, queue, handler)` sends `msg` and returns right away. Later, `handler` runs on `queue` with the reply, or with an error if expd died before replying.
5. Or wait for it: `xpc_connection_send_message_with_reply_sync(conn, msg)` blocks the thread until the reply arrives and returns it. Fine on a worker thread, but never on an app's main thread, where it would freeze the UI.
6. Close: `xpc_connection_cancel(conn)` ends the connection for good when you're done with it.

```c
dispatch_queue_t queue = dispatch_queue_create("com.example.app.expd", DISPATCH_QUEUE_SERIAL);
xpc_connection_t conn = xpc_connection_create_mach_service(
    "com.example.expd", queue, XPC_CONNECTION_MACH_SERVICE_PRIVILEGED);
xpc_connection_set_event_handler(conn, ^(xpc_object_t event) {
    // only errors arrive here (see When things go wrong)
});
xpc_connection_activate(conn);

xpc_object_t msg = xpc_dictionary_create_empty();
xpc_dictionary_set_string(msg, "experiment", "new_onboarding");
xpc_connection_send_message_with_reply(conn, msg, queue, ^(xpc_object_t reply) {
    if (xpc_get_type(reply) == XPC_TYPE_DICTIONARY) {
        const char *variant = xpc_dictionary_get_string(reply, "variant");   // "B"
    } else {
        // XPC_ERROR_CONNECTION_INTERRUPTED: expd died before replying
    }
});
xpc_release(msg);
```

### The calls you'll use

Most XPC code is these, grouped by job:

- Connect: `xpc_connection_create_mach_service()`, `xpc_connection_set_event_handler()`, `xpc_connection_activate()`. The same three on both sides. The listener flag makes it the server.
- Build a message: `xpc_dictionary_create_empty()`, then setters like `xpc_dictionary_set_string()`, `_set_int64()`, `_set_bool()`.
- Send: `xpc_connection_send_message()` when you don't need a reply, `xpc_connection_send_message_with_reply()` when you do.
- Handle an event: `xpc_get_type()` first, then getters like `xpc_dictionary_get_string()`.
- Reply: `xpc_dictionary_create_reply()`, then `xpc_connection_send_message()`.
- Clean up: `xpc_release()` for objects you made, `xpc_connection_cancel()` for a connection you're done with.
- Errors to check for: `XPC_ERROR_CONNECTION_INTERRUPTED` and `XPC_ERROR_CONNECTION_INVALID` (next section).

macOS 13 and 14 added a newer API for the same flow, `xpc_session_*` for clients and `xpc_listener_*` for servers. Most existing daemons, and this note, use `xpc_connection_*`.

## When things go wrong

An error arrives as an object of type `XPC_TYPE_ERROR`, in a connection's event handler or in a reply handler. Each error is a fixed object, so you compare it directly, like `event == XPC_ERROR_CONNECTION_INVALID`. Two matter:

- Interrupted: `XPC_ERROR_CONNECTION_INTERRUPTED` means the other side crashed or exited, but the connection still works. Pending reply handlers get this error, and the client's next message starts expd again through launchd. The new expd knows nothing about this client, so the client re-sends any state it had set up, like registering for change notifications.
- Invalid: `XPC_ERROR_CONNECTION_INVALID` means the connection is dead for good. A client gets it when the name isn't registered (a typo, the plist isn't loaded, or the sandbox blocks the lookup) or after it calls `xpc_connection_cancel()`. On the server, a peer gets it when that client exits. It's the XPC version of `EV_EOF` on a `conn_fd`, and the place to free that client's state.

A client's event handler that handles both:

```c
xpc_connection_set_event_handler(conn, ^(xpc_object_t event) {
    if (event == XPC_ERROR_CONNECTION_INTERRUPTED) {
        // expd restarted: re-send registrations
    } else if (event == XPC_ERROR_CONNECTION_INVALID) {
        // dead for good: stop using conn
    }
});
```

So when expd crashes, each client loses at most its pending replies, and no client has to reconnect by hand.

## Who is calling

expd decides who may talk to it by setting a requirement on the listener, before `xpc_connection_activate(listener)`. XPC then drops messages from any client that doesn't meet it.

- Require an entitlement: `xpc_connection_set_peer_entitlement_exists_requirement(conn, entitlement)` (macOS 14.4+, iOS 17.4+) accepts only clients that have `entitlement`, like `xpc_connection_set_peer_entitlement_exists_requirement(listener, "com.example.expd.client")`. An entitlement is a key signed into an app's code signature, so a client can't fake it.
- Require a signature: `xpc_connection_set_peer_code_signing_requirement(conn, requirement)` (macOS 12+) checks the client's code signature against a requirement string, like `anchor apple generic and certificate leaf[subject.OU] = "ABCDE12345"`, which means "signed by team `ABCDE12345`".

Pick one per connection. A client can set the same requirements on its own connection to check expd. If expd doesn't meet them, the client's reply handler gets `XPC_ERROR_PEER_CODE_SIGNING_REQUIREMENT` instead of a reply.

`xpc_connection_get_pid(peer)` returns the client's pid, like `4312`, which is fine for logging. Pids get reused (see the [sockets note](Sockets%20and%20kqueue.md#three-things-tcp-cant-do)), so don't base trust on it.

## Sending files and shared memory

Besides dictionaries, arrays, strings, numbers, bools and bytes (`xpc_dictionary_set_data()`), a message can carry two kernel objects. XPC moves them the way rights move in a message:

- An open file: `xpc_dictionary_set_fd(dict, key, fd)` puts an fd in a dictionary, like `xpc_dictionary_set_fd(reply, "snapshot", snapshot_fd)`, and `xpc_dictionary_dup_fd(dict, key)` gives the receiver its own fd to the same open file. It's `SCM_RIGHTS` on a UDS. expd opens the snapshot file and sends it, and the client can read it without permission to open the path.
- Shared memory: `xpc_shmem_create(addr, len)` wraps a memory region, and the receiver maps the same pages with `xpc_shmem_map(obj, &addr)`. For big data, share the region once, then send small messages saying what changed.

## Priority and idle exit

- Priority: each message carries the sender's QoS (quality of service), like `QOS_CLASS_USER_INITIATED` for something the user is waiting on, or `QOS_CLASS_BACKGROUND`. expd's handler runs at that QoS. While a foreground app waits on a reply, the kernel also raises expd's priority until the reply goes out. This is importance donation, and the plist turns it on with `ProcessType` set to `Adaptive`. A UDS `read()` carries none of this.
- Idle exit: a request expd hasn't replied to yet counts as an open transaction. With none open, expd is idle, and if its plist sets `EnablePressuredExit`, the system can kill it to free memory. launchd starts it again on the next message.

## Background work: xpc_activity

`xpc_activity` is the part of XPC that schedules work instead of sending messages. expd would use it to fetch new experiment configs. The scheduler daemon, `dasd`, picks the moment and batches work from many daemons, so the device wakes less often.

1. Describe when: a criteria dictionary says when the work may run, like "about once an hour, repeating, only on power".
2. Register: `xpc_activity_register(identifier, criteria, handler)` asks the system to run `handler` once the criteria hold, like `xpc_activity_register("com.example.expd.refresh", criteria, handler)`.
3. Check the state: `xpc_activity_get_state(activity)` returns `XPC_ACTIVITY_STATE_RUN` when it's time to do the work.
4. Give time back: `xpc_activity_should_defer(activity)` returns true when the system wants the time back, for example because the user started using the device. Call `xpc_activity_set_state(activity, XPC_ACTIVITY_STATE_DEFER)` and return, and the work runs again later.
5. Finish: returning from the handler marks the run as done.

```c
xpc_object_t criteria = xpc_dictionary_create_empty();
xpc_dictionary_set_int64(criteria, XPC_ACTIVITY_INTERVAL, XPC_ACTIVITY_INTERVAL_1_HOUR);
xpc_dictionary_set_bool(criteria, XPC_ACTIVITY_REPEATING, true);
xpc_dictionary_set_bool(criteria, XPC_ACTIVITY_ALLOW_BATTERY, false);

xpc_activity_register("com.example.expd.refresh", criteria, ^(xpc_activity_t activity) {
    if (xpc_activity_get_state(activity) != XPC_ACTIVITY_STATE_RUN) return;
    for (int i = 0; i < n_configs; i++) {
        if (xpc_activity_should_defer(activity)) {
            xpc_activity_set_state(activity, XPC_ACTIVITY_STATE_DEFER);   // try again later
            return;
        }
        // fetch config i, write it to disk
    }
});
xpc_release(criteria);
```

## Two kinds of XPC service

- launchd service: a name under `MachServices`, like expd. Any process whose sandbox allows the name can connect. On iOS, only Apple's own daemons can register one.
- XPC service bundle: a `.xpc` bundle inside an app's `Contents/XPCServices/`. Only that app can connect, with `xpc_connection_create(name, queue)`, like `xpc_connection_create("com.example.app.parser", queue)`. Apps use one to move risky work, like parsing untrusted files, into a separate sandboxed process that launchd starts and stops for them.

## UDS vs XPC

Both connect an app to a daemon like expd on the same machine. XPC's column is mostly "what you'd build yourself on a UDS, done for you".

| | UDS | XPC |
|---|---|---|
| Address | a path, like `/var/run/expd.sock` | a service name, like `com.example.expd` |
| Started on demand by | launchd, `Sockets` key | launchd, `MachServices` key |
| Server holds | `listen_fd`, plus a `conn_fd` per client | a listener, plus a peer connection per client |
| What travels | bytes you frame and parse | typed dictionaries |
| Receiving | your `kevent()` loop, then `read()` | libdispatch calls your handler |
| Replies | on the same `conn_fd`, matched by your own IDs | a reply handler per request |
| Checking the caller | `LOCAL_PEERTOKEN`, then check entitlements yourself | one `xpc_connection_set_peer_*_requirement()` call |
| Passing an fd | `SCM_RIGHTS` with `sendmsg()` | `xpc_dictionary_set_fd()` |
| Server crashes | `read()` returns `0`, and the client reconnects | `XPC_ERROR_CONNECTION_INTERRUPTED`, and the same connection keeps working |
| Priority | none carried | the sender's QoS travels with each message |
| Platforms | macOS and Linux | Apple only |

That's why the experimentation service uses a UDS (it also runs on Linux), and why XPC is the better default for an Apple-only design.

Linux contrast: there are no mach-ports. D-Bus does XPC's job: named services on a bus, typed messages, and systemd starts a service when its first message arrives.
