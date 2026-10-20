# Project Flow

The On-Device Experimentation Service from my resume, start to end. Each resume bullet covers one stretch of this flow: bullet 1 is the daemon, bullet 2 the read path, bullet 3 the exposure path.

1. Config push: a new experiment definition reaches the daemon, like `new_search_ranking` with control/treatment at 50/50 and salt `"k3f9"`.
2. Assign: expd, the C++ daemon, computes this device's variant for each experiment: `bucket = hash(salt + experiment + device_id) % 10000`, like `4821` → control (0–4999).
3. Publish: expd writes every assignment into a new file, `snapshot.tmp`, calls `fsync()`, then `rename()`s it over `snapshot`, then calls `notify_post("com.example.expd.snapshot")`. Each app's libexp gets that notification right away but only remaps (`open()` + `mmap()` the new file, `munmap()` the old) at a safe point, like the next launch or return to foreground, so a user's variant never changes mid-session. Until then its old mapping keeps showing the old snapshot, because `rename()` doesn't touch the old file. A kill switch is the exception: remap at once. (Decide: Darwin notification on macOS; on Linux, `inotify` on the directory.)
4. Read: an app calls `variant(for: "new_search_ranking")` in my Swift wrapper, which calls libexp, my C library. libexp binary-searches the snapshot it has `mmap()`ed into the app's address space: about 90 ns, no IPC.
5. Slow path: if no snapshot exists yet, libexp asks expd over the Unix domain socket instead: about 30 µs.
6. Exposure: when the app actually runs the changed code, libexp sends expd an exposure message, like "`new_search_ranking`, control, seen". (Decide: one message per exposure, or batched per process.)
7. Buffer: expd's event loop pushes the exposure into a ring buffer guarded by a mutex, then goes straight back to serving clients.
8. Flush: a writer thread wakes about every 60 s on a coalesced timer and appends the batch to a log file on disk.

The daemon does the writes and the background work; the library does the reads. That split answers most "why" questions.

When I don't know what the original code did, I give my reasoning ("I don't remember exactly; what I'd use is Y, because Z"), never a made-up specific. Items marked "Decide" get settled in the rebuild, and then I talk about what I built.
