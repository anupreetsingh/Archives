# Concurrency Models

At a high level, **concurrency is the goal or property** of having multiple tasks make progress over overlapping periods of time. There are several mechanisms for achieving it.

## Major Concurrency Mechanisms

### Multitasking with Processes

The operating system runs multiple processes concurrently.

```text
Chrome process  ───┐
Python process  ───┼── OS scheduler ──→ CPU
Spotify process ───┘
```

On one CPU core, the OS rapidly switches between them. On multiple cores, some can actually run in parallel.

### Multithreading

A single process creates multiple OS threads, which the OS scheduler manages.

```text
Server process
  ├── Thread 1 ──→ Request A
  ├── Thread 2 ──→ Request B
  └── Thread 3 ──→ Request C
```

Again, one core provides concurrency through interleaving; multiple cores can additionally provide **parallelism**.

### Async and Event-Loop Concurrency

Instead of creating a thread for every task, many tasks can share a thread and **yield when they are waiting**.

```text
One OS thread

Task A ── run ── await network ───────── resume
                  ↓
Task B            run ── await DB ────── resume
                         ↓
Task C                   run ───────────→
```

This is common in Python `asyncio`, JavaScript and Node.js, Rust async, and C# `async`/`await`.

### Lightweight Runtime-Managed Threads and Coroutines

This is where **goroutines** fit.

```text
Goroutine A ─┐
Goroutine B ─┤
Goroutine C ─┼── Go runtime ──→ OS threads ──→ CPU
Goroutine D ─┘
```

The language runtime schedules large numbers of lightweight concurrent tasks onto a smaller number of OS threads.

## Workload Types

The right concurrency model depends on what the program is waiting for.

| Workload type | Bottleneck | Good fit |
|---------------|------------|----------|
| **I/O-bound** | Network, disk, database, API calls | threads, async/await, event loops |
| **CPU-bound** | Computation, parsing, compression, image/video processing | processes, native code, parallel runtimes |
| **Mixed** | Some waiting, some computation | combine async/threads with worker processes |

## Python Concurrency

Python can achieve concurrency in several ways, but they do not all create the same kind of parallelism.

### Threads

Python threads are OS threads created through tools such as `threading` or `concurrent.futures.ThreadPoolExecutor`.

They are useful for I/O-bound work because while one thread waits for a slow operation, another thread can run.

```python
from concurrent.futures import ThreadPoolExecutor
import requests

urls = [
    "https://example.com",
    "https://example.org",
    "https://example.net",
]

def fetch(url):
    response = requests.get(url, timeout=5)
    return url, response.status_code

with ThreadPoolExecutor(max_workers=3) as pool:
    for url, status in pool.map(fetch, urls):
        print(url, status)
```

This is useful even with the GIL because most of the time is spent waiting on network I/O, not executing Python bytecode.

### Asyncio

`asyncio` uses cooperative multitasking. Tasks voluntarily pause at `await` points so the event loop can run another task.

```python
import asyncio
import httpx

urls = [
    "https://example.com",
    "https://example.org",
    "https://example.net",
]

async def fetch(client, url):
    response = await client.get(url)
    return url, response.status_code

async def main():
    async with httpx.AsyncClient(timeout=5) as client:
        tasks = [fetch(client, url) for url in urls]
        results = await asyncio.gather(*tasks)

    for url, status in results:
        print(url, status)

asyncio.run(main())
```

This is also best for I/O-bound work. It usually uses fewer threads than a threaded design, but every slow operation must be written using non-blocking async APIs. A blocking call inside an async function can freeze the event loop.

### Multiprocessing

`multiprocessing` and `concurrent.futures.ProcessPoolExecutor` use multiple OS processes.

Each process has its own Python interpreter and memory space, so CPU-bound work can run on multiple cores.

```python
from concurrent.futures import ProcessPoolExecutor

def count_primes(limit):
    count = 0

    for number in range(2, limit):
        for divisor in range(2, int(number ** 0.5) + 1):
            if number % divisor == 0:
                break
        else:
            count += 1

    return count

limits = [50_000, 60_000, 70_000, 80_000]

with ProcessPoolExecutor() as pool:
    print(list(pool.map(count_primes, limits)))
```

This is useful for CPU-bound Python code because each process has its own interpreter lock. The tradeoff is higher memory usage and slower communication because data must be copied, serialized, or sent through inter-process communication.

### Native Extensions

Some Python libraries run heavy work in native C, C++, Rust, or Fortran code.

Libraries such as NumPy, pandas, PyTorch, TensorFlow, `zlib`, and `hashlib` can perform expensive operations outside normal Python bytecode execution. Some native code releases the GIL while it runs, allowing other Python threads to make progress.

This is why Python can be effective for scientific computing and machine learning even though pure Python loops are relatively slow.

## Python GIL

The **GIL (Global Interpreter Lock)** is a lock inside the standard CPython interpreter that allows only one thread at a time to execute Python bytecode.

The GIL exists mainly to protect CPython's internal object model and memory management. For example, CPython uses reference counts to track object lifetime. If multiple threads updated those internal counts at the same time without coordination, interpreter memory could become corrupted.

Important implications:

- The GIL does **not** mean Python has no concurrency.
- The GIL does mean normal Python threads do not usually speed up CPU-bound pure Python code across multiple cores.
- Threads can still help I/O-bound Python programs because the GIL is released around many blocking I/O operations.
- Processes bypass the GIL limit because each process has its own interpreter and its own GIL.
- Native extensions can sometimes release the GIL while doing long-running non-Python work.
- The GIL is not a replacement for application-level locks. [Race conditions](Computer%20Architecture.md#race-conditions) can still happen when multiple threads update shared program data.

### Free-Threaded CPython

Starting with Python 3.13, CPython supports a special free-threaded build where the GIL can be disabled.

In a free-threaded build, multiple Python threads can execute Python bytecode in parallel on multiple cores. This is different from the normal/default CPython build where one thread executes Python bytecode at a time.

Practical caveats:

- Free-threaded Python is a build/runtime choice, not the historical default CPython behavior.
- Some third-party C extension modules may not be ready for free-threaded execution.
- Importing an extension that does not support free threading can cause the GIL to be re-enabled.
- Code with shared mutable state still needs locks or safer designs because true parallel thread execution increases the importance of thread safety.

## Other Language Models

Different languages expose concurrency differently.

| Language | Common model | Notes |
|----------|--------------|-------|
| **Python** | threads, `asyncio`, processes | GIL limits CPU-bound pure-Python threading in normal CPython |
| **JavaScript/Node.js** | event loop, async/await, worker threads | good for I/O-bound servers; CPU-heavy work can block the event loop unless moved to workers |
| **Go** | goroutines and channels | lightweight concurrent functions managed by the Go runtime |
| **Java** | OS threads, thread pools, virtual threads | strong JVM concurrency ecosystem; virtual threads make blocking-style code cheaper |
| **C++** | `std::thread`, atomics, mutexes, async libraries | high control and high responsibility; data races can cause undefined behavior |
| **Rust** | threads, async, ownership rules | type system prevents many shared-memory bugs at compile time |

## Choosing a Python Approach

| Situation | Usually choose |
|-----------|----------------|
| Many HTTP/API/database calls | `asyncio` or threads |
| Blocking libraries with no async API | threads |
| CPU-heavy pure Python loops | multiprocessing |
| CPU-heavy numerical work | NumPy/PyTorch/native libraries |
| Need shared mutable state | locks, queues, or redesign ownership |
| Need maximum parallel thread execution in Python bytecode | free-threaded CPython, if dependencies support it |
