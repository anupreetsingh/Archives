## XPC and mach-ports

XPC is Apple's library for sending messages between two processes on the same machine. The usual case is an app asking a daemon for something. Here an app asks expd (the experimentation daemon) which variant it's in: it sends the dictionary `{"experiment": "new_onboarding"}` and gets back `{"variant": "B"}`. It does the job a UDS does in the [sockets note](Sockets%20and%20kqueue.md#unix-domain-sockets), but only on Apple platforms.

What XPC does for you that you'd write yourself with a UDS:

1. Find the server: the client names it with a string like `com.example.expd`, and launchd looks the name up in its service table and returns a send right to the service port.
   UDS: both sides hardcode the same socket path, like `/var/run/expd.sock`.

2. Start it on demand: launchd holds the service port, starts expd when a message waits on it, and gets the port back through a port-destroyed notification after a crash.
   UDS: the plist's `Sockets` key, `launch_activate_socket()` in the daemon, and a fallback that creates the socket when it's run by hand.

3. Encode messages: you build typed dictionaries, and XPC turns each one into a single mach message.
   UDS: you design a wire format, add framing like a TLV format (a stream has no message boundaries), and parse it on the other side.

4. Match replies: each request carries a send-once right to its own reply port, and XPC runs that request's reply handler when the reply lands.
   UDS: you put a request ID in every message and keep a table matching replies to the callers waiting on them.

5. Report a dead peer: the kernel's dead-name notification becomes `XPC_ERROR_CONNECTION_INTERRUPTED` or `XPC_ERROR_CONNECTION_INVALID` in your handler.
   UDS: you notice `read()` returning `0` or an `EPIPE` / `ECONNRESET` error, then reconnect and resend yourself.

6. Check the caller: the kernel attaches the sender's audit token to every message, and XPC checks the caller's entitlement against it before your handler sees the message, so the caller can't fake it.
   UDS: on macOS, `getsockopt(LOCAL_PEERTOKEN)` gives the audit token, but reading the entitlement from it is code you write. On Linux, `SO_PEERCRED` gives only pid, uid and gid.

7. Donate priority: the client's priority (its QoS) travels with the message, and the kernel runs expd's handling at that priority, so a UI request isn't stuck behind expd's background work.
   UDS: no equivalent. The request runs at whatever priority expd's thread has, unless you send the priority in the message and set it on the thread yourself.

That's why, on Apple-only, a daemon would use XPC. expd uses a UDS instead so the same code runs on macOS and Linux.

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
                             libdispatch in expd wakes ──────────▶ expd handler gets the dictionary
                                                                     │  builds a reply, sends it
  your reply handler runs ◀── kernel: message on the reply port ◀────┘
```

The sections below follow the diagram in order: the mach-port the message travels on, launchd finding and starting expd, the reply coming back, and libdispatch running the handler on each side.

### Mach-ports: what the message travels on

A mach-port is a queue of messages inside the kernel. It's the channel underneath XPC, the way a socket is the channel underneath a UDS connection. Two things differ from a stream UDS:

- Whole messages: each send puts one message on the queue and each receive takes one whole message off, so there's no length prefix to add.
- One way: a mach-port has exactly one receiver, and everyone else can only send to it. So requests go to expd's mach-port, and replies need a second mach-port that the client receives on.

A process never holds a mach-port itself. It holds a right to one and refers to it by a small number, like `0x1103`, called the **mach-port-name**, the way it refers to a socket by fd `3`. The kernel keeps each process's rights in a table, like the fd table, so the same mach-port has a different number in each process. That table is the process's **IPC space** (`ipc_space`), which hangs off its Mach `task` the way the fd table hangs off its BSD `proc`. A mach-port-name like `0x1103` is an index into it.

There are three kinds of right:

- Receive right: take messages off the queue. Exactly one process holds it, and that process owns the mach-port. expd holds the receive right for its mach-port.
- Send right: put messages on the queue. Any number of processes can hold one. Every app talking to expd holds a send right to expd's mach-port.
- Send-once right: a send right that's used up by one message. Replies use it.

Holding the right is the permission. There's no separate check.

A right can also travel inside a message. The kernel takes it out of the sender's table and adds it to the receiver's under a new number, like the client's `0x1103` showing up as `0x2207` in expd. A raw number means nothing in another process, so this is the only way to hand over a right. It's how the client gives expd a way to reply, and how XPC sends an open fd: it wraps the fd in a **fileport**, which is a port right that points to the same open file object in the kernel (`fileglob`), and the receiver turns it back into its own fd.

When the process with the receive right to a mach-port exits, the kernel**destroys the mach-port** and tells everyone holding a send right to it. XPC turns that into an error event (see #When things go wrong).

That telling is a **Mach notification**: a message the kernel itself sends, to a port you chose, when something happens to a port. You ask for one with `mach_port_request_notification()`, and it arrives like any message, so a kqueue watching that port with `EVFILT_MACHPORT` wakes for it. Three kinds matter here:

- Dead-name: the mach-port you hold a send right to was destroyed. XPC asks for this one on every connection, and it's how the error event above happens.
- No-senders: no one holds a send right to your mach-port anymore, so you can clean up.
- Port-destroyed: the process holding a receive right died, and instead of destroying the mach-port, the kernel sends that receive right to the process that requested the notification. For example, launchd receives it when expd, a daemon launchd started, dies while holding the receive right to its service port (see below).

The syscall underneath is `mach_msg()`, which sends or receives one message. XPC calls it for you. App code almost never uses raw mach-ports because:

- You'd lay out every message's bytes by hand, and you'd have to give back every right by hand.
- A forgotten right shows up as a hang, not an error. A **hang** is a thread stuck waiting forever, with no error and no crash. For example, if expd returns early without replying and never gives back the client's send-once reply right, the kernel thinks a reply might still come, so the client's thread stays blocked in `mach_msg()` until expd exits.

### launchd: finding expd and starting it

A mach-port has no path, so a client can't find one in the filesystem. Instead, launchd keeps a **service table** that maps each service name, like `com.example.expd`, to a **service port**: an ordinary mach-port that launchd created for that name. A launchd job (a daemon or agent) registers its service names through the `MachServices` key in its plist. To reach launchd itself, every process starts out holding a send right to launchd's mach-port, called its **bootstrap port**, inherited from its parent. The bootstrap port is how a process asks launchd for a service port.

1. Register: expd's plist lists `com.example.expd` under `MachServices` (see The plist). At boot, launchd creates the service port, keeps the receive right, and records `"com.example.expd" → service port` in its service table. expd isn't running yet.
2. Look up: the client sends `com.example.expd` to launchd over its bootstrap port and gets back a send right to the service port. launchd checks the client's sandbox first, which can forbid the name. XPC does this step with `bootstrap_look_up()`.
3. Send: the client's message waits in the service port's queue, because nobody is receiving yet.
4. Launch: launchd watches the service port with `EVFILT_MACHPORT` in its kqueue (see [kqueue and kevent](Sockets%20and%20kqueue.md#kqueue-and-kevent)), sees the waiting message, and starts expd.
5. Check in: expd asks launchd over its own bootstrap port for the service port's receive right and gets it, with the message still queued. XPC does this step with `bootstrap_check_in()`. expd then receives the message as if it had been running all along.

If expd later crashes, the receive right goes back to launchd instead of being destroyed. Before handing it over in step 5, launchd requested a port-destroyed notification on the service port, so when expd dies the kernel puts the receive right in a message to launchd's own notification port, with the queued messages still on the service port. The clients' send rights stay valid, and the next message starts expd again.

launchd keeps a separate table, called a **domain**, for the whole system and for each logged-in user:

- System domain: daemons in `/Library/LaunchDaemons`, one per machine, running as root. expd is one.
- User domain: agents in `~/Library/LaunchAgents`, plus your apps.

A lookup searches the caller's own domain, then the system domain. A client of a system daemon passes `XPC_CONNECTION_MACH_SERVICE_PRIVILEGED` to search only the system domain, so an agent in the user's domain can't register `com.example.expd` and receive the client's messages.

### Replies: one mach-port per request

1. Make a port: when the client sends a request that expects a reply, XPC makes a new mach-port just for it. The client holds the receive right for the new per-request mach-port.
2. Attach: XPC puts a send-once right to that new mach-port in the request.
3. Move: delivering the request moves the send-once right into expd's table.
4. Reply: expd's reply goes out on that send-once right, which uses it up.
5. Deliver: the reply lands on the new per-request mach-port. XPC knows which request it belongs to, runs that request's reply handler, and destroys the mach-port.

Each request has its own mach-port, so replies can't get mixed up and there are no request IDs.

### Receiving: libdispatch runs your handlers

Neither side writes a receive loop. For each connection, XPC makes a dispatch-event-source for its mach-port : it calls `dispatch_source_create(DISPATCH_SOURCE_TYPE_MACH_RECV, port, 0, queue)` and `dispatch_source_set_event_handler()` with its own block. You never call these. You give XPC only two things (both in [Using XPC in code](#using-xpc-in-code)):

- Queue: you make a dispatch queue (`dispatch_queue_t`) with `dispatch_queue_create()` and pass it to `xpc_connection_create_mach_service()`.
- Handler: you pass your handler block to `xpc_connection_set_event_handler()`.

"Its mach-port" is the one port that side holds the receive right to. A connection is two mach-ports, one per direction, but each side receives on only one of them, so each side's dispatch-event-source watches only that one. The send right has nothing to watch:

```
client connection                        expd's peer connection
  receive right: port B  ← watched       receive right: port A  ← watched
  send right:    port A                  send right:    port B
```

- The client sends requests on port A. Its source watches port B, which carries only messages expd sends on its own, plus errors.
- expd's peer source watches port A, which carries the client's requests.
- Replies don't come on port B. Each comes on its request's own reply port (see [Replies](#replies-one-mach-port-per-request)), and XPC runs that request's reply handler, not the connection's handler.
- expd's listener has one port, the service port, so its source watches just that one.

Then each message goes like this:

1. Wait: no thread is running. The kernel watches the mach-port with `EVFILT_MACHPORT`, and the process uses no CPU.
2. Wake: a message arrives, the kernel wakes a thread from libdispatch's pool, and libdispatch puts the source's block, which is set as the handler for that XPC connection, on your queue.
3. Decode: XPC's block calls `mach_msg()` to take the message off and turns it into a dictionary.
4. Run: XPC's block calls your handler with the dictionary, on that same queue.

A handler is a block, Apple's C closure, like `^(xpc_object_t msg) { ... }`. On a serial queue, like expd's, handlers run one at a time, so expd's state needs no lock.

## Using XPC in code

Your code holds three kinds of XPC object and never deals with a mach-port directly:

- **Object:** `xpc_object_t`, a value XPC can send, usually a dictionary like `{"experiment": "new_onboarding"}`. Strings, `int64`s, bools, bytes and arrays are objects too. Objects are reference counted, and freeing one gives back any rights inside it.

- **Connection:** `xpc_connection_t`, one end of a two-way channel to one other process. The client holds one to expd. A mach-port carries messages one way only, so the channel is two mach-ports, one per direction, and each end holds the receive right to one and a send right to the other.

- **Listener:** also an `xpc_connection_t` on the server, but it's one mach-port, not two: expd holds the receive right for `com.example.expd`, and clients hold only send rights to it. It never carries messages. Each new client shows up on it as a new peer connection, like `accept()` returning a `conn_fd`, and from then on that client and expd talk over the peer. The listener and the service port are the same mach-port seen from two layers: the service port is the Mach object that launchd registered, and the listener is the XPC object that wraps its receive right inside expd. When expd exits, the listener is gone with it, and launchd holds the plain receive right again, with no listener.

The calls each side makes, in order (`xpc_connection_` dropped):

```
     client app                                     expd (server)

  create_mach_service(PRIVILEGED)                create_mach_service(LISTENER)
     │  nothing sent yet                            │  checks in: gets the receive right
     ▼                                              ▼
  set_event_handler(conn)    errors only         set_event_handler(listener)
     ▼                                              ▼
  activate(conn)                                 activate(listener)
     ▼                                              ▼
  send_message_with_reply(msg)                   dispatch_main(): waits for events
     │  encode dict, make reply port,               │
     │  mach_msg()                                  │
     │ ───────────── first message ───────────────▶ │
     │                                              ▼
     │                                           listener handler(peer)    like accept()
     │                                              ▼
     │                                           set_event_handler(peer), activate(peer)
     │                                              ▼
     │                                           peer handler(msg)
     │                                              │  create_reply(msg), send_message(peer, reply)
     ▼ ◀───────────── reply ─────────────────────── │
  reply handler(reply)                              │
     ▼                                              ▼
  client exits or cancels  ──▶                   peer handler(XPC_ERROR_CONNECTION_INVALID)
```

If expd isn't running when the first message is sent, the message waits while launchd starts expd. expd then runs its column from the top and finds the message already waiting.

The first message is also when the connection's two mach-ports (port A and port B in [Receiving](#receiving-libdispatch-runs-your-handlers)) reach expd. Creating and activating the client connection sends nothing. On the first send, the client's XPC looks up the service port with launchd, makes the port pair, and sends a first message to the service port carrying the receive right to port A and a send right to port B. expd's listener source fires, and XPC wraps those two rights in the new peer it hands to `listener handler(peer)`. libxpc doesn't document this handshake, but this is the model.

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

Calls, in the order the client uses them. The first four mirror the server:

1. Make a queue: `dispatch_queue_create("com.example.client.xpc", DISPATCH_QUEUE_SERIAL)`, the client's own dispatch queue, separate from expd's. The connection's handler and reply handlers run on it. An app that updates its UI with the reply can pass `dispatch_get_main_queue()` instead.
2. Connect: `xpc_connection_create_mach_service("com.example.expd", queue, XPC_CONNECTION_MACH_SERVICE_PRIVILEGED)`, the same call without the listener flag, so you get a send right instead of the receive right. Nothing is sent yet: the lookup and the rights expd needs for this connection go out with the first message (see the paragraph under the diagram).
3. Set the handler: `xpc_connection_set_event_handler(conn, handler)`, as on the server. A client that only sends requests gets only errors here, so the handler checks `xpc_get_type(event) == XPC_TYPE_ERROR`.
4. Start: `xpc_connection_activate(conn)`, as on the server.
5. Build the request: `xpc_dictionary_create_empty()` makes an empty dictionary. Fill it with setters, like `xpc_dictionary_set_string(msg, "experiment", "new_onboarding")`.
6. Send and get the reply, in one of two ways:
   - Async: `xpc_connection_send_message_with_reply(conn, msg, queue, handler)` sends `msg` and returns right away. Later, `handler` runs on `queue` with the reply, or with an error if expd died before replying. `queue` is usually the one from step 1.
   - Sync: `xpc_connection_send_message_with_reply_sync(conn, msg)` blocks the thread until the reply arrives and returns it. Fine on a worker thread, but never on an app's main thread, where it would freeze the UI.
7. Release: `xpc_release(msg)` once it's sent, since the client made it.
8. Close: `xpc_connection_cancel(conn)` ends the connection for good when you're done with it.

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

Underneath, both start as Mach notifications (see [Mach-ports: what the message travels on](#mach-ports-what-the-message-travels-on)). A crash on either side destroys that side's ports, and the other side gets a dead-name notification. What separates the two errors is port-destroyed: launchd holds it on the service port, so when expd dies the service port survives and the client's connection still works (interrupted). A client's ports have no such backup, so when a client dies its peer is gone for good (invalid). libxpc's exact internals aren't public, but this is the mapping.

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

`xpc_activity` is the part of XPC that schedules work instead of sending messages. expd would use it to fetch new experiment configs. The scheduler daemon: `dasd`, picks the moment and batches work from many daemons, so the device wakes less often.

It's called an XPC activity because the wake-up is IPC. When dasd decides it's time, launchd starts expd if it isn't running, and libxpc runs the handler expd registered with `xpc_activity_register()`. XPC carries only "run now" in, and expd's state (defer, done) back out. The fetch itself is the handler's own code, like an HTTPS request over a TCP socket, not an XPC message. Declaring the activity in the plist under `LaunchEvents` → `com.apple.xpc.activity` lets it fire even when expd isn't running; expd registers the same identifier at startup to receive it.

1. Describe when: a criteria dictionary says when the work may run, like "about once an hour, repeating, only on power".
2. Register: `xpc_activity_register(identifier, criteria, handler)` asks the system to run `handler` once the criteria holds, like `xpc_activity_register("com.example.expd.refresh", criteria, handler)`.
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
