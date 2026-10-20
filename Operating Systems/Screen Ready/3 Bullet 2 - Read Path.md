# Bullet 2: The Read Path

> Wrote a C client library with a Swift wrapper whose one-call variant(for:) reads an mmap'd snapshot the daemon swaps atomically via rename on each config push, keeping lookups lock-free at ∼90 ns versus 30 µs over the socket.

## "C client library with a Swift wrapper"

libexp is the library each app links. Its public interface is plain C functions, like `int exp_variant(const char *experiment, char *out, size_t out_len)`, which copies the variant name into `out` and returns `0`, or `-1` if there's none. Inside, it can be C++ behind `extern "C"`. The Swift wrapper imports that C header and adds `variant(for:)` on top.

Why C at the boundary:

- Anyone can call it: C, C++, Objective-C and Swift all call C functions directly.
- Stable ABI: a C function's calling convention doesn't change between compiler versions. A C++ interface breaks: name mangling differs, `std::string`'s layout differs between libc++ and libstdc++, and exceptions can't cross into C or Swift.
- Apple ships its own frameworks the same way: a C or Objective-C core with a Swift layer.

Instead of: a C++ library (breaks across compilers, no exceptions across the boundary), or Swift only (C and C++ daemons couldn't use it).

He'll ask: "Why the Swift wrapper at all?" Feature teams write Swift. The wrapper turns a `char` buffer and an error code into a Swift `String` with a default, so nobody handles C memory.

## "one-call variant(for:)"

`variant(for: "new_search_ranking")` → `"treatment"`. One call: no setup, no async, never throws.

Why:

- Easy to use: the JD asks for "high-performance libraries that are easy to use by developers."
- Hard to misuse: if the device isn't enrolled, there's no snapshot yet, or the file is corrupt, it returns the default, `"control"`, so the feature always works.
- Safe on any thread, including the main thread: it never blocks (no IPC, no lock).
- Lazy: the first call maps the snapshot; later calls only read.

He'll ask:

- "Does it log the exposure?" Decide: automatically on the first call per experiment per session (easy, but also logs code that only prefetched), or an explicit `logExposure()` call (accurate, but easy to forget). Apple.md's design logs automatically, with an opt-out for prefetching.
- "Can a user's variant change while they use the app?" No. libexp pins one snapshot per session and remaps only at a safe point (step 3 of [Project Flow](1%20Project%20Flow.md)).

## "reads an mmap'd snapshot"

The snapshot is a file expd writes, holding this device's variant for every active experiment. Decide its layout. A simple one:

1. Header: a magic number (like `"EXPS"`), a format version (like `1`), the entry count (like `40`) and a checksum.
2. Entries: fixed-size records sorted by experiment name, each holding the name and the variant, like `{"new_search_ranking", "treatment"}`.

What happens when libexp maps it. `open()` and `mmap()` are POSIX: [Files](../Foundations/POSIX.md#files), [Memory](../Foundations/POSIX.md#memory).

1. Open: `open("snapshot", O_RDONLY)` gives an fd. It points at the open-file record (`fileglob`), which points at the file's vnode.
2. Map: `mmap(NULL, size, PROT_READ, MAP_SHARED, fd, 0)` adds an entry to the app's VM map, pointing a range of its addresses at the vnode's VM object (`vm_object`), the kernel structure recording which physical pages hold the file's data. Those pages are the file's page cache.
3. Check once: libexp checks the magic, version and checksum here, not on every lookup. Bad file → defaults.
4. First touch: the first read of each page page-faults. The kernel finds the page in that VM object and maps it: about a microsecond if the page is already in the page cache, much longer if it has to be read from disk. Once per page.
5. Lookup: after that, `variant(for:)` is a binary search over the sorted entries: plain memory reads, no syscall. 40 entries take about 6 comparisons.

Why:

- No syscall or IPC per lookup.
- One copy in RAM: every app that maps the file points at the same VM object, so 10 apps share the same physical pages.
- Cheap memory: the pages are clean (never modified), so under pressure the kernel just drops them and reads them back later. On iOS, clean pages don't count toward the jetsam footprint (footprint = dirty + compressed).

Instead of:

- `read()` into a heap buffer: each app gets a private copy, and those are dirty pages that count against its memory limit.
- Asking expd every time: about 30 µs, and expd must be running.

He'll ask:

- "What if expd truncates the file while apps have it mapped?" An app touching a page past the new end gets `SIGBUS` and crashes. That's why expd never modifies or truncates the live file; it writes a new one and renames it.
- "Why binary search, not a hash table?" Tens to hundreds of experiments means 6–8 comparisons, a simple layout, and an order that's easy to check. A perfect hash only pays off at thousands.

## "the daemon swaps atomically via rename on each config push"

On each new config, expd does four steps. `fsync()` and `rename()` are POSIX: [Files](../Foundations/POSIX.md#files), [Paths and directories](../Foundations/POSIX.md#paths-and-directories).

1. Write: the full snapshot goes to `snapshot.tmp`, in the same directory.
2. Sync: `fsync()` on it, so the data is on disk before the name changes. On macOS, `fcntl(fd, F_FULLFSYNC)` also flushes the drive's own cache; plain `fsync()` doesn't.
3. Swap: `rename("snapshot.tmp", "snapshot")` points the name `snapshot` at the new file's vnode in one step.
4. Tell apps: `notify_post("com.example.expd.snapshot")`. Each libexp remaps at its next safe point.

What readers see:

- Anyone opening `snapshot` after the rename gets the new file; before it, the old one. Never half of each.
- Apps that already mapped the old file keep reading it: their VM map still points at the old vnode's VM object, so the old file's data stays alive until the last app unmaps it, even though no name points to it anymore.

Why: no locks between processes, and no reader ever sees a half-written file.

Instead of: editing the live file in place, which needs a cross-process lock, lets readers see half-written entries, and risks `SIGBUS` on truncation.

He'll ask:

- "Why `fsync()` before `rename()`?" Without it, a crash can leave the name `snapshot` pointing at a file whose data never reached disk: an empty or partial snapshot.
- "Does `rename()` work across volumes?" No, it fails with `EXDEV`. That's why `snapshot.tmp` sits in the same directory.
- "How does an app learn there's a new file?" A Darwin notification (`notify_post()`); on Linux, `inotify` on the directory. Not `stat()` on every lookup: that's a syscall each time and would destroy the 90 ns.

## "keeping lookups lock-free"

libexp keeps the current mapping behind one atomic pointer, like `std::atomic<const Snapshot*> current`:

1. Lookup: `current.load(std::memory_order_acquire)`, then binary search. No lock.
2. Remap: map the new file and check it fully, then `current.store(new_snapshot, std::memory_order_release)`.

Acquire and release pair up: a thread that sees the new pointer also sees everything libexp wrote before storing it, like the checked header. So no reader ever sees a half-set-up snapshot.

Why:

- No waiting: lookups never block, so they're safe on the main thread.
- No priority inversion: a high-priority UI thread can't get stuck behind a low-priority thread holding a lock.
- No contention: readers only read the pointer, so no cache line bounces between cores.

Instead of: `std::shared_mutex` (a readers-writer lock). Even readers must atomically update a shared counter to take it, so that counter's cache line bounces between cores, and a reader can block behind a writer.

He'll ask: "When is it safe to `munmap()` the old snapshot?" Another thread might be mid-lookup in it. Options:

1. Keep old mappings until the process exits. Snapshots are a few KB and change about once a day, so the leak is tiny. Simplest, and what I'd do at this scale.
2. Epoch-based reclamation: readers record which generation they're reading, and the old mapping is unmapped once every reader has moved on. The scalable answer.
3. A reference count per lookup: correct, but every lookup does an atomic increment and decrement on a shared counter, which costs more than the lookup itself.

## "∼90 ns versus 30 µs over the socket"

90 ns, the mmap path: comparing the experiment name and a binary search touching a few cache lines, with no syscall.

30 µs, the socket path: the app's `send()`, waking expd's thread, a switch into expd's process, expd's `read()`, the lookup, expd's `write()`, then a switch back to the app. Most of it is waking and switching to another process, not the syscalls themselves.

He'll ask:

- "How did you measure?" A loop of many lookups timed with `std::chrono::steady_clock`, divided by the count, after a warm-up so first-touch page faults don't count. If I don't remember the exact setup, I explain how such a number is measured and what it includes, rather than inventing one.
- "What's the first lookup's cost?" Higher: mapping the file plus page faults, microseconds, once.
- "Could it be faster?" Yes: the Swift wrapper could find an experiment's entry once and reuse its index, skipping the name comparisons.
