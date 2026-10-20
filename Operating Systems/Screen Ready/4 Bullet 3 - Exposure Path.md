# Bullet 3: The Exposure Path

> Moved exposure logging off the event loop into a mutex-guarded ring buffer flushed by a writer thread on a coalesced 60 s timer, cutting wakeups 12× at under 2 MB RSS, tested under GoogleTest and ThreadSanitizer.

## "exposure logging"

An exposure is a record that this device actually reached the code that differs between variants, like `{"new_search_ranking", "treatment", 2026-10-20}`. It's logged when the app runs that code, not when the device is assigned.

Why: the analysis compares only devices that saw the change. Counting every assigned device dilutes the effect with devices that never opened the feature.

How exposures reach expd: reads never touch expd, so libexp sends exposures as messages over the Unix domain socket. libexp dedupes per experiment per session, so a feature called 1,000 times sends one exposure. (Decide: one message per exposure, or batched per process. Batching means fewer wakeups for expd.)

## "moved off the event loop"

Before: the event loop wrote each exposure to disk itself. A disk write can block for milliseconds, and while it blocks, no client gets served.

After: the event loop only appends to a buffer in memory, which takes nanoseconds, and a separate writer thread does the disk I/O. This is bullet 1's rule: add a thread only for blocking work.

He'll ask: "Why not just a non-blocking write?" `O_NONBLOCK` doesn't apply to regular files: kqueue reports a file as always ready, and a `write()` to it can still wait on the disk. So disk I/O gets its own thread.

## "mutex-guarded ring buffer"

A fixed-size circular array, like 4,096 slots of 64 bytes (256 KB), with a head index (next slot to read) and a tail index (next slot to write) that wrap back to 0 at the end.

1. Push (event loop): lock the `std::mutex`, write the exposure at `tail`, advance `tail`, unlock.
2. Drain (writer thread): lock, copy everything between `head` and `tail` into a local batch, reset the indices, unlock, then write the batch to disk with the lock released.

Why:

- Bounded memory: the buffer never grows, which is part of how expd stays under 2 MB.
- A mutex is enough: one producer, one consumer, and the writer takes the lock about once a minute, so it's almost never contended. Each critical section is a few copies. Simple and easy to prove correct.
- The disk write happens outside the lock, so the event loop never waits on the disk.

Instead of: a lock-free single-producer/single-consumer queue (faster under heavy contention, but the memory ordering is harder to get right, and there's no contention here to win back), or an unbounded `std::vector` (grows without limit if the disk stalls).

He'll ask:

- "What happens when it's full?" Decide: drop the newest or the oldest, and count every drop so the analysis knows data is missing. Never block the event loop.
- "How does the writer know to flush early?" The event loop calls `notify_one()` on a `std::condition_variable` when the buffer passes a high-water mark, like 75% full, so the writer flushes early instead of dropping.

## "flushed by a writer thread on a coalesced 60 s timer"

The writer thread sleeps until its timer fires about every 60 s, drains the buffer, and appends the batch to a log file. It also flushes early at the high-water mark, and once more on `SIGTERM` before expd exits.

Coalesced means the timer has leeway, like 60 s give or take 10 s. The kernel can then fire it together with other timers that are already waking the CPU, so the CPU leaves its low-power idle state fewer times across the whole system.

- macOS: `dispatch_source_set_timer(timer, start, interval, leeway)` sets a dispatch timer source's schedule, like `dispatch_source_set_timer(t, start, 60 * NSEC_PER_SEC, 10 * NSEC_PER_SEC)`.
- Linux: timer slack, set with `prctl(PR_SET_TIMERSLACK, ...)`.
- `std::condition_variable::wait_for()` has no leeway, which is one reason to use a dispatch timer on macOS.

He'll ask:

- "What's lost if expd is killed?" Up to 60 s of exposures still in memory: `SIGKILL` (jetsam, idle exit) gives no chance to flush. That's fine if the loss is random across variants. It's harmful if one variant crashes more, because that variant loses more exposures and the split looks wrong (a sample ratio mismatch, which the analysis checks for).
- "Why not flush every exposure right away?" Each flush wakes the CPU and touches the disk. Batching trades a little freshness for far fewer wakeups.

## "cutting wakeups 12× at under 2 MB RSS"

Wakeups: how often expd's threads leave the blocked state to run. Each one pulls a CPU core out of idle, which costs battery. 12× means expd woke 12 times less often after the change. Know the before and after. If I don't remember the baseline, I say how I'd measure it: `top -stats pid,command,idlew` or `powermetrics` on macOS, and `voluntary_ctxt_switches` in `/proc/<pid>/status` on Linux.

RSS (resident set size): the physical memory currently mapped into expd, like `1.6 MB`, from `ps -o rss -p <pid>`. On Apple, the number jetsam uses is the footprint (dirty + compressed memory), from `footprint <pid>`.

What keeps it small: one event-loop thread, one writer thread, a fixed 256 KB ring buffer, and a small assignment table.

He'll ask: "RSS vs footprint?" RSS also counts clean pages, including shared library pages other processes use too. Footprint counts only memory the system can't take back without killing the process, so it's what matters on iOS.

## "tested under GoogleTest and ThreadSanitizer"

GoogleTest is the C++ unit-test framework. The tests that matter here:

1. Golden values: fixed inputs always give the same bucket, like `H("k3f9" + "new_search_ranking" + "6F1C…") % 10000 == 4821`. This catches anyone swapping the hash.
2. Distribution: 1,000,000 random IDs at 10% treatment land within about ±0.1% of 10%.
3. Snapshot: corrupt or truncated files are rejected, and lookups return the default.
4. Ring buffer: wrap-around, a full buffer, drop counting.

ThreadSanitizer (TSan, built with `-fsanitize=thread`): the compiler instruments every memory access, and at runtime TSan reports a data race when two threads touch the same memory, at least one writes, and nothing orders them (no lock, no atomic). Here, the tests to run under it: the event loop pushing while the writer drains, and lookups while the snapshot pointer swaps.

He'll ask:

- "Did TSan find a real bug?" Only answer with one I actually remember. Otherwise, say what kind it catches here, like a plain `const Snapshot*` read on one thread while another swaps it, which `std::atomic` fixes.
- "What can't TSan find?" Races on code paths the tests never run. It only checks what executes.
