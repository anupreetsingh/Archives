## Sockets

A socket is a kernel I/O object a process uses as an endpoint to talk to another process, either on the same machine or over the network. It's the same kind of object as an open file or a pipe. The kernel holds it in memory the process can't touch, and gives the process a small integer called a file descriptor (fd), like `3`, as a handle. Since all three are reached through fds, the same calls work on each: `read(3, ...)`, `write`, `close`. The process passes the fd, and the kernel does the work on the socket.

`socket(domain, type, protocol)` creates a socket and returns its fd, or `-1` on error. The TCP server below calls `listen_fd = socket(AF_INET, SOCK_STREAM, 0)`.

- `domain` chooses what kind of address the socket uses. `AF` stands for address family:
  - `AF_INET`: an IPv4 address and port, like `17.253.144.10:443`. The two processes can be on any two machines.
  - `AF_INET6`: an IPv6 address and port.
  - `AF_UNIX`: a file path, like `/var/run/expd.sock`. Both processes are on the same machine. This is a Unix domain socket.
- `type` chooses how the data moves:
  - `SOCK_STREAM`: a byte stream, which is TCP when the domain is `AF_INET`.
  - `SOCK_DGRAM`: separate datagrams, which is UDP when the domain is `AF_INET`.
- `protocol` is usually `0`, which lets the kernel pick the obvious one: TCP for a stream, UDP for datagrams.

The domain and the type together decide what kind of socket you get:

```
                     SOCK_STREAM        SOCK_DGRAM
AF_INET / AF_INET6   TCP                UDP
AF_UNIX              stream UDS         datagram UDS
```

This note is mostly about the `SOCK_STREAM` column: internet sockets with TCP, then Unix domain sockets as what changes from TCP. Each section has a short paragraph on its `SOCK_DGRAM` version. All of them use the same calls (`bind`, `connect`, `read`, `write`, `close`, plus `listen` and `accept` for streams), and the same `kqueue` loop waits on any of them. The last part covers how one thread waits on many sockets.

### Network sockets

`AF_INET` or `AF_INET6` with `SOCK_STREAM`, which is TCP. UDP (`SOCK_DGRAM`) is the paragraph at the end of Exchanging data.

```
      Server process                         Client process

        socket()                               socket()
           │                                      │
           ▼                                      ▼
        bind()                                 connect() ──┐
           │   ◀── error (ECONNREFUSED) ───────────────────┤ connect before listen()
           ▼                                      │        │
        listen()                                  │        │
           │   ◀── accept() will not block ────────────────┤ connect after listen()
           ▼                                      │        │
        accept()                                  │        │
           │   ◀── accept() blocks until here ─────────────┘ connect after accept()
           │                                      │
   ─ ─ ─ ─ ┼ ─ server switches to conn_fd from accept() ─ ─ ┼ ─ ─ ─ ─
           ▼                                      ▼
        read()   ◀┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈  write()
           │                                      │
           ▼                                      ▼
        write()  ┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈▶  read()
           │              ...                     │
           ▼                                      ▼
        close()                                close()
```

Server:

1. `listen_fd = socket(AF_INET, SOCK_STREAM, 0)` creates a TCP socket and returns its fd, like `3`.
2. `bind(listen_fd, server_ip, server_port)` attaches the server's address to `listen_fd`. `server_ip` is like `17.253.144.10` (`0.0.0.0` means all of the server's IPs). `server_port` is like `443`, usually fixed in a config file. Fails with `EADDRINUSE` if another socket already has that IP and port. `bind()` only gives the socket an address; it is not listening yet, so a client connecting now gets `ECONNREFUSED`.
3. `listen(listen_fd, backlog)` makes `listen_fd` a listening socket: the kernel starts completing handshakes for it. Separate from `bind()` because a client can also `bind()` to pick its own client port, then `connect()` instead of `listen()`. `backlog` is like `128`: the max number of finished handshakes the kernel queues while waiting for `accept()`. When full, new clients wait and retry.
4. `conn_fd = accept(listen_fd)` takes one connection off the queue and returns a new fd for the connected socket associated with that client, like `4`. The state of the server's thread changes to blocked if the queue is empty. From here on, the server uses two kinds of fd:
   - `listen_fd`: one per server port. It never carries data; it only produces new connections through `accept()`.
   - `conn_fd`: one per client. All `read()`/`write()` for that client go through it. It is the server's end of the connection; `client_fd` is the client's end.

   So a server with 1,000 clients holds 1,001 socket fds: one `listen_fd` and 1,000 `conn_fd`s. Each is an ordinary fd table entry (see Day 1, File descriptors), which is why `read()`, `write()` and `close()` work on sockets and files alike.

Client:

1. `client_fd = socket(AF_INET, SOCK_STREAM, 0)` creates the client's socket, like fd `3` in the client process.
2. `connect(client_fd, server_ip, server_port)` connects to the server, e.g. `17.253.144.10:443`. The client's kernel picks the client port itself, like `52814` (an ephemeral port). Returns when the TCP handshake finishes.

The arrows in the figure:

- If the client connects before the server calls `listen()`, nothing is accepting on that server port, so the server's kernel refuses it: `ECONNREFUSED`.
- If the client connects after `listen()` but before `accept()`, the kernel queues the connection, so `accept()` returns immediately.
- If `accept()` runs first, it blocks until a client connects.

#### Exchanging data

Once the connection is established, both sides switch from setting up to moving bytes. Everything in this part, the diagram included, describes TCP; UDP is covered briefly at the end.

One connection has two sockets, one on each end: the client's socket (`client_fd`) in the client machine's kernel, and the server's socket (`conn_fd`) in the server machine's kernel. Each socket has two buffers: a send buffer for outgoing bytes and a receive buffer for bytes that have arrived. Bytes written into one socket's send buffer are carried by the two kernels over TCP into the other socket's receive buffer. TCP resends anything lost and delivers the bytes in order:

```
  client machine                          server machine
  socket (client_fd)                      socket (conn_fd)
    send buffer     ───── network ─────▶    receive buffer
    receive buffer  ◀──── network ──────    send buffer
```

Each process only touches its own socket, through its fd, using `read()` and `write()`. These aren't socket-specific: they work on any fd, like `read(file_fd, buf, 4096)` on an open file or `write(pipe_fd, "hi", 2)` on a pipe. Here they are used on sockets, where they move bytes in and out of the socket's buffers. A request and reply go like this:

1. `write(fd, data, len)` sends bytes. It copies `len` bytes from `data` into the socket's send buffer and returns how many it copied; the kernel then transmits them to the other side. If the send buffer is full, the thread blocks until there is room. The client sends its request this way, like `write(client_fd, "hello", 5)`, which returns `5`.
2. `read(fd, buf, len)` receives bytes. `buf` is memory the caller provides, like a 4096-byte array, and `len` is its size. `read()` copies up to `len` bytes from the socket's receive buffer into `buf` and returns how many it copied. If nothing has arrived yet, the thread blocks until data comes in. The server receives the request this way, like `read(conn_fd, buf, 4096)`, which returns `5` for `hello`.
3. The server replies with `write(conn_fd, reply, reply_len)`, the same way as step 1.
4. The client receives the reply with `read(client_fd, buf, 4096)`, the same way as step 2.
5. `close(fd)` ends the connection and frees the fd. Each side calls it on its own fd, like `close(conn_fd)` on the server. The other side's next `read()` returns `0`, which is how it learns the connection is over.

`recv(fd, buf, len, flags)` and `send(fd, data, len, flags)` do the same job as `read()` and `write()`, but only on sockets. The extra argument, `flags`, turns on socket-only options. For example, `recv(conn_fd, buf, 4096, MSG_PEEK)` looks at waiting bytes without removing them from the receive buffer. With `flags` set to `0`, `recv()` is exactly `read()` and `send()` is exactly `write()`.

UDP doesn't work this way. There is no connection: no handshake, no `listen()`/`accept()`, and the server uses one socket for all clients instead of a `conn_fd` per client. `sendto(fd, data, len, flags, dest_addr)` names the destination on every call, like `17.253.144.10:53`, and sends the data right away as one packet, called a datagram. `recvfrom(fd, buf, len, flags, src_addr)` returns one whole datagram and fills `src_addr` with the sender's address, so the server knows where to reply. Nothing is resent: a lost datagram is gone, and datagrams can arrive out of order.

#### Ports and the 5-tuple

A port is a 16-bit number that names an endpoint on one machine. The IP address picks the machine, and the port picks the socket on it.

Every connection has two ports:

- Server port: chosen by the server with `bind()`, such as 443. Clients pass it to `connect()`.
- Client port: an ephemeral port the client's kernel picks during `connect()`, such as 52814.

All 1,001 sockets on the server use server port 443: `listen_fd` plus 1,000 `conn_fd`s. `listen_fd` has no client, so it only has `TCP 17.253.144.10:443`. Each `conn_fd` is told apart by its 5-tuple:

```
(protocol, client IP, client port, server IP, server port)
e.g. (TCP, 10.0.0.5, 52814, 17.253.144.10, 443)
```

Protocol is part of it because TCP and UDP have separate port spaces, so TCP port 53 and UDP port 53 are different sockets. You'll often see the 4-tuple quoted instead; that's the same thing with the protocol left implied.

The same connection has the same 5-tuple on both machines. Only the labels "source" and "destination" flip: a packet from the client has the client port as its source port, and a reply from the server has the client port as its destination port.

When a packet arrives at the server, the server's kernel looks up its 5-tuple to find the `conn_fd`, copies the bytes into that socket's receive buffer, and wakes any thread blocked on it. If no `conn_fd` matches and it's a new connection, it goes to `listen_fd`.

Ports below 1024 are privileged (traditionally root only). Client-side ephemeral ports on macOS come from 49152–65535.

### Unix domain sockets

`AF_UNIX` with `SOCK_STREAM`. The datagram version (`SOCK_DGRAM`) is the paragraph at the end of Exchanging data.

With TCP, the client and server find each other by IP and port, and two kernels carry the bytes over the network. A **UDS** (Unix domain socket) connects two processes on the same machine, like the experimentation daemon and an app asking it for a variant. Two things change: the address is a file path instead of an IP and port, and the bytes never go through the network stack.

```
      Server process              Filesystem                     Client process

        socket()                                                    socket()
           │                                                           │
           ▼                                                           │
        bind(path) ── creates ──▶ /var/run/expd.sock                   │
           │                      type: socket, holds no data          │
           │                      linked to listen_fd's socket         │
           ▼                               ▲                           ▼
        listen()                           └─── 1. look up path ─── connect(path)
           │                                                           │
           ▼                                                           │
        accept()  ◀─────────── 2. kernel queues the connection ────────┘
           │                                                           │
   ─ ─ ─ ─ ┼ ─ ─ ─ ─ ─ ─ ─ ─ path is not used after this ─ ─ ─ ─ ─ ─ ─ ┼ ─ ─ ─
           ▼                                                           ▼
        read()   ◀┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈ kernel copies bytes ┈┈┈┈┈┈┈┈┈┈┈┈┈┈  write()
           │                                                           │
           ▼                                                           ▼
        write()  ┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈ kernel copies bytes ┈┈┈┈┈┈┈┈┈┈┈┈┈▶  read()
           │                                                           │
           ▼                                                           ▼
        close()             file stays until unlink()               close()
```

#### The socket file

The socket file is a name for the server's listening socket. Its path, the socket path, is what other processes pass to `connect()` to reach the server, the way a TCP client passes an IP and port. Only processes with write permission on the file are allowed to connect, because connecting lets you send bytes into the server, which counts as writing to the server, even though nothing is ever written to the file itself. So the file does two jobs: it lets clients find the server, and its permissions decide who may connect.

The file itself takes up no space for contents. It is only an entry in the filesystem. A path lookup goes through these parts:

```
path "/var/run/syslog"
  → directory entry   "syslog" → inode 104140641   (stored in /var/run's data)
  → inode             type socket, rw-rw-rw-, owner root, size 0
  → contents          none
```

- Directory entry: the name. A directory's contents are a list of name → inode number pairs, so the name is stored in the parent directory, not in the file.
- Inode: the filesystem's record about one file. It holds everything except the name and the contents: type, size, owner, permissions, timestamps, and where the contents are on disk. `ls -i` shows its number.

A socket file has both but no contents: `stat -f "%HT %z" /var/run/syslog` prints `Socket 0`.

How the file leads to the socket, following the figure:

1. `bind(path)` creates the socket file, and the kernel links it to `listen_fd`'s socket. The link is only in kernel memory, not on disk: on macOS, the file's vnode keeps a pointer to the socket.
2. `connect(path)` makes the kernel look up the path the same way `open()` would, including the permission check, and follow the link to the listening socket. The kernel creates a new socket on the server's side, connects it to `client_fd`, and queues it on the listening socket.
3. `accept()` returns that new socket as `conn_fd`. Bytes go from one socket to the other and never touch the file. Deleting the file at this point doesn't affect the connection; only new clients can't find the server. The listening socket stays linked to the path, so the next client gets its own `conn_fd`.
4. When the server exits, the in-memory link is gone, but the name and inode stay on disk until something calls `unlink()`. A file left behind like this leads nowhere.

A path is used because both processes already share the filesystem, so it's a name both can look up, and file permissions decide who may connect.

#### Setting up

Same steps as the TCP figure. Only the address changes.

Server:

1. `listen_fd = socket(AF_UNIX, SOCK_STREAM, 0)` creates the socket. Same call as TCP, with `AF_UNIX` as the domain.
2. `bind(listen_fd, addr, addrlen)` gives the socket its address. For a UDS, `addr` is a `struct sockaddr_un` holding a path, like `/var/run/expd.sock`:

   ```c
   struct sockaddr_un addr = {0};
   addr.sun_family = AF_UNIX; // sun is for socket+unix
   // size_t strlcpy(char *dst, const char *src, size_t dstsize);
   // Copies string literal to addr.sun_path buffer.
   strlcpy(addr.sun_path, "/var/run/expd.sock", sizeof addr.sun_path); // max 104 bytes on macOS, 108 on Linux
   unlink(addr.sun_path);   // delete a stale socket file from a previous run, or bind() fails
   bind(listen_fd, (struct sockaddr *)&addr, sizeof addr);
   ```

   `bind()` creates the socket file described above. `ls -l` shows it with `s` as the first letter, like `srw-rw-rw- ... /var/run/syslog`.

   The file is not deleted when the server exits or crashes. It stays on disk, but the socket it pointed to is gone, so it's a stale name. The next `bind()` to that path then fails with `EADDRINUSE`, which here just means "a file already exists at this path". That's why servers `unlink()` the stale file first, and `bind()` then creates a new one linked to the new socket.
3. `listen(listen_fd, backlog)` and `accept(listen_fd, NULL, NULL)` work the same as with TCP. There's no handshake: `connect()` puts the client straight on the queue. One difference on macOS: when the backlog is full, a UDS `connect()` fails right away with `ECONNREFUSED` instead of waiting.

Client:

1. `client_fd = socket(AF_UNIX, SOCK_STREAM, 0)`.
2. `connect(client_fd, (struct sockaddr *)&addr, sizeof addr)` with the same path. There are no ports at all, so there's no ephemeral port either. Errors:
   - `ENOENT`: there is no file at that path (the server never started).
   - `ECONNREFUSED`: the file exists but nothing is listening on it (the server crashed and left the file behind).
   - `EACCES`: the client doesn't have write permission on the file.

That last error is the access control. To connect, a process needs write permission on the socket file, so ordinary file permissions decide who can talk to the server. `/var/run/syslog` is `rw-rw-rw-`, so any process can send log messages to it. A daemon that only root should reach makes its socket `rw-------` with `chmod(path, 0600)` after `bind()`.

#### Exchanging data

The calls are the same as with TCP. What changes is the path the bytes take between the two sockets: they skip the send buffer.

```
                        one machine, one kernel
  server process                                   client process
  socket (conn_fd)                                 socket (client_fd)
    read() ← receive buffer  ◀──── kernel copies ────  write()
    write()  ──── kernel copies ────▶  receive buffer → read()
```

`write(client_fd, "hello", 5)` copies the 5 bytes from the client's memory into the kernel, and the kernel appends them straight to the receive buffer of the server's `conn_fd`. The client's send buffer isn't used, unlike with TCP, and nothing runs after `write()` returns, because the bytes are already in place. `read(conn_fd, buf, 4096)` copies them out. There are no packets, IP headers, checksums, acks or resends, and no 5-tuple lookup either, because the kernel already knows which socket is connected to which. That's why a UDS is faster than TCP to `127.0.0.1` (loopback): with loopback both processes are also on one machine, but every byte still goes through the whole TCP/IP stack on the way out and on the way back in.

Two things carry over from TCP:

- `SOCK_STREAM` is a byte stream with no message boundaries. Two `write()`s of 5 bytes can come back as one `read()` of 10. To send messages, frame them yourself, for example by putting a 4-byte length in front of each one.
- A full receive buffer on the other side blocks `write()`. The client's send buffer holds no bytes here, but the kernel still uses it for this. As the server's receive buffer fills, the kernel lowers the send buffer's limit, and `write()` blocks when the limit reaches zero. With `kqueue` (below), `EVFILT_WRITE` tells you when there's room again.

`socket(AF_UNIX, SOCK_DGRAM, 0)` gives a datagram UDS, used with `sendto()`/`recvfrom()` like UDP. Unlike UDP, nothing gets lost or reordered, because the bytes never cross a network. Each datagram arrives as one whole message. `/var/run/syslog` is a datagram socket: each log line is one datagram.

#### Three things TCP can't do

All three work because both ends are in the same kernel.

1. Find out who is on the other end. The kernel records the client's identity when it calls `connect()`, and the server reads it from `conn_fd`:
   - `getpeereid(fd, &uid, &gid)` returns the client's user and group IDs, like uid `501` (the first user account on a Mac) or `0` (root). Works on macOS and Linux.
   - `getsockopt(fd, SOL_LOCAL, LOCAL_PEERPID, &pid, &len)` returns the client's process ID, like `4312`. macOS only.
   - `getsockopt(fd, SOL_LOCAL, LOCAL_PEERTOKEN, &token, &len)` returns the client's audit token (`audit_token_t`), which holds its uid, gid, pid and a pid version. macOS only.

   Use the audit token, not the pid. Pids get reused: if the client exits, a different program can get pid `4312`, and the server would end up checking the wrong program. The pid version tells the two apart. With the token, `SecTaskCreateWithAuditToken()` and `SecTaskCopyValueForEntitlement()` let the server check the client's code-signing entitlements, like "only processes with the `com.example.expd.client` entitlement may ask for variants". XPC does the same check (see [XPC and Mach ports](XPC%20and%20Mach%20ports.md#who-is-calling)).

   Over TCP the server only sees an IP and port, so it would need a password or certificate to know who's calling. Linux: `getsockopt(fd, SOL_SOCKET, SO_PEERCRED, &cred, &len)` returns pid, uid and gid in one call.

2. Pass an open fd to another process. `sendmsg(fd, &msg, flags)` sends normal bytes plus extra control data described by a `struct msghdr`, and `recvmsg(fd, &msg, flags)` receives both. A control message of type `SCM_RIGHTS` carries fds. The kernel adds a new entry to the receiver's fd table that points at the same open file, like the sender's fd `7` showing up as fd `5` in the receiver. Example: the daemon opens the snapshot file and sends that fd, and the client `mmap`s it without needing permission to open the path itself. The kernel has to do this: an fd number means nothing in another process's fd table, so sent over TCP it would just be 4 bytes.

3. Make an already-connected pair. `socketpair(AF_UNIX, SOCK_STREAM, 0, fds)` creates two sockets connected to each other, like `fds[0] = 3` and `fds[1] = 4`, with no path and no `bind`/`listen`/`accept`. A parent calls it before `fork()` or `posix_spawn()`: the child inherits one end and the parent keeps the other. It's like a pipe that works in both directions.

## Waiting on many sockets

### The problem: one thread, one fd

A server thread blocks in two places while serving one client:

1. It calls `accept(listen_fd)` and blocks until a client connects.
2. It gets back `conn_fd` for that client.
3. It calls `read(conn_fd, buf, 4096)` and blocks until that client sends something.

While the thread waits in step 3 on client A, it can't also be in `accept()`, so a new client B waits in the backlog queue. It also can't be in `read()` on client C's `conn_fd`, so C's data sits unread in its receive buffer. A blocked thread can only wait on one fd at a time.

There are two simple ways around this, and both are bad:

### Bad option 1: a thread per client

Bad option 1, a thread per client. One thread loops on `accept()`, and each new `conn_fd` gets its own thread that loops on `read()`. With 1,000 clients that is 1,001 threads. It wastes memory:

- Each thread costs a stack (512 KB by default for secondary threads on macOS) plus kernel bookkeeping.
- Most of those threads are idle, waiting on clients that send nothing.
- Every wakeup is a context switch.

### Bad option 2: polling

Bad option 2, polling. Polling means one thread keeps asking every fd "anything ready?" in a loop, instead of sleeping until something is. The fds are set non-blocking (`O_NONBLOCK`), so `accept()` and `read()` return right away when nothing is ready instead of blocking. The thread tries `listen_fd`, then every `conn_fd`, then starts over. It wastes CPU: the thread never sleeps, so it uses a full core even when no client is sending anything.

### The fix: readiness notification

What you want is one thread that sleeps until any of its fds (`listen_fd` plus all the `conn_fd`s) is ready, then handles just those. This is called **readiness notification**.

It works by turning polling around. In bad option 2 the thread holds the list of fds and goes to each one asking "anything ready?". With readiness notification, each socket holds a list of who is waiting on it, called its wait list, and the socket's side does the waking:

1. The thread hands the kernel the fds it cares about and goes to sleep. The kernel adds the thread to the wait list of each of those sockets.
2. Bytes arrive for one of those sockets. For TCP, a packet comes in and the kernel's network code finds the socket by its 5-tuple. For a UDS, the other process calls `write()` and the kernel already knows the connected socket. Either way, the kernel puts the bytes in that socket's receive buffer.
3. While it's there, the kernel looks at that socket's wait list and wakes whoever is on it.

Nobody loops. The kernel handles every incoming packet anyway, and waking the waiters is one extra step while it does. Plain blocking `read()` and `accept()` work the same way, with one thread on one socket's wait list.

There are two generations of readiness notification:

- `select()` and `poll()` are the older calls. Despite the name, `poll()` is not bad option 2: the thread does sleep. But on every call you pass the kernel your full list of fds, the kernel adds the thread to all of their wait lists and removes it again when the call returns, and you then check the whole list to find the ready ones. With 1,000 clients and only 2 sending data, that is 1,001 fds handled on every call.
- `kqueue` (macOS and the BSDs) and epoll (Linux) fix this. You give the kernel each fd once, and the `kqueue` attaches to those sockets and stays attached. When a socket gets data, it notifies the `kqueue`, the `kqueue` adds the event to its ready list, and then it wakes the thread blocked in `kevent()`. Each call returns only what is on that ready list, so the work depends on how many fds are ready, not on how many clients you have.

## kqueue and kevent

Flow: register interest once, then block in `kevent()` until the kernel hands back a list of ready events.

1. `kq = kqueue()` creates an event queue in the kernel and returns an fd for it, like `5`. `kq` is an ordinary fd, but it refers to that event queue rather than a socket, so you use it only with `kevent()`, not `read()` or `write()`. The server now holds three kinds of fd: `listen_fd`, the `conn_fd`s, and `kq`.
2. Register each socket fd you care about (`listen_fd`, then each `conn_fd`) with a `struct kevent` (built with `EV_SET`) naming the fd in its `ident` field, the filter (what to watch) and flags such as `EV_ADD`.
3. Call `kevent(kq, changes, nchanges, events, nevents, timeout)`. It applies the changes, then blocks the thread until at least one event fires or the timeout runs out. A blocked thread uses no CPU.
4. It returns the number of ready events and fills `events`. Each one says which fd is ready (`ident`), which filter, and extra data, such as how many bytes are readable.

kevent(kq,  changelist, nchanges,   eventlist, nevents,   timeout);
//          └── job 1: register ──┘  └── job 2: report ──┘

A minimal server loop:

```c
#include <sys/event.h>   // Gives us kqueue, kevent, struct kevent, EV_SET
#include <sys/socket.h>  // accept
#include <unistd.h>      // read, close

// ---- Calls used, in order ----
//
// int kqueue(void);
//   Creates an event queue in the kernel. Returns its fd, or -1 on error.
//
// EV_SET(&kev, ident, filter, flags, fflags, data, udata);
//   Macro that fills in a struct kevent. Doesn't talk to the kernel.
//   kev    - the struct kevent to fill
//   ident  - what to watch, usually an fd
//   filter - which event: EVFILT_READ, EVFILT_WRITE, ...
//   flags  - what to do: EV_ADD (register), EV_DELETE (remove), EV_ONESHOT (fire once)
//   fflags - filter-specific options (0 for sockets)
//   data   - filter-specific value (0 for sockets)
//   udata  - your own pointer, handed back untouched when the event fires (NULL if unused)
//
// int kevent(int kq,
//            const struct kevent *changelist, int nchanges,  // in: registrations to apply
//            struct kevent *eventlist, int nevents,          // out: ready events
//            const struct timespec *timeout);                // NULL = wait forever
//   Applies changelist, then blocks until an event is ready or timeout runs out.
//   Returns how many entries it filled in eventlist (0 on timeout, -1 on error).
//   Register only:  kevent(kq, &ev, 1, NULL, 0, NULL)
//   Wait only:      kevent(kq, NULL, 0, ready, 64, NULL)
//   Each returned struct kevent has: ident (the ready fd), filter,
//   flags (EV_EOF set if the peer hung up), data (bytes readable).
//
// int accept(int listen_fd, struct sockaddr *addr, socklen_t *addrlen);
//   Returns conn_fd for the next queued client. addr/addrlen get the client's
//   address; NULL, NULL if you don't need it.
//
// ssize_t read(int fd, void *buf, size_t len);
//   Copies up to len bytes into buf. Returns bytes read, 0 at EOF, -1 on error.
//
// int close(int fd);
//   Frees the fd. Closing a socket also removes it from every kqueue.

int kq = kqueue(); // fd for the event queue object in kernel
struct kevent ev; // Declaring an instance of the struct kevent, ev for Event
EV_SET(&ev, listen_fd, EVFILT_READ, EV_ADD, 0, 0, NULL); // the macro EV_SET fills in the fields of ev
kevent(kq, &ev, 1, NULL, 0, NULL);          // register the listening socket

struct kevent ready[64]; // An array of 64 struct kevent's
for (;;) {
    int n = kevent(kq, NULL, 0, ready, 64, NULL);   // block here
    for (int i = 0; i < n; i++) {
        int ready_fd = (int)ready[i].ident;          // listen_fd or some conn_fd
        if (ready_fd == listen_fd) {                 // a new client is queued
            int conn_fd = accept(listen_fd, NULL, NULL);
            EV_SET(&ev, conn_fd, EVFILT_READ, EV_ADD, 0, 0, NULL);
            kevent(kq, &ev, 1, NULL, 0, NULL);
        } else if (ready[i].flags & EV_EOF) {        // client hung up
            close(ready_fd);                         // a conn_fd; closing also removes it from kq
        } else {                                     // data is waiting
            char buf[4096];
            ssize_t len = read(ready_fd, buf, sizeof buf); // a conn_fd; won't block
            // handle the request, write() the reply
        }
    }
}
```

On `listen_fd`, `EVFILT_READ` means "a connection is waiting", so `accept()` won't block. On a `conn_fd` it means "bytes are waiting", and `data` holds how many. This holds for both kinds of socket: `listen_fd` can be TCP or UDS, and the loop doesn't change. A daemon can even register a TCP `listen_fd` and a UDS `listen_fd` in the same `kqueue`.

Filters beyond sockets, which is where `kqueue` goes past epoll:

- `EVFILT_READ` / `EVFILT_WRITE`: fd readable or writable.
- `EVFILT_VNODE`: a file was written, renamed or deleted.
- `EVFILT_PROC`: a process exited or forked.
- `EVFILT_SIGNAL`: a signal arrived.
- `EVFILT_TIMER`: a timer fired.
- `EVFILT_MACHPORT`: a message arrived on a Mach port. Mach ports aren't fds, so they need their own filter.

So one `kevent()` call can wait on sockets, files, child processes, timers and Mach IPC together.

### Where this shows up on macOS

- GCD dispatch sources (`dispatch_source_create`) are built on `kqueue`. A dispatch source for a socket is a `kevent` registration with the handler run on a queue.
- launchd daemons sit in `kevent()` waiting for XPC messages, which arrive on Mach ports (`EVFILT_MACHPORT`).
- libdispatch, `CFRunLoop` and Network.framework all end up blocked in `kevent()` (or `mach_msg()`) when idle. That's why an idle daemon uses zero CPU.

Linux contrast: `epoll_create1()`, `epoll_ctl()` to register, `epoll_wait()` to block. It handles fds only, so other event types need wrapping as fds first (`timerfd`, `signalfd`, `pidfd`).

### Socket activation with launchd

The daemon doesn't have to create its own listening socket. The `Sockets` key in its launchd plist tells launchd to create it:

```xml
<key>Sockets</key>
<dict>
    <key>Listeners</key>
    <dict>
        <key>SockPathName</key>
        <string>/var/run/expd.sock</string>
    </dict>
</dict>
```

1. At boot, launchd itself calls `socket()`, `bind()` and `listen()` on that path, and adds the `listen_fd` to its own `kqueue` with `EVFILT_READ`. The daemon is not running yet.
2. A client calls `connect()`. The connection waits in the backlog queue, and launchd's `kevent()` returns because `listen_fd` is readable.
3. launchd starts the daemon. The daemon calls `launch_activate_socket(name, &fds, &count)`, which gives the daemon its own fd for each listening socket launchd made for the entry called `name`. It returns an array because one entry can make several sockets (see More than one listening socket below). Here there's one: `launch_activate_socket("Listeners", &fds, &count)` gives `count = 1` and something like `fds[0] = 3`. It's the same socket object: launchd keeps its own fd to it open, and `fds[0]` is a second entry in the daemon's fd table.
4. The daemon adds `fds[0]` to its own `kqueue` and `accept()`s the waiting client, the same loop as in the kqueue section. launchd stops watching its fd while the daemon runs, so every later connection wakes the daemon's `kevent()`, not launchd's. launchd never calls `accept()`.
5. When the last client disconnects, the daemon keeps running, blocked in `kevent()` on `fds[0]`. That uses 0 CPU but still holds memory. It exits on its own idle timeout, or, if its plist sets `EnablePressuredExit`, jetsam (the kernel's memory-pressure killer) can kill it while it has no work in progress. Its fds close, launchd's fd keeps the socket alive, and launchd watches it again until the next client connects.

How launchd knows the daemon is gone, so it can watch `listen_fd` again: it watches the daemon's pid in the same `kqueue`.

1. When launchd starts the daemon, it removes `listen_fd` from its kqueue with `EV_DELETE`. It has to: the waiting client keeps `listen_fd` readable until the daemon `accept()`s it, so launchd's `kevent()` would keep returning and start the daemon again and again.
2. It adds the daemon's pid with `EVFILT_PROC` and `NOTE_EXIT`. Here `ident` is a pid, not an fd:

   ```c
   EV_SET(&ev, pid, EVFILT_PROC, EV_ADD, NOTE_EXIT, 0, NULL);   // tell me when pid exits
   ```

3. The daemon exits or crashes, and launchd's `kevent()` returns that event. launchd calls `waitpid()` to reap the daemon.
4. launchd adds `listen_fd` back with `EVFILT_READ`. If clients are already waiting in the backlog, it's readable right away and launchd starts the daemon again.

The old open-source launchd called these two steps `socketgroup_ignore()` and `socketgroup_watch()`. Apple stopped publishing launchd's source around macOS 10.10, so today's launchd works this way in concept, not necessarily line for line.

The daemon costs nothing until someone connects. A crash works like step 5: new clients wait in the queue instead of getting `ECONNREFUSED`, and launchd starts the daemon again.

ssh-agent on your Mac works this way. `/System/Library/LaunchAgents/com.openssh.ssh-agent.plist` has a `Sockets` → `Listeners` entry, and `echo $SSH_AUTH_SOCK` prints the path launchd made, like `/var/run/com.apple.launchd.eSvxQXvniH/Listeners`. ssh-agent doesn't start until `ssh` first connects to it.

Linux contrast: a systemd `.socket` unit does what the `Sockets` key does, and the daemon gets the fds with `sd_listen_fds()`, starting at fd `3`.

#### More than one listening socket

A socket is bound to exactly one address: `bind()` takes one, and a second `bind()` on the same socket fails with `EINVAL`. So a daemon reachable at several addresses has one listening socket per address. What counts as an address differs:

- TCP: domain + IP + port. `0.0.0.0:443` is one address, even though the wildcard `0.0.0.0` covers every IPv4 address on the machine. A daemon needs several sockets for several ports, like 80 and 443, or for IPv4 and IPv6 on the same port, because a socket's domain (`AF_INET` or `AF_INET6`) is fixed by `socket()`. A launchd entry with `SockServiceName` set to `443` makes both, so `count = 2`. `lsof -nP -iTCP -sTCP:LISTEN` shows this as two lines, `IPv4` and `IPv6`, for one port.
- UDS: a path. One `SockPathName` is one socket, so `count = 1`. Several UDS sockets mean several paths, usually for different permissions, like `/var/run/expd.sock` at `rw-rw-rw-` for any app and `/var/run/expd-admin.sock` at `rw-------` for root only. Each gets its own entry in the `Sockets` dict and its own `launch_activate_socket()` call:

  ```xml
  <key>Sockets</key>
  <dict>
      <key>Listeners</key>   <!-- launch_activate_socket("Listeners", ...) -->
      <dict>
          <key>SockPathName</key>
          <string>/var/run/expd.sock</string>
      </dict>
      <key>Admin</key>       <!-- launch_activate_socket("Admin", ...) -->
      <dict>
          <key>SockPathName</key>
          <string>/var/run/expd-admin.sock</string>
      </dict>
  </dict>
  ```

One daemon can also mix both kinds. PostgreSQL listens on a UDS path like `/tmp/.s.PGSQL.5432` for local clients and on TCP port `5432` for remote ones. Under launchd that's two entries, one with `SockPathName` and one with `SockServiceName`, so the daemon gets 3 listening fds: 1 for the path, plus IPv4 and IPv6 for the port. After `accept()`, it handles both kinds of `conn_fd` with the same `read()` and `write()`.

The steps above work the same with several sockets: launchd watches all of them, a `connect()` on any one starts the daemon, and the daemon adds all its fds to one `kqueue`. The `ident` of each returned event says which socket the client came in on.

### Shutting down with SIGTERM

A signal is a small number the kernel delivers to a process, like `15` for `SIGTERM` ("please exit") or `9` for `SIGKILL` ("die now", which can't be caught or ignored). OS Processes → Signals has the basics. For a daemon, what matters is how launchd stops it:

1. Something asks launchd to stop expd: `launchctl bootout system/com.example.expd`, an installer replacing it, or the machine shutting down.
2. launchd sends `SIGTERM`. Its default action ends the process on the spot, so a daemon that wants to finish its work has to change that.
3. expd stops accepting new clients, finishes or drops the requests in progress, saves any state it needs, and exits.
4. If expd is still running after `ExitTimeOut` seconds (a plist key, about 20 by default), launchd sends `SIGKILL`.

**jetsam** sends `SIGKILL` with no `SIGTERM` first, and a crash gives no warning at all. So expd's state on disk has to be consistent at every moment (write a new file, `fsync()`, then `rename()` over the old one). `SIGTERM` is only for finishing in-flight work tidily.

The classic way to catch a signal is a handler. `signal(sig, handler)` sets what happens when `sig` arrives: run `handler`, a function that takes the signal number; `SIG_IGN` to ignore it; or `SIG_DFL` for the default action. The handler runs on whatever thread the signal interrupts, wherever it is, maybe halfway through `malloc()` while holding malloc's lock. If the handler then calls `malloc()`, or `printf()`, which mallocs, it waits on a lock its own thread holds, and the process hangs. So a handler may only call async-signal-safe functions, a short list that includes `write()` and `_exit()`. In practice it only sets a flag for the main loop to check.

With `kqueue` you skip the handler and get the signal as an event instead:

1. `signal(SIGTERM, SIG_IGN)` turns off the default action, so `SIGTERM` no longer ends the process.
2. `EV_SET(&ev, SIGTERM, EVFILT_SIGNAL, EV_ADD, 0, 0, NULL)` registers for it. Here `ident` is the signal number, `15`, not an fd. kqueue still records a signal that is set to `SIG_IGN`.
3. When launchd sends `SIGTERM`, `kevent()` returns an event whose `filter` is `EVFILT_SIGNAL`. The shutdown code runs on the loop's thread like any other event, so it can call anything.

Added to the server loop from the kqueue section:

```c
signal(SIGTERM, SIG_IGN);                                  // don't exit on SIGTERM
EV_SET(&ev, SIGTERM, EVFILT_SIGNAL, EV_ADD, 0, 0, NULL);   // ident = 15, a signal number
kevent(kq, &ev, 1, NULL, 0, NULL);

// inside the for loop, checked first:
if (ready[i].filter == EVFILT_SIGNAL) {    // SIGTERM arrived
    close(listen_fd);                      // stop taking new clients
    // finish the requests in progress, save state
    exit(0);
}
```

Check `filter` before treating `ident` as an fd. An `ident` of `15` here means `SIGTERM`, but fd `15` could also be one of the `conn_fd`s.

`dispatch_source_create(DISPATCH_SOURCE_TYPE_SIGNAL, SIGTERM, 0, queue)` does the same registration through GCD and runs a handler block on `queue`. You still call `signal(SIGTERM, SIG_IGN)` first. An XPC daemon uses this one, because it sits in `dispatch_main()` instead of its own `kevent()` loop.

A socket daemon also has to deal with `SIGPIPE`. A `write()` to a socket whose other end has closed raises `SIGPIPE`, and its default action ends the process, so one client hanging up mid-reply would take expd down. `signal(SIGPIPE, SIG_IGN)` at startup, or `setsockopt(fd, SOL_SOCKET, SO_NOSIGPIPE, &one, sizeof one)` per socket on macOS, turns it into a plain error: `write()` returns `-1` with `errno` set to `EPIPE`.

Linux contrast: block the signal with `sigprocmask()`, then `signalfd()` gives an fd that becomes readable when the signal arrives, and that fd goes into epoll. systemd sends `SIGTERM`, then `SIGKILL` after `TimeoutStopSec` (90 s by default).
