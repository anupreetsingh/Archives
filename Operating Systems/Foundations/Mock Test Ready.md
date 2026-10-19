## Stop 0

Stop 0: What happens when a program runs
The flow

launchd ──posix_spawn()──► the kernel creates a PROCESS
                           (address space, fd table, credentials, first thread)
                                     │
                                     ▼
                  a THREAD runs your code in USER MODE  ◄─────────────────┐
                  (ordinary memory reads and math: nanoseconds)            │
                                     │                                     │
                                     │ needs a file, socket, other process │
                                     ▼                                     │
                  SYSCALL: a trap into KERNEL MODE                         │
                                     │                                     │
                 ┌───────────────────┴──────────────────┐                  │
          can finish now                    has to wait (data, disk)       │
                 │                                       │                 │
          return (~0.1–1 µs)              thread BLOCKS; the scheduler     │
                 │                        CONTEXT-SWITCHES to another      │
                 │                        thread until data arrives        │
                 └───────────────────┬───────────────────┘                 │
                                     └───────── back to user mode ─────────┘
Every OS idea this week builds on this loop. Here are the pieces, in the order the flow reaches them.

1. Process: the unit of isolation and ownership
When launchd starts a program, the kernel creates a process. A process is a container that owns:

A virtual address space. The process sees its own memory and nobody else's. Stop 5 explains how.
A file descriptor table. This holds its open files, sockets and pipes.
Credentials. These are the user and group IDs, and on Apple platforms also the code signature and entitlements. Entitlements are how a system service decides which processes may talk to it.
One or more threads.
Isolation is the main point. A crash or memory bug in one process can't corrupt another. The cost is that two processes can't just share variables. To cooperate they need IPC (Stop 2).

2. Thread: the unit the scheduler runs
A thread is a path of execution. Each thread has its own registers (including the program counter and stack pointer), its own stack, and its own scheduling priority. Everything else, including heap memory, globals and open fds, is shared with the other threads in the same process.

Sharing makes communication between threads almost free. It's also why threads need synchronization: two threads writing the same memory without a lock is a data race (Stop 4).

In C++, std::thread sits on top of pthread_create(). On macOS each pthread is a Mach thread. On Linux it's a task created with clone().

3. User mode vs kernel mode
The CPU runs your code in a restricted mode: EL0 on ARM64 (Apple silicon), ring 3 on x86. In that mode the code can compute and touch its own memory, but it can't reach hardware, other processes' memory or kernel data. The kernel runs in a privileged mode (EL1 on ARM64, ring 0 on x86).

4. Syscalls: the doorway between the two
When code needs something outside its own memory, it makes a syscall. Here's what happens during read(fd, buf, n):

The libc wrapper puts the syscall number and arguments in registers.
A trap instruction (svc on ARM64) switches the CPU into kernel mode.
The kernel looks up fd in this process's fd table and checks permissions.
If the data is ready (say, already cached in memory), the kernel copies it into buf and returns. Otherwise the thread blocks (point 6).
The return value goes back in a register, and the CPU drops back to user mode.
Cost: a simple syscall takes roughly 100 ns to 1 µs, compared with about 1 ns for an add and well under 100 ns for a memory read. Crossing into the kernel is cheap, but not free.

A detail interviewers like: reading the clock doesn't enter the kernel. The kernel maps a read-only page into every process holding time data. Apple calls it the commpage; Linux calls it the vDSO. So mach_absolute_time() and clock_gettime() cost tens of nanoseconds.

5. File descriptors: one handle for everything
An fd is a small integer that indexes the process's fd table. Each entry points to a kernel open file description (offset, flags), which points to the actual object: a file, a socket or a pipe. On Unix almost everything is an fd. That's why a single event loop (kevent on macOS, epoll_wait on Linux) can wait on files, sockets and pipes at once (Stop 3).

macOS exception: Mach ports aren't fds. That's why kqueue (kernel queue) has a separate EVFILT_MACHPORT filter.

Connection to your resume: flock() locks belong to the open file description, not the fd. That detail is what makes Role Radar's PID lock behave the way it does.

6. Blocking, scheduling and context switches
A thread is always in one of three states:

running: on a CPU core right now
runnable: ready, waiting for a core
blocked: waiting for something (data, a lock, a timer)
A blocked thread uses zero CPU. So the right way to wait is to block in the kernel (kevent, recv, std::condition_variable::wait), never to poll in a loop. Polling wakes the CPU over and over, which wastes power. Apple cares about this a great deal.

When the running thread blocks or uses up its time slice, the scheduler context-switches: it saves that thread's registers and loads another's.

Between threads of one process: about 1–2 µs.
Between processes: the kernel also switches page tables. The CPU's caches and TLB (the cache of address translations, covered in your Computer Architecture note) are full of the old process's data, so the new process starts cold. The real cost is a few µs plus a slowdown afterwards.
Tying it to your project: 90 ns vs 30 µs
The mmap lookup (~90 ns). The snapshot is mapped into the app's own address space. A lookup is a hash plus a few memory reads, all in user mode: no syscall, no switch. Only the first touch of a page costs more, because of a page fault (Stop 5).

The socket lookup (~30 µs).

The app calls send(), a syscall. The kernel copies the bytes into the socket buffer.
The daemon's thread is blocked in kevent(). The kernel marks it runnable.
The scheduler switches to the daemon's process: a context switch, starting cold.
The daemon's kevent() returns, then recv() (another syscall) copies the request in, the daemon does the lookup, and send() (another syscall) sends the reply.
The app's blocked recv() thread wakes up: a switch back.
That's at least four syscalls, two process switches, two thread wakeups and two data copies. That's where 300× comes from, and it's why the read path should stay inside the caller's process.

macOS specifics, with Linux for contrast
XNU, the kernel of macOS and iOS, is a hybrid of three parts:

Mach: tasks, threads, virtual memory, scheduling, and Mach ports (kernel-managed message queues; XPC and launchd's MachServices are built on them).
BSD: processes, PIDs, POSIX, fds, sockets, signals, file systems.
IOKit: the driver framework.
So a macOS process is a BSD process on top of a Mach task. iOS uses the same kernel with tighter rules (no swap, jetsam, sandboxing everywhere), which come up later.

Linux is one monolithic kernel, and processes and threads are both "tasks." clone() flags decide what's shared: share the address space (CLONE_VM) and you have a thread; share nothing (copy-on-write) and you have a new process, which is what fork() gives you.

Sound bite for the interview
"A process is the unit of isolation and resource ownership; a thread is the unit of scheduling. Crossing a process boundary costs syscalls and context switches, so the hot read path for experiment values should stay inside the caller's process, and only rare work like config updates and exposure batches should cross into the daemon."

Check questions (answer in your own words, without notes)
Why is the experimentation logic a separate daemon process instead of code linked into every app that reads experiment values? Think about what's isolated and what's shared.
Name what happens in the 30 µs socket lookup that doesn't happen in the 90 ns mmap lookup.
Two threads in one process, compared with two separate processes: which pair can read each other's memory directly, and what does each pair need to coordinate safely?
Your daemon has no clients right now and is sitting in kevent(). How much CPU is it using, what state is its thread in, and why is that better than checking for clients every 10 ms?

## Stop 1
