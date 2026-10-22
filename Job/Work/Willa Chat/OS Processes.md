## Kernel

Kernel is the most fundamental software of your OS. It manages resources, processes, files, and networking.

Here is a brief timeline of important OS Development:

```txt
                    Bell Labs (1925, AT&T's research lab)
                      │   transistor (1947), information theory (1948)
                      ▼
                AT&T Bell Labs creates UNIX (1969) + C
                      │
        ┌─────────────┴──────────────────────────┐
        ▼                                        ▼
 Cheap licenses to universities           AT&T breakup (1984)
 (antitrust ruling stopped AT&T           → Unix goes commercial,
  selling computer products)                expensive, closed
        │                                        │
        ▼                                        ├─► SunOS/Solaris, AIX, HP-UX
 Berkeley: BSD (1977)                            │
 "Berkeley Software Distribution"                ▼
 starts as Unix add-ons,                  Reaction: free alternatives
 grows into a full Unix                          │
        │                          ┌─────────────┴────────────┐
        │                          ▼                          ▼
        │                  GNU (1983, Stallman)      Linux kernel (1991, Linus)
        │                  free tools, but its       Unix-LIKE, written from
        │                  kernel (Hurd) never       scratch, no AT&T code
        │                  got finished                       │
        │                          └────────────┬─────────────┘
        │                                       ▼
        │                              GNU/Linux → distros
        │                              (Ubuntu, Fedora, Arch:
        │                               one shared kernel)
        │
        ├─► AT&T lawsuit (1992–94) puts BSD under a legal cloud → Linux takes the lead
        │
        ├─► FreeBSD / OpenBSD / NetBSD (separate OSes, each with its own kernel)
        │
        ▼
 Mach microkernel (CMU, 1980s)  ─┐
 BSD code (FreeBSD)              ├─► NeXTSTEP (1989)
                                 │   Jobs pushed out of Apple (1985) → founds NeXT
                                 ▼
                    Apple buys NeXT (1997, ~$429M) → Jobs returns
                                 │
                                 ▼
                    Mac OS X (2001) → macOS today
                                 │
                                 ▼
          macOS = Apple UI/frameworks  on  Darwin (open source)
                                              │
                                              ▼
                        XNU kernel ("X is Not Unix"), a hybrid kernel
               ┌──────────────────┬───────────┴──────────┐
               ▼                  ▼                      ▼
         Mach part           BSD part                 IOKit
         tasks, threads,     processes, files,        drivers
         memory              networking
               │                  │
               ▼                  ▼
    ═════════════ IPC (processes talking to each other) ═════════════
               │                  │
         Mach ports           Pipes, Unix sockets, network sockets,
         (kernel queues,      shared memory, signals
          send/receive        (byte streams, portable to any Unix)
          rights)
               │
               ▼
         XPC (Cross-Process Communication)
         named services, launchd starts them on demand,
         typed messages, code-signature checks
               │
               ▼
     Willa Guardian: Electron app ⇄ native helper ⇄ Network Extension (over XPC)


 Side branch, what happened to Bell Labs:
   AT&T monopoly money → 1984 breakup → Lucent (1996) → telecom crash (2001)
   → Alcatel-Lucent (2006) → Nokia (2016): a smaller lab focused on products

```

**XNU** (X is NOT Unix) is the macOS kernel. There are 3 parts in XNU that run together.

| Part | Comes from | Handles |
|---|---|---|
| Mach | Carnegie Mellon | Tasks, threads, virtual memory, scheduling, Mach ports / IPC |
| BSD layer | FreeBSD code | Processes (PIDs), users and permissions, file systems, networking stack, sockets, pipes, signals, POSIX APIs |
| IOKit | Apple | Device drivers |

### Darwin

Darwin is the core OS under macOS and iOS: XNU, libSystem , and core daemons like launchd and notifyd. Everything above it, like AppKit, Finder, and private frameworks like `Trial.framework`, is macOS, not Darwin.

## IPC

Since each process has its private memory, **IPC** (Inter-Process Communication) is the general idea for set of OS mechanisms that let processes communicate with each other.

Each IPC mechanism does it a bit differently but in general it flows like:

Process A → [some IPC mechanism] → kernel → Process B

### Mach Port and XPC

Deep dive: [XPC and Mach ports](../Foundations/XPC%20and%20Mach%20ports.md)

On MacOS specifically, you have **Mach ports** serving as the lower level IPC built into the kernel. They serve as the building block of MacOS but are awkward to talk to and easy to get wrong so app code rarely interacts with it directly.

**XPC** (Cross Process Communication) is a higher level IPC API built on top of Mach ports. You can use it from C, objective-C or Swift.

This is how a message flows:
> Client process → XPC library → Mach port → kernel → Mach port → XPC library → service process

This is kind of like how an HTTP message flows through the OSI layers.

XPC turns your message, a dictionary of typed values, into a Mach message.

The Mach port is where that message goes. A Mach port is a message queue owned by the kernel. A process never holds the port itself; it holds a right to it:

- A send right lets you put messages in.
- A receive right lets you take messages out. Only one process can hold it.

The kernel copies the message across from one process's address space to the other's. This is how one process grants another access to something.

`launchd` handles the lookup. Services register their names with it, and when a client asks for com.willa.guardian.helper, launchd gives it a send right to that service's port. If the service isn't running, launchd starts it on demand.

The service process receives the message through its XPC listener.

### Pipes

> Process A writes → kernel pipe buffer → Process B reads

**How it's created:**

This example uses a parent and child, the simplest case. Any processes that inherit the fds can share a pipe, including two siblings (see `ls | grep` below).

```
   Parent                                    Child
┌──────────┐      ┌───────────────┐      ┌──────────┐
│  fds[1]  │ ───▶ │ kernel buffer │ ───▶ │  fds[0]  │
│ (write)  │      └───────────────┘      │  (read)  │
└──────────┘                             └──────────┘
```

1. A process calls `pipe(fds)`, where `fds` is an array of two ints. The kernel creates a small buffer (about 16–64 KB on macOS) and fills `fds` with two file descriptors: `fds[0]` is the read end, like `3`, and `fds[1]` is the write end, like `4`. Both ends share that one buffer.
2. The process calls `fork()`. The child gets copies of both file descriptors.
3. Each side closes the end it doesn't use: the parent keeps the write end, the child keeps the read end.
4. Now the parent's `write(fds[1], "hi", 2)` puts bytes into the kernel buffer, and the child's `read(fds[0], buf, 4096)` copies them out into `buf`.

Each end only works one way. `write(fds[0], ...)` on the read end fails, returning `-1` with `EBADF` (bad file descriptor). This differs from a socket, where with one fd you can both read and write on the same end because it has its own send and receive buffers.

**What happens with the shell's `ls | grep foo`:** the shell makes a pipe and forks twice. It connects `ls`'s stdout to the write end and `grep`'s stdin to the read end. Neither program knows it's using a pipe; each one just reads stdin or writes stdout.

**How it behaves:**

- One-way. For two-way communication you need two pipes.
- It's just a byte stream. If A writes "hello" and then "world", B might read "helloworld" in one go, or "hel" then "loworld". There are no message boundaries, so you have to add your own framing (newlines, length prefixes).
- Built-in back-pressure. If the buffer is full, the writer **blocks** until the reader catches up. If the buffer is empty, the reader blocks.
- End of stream: when every write end is closed, the reader gets EOF (end of file). If the reader is gone, the writer gets `SIGPIPE`.
- Related processes only, because the file descriptors have to be inherited through `fork()`.

**Named pipe (FIFO):** made with `mkfifo /tmp/myfifo`, it shows up in the file system as a special file. Any process with permission can `open()` it, so unrelated processes can use it. The data still goes through a kernel buffer and is never written to disk.

**Where you'd see it:** Electron spawning a child process and reading its `stdout`/`stderr` is pipes under the hood. The same goes for shell pipelines and `posix_spawn` with output capture.

**When to pick a pipe:** you need a one-way byte stream to or from a process you spawned. It's one call to set up, it plugs straight into stdin/stdout, and EOF, `SIGPIPE`, and cleanup come for free.

**When something else fits better:**

- Two-way communication, unrelated processes, or one server with many clients: use a Unix domain socket (next).
- Structured messages to macOS system services, with security checks and on-demand launch: use Mach ports / XPC.
- Large data where copying through the kernel is too slow: use shared memory.

Pipes stop at one direction, no names, and inherited fds. Unix domain sockets keep the same `read()`/`write()` on an fd, but go both ways and let any process connect by path.

### Unix domain sockets

Deep dive: [Sockets and kqueue](../Foundations/Sockets%20and%20kqueue.md#unix-domain-sockets)

> Client connects to a socket file → kernel links the two sockets → two-way stream (or messages)

**How it's created:**

1. Server: `socket(AF_UNIX, SOCK_STREAM)` → `bind("/tmp/app.sock")`, which creates a socket file → `listen()` → `accept()` and wait.
2. Client: `socket(AF_UNIX, …)` → `connect("/tmp/app.sock")`.
3. The kernel pairs the two sockets. `accept()` returns a new file descriptor just for this client, and both sides can `read()` and `write()`.

**How it behaves:**

- Two-way, and works between **unrelated** processes on the same machine.
- Same API as network sockets, but the data **never goes through the network stack**. There's no TCP/IP, no checksums and no routing; the kernel copies buffer to buffer. That makes it noticeably faster than TCP over localhost.
- Types: `SOCK_STREAM` is a byte stream like TCP. `SOCK_DGRAM` keeps message boundaries like UDP but is reliable on the local machine.
- Access control uses file permissions on the socket file, so `chmod` decides who can connect.
- You can ask who the caller is: `getpeereid()` / `LOCAL_PEERCRED` on macOS gives the peer's UID/GID. This is only the user, not *which app*, which is weaker than XPC's code-signature check.
- It can pass file descriptors (`SCM_RIGHTS`): one process can hand an open file or socket to another. This is the BSD version of passing Mach port rights.

**Where you'd see it:** Docker (`/var/run/docker.sock`), Postgres (`/tmp/.s.PGSQL.5432`), `ssh-agent`, and many local daemons.

### Network sockets (TCP / UDP)

Deep dive: [Sockets and kqueue](../Foundations/Sockets%20and%20kqueue.md#network-sockets)

> Process A → socket → kernel TCP/IP stack → (loopback or network card) → kernel TCP/IP stack → socket → Process B

**How it's created:** the same calls as Unix sockets, with addresses: `socket(AF_INET, SOCK_STREAM)` → `bind(0.0.0.0:8000)` → `listen` → `accept`. The client does `connect(127.0.0.1:8000)`.

**What changes compared with Unix sockets:**

- The data goes through the **full network stack**: TCP segments, sequence numbers, ACKs, checksums. On `127.0.0.1` it goes through the loopback interface and never leaves the machine, but you still pay for that processing.
- TCP is a reliable, ordered byte stream (the same framing problem as pipes). **UDP** keeps message boundaries but can drop or reorder messages.
- Identity is weak: you only know the remote IP:port. **Any local process** can connect to a port listening on localhost, whatever user or app it is. That's why local dev servers sometimes have security bugs.
- It's the only one here that crosses machines.

**Where you'd see it:** in Willa, the frontend talks to FastAPI over HTTP, and the backend talks to Postgres and Redis over TCP. When they run in separate containers, that's network-socket IPC.

### Shared memory

> Process A writes to memory ⇄ the same physical pages ⇄ Process B reads directly (no kernel copy)

**How it's created:**

1. Process A calls `shm_open("/myshm", O_RDWR | O_CREAT, 0600)` (POSIX), which creates a named shared-memory object in the kernel and returns an fd, like `6`. Then `ftruncate(6, 4096)` gives it a size. The memory belongs to this kernel object, not to A, and isn't in any process's address space yet. No physical pages exist until someone touches them.
2. Both processes call `mmap()` on it with `MAP_SHARED`. B first opens the same name with `shm_open("/myshm", O_RDWR, 0)` to get its own fd. Each `mmap()` adds a new region to that process's own address space, and the virtual addresses can differ, like `0x104a00000` in A and `0x10c200000` in B. The kernel points both processes' page tables at **the same physical RAM pages**, allocated zero-filled on the first page fault.
3. From then on, a write by A is instantly visible to B, because both addresses lead to the same RAM. There's no system call for each message: after the setup, it's plain memory reads and writes.

`shm_unlink("/myshm")` removes the name, but the memory lives on until the last process unmaps it. So it can outlive A, as long as B still has it mapped.

**How it behaves:**

- The fastest IPC there is, because nothing is copied. Every other mechanism copies data into the kernel and back out.
- No synchronization built in. If A is halfway through writing while B reads, B sees half-written data. You need **semaphores, mutexes or atomics** to coordinate, and that's where the bugs come from.
- No notification either. B doesn't know new data arrived unless you add a signal: a semaphore, a pipe byte, or a Mach message saying "go look".
- On macOS there's also the Mach way: a **memory entry port** sent over a Mach message, or XPC's `xpc_shmem`. Big data goes into shared memory, and a small message announces it.

**Where you'd see it:** video and audio pipelines, browsers (Chrome's GPU process), databases (Postgres shared buffers). It's usually paired with another IPC that sends the "it's ready" signal.

### Signals

Deep dive: [Shutting down with SIGTERM](../Foundations/Sockets%20and%20kqueue.md#shutting-down-with-sigterm)

> Sender calls `kill(pid, SIGTERM)` → kernel marks the signal pending on the target → the target is interrupted → its signal handler runs (or the default action)

**How it behaves:**

- A number, no data. SIGTERM (15) means "please exit". SIGKILL (9) means "die now" and can't be caught. SIGINT (2) is Ctrl+C. SIGHUP is often used for "reload your config". SIGUSR1/2 are free for apps to use.
- Asynchronous: the target is interrupted wherever it happens to be, so handlers can only safely do very little (set a flag, mostly).
- Permission: you can only signal processes owned by your own user, unless you're root.
- No reply and no queue. Two identical signals arriving close together can merge into one.
- On macOS the kernel actually turns hardware faults into **Mach exceptions** first, then into BSD signals like SIGSEGV. You can see both layers there.

**Where you'd see it:** `kill`, Ctrl+C, Docker stopping containers (SIGTERM, then SIGKILL after a timeout), Electron and Node `process.on('SIGTERM')`.

## /Usr vs /System

`/bin`, `/sbin` and `/usr` are the Unix part of macOS (Free BSD derived).

- `/bin` → essential commands needed to boot and recover, such as `ls`, `cp`, `sh`
- `/sbin` → essential system binaries such as `launchd` (PID 1), `mount`, `fsck`
- `/usr/bin` → other command-line tools such as `grep`, `python3`, `ssh`
- `/usr/sbin` → system binaries for daemons and admin tools, including `mDNSResponder` and `networksetup`
- `/usr/lib` → shared C libraries (`.dylib`)
- `/usr/share` → man pages and other data
- `/usr/local` → tools you install yourself, and Homebrew on Intel Macs

`/System` is the Apple-specific part.

- `/System/Library/Frameworks` → Apple frameworks such as Foundation, AppKit, NetworkExtension
- `/System/Library/CoreServices` → Finder, the Dock and login components
- `/System/Library/LaunchDaemons` → plists that tell `launchd` which system daemons to start
- `/System/Library/LaunchAgents` → plists for Apple's per-user agents
- `/System/Library/Extensions` → kernel extensions (kexts)
- `/System/Applications` → built-in apps such as Safari and Mail

`/Library`, `/Applications` and `~/Library` are the writable part, where third parties and users install things.

- `/Library/LaunchDaemons` → plists for third-party system daemons, such as Guardian's `com.willachat.guardian.dns.plist`. `launchd` loads these at boot just like `/System/Library/LaunchDaemons`; both feed the same system domain
- `/Library/LaunchAgents` → plists for third-party agents that run for every user. When a user logs in, `launchd` loads these into that user's domain, along with `/System/Library/LaunchAgents`
- `~/Library/LaunchAgents` → plists for agents that run only for you. `launchd` loads them into your user domain at login, same as the two folders above
- `/Library/Application Support` → data and helper tools that apps install for all users
- `/Applications` → apps you install, such as Xcode
- `/opt/homebrew` → Homebrew on Apple Silicon Macs

So the same kind of folder can exist in both parts: Apple's daemon plists go in `/System/Library/LaunchDaemons`, and yours go in `/Library/LaunchDaemons`.

macOS keeps these parts on two volumes:

- System volume: `/System`, `/bin`, `/sbin` and `/usr`. It's the Signed System Volume, sealed by Apple and mounted read-only, so not even `sudo` can change it. Only a macOS update replaces it.
- Data volume: everything writable, like `/Library`, `/Applications`, `/Users`, `/usr/local` and `/opt`.

Firmlinks join the two so they look like one tree. `/usr/local` sits inside `/usr` but is a firmlink to the Data volume, which is why you can write to it.

## Daemon

A daemon is a Unix convention for a background process: no terminal, long-running,
waiting to serve requests. Names often end in "d" (`sshd`, `launchd`, `httpd`).

### launchd

`launchd` ("launch daemon") is the first user-space process (PID 1); the kernel (`kernel_task`, PID 0) starts it. launchd starts and manages everything else.

Each job is described by a `.plist` (property list) file: which binary to run, as
which user, and when (at load, on demand, on a schedule), plus whether to restart it
if it dies (`KeepAlive`). Each job has a unique label, e.g. `com.apple.mDNSResponder`.

At boot, `launchd` loads every plist in the system daemon folders:
`/System/Library/LaunchDaemons` (Apple's) and `/Library/LaunchDaemons` (third-party,
e.g. Guardian's `com.willachat.guardian.dns.plist`). Jobs marked `RunAtLoad`/`KeepAlive`
start immediately; the rest start on demand.

A process managed by launchd is called a **service**. The service outlives any one process: if it dies, launchd starts a new one (new PID).

`launchctl` is the command-line tool you use to talk to launchd.

launchd is the root of the user-space process tree, but it doesn't directly start every process. It starts daemons and agents; most other processes are created by their parent with `fork()` + `exec()` or `posix_spawn()` (e.g. `Terminal → zsh → ls`). Every process traces back to launchd, and if a parent exits first, its orphaned children are reparented to launchd, which reaps them so they don't stay zombies.

### Agent

An agent is a launchd job that runs for one logged-in user, where a daemon runs for the whole system.

1. You log in, and launchd creates a per-user session (a "domain") for you.
2. It loads the agent plists from `~/Library/LaunchAgents`, `/Library/LaunchAgents` and `/System/Library/LaunchAgents`.
3. It starts each agent as your user, inside your GUI session, at load or on demand.
4. When you log out, your agents are stopped.

| | Daemon | Agent |
|---|---|---|
| Runs as | root or a system user | the logged-in user |
| When | from boot, before login | only while that user is logged in |
| GUI access | no | yes (WindowServer, UI, the user's files and keychain) |
| Plist folder | `LaunchDaemons` | `LaunchAgents` |
| Example | `logd`, `mDNSResponder` | `cloudd`, app menu-bar helpers |

Agents exist because some background work belongs to a user: it needs that user's files, keychain or screen. A root daemon has no user session, and running that work as root would be a security risk.

### dasd

`dasd` (Duet Activity Scheduler Daemon) decides when deferrable background work should run. "Duet" is CoreDuet, the framework that learns your usage patterns, like when you charge the device and which apps you open.

The split with launchd: launchd decides how a process runs (spawn, keep alive, start on demand), and dasd decides when it's a good time to run it.

> Process registers an activity → dasd queues it → dasd waits for good conditions → launchd starts the process if needed → handler runs

1. A process registers work with `xpc_activity_register(identifier, criteria, handler)`. The criteria are an XPC dictionary, like `XPC_ACTIVITY_INTERVAL` set to one day plus `XPC_ACTIVITY_REQUIRE_NETWORK_CONNECTIVITY`. A plist can also declare it under `LaunchEvents` → `com.apple.xpc.activity`, so it exists even when the process isn't running.
2. dasd weighs each activity against battery, charging, thermal pressure, network cost, whether the device is idle, and when CoreDuet predicts you'll use it next. A backup might wait until the Mac is plugged in and idle at 3 a.m.
3. When conditions fit, launchd starts the process if it isn't running, and the handler gets `XPC_ACTIVITY_STATE_RUN`.
4. While working, the handler can check `xpc_activity_should_defer()` (true if the user just picked up the device, say). It finishes with `xpc_activity_set_state()`, set to `DONE` or `DEFER`. A deferred activity goes back in dasd's queue.

iOS `BGTaskScheduler` and macOS Spotlight indexing, Photos analysis and Time Machine maintenance all end up in dasd.

Where it overlaps with launchd: a plist's `StartInterval` / `StartCalendarInterval` is launchd's own timer. It fires at the set time regardless of battery or heat. Use it when the job must run at an exact time; use an XPC activity when any sensible time will do.

Watch its decisions with `log stream --predicate 'process == "dasd"'`.

### apsd

`apsd` (Apple Push Service daemon) receives Apple Push Notification service (APNs) messages for every app and daemon on the device. APNs is Apple's server-side push service: a server hands a small message (up to about 4 KB) to Apple, and Apple delivers it to the device. It's a system daemon, from `/System/Library/LaunchDaemons/com.apple.apsd.plist`.

One daemon owns it because an open connection costs battery and radio wake-ups, so the device keeps a single long-lived TLS-over-TCP connection to APNs instead of one per app.

> Server sends a push to APNs → APNs sends it down the device's open connection → apsd receives it → apsd delivers it to the client over XPC → launchd starts the client if needed

1. A server, like an app's backend or an Apple service like Trial, sends the push to APNs, addressed with the device's token.
2. APNs sends it down the connection apsd holds. If the device is offline, APNs keeps the latest one and delivers it on reconnect.
3. apsd routes it to the local client. A daemon gets it on a Mach service named `com.apple.aps.<client>`, like triald's `com.apple.aps.triald`. Because launchd holds that port, a push can start a client that isn't running.
4. The client usually treats the push as a wake-up hint and fetches the real data over HTTPS, since a push is small.

So the network part (steps 1–2) is TCP, and the hand-off on the device (step 3) is XPC.

Watch deliveries with `log stream --predicate 'process == "apsd"'`.

### notifyd

`notifyd` delivers Darwin notifications: system-wide broadcasts from one process on the device to every process that subscribed to a name, like `com.apple.language.changed`. There's no network and no payload, only the name and an optional 64-bit state number. It's a system daemon, from `/System/Library/LaunchDaemons/com.apple.notifyd.plist`, and the C API is in `<notify.h>`.

> Process posts a name → notifyd looks up subscribers → notifyd wakes each one → launchd starts a subscriber that isn't running

1. A process posts a name with `notify_post(name)`, like `notify_post("com.apple.language.changed")` after you change the system language.
2. notifyd finds every process that registered for that name.
3. It wakes each one through what it registered with. `notify_register_dispatch(name, &token, queue, handler)` runs a block on a dispatch queue, like `notify_register_dispatch("com.apple.language.changed", &token, q, ^(int t) { recheck_targeting(); })`. There are also Mach port, file descriptor and signal variants.
4. A process that isn't running can still be started: its plist lists names under `LaunchEvents` → `com.apple.notifyd.matching`, and launchd starts it when one is posted. triald does this to re-check targeting.

Because there's no payload, the subscriber reads the current state itself after it wakes, like the new language setting.

It's a different thing from apsd: notifyd is local and broadcast, apsd brings a message from a server to one client. Neither shows anything on screen; banners are user notifications, a separate system.

Try it with `notifyutil -w com.example.test` in one terminal and `notifyutil -p com.example.test` in another.

### mDNS Responder

**mDNSResponder**(Multi Cast DNS Responder) is a daemon that starts at boot and is MacOS's single DNS serice for every app on the system.

#### What mDNSResponder does

The clearest way to see mDNSResponder's jobs is to follow a single lookup from the app that starts it to the answer that comes back.

1. **An app asks for a name.** A browser that wants `example.com` calls a standard system function such as `getaddrinfo`. That function lives in macOS's system library, but it doesn't do any DNS work itself. It forwards the request over a local socket at `/var/run/mDNSResponder` to the daemon. This hand-off is why **one daemon serves every app** on the system.

2. **mDNSResponder checks its cache.** Every DNS answer comes with a **TTL** (time to live), which says how many seconds the answer stays valid.
   - If a cached answer exists and its TTL hasn't run out, mDNSResponder returns it immediately and nothing goes onto the network.
   - This makes repeat lookups almost instant. It's also why a changed DNS record can take a while to show up on your Mac.
   - You can clear the cache by hand with `sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder`.

3. **mDNSResponder decides which DNS server to ask.** A Mac can have several DNS configurations active at once:
   - Wi-Fi supplies one.
   - A VPN may add another for its company domains.
   - An encrypted-DNS profile can replace both.
   - Files in `/etc/resolver/` can send a particular domain to a server you choose.

   macOS merges all of these into one list of resolvers, and mDNSResponder picks the **most specific one** that matches the name. You can see that list with `scutil --dns`. This step is where an `/etc/resolver` file takes effect, and because every app goes through this daemon, the file **applies to all apps at once**.

4. **mDNSResponder sends the query**, and the name decides how it goes:
   - An ordinary name like `example.com` goes by **unicast** to the chosen server. Usually that's UDP on port 53. If the answer is too large for UDP, it retries over TCP, and if an encrypted-DNS profile is installed, it uses DNS-over-HTTPS or DNS-over-TLS instead.
   - A name ending in `.local` has no server to ask, because these names exist only on your local network. mDNSResponder **multicasts** the question to 224.0.0.251:5353, and the device that owns the name answers. Apple calls this **Bonjour**, and it's how your Mac finds `Living-Room-TV.local` without anyone setting up DNS.

5. **mDNSResponder caches the answer and returns it.** It keeps the answer for the length of its TTL and hands it back to the app, which can then open its connection.

#### Bonjour also finds services, not just names

Through **DNS Service Discovery (DNS-SD)**, devices announce what services they offer over the same multicast group.

- AirPlay speakers announce `_airplay._tcp`, and printers announce `_ipp._tcp`.
- That's how the AirPlay and print menus fill themselves in.
- You can watch these announcements live with `dns-sd -B _services._dns-sd._udp`.

#### Some tools skip mDNSResponder, which matters for testing

> **`dig` and `nslookup` don't go through mDNSResponder.** They read `/etc/resolv.conf` and send their own queries straight to a server, so they skip the cache and ignore `/etc/resolver` files. A test with `dig` can therefore disagree with what real apps see.

To check the path apps actually take, use one of these:

- `dscacheutil -q host -a name example.com`
- `dns-sd -G v4 example.com`

This is worth remembering when testing Guardian's DNS blocking in the VM macOS.
