POSIX (Portable Operating System Interface) is a standard for Unix-like OS interfaces, specified mostly as C APIs. It's a document, not code: it says "a conforming OS provides `fork()`, `open()`, `read()`, `posix_spawn()` with these signatures and these behaviors."

It covers:

- files: `open()`, `read()`, `write()`, `close()`
- processes: `fork()`, the `exec` family (e.g. `execve()`), `waitpid()`
- memory: `mmap()`, `munmap()`
- shared memory: `shm_open()`, `sem_open()`
- threads: `pthread_create()`, `pthread_mutex_lock()`
- signals: `sigaction()`, `kill()`
- paths and directories, time, sockets, and other Unix-style interfaces

POSIX builds on ISO C (the C language standard and its library, e.g. `printf`, `malloc`, `strlen`) and includes all of it.

## libc

`libc` is the library that implements these standards, usually by making system calls into the kernel. "libc" names the role; each OS ships its own:

- `glibc`, the GNU C Library, default on most Linux.
- `libSystem`, Apple's, default on macOS / iOS. It calls into XNU kernel for kernel mode commands. Besides libc, it bundles libraries that every Mac program gets: `kqueue()` (BSD event notification), libdispatch (GCD, like `dispatch_async()`) and libxpc (XPC).

All provide the same standard interfaces, so `printf` and `read` behave the same on each. They differ in:

- The kernel underneath: `glibc` makes Linux syscalls; `libSystem` makes XNU BSD syscalls and Mach traps. This is the main reason they can't be swapped.
- Extras beyond the standards: `glibc` adds GNU extensions (e.g. `asprintf()`, enabled via `_GNU_SOURCE`); `libSystem` adds the libraries above, plus `os_log()`.

## Function vs. syscall

A POSIX function and the syscall behind it often share a name: `read()` in libc is a thin wrapper that invokes the `read` syscall. But not every function maps 1:1 to a syscall: `printf()` formats and buffers in userspace, and only enters the kernel when it calls `write()`.

## Core calls: how a shell runs `ls | grep md`

The process calls are easiest to follow through one command. When you type `ls | grep md` in zsh, the flow is: zsh makes a pipe, starts two children, wires each child's stdout or stdin to the pipe, swaps each child into its program, then waits for both to finish.

1. `pipe(fds)` is for sending bytes from one process to another, usually between processes that share a parent. It creates a one-way byte channel in the kernel and fills `fds` with two fds for it, like `fds[0] = 3` (read end) and `fds[1] = 4` (write end). Bytes written to `fds[1]` come out of `fds[0]`.

2. `fork()` is how a process starts another process. It creates a child that is a copy of the caller: same program, same memory (copied lazily, copy-on-write), same open fds, same bootstrap port. It returns twice: `0` in the child, and the child's PID, like `812`, in the parent. zsh forks once for `ls` and once for `grep`, and both children inherit fds `3` and `4`.

   Only the thread that called `fork()` exists in the child. On macOS, that's why a forked child must `exec()` before using frameworks like CoreFoundation, which crash with `__THE_PROCESS_HAS_FORKED_AND_YOU_CANNOT_USE_THIS_COREFOUNDATION_FUNCTIONALITY___YOU_MUST_EXEC__`.

3. `dup2(oldfd, newfd)` is for redirecting: making an fd number a program already uses, like stdout, point somewhere else. It makes `newfd` refer to the same open file as `oldfd`, closing whatever `newfd` was before. The open file is the kernel's record of one `open()` (or `pipe()`), holding the current offset and flags; in XNU it's a `fileglob`. After `dup2()`, both fds share that record. In the `ls` child, `dup2(4, 1)` points stdout (fd `1`) at the pipe's write end. In the `grep` child, `dup2(3, 0)` points stdin (fd `0`) at the read end. Each child then calls `close(fd)` on `3` and `4`, since it no longer needs the originals.

4. `execve(path, argv, envp)` is how a process runs a different program. It replaces the calling process's program with a new one: `path` is the file to run, `argv` its arguments, `envp` its environment, like `execve("/bin/ls", (char*[]){"ls", NULL}, environ)`. It doesn't return on success. The PID stays the same, and open fds survive unless they were opened with `O_CLOEXEC`. So `ls` starts with stdout already on the pipe, and never knows it isn't writing to a terminal. The other `exec` variants, like `execvp()`, are libc wrappers that search `PATH` and then call `execve()`.

5. `exit(status)` ends the calling process and tells its parent how it went: `0` for success, nonzero for failure, like `exit(1)`. It runs `atexit()` handlers and flushes stdio buffers, then calls `_exit(status)`, the syscall that actually ends the process. The kernel closes its fds; when `ls` exits, the pipe's write end closes and `grep` reads end-of-file.

6. `waitpid(pid, &status, options)` is how a parent learns that a child finished and how it went. It blocks until the child ends and collects its exit status, like `waitpid(812, &status, 0)`. `options` is `0` to block, or `WNOHANG` to return right away if the child is still running. `WIFEXITED(status)` says it exited normally and `WEXITSTATUS(status)` gives the code, like `0`. Until the parent calls it, a dead child stays as a zombie: an entry in the process table holding only its exit status. zsh waits on both children, then prints the next prompt.

`posix_spawn(&pid, path, file_actions, attrp, argv, envp)` is for starting a new program in a child process with one call. It does steps 2 to 4 together: `file_actions` lists the `dup2()`s and `close()`s to do in the child, and `attrp` sets things like the signal mask. For example, `posix_spawn_file_actions_adddup2(&fa, 4, 1)` then `posix_spawn(&pid, "/bin/ls", &fa, NULL, argv, environ)`. On macOS it's a real XNU syscall and the one Apple prefers: it never copies the parent, so it's faster and avoids the fork-then-frameworks problem above.

## Other calls

Almost every call below returns `-1` on failure and sets `errno` to say why, like `ENOENT` (no such file). `strerror(errno)` turns it into a message, like `"No such file or directory"`.

### Files

1. Open: `open(path, flags, mode)` is how you get access to a file. It returns an fd, a small number the other file calls take to say which file. The fd is an index into the process's fd table, and that slot points at the kernel's open-file record (offset and flags), which points at the file's vnode. `flags` says how to open it, like `O_RDWR` (read and write), `O_CREAT` (create it if missing) and `O_CLOEXEC` (close it on `execve()`). `mode` sets permissions if the file gets created. Like `open("/tmp/log", O_RDWR | O_CREAT | O_CLOEXEC, 0644)` → `5`.
2. Write: `write(fd, buf, len)` sends bytes out through an fd, into a file, pipe or socket. It copies `len` bytes from `buf` at the fd's current offset, moves the offset forward, and returns how many it wrote. Like `write(5, "hi\n", 3)` → `3`. For a file, the bytes land in the page cache, and the kernel writes them to disk later.
3. Seek: `lseek(fd, offset, whence)` is for jumping to a position in a file, since `read()` and `write()` always work at the fd's current offset. It moves the offset and returns the new one. `whence` is `SEEK_SET` (from the start), `SEEK_CUR` (from here) or `SEEK_END` (from the end). Like `lseek(5, 0, SEEK_SET)` → `0`, back to the start.
4. Read: `read(fd, buf, len)` brings bytes in through an fd, from a file, pipe or socket. It copies up to `len` bytes into `buf` and returns how many it got, with `0` meaning end-of-file. Like `read(5, buf, 4096)` → `3`.
5. Sync: `fsync(fd)` is for making sure data survives a crash or power loss. It blocks until the file's data is on disk, not just in the page cache, like `fsync(5)`. On macOS, `fcntl(5, F_FULLFSYNC)` is needed to also flush the drive's own cache.
6. Close: `close(fd)` tells the kernel you're done with an fd. It frees the fd number for reuse, like `close(5)`. When the last fd for a pipe's write end closes, the reader gets end-of-file.

The same calls work on pipes and sockets, because they're all fds.

Two calls work on any open fd:

- Info: `fstat(fd, &st)` is for learning about a file without reading it, like its size before allocating a buffer. It fills `st` with the file's details: `st.st_size` (bytes, like `3`), `st.st_mode` (type and permissions, like `0644`) and `st.st_mtime` (last change). `stat(path, &st)` does the same by path.
- Settings: `fcntl(fd, cmd, arg)` is for changing how an fd behaves after it's open. It reads or changes the fd's settings, like `fcntl(5, F_SETFL, O_NONBLOCK)` so `read()` returns `-1` with `EAGAIN` instead of blocking when there's no data.

### Paths and directories

- Remove: `unlink(path)` deletes a file by removing its name from its directory, like `unlink("/tmp/log")`. It's what `rm` calls. The data is freed once no name and no open fd refer to it.
- Rename: `rename(old, new)` moves or renames a file in one atomic step, replacing `new` if it exists. Like `rename("/tmp/log.tmp", "/tmp/log")`: apps write a temp file and rename it so readers never see a half-written file.
- Permissions: `chmod(path, mode)` controls who can read, write or run a file. `mode` is octal digits for owner, group and others, each adding read `4`, write `2` and execute `1`. Like `chmod("/tmp/log", 0600)` (owner read/write only).
- Make: `mkdir(path, mode)` creates a directory, like `mkdir("/tmp/cache", 0755)`. `rmdir(path)` removes one, but only if it's empty.
- Working directory: every process has a current directory that relative paths start from. `getcwd(buf, len)` writes it into `buf`, like `"/Users/manpreetsingh"`, and `chdir(path)` changes it, like `chdir("/tmp")`. The shell's `cd` is `chdir()`.

Listing a directory is for finding out what's in it, the way `ls` does:

1. Open: `opendir(path)` opens a directory for reading and returns a `DIR*` handle, like `opendir("/tmp")`.
2. Read: `readdir(dir)` returns the next entry, with `entry->d_name` like `"log"`, or `NULL` when there are no more. You call it in a loop.
3. Close: `closedir(dir)` frees the handle.

### Memory

Every process has its own virtual address space, and `mmap()` adds a new region to that space. Most programs don't call these directly: `malloc()`, the dynamic loader and file APIs call them underneath.

1. Map: `mmap(addr, len, prot, flags, fd, offset)` maps a kernel VM object (`vm_object`) into your address space, so you can use its pages through a pointer instead of `read()` and `write()`. A VM object is the kernel object that records which physical pages hold some data, and where to get a page that isn't in RAM yet. Which VM object gets mapped depends on what `fd` refers to:

   - A regular file, like `/tmp/log`: the VM object attached to the file's vnode, the kernel object for that file. Pages not in RAM are read from the file on disk.
   - A shared-memory object from `shm_open()` (below): the VM object it holds. Pages not in RAM yet are zero-filled.
   - Nothing, with `MAP_ANONYMOUS` and fd `-1`: a new VM object made just for this mapping. Pages are zero-filled.

   XNU adds an entry to the process's VM map, the kernel object that lists the process's address ranges, pointing that range at the VM object. `mmap()` returns a pointer to the start of the range.

   `addr` is usually `NULL` (the kernel picks a free gap), `len` is how many bytes, `prot` sets access like `PROT_READ | PROT_WRITE`, `flags` is `MAP_PRIVATE` (writes go to your own copy of the page) or `MAP_SHARED` (writes go to the shared pages, so every process mapping the same file or shared-memory object sees them, and a file's changes are written back to it), and `offset` is where in the file the mapping starts, a multiple of the page size. Like `mmap(NULL, 4096, PROT_READ | PROT_WRITE, MAP_SHARED, 5, 0)` → `0x104a00000`.

   Nothing is read when you call it. Each page is filled on first touch, through a page fault. For a file, that page comes from the page cache, so there are no `read()` calls or copies.
2. Protect: `mprotect(addr, len, prot)` is for changing what the process may do with memory it already has mapped. It changes the access on those pages, like `mprotect(p, 4096, PROT_READ)` to make them read-only, and a write after that crashes with a signal. Guard pages use it to catch overflows, and JIT compilers use it to switch code pages between writable and executable.
3. Advise: `posix_madvise(addr, len, advice)` is for helping the kernel manage your pages. It tells the kernel how you'll use them, like `POSIX_MADV_SEQUENTIAL` so it reads ahead, or `POSIX_MADV_DONTNEED` when you're done with them for now. It's only a hint, so nothing breaks if the kernel ignores it.
4. Lock: `mlock(addr, len)` is for memory that must never be written out to disk. It keeps the pages in RAM so they're never paged out, like `mlock(key, 32)` so a secret key never lands in swap. `munlock(addr, len)` undoes it.
5. Sync: `msync(addr, len, flags)` is the `fsync()` of mapped files: it makes sure changes made through a `MAP_SHARED` file mapping reach the file. It writes the changed pages back, like `msync(p, 4096, MS_SYNC)`, which blocks until done.
6. Unmap: `munmap(addr, len)` is for giving memory back when you're done. It removes the region from your address space, like `munmap(p, 4096)`, and touching it after that crashes. `free()` calls it for large blocks.

Besides the shared-memory calls below, these are all of POSIX's memory calls. `malloc()` and `free()` come from ISO C, and libc builds them on anonymous `mmap()`. The old heap calls `brk()` and `sbrk()` were removed from POSIX.

### Shared memory

A shared-memory object is a kernel structure that holds three things: a name, like `/myshm`, a size, and a VM object with the memory's pages. No process owns it. Two processes share memory by both mapping it, so both of their address ranges point at its physical pages. How processes use it is in OS Processes.

1. Create: `shm_open(name, oflag, mode)` is for making memory that other processes can find by name. It creates the shared-memory object, or opens the existing one with that name, and returns an fd for it that you can `mmap()`. `name` starts with `/`, `oflag` is like `open()`'s flags, and `mode` sets permissions. Like `shm_open("/myshm", O_RDWR | O_CREAT, 0600)` → `6`.
2. Size: `ftruncate(fd, len)` sets the size of a file or shared-memory object, like `ftruncate(6, 4096)`. A new shared-memory object starts at size `0`, so it has to be sized before it's useful. On macOS this works only once per object.
3. Map: each process calls `mmap()` on the fd with `MAP_SHARED`, like `mmap(NULL, 4096, PROT_READ | PROT_WRITE, MAP_SHARED, 6, 0)`. A write by one process shows up in the other with no copying.
4. Unmap: `munmap(addr, len)` when a process is done, like `munmap(p, 4096)`.
5. Remove: `shm_unlink(name)` is for cleanup. It removes the name so no new process can open it, like `shm_unlink("/myshm")`. The memory is freed once every process has unmapped it.

Shared memory has no locking of its own. To stop two processes writing at once, use a named semaphore, a counter that processes wait on and post:

1. Open: `sem_open(name, oflag, mode, value)` creates or opens a named semaphore, so separate processes can find the same one. It returns a `sem_t*`. `value` is the starting count, like `sem_open("/mysem", O_CREAT, 0600, 1)` for one process at a time.
2. Wait: `sem_wait(sem)` is how a process takes its turn. It blocks while the count is `0`, then decrements it. Call it before touching the shared memory.
3. Post: `sem_post(sem)` is how a process gives its turn back. It increments the count and wakes one waiter. Call it when done.
4. Close: `sem_close(sem)` is for when this process is done with the semaphore. It frees this process's handle, and the semaphore stays for the others.
5. Remove: `sem_unlink(name)` removes the name, like `sem_unlink("/mysem")`. The semaphore is destroyed once every process has closed it.

On macOS, only named semaphores work. `sem_init()`, which makes an unnamed one, fails with `ENOSYS`.

POSIX also keeps an older System V version, which uses numeric keys instead of names:

- Create: `shmget(key, size, flags)` creates a segment, or finds the one with that key, and returns its ID. The processes agree on the key ahead of time. Like `shmget(1234, 4096, IPC_CREAT | 0600)` → `65536`.
- Attach: `shmat(id, NULL, 0)` maps the segment into your address space, like `mmap()`, and returns a pointer.
- Detach: `shmdt(addr)` unmaps it, like `munmap()`.
- Remove: `shmctl(id, IPC_RMID, NULL)` marks it for removal, and it's freed once every process has detached. Without this, the segment stays even after every process exits.

macOS limits System V segments to 4 MB by default, so `shm_open()` is the one to use.

### Process info

- PID: `getpid()` returns your process's ID, like `812`, for log lines or naming a temp file. `getppid()` returns your parent's, like `640` for zsh.
- User: `getuid()` returns the ID of the user running the process, like `501` for the first user on a Mac. Code uses it to check who it's running as, like `0` for root.
- Environment: every process has environment variables, name-value strings that configure it, like `HOME` and `PATH`. `getenv(name)` reads one, like `getenv("HOME")` → `"/Users/manpreetsingh"`, or `NULL` if it isn't set. `setenv(name, value, overwrite)` sets one, like `setenv("LANG", "en_US.UTF-8", 1)`. With `overwrite` set to `0`, it leaves an existing value alone. Children get a copy through `execve()`'s `envp`.

### Signals

1. Handle: `sigaction(sig, &act, &old)` is for deciding what your process does when a signal arrives, like cleaning up on `SIGTERM` instead of dying right away. `act.sa_handler` is the function to run, and `old` gets the previous setting (or pass `NULL`). Like `sigaction(SIGTERM, &act, NULL)`. A handler can interrupt any code, so it may only call async-signal-safe functions like `write()`, not `printf()` or `malloc()`.
2. Block: `sigprocmask(how, &set, &old)` is for holding signals off during code that mustn't be interrupted. It blocks or unblocks the signals in `set`. `how` is `SIG_BLOCK`, `SIG_UNBLOCK` or `SIG_SETMASK`, and `set` is built with `sigemptyset(&set)` then `sigaddset(&set, SIGTERM)`. A blocked signal waits as pending until it's unblocked. In a threaded program, use `pthread_sigmask()`, which takes the same arguments and applies to the calling thread.
3. Send: `kill(pid, sig)` is how one process signals another, like asking it to quit. Despite the name, it can send any signal, like `kill(812, SIGTERM)`. `kill(812, 0)` sends nothing and just checks that the process exists. `raise(sig)` sends a signal to yourself.

More in OS Processes and in the sockets note under Shutting down with SIGTERM.

### Threads

1. Start: `pthread_create(&tid, attr, fn, arg)` is for running work at the same time as the rest of your program, in the same address space. It starts a thread running `fn(arg)`. `tid` gets the thread's ID, and `attr` is `NULL` for defaults. Like `pthread_create(&tid, NULL, worker, &job)`.
2. Self: `pthread_self()` returns the calling thread's ID, for logging which thread did something or comparing against a saved `tid`.
3. End: `pthread_exit(ret)` is for ending a thread early, from anywhere inside it. It ends the calling thread with return value `ret`, the same as returning `ret` from `fn`. Only that thread ends, and the process keeps running.
4. Wait: `pthread_join(tid, &ret)` is for waiting until a thread's work is done before using its results. It blocks until the thread ends, `ret` gets its return value, and the thread's resources are freed. Like `pthread_join(tid, NULL)`. If no one will wait, call `pthread_detach(tid)` so the resources are freed as soon as it ends. Otherwise a finished thread lingers, like a zombie process.

Locks are in the C++ Concurrency notes.

### Time

- Clock: `clock_gettime(clock, &ts)` is for reading the time, either the time of day or a timer for measuring how long something takes. It fills `ts` with seconds and nanoseconds. `CLOCK_REALTIME` is the wall clock, which can jump when the clock is set; `CLOCK_MONOTONIC` only moves forward, so use it to measure durations. Like `clock_gettime(CLOCK_MONOTONIC, &ts)` → `ts.tv_sec = 5231`, `ts.tv_nsec = 120000`.
- Sleep: `nanosleep(&req, &rem)` is for pausing the calling thread for a set time, like waiting before a retry. It sleeps for `req`, like `{.tv_sec = 0, .tv_nsec = 500000000}` for half a second. If a signal wakes it early, it returns `-1` with `EINTR` and `rem` holds the time left.
