# Bullet 1: The Daemon

> Built a C++ experiment-assignment daemon that launchd (macOS) and systemd (Linux) start on demand by socket activation, serving client processes over Unix domain sockets from one non-blocking kqueue/epoll event loop and bucketing each device by a salted hash of its ID, so it keeps its variant across reboots with no server round trip.

## "experiment-assignment daemon"

A background process with one job: answering "which variant is this device in for experiment X?"

Why a daemon instead of a library in every app:

- One owner: the config is fetched and evaluated once per device, and every process gets the same answer.
- Runs with no app open: fetching and flushing happen whether or not any app is running.
- Privilege: only the daemon holds the device ID and can write the snapshot.
- Fault isolation: if a bad config crashes the parser, expd crashes and launchd restarts it; the app keeps going.

He'll ask: "Then why do reads skip the daemon?" Reads are the hot path, so they stay inside the app's process (bullet 2). The daemon does the writes and background work.

## "start on demand by socket activation"

launchd (or systemd) creates the listening socket itself and starts expd only when the first client connects. expd then takes over the socket's fd: `launch_activate_socket("Listeners", &fds, &n)` on macOS, `sd_listen_fds(0)` on Linux. Details: [Socket activation with launchd](../Foundations/Sockets%20and%20kqueue.md#socket-activation-with-launchd).

Why:

- Zero idle cost: no memory or CPU until someone connects.
- No lost connections: while expd is dead or restarting, launchd keeps the socket open, and new connections wait in its backlog queue.
- No startup order: clients never wait for expd to come up first.

Instead of: running all the time with `KeepAlive` (costs memory forever), or expd creating its own socket (connections fail while it restarts).

He'll ask:

- "What if you run it by hand?" `launch_activate_socket()` fails, so expd creates the socket itself.
- "How does it stop?" launchd sends `SIGTERM`, waits `ExitTimeOut` seconds (like `20`), then sends `SIGKILL`. expd handles `SIGTERM` as a kqueue event, not inside a signal handler: [Shutting down with SIGTERM](../Foundations/Sockets%20and%20kqueue.md#shutting-down-with-sigterm).

## "over Unix domain sockets"

A socket whose address is a file path, like `/var/run/expd.sock`, instead of an IP and port. No TCP/IP runs underneath: the kernel copies the bytes from one side's `send()` straight into the peer socket's receive buffer (`so_rcv`, a `struct sockbuf`). Details: [Unix domain sockets](../Foundations/Sockets%20and%20kqueue.md#unix-domain-sockets).

Why:

- Same code on macOS and Linux.
- Fast for local IPC.
- The socket file's permissions decide who can connect, and expd can ask who connected: `getpeereid()` gives the client's uid and gid (Linux: `SO_PEERCRED`).

Instead of:

- XPC: Apple-only, but adds typed messages, launchd integration, priority donation, and the audit token for checking the client's entitlements. See [UDS vs XPC](../Foundations/XPC%20and%20Mach%20ports.md#uds-vs-xpc).
- TCP on localhost: a network stack for nothing, and any local process can connect.
- Pipes: one-way, and usually only between a parent and its child.

He'll ask:

- "Why not XPC?" Portability to Linux. If it were Apple-only, I'd use XPC, mainly for the audit-token entitlement check and priority donation.
- "A stream socket has no message boundaries. How do you split messages?" TLV (type-length-value) frames: a 1-byte type, a 4-byte length, then that many bytes of value. The header is always 5 bytes, so expd waits for 5 bytes, reads the length, then waits until it has `5 + length` bytes in that client's buffer before handling the message. Two types:

  ```
  [ 01 lookup   ][ length ][ "com.anupreet.roleradar/new_onboarding" ]            → reply [ 01 ][ length ][ "B" ]
  [ 02 exposure ][ length ][ key | variant | timestamp ]                          → no reply, goes in the ring buffer
  ```

  The length lets expd skip a type it doesn't know, so an older expd stays in sync with a newer library. It also rejects any length over a maximum, like 64 KB, and closes a client that stalls mid-frame.

## "one non-blocking kqueue/epoll event loop"

One thread loops. `kevent()` (Linux: `epoll_wait()`) blocks until any registered fd is ready, then the thread handles each ready fd:

1. Listening socket ready: `accept()` the new client.
2. Client socket ready: `read()` into that client's buffer, and handle the request once it's complete.

Every fd is non-blocking (`fcntl(fd, F_SETFL, O_NONBLOCK)`), so `read()` and `accept()` return `EAGAIN` ("try again") instead of putting the thread to sleep. The kqueue (`struct kqueue`) holds one `knote` per registered fd or event. Details: [kqueue and kevent](../Foundations/Sockets%20and%20kqueue.md#kqueue-and-kevent).

Why one thread:

- Each request is a lookup costing microseconds of CPU.
- No locks on the assignment table, since only one thread touches it.
- Small memory: no extra thread stacks.
- Add a thread only for blocking or heavy work. Bullet 3 does exactly that for disk writes.

Why non-blocking: with a blocking `read()`, one client that sends half a message freezes the loop for everyone.

kqueue vs epoll:

- Calls: `kevent()` registers changes and waits in one syscall. epoll needs one `epoll_ctl()` per change, then `epoll_wait()`.
- What they watch: kqueue also watches timers (`EVFILT_TIMER`), signals (`EVFILT_SIGNAL`), file changes (`EVFILT_VNODE`) and processes (`EVFILT_PROC`). epoll only watches fds, so timers and signals need `timerfd` and `signalfd`.

He'll ask:

- "A client sends half a request and stalls." Its bytes wait in its own buffer; the other clients are unaffected.
- "A client stops reading replies." `write()` returns `EAGAIN`, so expd keeps the reply in that client's buffer and registers `EVFILT_WRITE`. Cap that buffer, and disconnect the client past it.
- "A client crashes." `read()` returns `0` (end of file), so expd calls `close()`, which also removes the fd's knotes.
- "Level- or edge-triggered?" Level, kqueue's default and the simplest. Edge (`EV_CLEAR`, `EPOLLET`) forces reading until `EAGAIN` every time.
- "On Apple?" Dispatch sources on a serial queue, which libdispatch builds on kqueue: [Event sources](../Foundations/libdispatch.md#event-sources-libdispatch-replaces-your-kevent-loop).

## "bucketing each device by a salted hash of its ID"

`bucket = H(salt + experiment_id + device_id) % 10000`, like `H("k3f9" + "new_search_ranking" + "6F1C…") % 10000` → `4821`. The variant is whichever range the bucket falls in, like 0–4999 control and 5000–9999 treatment.

Why a hash instead of a random draw that's stored:

- Deterministic: no stored assignment, no server state, the same answer after every reboot.
- Ramps stay consistent: widening treatment from `< 1000` to `< 5000` keeps every device already in treatment.

Why a salt: if only the device ID were hashed, every experiment would put the same devices in treatment, so experiments wouldn't be independent and their effects would mix. Putting the experiment name in the hash already prevents that. A separate salt adds one more thing: a team can reshuffle an experiment by changing its salt, without renaming the experiment.

Where each input comes from: the salt and the experiment name come with the config, one salt per experiment, the same on every device. The device ID is the only per-device input, and it stays on the device. So for one experiment, a device always gets the same variant, until someone changes that experiment's salt in the config.

What the hash must be:

- Stable across compilers, platforms and versions.
- Uniform, so 10% really is 10%.
- Fast enough, which is easy: expd hashes once per experiment per config push, not on every lookup.

Never `std::hash`: the standard only requires it to be consistent within one run of a program, and libc++ and libstdc++ give different results.

Decide:

- Hash: SHA-256 truncated to 64 bits (`CC_SHA256()` on macOS) is stable, uniform, and fast enough once per push. Also know FNV-1a by heart for CoderPad.
- Device ID: `IOPlatformUUID` (a hardware ID, read through IOKit), `/etc/machine-id` on Linux, or a random UUID made on first run and saved to disk. The random UUID is the privacy-friendly one: no hardware ID, and it never leaves the device.

He'll ask:

- "What if the ID file is deleted?" New ID, so the device re-buckets and contaminates the data. So the ID lives somewhere durable; resetting it can also be a deliberate privacy feature.
- "What if the salt changes?" It only changes when someone edits it in the config, never on its own. Then every device is reshuffled for that experiment. Teams do that on purpose to restart an experiment cleanly, like after a bug ruined the first run.

## "keeps its variant across reboots with no server round trip"

This follows from deterministic hashing: every device gets the same config, and each one computes its own variant.

Why it matters:

- Scale: one identical config can sit on a CDN, and the server keeps no per-device state.
- Privacy: the server never needs to know who is in which arm.
- Offline: it works with no network.

He'll ask: "The split goes from 10% to 50%." expd recomputes on the new config. Devices already in treatment stay there; new ones join.
