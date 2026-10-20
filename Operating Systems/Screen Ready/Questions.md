# Questions

Rajan-style questions from each lesson, in order, with the answers.

## Bullet 1: The Daemon

1. Why does each experiment need its own salt? What goes wrong without one?

   Something per-experiment has to go into the hash. If only the device ID were hashed, a device would get the same bucket in every experiment, so the same devices would be in treatment everywhere. The experiments' effects would mix and couldn't be told apart. The experiment name already makes the input different per experiment. A separate salt lets a team reshuffle an experiment (restart it cleanly after a bug) without renaming it.

2. Why not `std::hash`? And why doesn't the hash's speed matter much here?

   `std::hash` is implementation-defined: the standard only requires the same result within one run of a program, and libc++ and libstdc++ differ. A compiler or library update could silently move every device to a different bucket and ruin running experiments. I need an algorithm whose output is fixed by its spec, like SHA-256 or FNV-1a. Speed barely matters because expd hashes once per experiment per config push, about once a day. Lookups read the precomputed variant from the snapshot and never hash.

3. One client sends half a request, then stops. What happens to the other clients, and why?

   Nothing. Every fd is non-blocking, so `read()` returns the partial bytes, which go into that client's own buffer, and the loop goes back to `kevent()`. The request is handled only once the whole frame has arrived (the length prefix says how many bytes). Other clients keep being served. With a blocking `read()`, the loop would sleep waiting for the rest and freeze everyone. To be safe, also drop clients that stall too long (an `EVFILT_TIMER` timeout) and cap the frame size, so a client can't make expd allocate a huge buffer.

4. Why did you use one thread for the event loop? When would you add another?

   Each request is a lookup costing microseconds of CPU, so one thread handles thousands per second. With one thread, only one thread touches the assignment table, so it needs no locks and can't have data races. It also saves memory and context switches. I'd add a thread for work that blocks or takes long: disk I/O (bullet 3's writer thread), heavy CPU like checking and evaluating a big config on a push, or if measurements showed one core saturated.
