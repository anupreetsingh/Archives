## libdispatch

libdispatch, also called **GCD** (Grand Central Dispatch), is Apple's C library for running work on threads. It ships in libSystem as one of Apple's extras beyond POSIX, so every macOS and iOS process has it. Instead of creating threads with `pthread_create()`, you hand it blocks of code and it runs them on threads it manages. XPC is built on it.

Flow: you submit a block → it waits on a queue → libdispatch takes a free thread from its pool → the thread runs the block → the thread goes back to the pool.

The pieces, in that order:

- Block: Apple's C extension for closures, a function written inline that can use the local variables around it, like a C++ lambda. `^{ printf("%d\n", n); }` captures `n`. A block is a function pointer plus its captured variables, which are addresses in the process that made it, so it only means anything inside that process.
- Queue (`dispatch_queue_t`): a FIFO list of blocks waiting to run.
  - Ownership: it's held in the process's memory, so it only ever holds that process's blocks. A process can have many queues, each holding many blocks. Another process can't add to them: an XPC client sends a message, which is data, not code, and expd's own handler block goes on expd's queue.
  - Kinds: a serial queue runs one block at a time, in order. A concurrent queue can run several at once. The main queue belongs to the main thread, where an app's UI runs.
- Thread pool: threads created once and reused. Each one waits at zero CPU, runs a block, then goes back to waiting, which saves creating a thread (a syscall and a 512 KB stack) per block. There's one thread pool per process, and every queue in that process lines up for it. The kernel sizes each pool to the number of cores and adds threads when some are blocked, so 50 serial queues on an 8-core Mac might share about 8 threads. It also balances threads across processes by QoS.

The queue holds the work: what to run, in what order, and the serial or concurrent rule. The pool holds the workers that actually run it.

```
 inside one process, e.g. expd

 queue A (serial)      [a1][a2][a3]  ─┐
 queue B (serial)      [b1][b2]      ─┼─▶  libdispatch picks the next block  ─▶  pool: T1 T2 T3 ... T8
 queue C (concurrent)  [c1][c2][c3]  ─┘    that's allowed to run
```

`dispatch_async(A, a1)` adds `a1` to queue A. A isn't running anything, so libdispatch hands `a1` to an idle thread, like T3. When T3 finishes, it takes the next allowed block from any queue. Serial queue A gets at most one thread at a time, so `a2` waits for `a1` even if other threads are idle, and may then run on T5. Concurrent queue C can run `c1`, `c2` and `c3` on three threads at once.

So a serial queue promises that its blocks run one after another, not that they all run on the same thread. Because a serial queue never runs two of its blocks at once, it can **guard state** the way a mutex does: if every access goes through one serial queue, no lock is needed. expd's XPC server runs all its handlers on one serial queue for this reason (see [XPC and Mach ports](XPC%20and%20Mach%20ports.md#server)).

### Running a block

1. `dispatch_queue_create(label, attr)` creates a queue, like `dispatch_queue_create("com.example.worker", DISPATCH_QUEUE_SERIAL)`. The label shows up in crash logs and the debugger.
2. `dispatch_async(queue, block)` puts the block on the queue once and returns right away, like `dispatch_async(q, ^{ load_file(path); })`. The block runs once later, on a pool thread. Use it when you already have the work in hand. For work that should run each time something happens, use a source (below).
3. `dispatch_get_main_queue()` returns the main queue. An app uses it to get back onto the main thread after background work.

```c
dispatch_queue_t q = dispatch_queue_create("com.example.worker", DISPATCH_QUEUE_SERIAL);
dispatch_async(q, ^{
    char *data = load_file(path);                  // slow, off the main thread
    dispatch_async(dispatch_get_main_queue(), ^{
        show(data);                                // back on the main thread for UI
    });
});
```

### Event sources: libdispatch replaces your kevent() loop

A source (`dispatch_source_t`) is a libdispatch object that watches one thing for one kind of event, like "`conn_fd` has data to read". Like queues, it lives in the process's memory. A source holds four things:

1. What it watches (the handle): an fd like `conn_fd`, a mach-port name like `0x2207`, a pid, or a signal number. It ends up in the kevent's `ident` field.
2. Which event: the type and mask, like `DISPATCH_SOURCE_TYPE_READ` or `DISPATCH_PROC_EXIT`. These become the kevent's `filter` and `fflags`.
3. The handler block: the code to run.
4. The target queue: where that block runs.

What a source does, in order:

1. Watch: the source watches its handle for its event, like `conn_fd` getting data.
2. Event fires: the event happens on the handle, not in the block. The block hasn't run yet: it's the response the source holds.
3. Enqueue: the source puts its block on its target queue, the one set by the last argument of `dispatch_source_create()`. You never call `dispatch_async()` for it: libdispatch does that on each event.
4. Run: when a pool thread is free and the queue's rule allows it (a serial queue runs one block at a time), the thread takes the block and runs it.
5. Repeat: the source keeps watching, so the next event puts the block on the queue again.

Behind the scenes, creating a source adds one registration to a kqueue, like those in the kqueue note: the handle, the filter for its event, and the source's address in `udata`. Every source in the process goes into the same kqueue, so `conn_fd` with `EVFILT_READ`, mach-port `0x2207` with `EVFILT_MACHPORT` and `SIGTERM` with `EVFILT_SIGNAL` can sit in it at once. When an event fires, the kernel wakes a pool thread, libdispatch uses `udata` to find the source, and it puts the source's block on its queue. So libdispatch takes the place of your `for (;;)` loop. On macOS the kernel creates this kqueue for the process, so it has no fd and no thread waits in `kevent()`.

| kqueue note | libdispatch |
|---|---|
| `EV_SET` + `kevent()` to register | `dispatch_source_create()` |
| the code after `kevent()` returns | the handler block |
| your `for (;;)` loop | the kernel waking a pool thread when an event fires |

#### Using a source in code

Calls, in the order you use them:

1. Create: `dispatch_source_create(type, handle, mask, queue)` creates a source. `handle` is what to watch, `mask` narrows which events (`0` for fds and mach-ports), and `queue` is where the handler runs, like `dispatch_source_create(DISPATCH_SOURCE_TYPE_READ, conn_fd, 0, q)`. Underneath, it's an `EV_SET` with `EVFILT_READ`.
2. Set the handler: `dispatch_source_set_event_handler(source, block)` sets the block that runs on `queue` each time the event fires. If the event fires again before the block gets a thread, the block runs once, not once per event. `dispatch_source_get_data(source)` says how much is waiting: for a read source, the bytes available.
3. Start: `dispatch_resume(source)` starts the source. Sources start suspended so you can set the handler first.
4. Stay alive: `dispatch_main()` parks the main thread for good. A daemon that only reacts to events calls it to stay alive while libdispatch runs the handlers.

```c
dispatch_source_t src = dispatch_source_create(DISPATCH_SOURCE_TYPE_READ, conn_fd, 0, q);
dispatch_source_set_event_handler(src, ^{
    char buf[4096];
    ssize_t len = read(conn_fd, buf, sizeof buf);  // won't block: data is waiting
    // handle the request
});
dispatch_resume(src);
dispatch_main();   // never returns
```

XPC is a dispatch source with a mach-port where the fd was. When you create a connection, XPC registers its mach-port with libdispatch using `EVFILT_MACHPORT`. When a message arrives, the kernel wakes a pool thread, XPC calls `mach_msg()` to receive it and decodes it into a dictionary, and libdispatch runs your handler block on the queue you gave the connection.

Linux contrast: libdispatch is open source and ships with Swift on Linux. There, and on older macOS, it uses the model from the kqueue note directly: one ordinary fd made with `kqueue()` (`epoll_create1()` on Linux), and a manager thread that waits in `kevent()` (`epoll_wait()`) and hands events to the pool.
