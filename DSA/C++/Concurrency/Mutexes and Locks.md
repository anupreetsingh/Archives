# Mutexes and Locks in C++

When several threads run inside one program, they share the same memory. That's what makes threads useful, because there's no copying between them, and also what makes them dangerous: two threads touching the same variable at the same time is a **data race**, and in C++ a data race is undefined behavior.

A **mutex** (from *mutual exclusion*) fixes this by letting only one thread at a time into the code that touches shared data. A **lock** is the act of holding a mutex, and in C++ it is also the RAII object that does the holding for you: lock in the constructor, unlock in the destructor.

Python has the same tools in `threading.Lock` and `with lock:`. Because Python has the GIL, many races there only give wrong answers instead of corrupting memory, so it's easy to get away without locks. C++ has no GIL, so every piece of shared mutable data needs protecting.

---

## 1. The Problem: Threads Share Memory

### Starting a Thread

`std::thread` (in `<thread>`) starts running a function on a new thread right away. `join()` waits for it to finish:

```cpp
#include <thread>

void work() { /* ... */ }

int main() {
    std::thread a(work);     // a starts running work() now
    std::thread b(work);     // b too, at the same time as a and main
    a.join();                // main waits here until a finishes
    b.join();
}
```

Every `std::thread` must be joined (or detached) before it is destroyed, or the program calls `std::terminate`. C++20's `std::jthread` joins automatically in its destructor, so it's the RAII version (see [Object Lifetime and RAII](<../OOP/Object Lifetime and RAII.md#raii-everywhere-in-the-standard-library>)).

### Lost Updates

Here two threads each add 100,000 to a shared counter:

```cpp
int counter = 0;

void add() {
    for (int i = 0; i < 100'000; ++i) ++counter;
}

// main: start two threads running add(), join both, print counter
```

```text
100000
101358
100000          <- three runs, three different wrong answers; 200000 was expected
```

`++counter` looks like one step, but the CPU does it in three: **read** the value into a register, **add** 1, **write** it back. When the threads interleave, both can read the same old value:

```text
counter = 5
Thread A: read 5
Thread B: read 5          <- B reads before A has written
Thread A: write 6
Thread B: write 6         <- A's increment is lost; counter should be 7
```

### Data Race vs Race Condition

These two terms are related but mean different things:

- A **data race** is a language-level fault: two threads access the same memory location, at least one of them writes, and nothing synchronizes them. In C++ this is [undefined behavior](<../Memory/Undefined Behavior and Sanitizers.md#3-common-sources-of-ub>). The program is invalid even if it happens to print the right number.
- A **race condition** is a logic bug: the result depends on how the threads happen to be scheduled. Code can be free of data races (every access is locked) and still have race conditions, for example "check the balance, unlock, then withdraw".

ThreadSanitizer finds data races at runtime, and it works with Apple Clang on macOS. It can't be combined with ASan, so build a separate binary:

```bash
clang++ -std=c++20 -g -O0 -fsanitize=thread race.cpp -o race_tsan && ./race_tsan
```

```text
WARNING: ThreadSanitizer: data race (pid=36552)
  Write of size 4 at 0x000100edc040 by thread T2:
    #0 add_unsafe() race.cpp:8
  Previous write of size 4 at 0x000100edc040 by thread T1:
    #0 add_unsafe() race.cpp:8
  Location is global 'counter' at 0x000100edc040
```

Read it the same way as an ASan report: what happened (two writes), where (line 8, both times), and to what (the global `counter`).

## 2. "Lock" and "Mutex": Two Meanings

In general computer science, **lock** and **mutex** are mostly the same idea: something only one thread can hold at a time. Other synchronization tools build on it:

| Concept | What it allows |
|---|---|
| **Mutex / lock** | One holder at a time; others **sleep** until it's free |
| **Spinlock** | One holder at a time; others **busy-wait** in a loop. Cheap for very short waits, wasteful for long ones |
| **Read-write lock** | Many readers at once, **or** one writer |
| **Semaphore** | Up to *N* holders at once (a mutex is roughly a semaphore with *N* = 1) |
| **Condition variable** | Lets a thread sleep until another thread says "something changed" |

The C++ standard library splits the word in two:

- A **mutex** (`std::mutex`) is the thing that can be locked. It has `lock()` and `unlock()`, and that's about all.
- A **lock** (`std::lock_guard`, `std::scoped_lock`, `std::unique_lock`) is an RAII object that **owns** a locked mutex for as long as the lock object lives.

So in C++ code, "take a lock" means "construct a lock object on a mutex".

## 3. `std::mutex`

### How Locking Works

A mutex is a gate in front of a **critical section**, the code that touches shared data. Here's what happens when two threads reach it:

```text
Thread A: m.lock()   -> mutex free, A takes it, A enters the critical section
Thread B: m.lock()   -> mutex held, B sleeps
Thread A: ...works on the shared data...
Thread A: m.unlock() -> mutex free, the OS wakes B
Thread B:            -> B takes the mutex and enters
```

Locking also makes **memory visible**. Whatever A wrote before `unlock()` is guaranteed to be seen by B after its `lock()`. Without the mutex, B could see stale values even without a lost update, because of CPU caches and compiler reordering.

### Manual `lock()` / `unlock()` Is Fragile

```cpp
std::mutex m;
std::vector<int> scores;

void add_score(int s) {
    m.lock();
    if (s < 0) return;          // BUG: returns while still holding m
    scores.push_back(s);        // BUG: if this throws, m is never unlocked either
    m.unlock();
}
```

After either bug, `m` stays locked forever, and the next thread to call `add_score` sleeps forever. This is the same "every exit path needs cleanup" problem that [RAII](<../OOP/Object Lifetime and RAII.md#3-the-problem-raii-solves>) solves, and the fix is the same: use a lock object.

### A Mutex Protects Data Only by Convention

The compiler has no idea which data a mutex is meant to protect. It's your job to lock it before **every** access to that data, reads included. The usual pattern is to keep the mutex next to the data, both private, so outside code can't skip the lock:

```cpp
class Scoreboard {
public:
    void add(int s) {
        std::lock_guard<std::mutex> lock(m_);
        scores_.push_back(s);
    }

    std::size_t count() const {
        std::lock_guard<std::mutex> lock(m_);   // reads need the lock too
        return scores_.size();
    }

private:
    mutable std::mutex m_;      // mutable: locking changes the mutex, even in a const function
    std::vector<int> scores_;   // guarded by m_
};
```

`mutable` lets a `const` member function lock the mutex (see [Const Correctness](<../Memory/Const Correctness.md#mutable-the-escape-hatch>)). A `std::mutex` can't be copied or moved, so a class that holds one can't be copied or moved either unless you write those operations yourself.

## 4. Lock Types: RAII for Mutexes

### `std::lock_guard`: the Simplest

It locks in its constructor and unlocks in its destructor, and you can't do anything else with it:

```cpp
void add_score(int s) {
    std::lock_guard<std::mutex> lock(m);   // locks m
    if (s < 0) return;                     // unlocks m
    scores.push_back(s);                   // throws? unlocks m during stack unwinding
}                                          // unlocks m
```

Since C++17 the template argument can be deduced: `std::lock_guard lock(m);`.

Give the lock object a name. `std::lock_guard<std::mutex>{m};` creates a temporary that unlocks at the `;`, so it protects nothing (see [Unnamed Temporaries Die Immediately](<../OOP/Object Lifetime and RAII.md#unnamed-temporaries-die-immediately>)).

### `std::scoped_lock` (C++17): One or More Mutexes

It works like `lock_guard`, but it can lock **several mutexes at once** without risk of deadlock (see section 6):

```cpp
std::scoped_lock lock(from.m, to.m);   // locks both, in a deadlock-free way
```

With a single mutex it behaves exactly like `lock_guard`, so many codebases use `scoped_lock` everywhere.

### `std::unique_lock`: Flexible

`std::unique_lock` owns a mutex too, but it adds control over when the mutex is locked:

```cpp
std::unique_lock<std::mutex> lock(m);   // locked
// ...
lock.unlock();                           // release early, before slow work
expensive_computation();
lock.lock();                             // take it back
// destructor unlocks only if it currently holds the mutex
```

It can also be constructed unlocked (`std::defer_lock`), try to lock without blocking (`std::try_to_lock`), and be moved to another scope. `std::condition_variable` requires a `unique_lock`, because waiting has to unlock and relock the mutex.

| Lock type | Mutexes | Unlock early / relock | Movable | Use it for |
|---|---|---|---|---|
| `std::lock_guard` | 1 | No | No | The simple case |
| `std::scoped_lock` | 1 or more | No | No | The default in C++17 and later; required for several mutexes |
| `std::unique_lock` | 1 | Yes | Yes | Condition variables, deferred or early unlocking |

## 5. Keep Critical Sections Small, but Complete

While one thread holds a mutex, every other thread that needs it waits. So hold the mutex for as little code as possible: no file I/O, no sleeping, and no calls into unknown code while locked. An extra pair of braces ends the lock's scope early:

```cpp
void record(int k, int value) {
    Result r = compute(value);                   // slow work, done without the lock
    {
        std::lock_guard<std::mutex> lock(m);
        results[k] = r;                          // only the shared write is locked
    }                                            // m unlocked here
    log("recorded");                             // no lock needed
}
```

Small, but not too small. Any access **after** the braces close is unprotected:

```cpp
int fn(int k, int f) {
    static std::mutex mutex;
    static std::map<int, int> data;

    {
        std::lock_guard<std::mutex> guard{mutex};
        data[k] = f;
    }                                            // mutex released

    return data[k];                              // unprotected read
}
```

This function can't deadlock: it has a single mutex and nothing runs while the mutex is held, and the guard always releases it. But it isn't thread-safe:

- Another thread can write `data[k]` between the closing brace and the `return`, so the function may not return `f`.
- More seriously, that unlocked read can happen at the same moment as another thread's locked write. That's a data race, so it's undefined behavior. For a `std::map`, `operator[]` may even insert a node and rebalance the tree while another thread is reading it. TSan flags it immediately:

```text
WARNING: ThreadSanitizer: data race
  Write of size 4 at 0x00010b401730 by thread T2 (mutexes: write M0):
  Previous read of size 4 at 0x00010b401730 by thread T1:
```

The report shows the problem: the write held mutex `M0`, the read held nothing. A lock only helps if **every** access takes it. Here the fix is to return `f` directly, or to do the read while the lock is still held.

The lesson in general: a sequence of steps that must behave as one ("write then read back", "check then act") has to happen under **one** continuous hold of the lock.

## 6. Deadlock

A **deadlock** is when threads wait for each other forever, so none of them can make progress. With mutexes it happens in three main ways.

### 1. Locking the Same Mutex Twice on One Thread

`std::mutex` isn't recursive. If a thread locks a mutex it already holds, the behavior is undefined, and in practice the thread waits forever for itself:

```cpp
std::lock_guard<std::mutex> a(m);
std::cout << "locked once\n";
std::lock_guard<std::mutex> b(m);       // hangs here
std::cout << "locked twice\n";          // never printed
```

This usually happens indirectly: a locked member function calls another member function that takes the same lock. A common fix is a private helper that assumes the lock is already held, called by public functions that take the lock. `std::recursive_mutex` allows the same thread to lock again, but needing one often means the locking design is muddled.

### 2. Two Mutexes Locked in Opposite Orders

This is the classic case:

```cpp
struct Account { std::mutex m; int balance = 100; };

void transfer(Account& from, Account& to, int amount) {
    std::lock_guard<std::mutex> a(from.m);
    std::lock_guard<std::mutex> b(to.m);
    from.balance -= amount;
    to.balance += amount;
}

// thread 1: transfer(x, y, 10)   locks x.m, then waits for y.m
// thread 2: transfer(y, x, 20)   locks y.m, then waits for x.m
```

```text
Thread 1 holds x.m, wants y.m  -> waits for thread 2
Thread 2 holds y.m, wants x.m  -> waits for thread 1     (forever)
```

Run it and the program hangs. There are two fixes:

- **Lock all mutexes at once** with `std::scoped_lock lock(from.m, to.m);`. It uses a deadlock-avoidance algorithm, so the order in which the arguments are passed doesn't matter. With that change the same program finishes and prints `110 90`.
- **Always lock in one global order**, for example by account ID or by address. If every thread takes `x.m` before `y.m`, nothing can wait in a cycle.

### 3. A Lock That Is Never Released

A manual `lock()` with a missed `unlock()` (section 3) blocks every later locker forever. RAII lock objects prevent this entirely.

### Rules of Thumb

- Use RAII lock objects, never bare `lock()` / `unlock()`.
- Lock several mutexes in one `std::scoped_lock`, or in one fixed order.
- Don't call unknown code (callbacks, virtual functions, user lambdas) while holding a lock. It might take the same lock or another one.
- Hold locks briefly.

## 7. Other Tools, Briefly

| Tool | Header | What it's for |
|---|---|---|
| `std::recursive_mutex` | `<mutex>` | The same thread may lock it several times; it must unlock the same number of times |
| `std::timed_mutex` | `<mutex>` | `try_lock_for(duration)`: give up after waiting a while |
| `std::shared_mutex` + `std::shared_lock` | `<shared_mutex>` | Read-write lock: many readers with `shared_lock`, or one writer with `unique_lock` / `lock_guard` |
| `std::condition_variable` | `<condition_variable>` | Sleep until another thread signals; the basis of producer/consumer queues |
| `std::atomic<T>` | `<atomic>` | A single variable that threads can safely read and modify without a mutex: `std::atomic<int> counter; ++counter;` fixes the counter in section 1 |
| `std::once_flag` + `std::call_once` | `<mutex>` | Run an initializer exactly once across threads |

`std::atomic` is the right tool for one counter or flag. When several variables have to change together, use a mutex. Two individually atomic variables don't give you an atomic update of the pair.

`static` locals are already initialized thread-safely (see [`static` Local Variables](<../Fundamentals/Functions.md#static-local-variables>)). That protects only their first initialization, not later reads and writes.

## 8. In Unreal Engine

Most gameplay code runs on the **game thread**, and UObjects generally shouldn't be touched from other threads. Locks matter when you use background work (async tasks, `ParallelFor`, worker threads) that shares data with the game thread. Unreal has its own names for the same tools:

| Standard C++ | Unreal |
|---|---|
| `std::mutex` | `FCriticalSection` |
| `std::lock_guard` | `FScopeLock Lock(&CriticalSection);` |
| `std::shared_mutex` | `FRWLock`, with `FReadScopeLock` / `FWriteScopeLock` |
| `std::atomic<T>` | `std::atomic<T>` (Unreal now uses the standard one) |

```cpp
FCriticalSection ResultsLock;
TArray<int32> Results;

void AddResult(int32 Value)
{
    FScopeLock Lock(&ResultsLock);   // same RAII idea: unlocks when Lock goes out of scope
    Results.Add(Value);
}
```

## Summary

```text
Threads        -> share memory; unsynchronized access with at least one write = data race = UB
Mutex          -> one thread at a time in the critical section; also makes writes visible to the next locker
Lock object    -> RAII owner of a locked mutex: lock_guard (simple), scoped_lock (several), unique_lock (flexible)
Every access   -> reads included, must hold the lock; a "write then read" must be one continuous hold
Deadlock       -> self-relock, opposite lock orders, or a lock never released
Avoid it       -> RAII locks, scoped_lock for several mutexes, fixed order, no unknown calls while locked
Find races     -> -fsanitize=thread (separate build from ASan)
Unreal         -> FCriticalSection + FScopeLock
Python         -> threading.Lock + `with lock:`; the GIL hides many (not all) races that C++ exposes
```
