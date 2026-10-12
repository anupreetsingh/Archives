# Undefined Behavior and Sanitizers

**Undefined behavior (UB)** is the C++ standard's term for operations it places **no requirements on at all**. If a program performs one, the standard says nothing about what happens, not just at that line but for the whole run.

In practice the program might:

- Crash immediately (the lucky case)
- Print garbage
- **Appear to work perfectly**, then break later, on another machine, or after a compiler upgrade
- Behave differently with optimizations on (`-O2`) than off (`-O0`)

Python has almost no UB. The interpreter checks every index, every name, every attribute, and raises an exception when something is wrong. C++ skips those checks to be fast (see [What is C++](<../Fundamentals/What is C++.md#6-undefined-behavior>)), and leaves it to you not to do these things.

---

## 1. Why "It Works on My Machine" Means Nothing

The same program, built without any checking, runs four kinds of UB and reports **success** every time:

```cpp
// mode 1: read one past the end of a heap array
int* arr = new int[3]{1, 2, 3};
std::cout << arr[3];                 // printed 0: garbage that happened to be there

// mode 2: read after delete
int* p = new int(5);
delete p;
std::cout << *p;                     // printed 5: the freed memory hadn't been reused yet

// mode 3: write through a reference invalidated by vector growth
std::vector<int> v = {1, 2, 3};
int& first = v[0];
v.push_back(4);                      // reallocates: elements move to a new block
first = 10;
std::cout << v[0];                   // printed 1: the write went into freed memory, silently

// mode 5: read one past the end of a stack array
int a[3] = {1, 2, 3};
std::cout << a[3];                   // printed 1: whatever was next on the stack
```

```text
exit=0, exit=0, exit=0, exit=0
```

No crash and no error. Mode 2 even printed the "right" answer. This is why UB is so dangerous: the bug is there, but nothing tells you.

## 2. The Optimizer Assumes UB Never Happens

The compiler is allowed to assume your program has no UB, and optimizers use that assumption to generate faster code. Signed integer overflow is UB, so:

```cpp
bool will_overflow(int x) {
    return x + 1 < x;   // "if adding 1 made it smaller, it overflowed"
}

will_overflow(INT_MAX);
```

```text
-O0:  1  (true)   the addition wrapped around at runtime
-O2:  0  (false)  ...the optimizer reasoned "x + 1 < x can never be true without UB"
```

At `-O2`, the whole function compiled to:

```text
__Z13will_overflowi:
    mov  w0, #0      ; return false
    ret
```

The check you wrote was **deleted**. That's legal, because a program with UB has no meaning to preserve. (To check for overflow correctly, compare **before** adding: `x == INT_MAX`, or use `__builtin_add_overflow`.)

## 3. Common Sources of UB

| UB | Example | Python would |
|---|---|---|
| Out-of-bounds access | `arr[10]` on a 5-element array, `v[v.size()]` | Raise `IndexError` |
| Reading an uninitialized variable | `int x; std::cout << x;` | Raise `NameError` |
| Null pointer dereference | `Player* p = nullptr; p->health;` | Raise `AttributeError` on `None` |
| Use after free / dangling pointer or reference | `delete p; *p;` / returning `&local` | Impossible: objects live while referenced |
| Double free / mismatched `delete` | `delete p; delete p;` / `delete` on `new[]` | Impossible |
| Signed integer overflow | `INT_MAX + 1` | Just a bigger int |
| Integer division by zero | `x / 0` | Raise `ZeroDivisionError` |
| Shift by too many bits | `1 << 40` on a 32-bit `int` | Fine, ints grow |
| Modifying a variable twice without sequencing | `i++ + i++` | Well-defined, left to right |
| Iterator / reference invalidation | Using an iterator after `push_back` or `erase` | Defined behavior (though often a logic bug) |
| Data race | Two threads writing the same variable without a lock | The GIL makes most of these safe |
| Missing `return` in a non-`void` function | `int f() { }` | Returns `None` |
| Writing to a `const` object through `const_cast` | | n/a |

**Implementation-defined** behavior is different. The behavior varies by platform, but each implementation documents it and the program is still valid, like `sizeof(long)` being 8 or 4. **Unspecified** behavior is one of several valid outcomes, like the order in which function arguments are evaluated.

## 4. Compiler Warnings: Your First Line of Defense

Many UB bugs are visible to the compiler. Always build with warnings on:

```bash
clang++ -std=c++20 -Wall -Wextra -Wshadow -Wconversion main.cpp
```

Warnings seen throughout these notes, each of which points at UB or a near-UB bug:

```text
warning: variable 'x' is uninitialized when used here                       [-Wuninitialized]
warning: reference to stack memory associated with local variable returned  [-Wreturn-stack-address]
warning: multiple unsequenced modifications to 'i'                          [-Wunsequenced]
warning: shift count >= width of type                                       [-Wshift-count-overflow]
warning: non-void function does not return a value in all control paths     [-Wreturn-type]
warning: comparison of integers of different signs                          [-Wsign-compare]
```

Add `-Werror` to make warnings stop the build, so they can't be ignored.

## 5. Sanitizers: Catching UB at Runtime

Warnings only catch what's visible at compile time. **Sanitizers** add checks to the compiled program, so UB is caught at the moment it happens, with a report that says exactly where. They're the closest thing C++ has to Python's tracebacks.

```bash
clang++ -std=c++20 -g -O0 -fsanitize=address,undefined main.cpp -o main
./main
```

- `-g` includes debug info, so reports show file names and line numbers.
- `-fsanitize=address` turns on **AddressSanitizer (ASan)**: memory errors.
- `-fsanitize=undefined` turns on **UndefinedBehaviorSanitizer (UBSan)**: overflow, bad shifts, misaligned access, and more.

Programs run about 2x slower under ASan, which is fine for development and testing.

### AddressSanitizer Reports

Running the four "silent" examples from section 1 again, built with sanitizers:

**Heap buffer overflow** (mode 1, `arr[3]` on a 3-element heap array):

```text
==47140==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x6020000000dc
READ of size 4 at 0x6020000000dc thread T0
    #0 0x000104928dec in main ub.cpp:7                         <- where the bad read happened
0x6020000000dc is located 0 bytes after 12-byte region [...)   <- just past a 12-byte (3-int) block
allocated by thread T0 here:
    #1 0x000104928b7c in main ub.cpp:7                         <- where that block was allocated
```

**Use after free** (mode 2, `*p` after `delete p`):

```text
==47150==ERROR: AddressSanitizer: heap-use-after-free on address 0x6020000000d0
READ of size 4 at 0x6020000000d0 thread T0
    #0 0x00010099900c in main ub.cpp:8
0x6020000000d0 is located 0 bytes inside of 4-byte region [...)
freed by thread T0 here:                                       <- where it was deleted
    #1 0x000100998f80 in main ub.cpp:8
```

**Invalidated reference** (mode 3, writing through `first` after `push_back`). ASan sees it as a use-after-free, and shows that the vector's own allocator freed the block:

```text
==47155==ERROR: AddressSanitizer: heap-use-after-free on address 0x6020000000d0
WRITE of size 4 at 0x6020000000d0 thread T0
    #0 0x000100c4d1f4 in main ub.cpp:9
freed by thread T0 here:
    #2 ... in std::__1::allocator<int>::deallocate(int*, unsigned long) allocator.h:120
```

**Double free**:

```text
==47170==ERROR: AddressSanitizer: attempting double-free on 0x6020000000d0 in thread T0
```

**Stack buffer overflow** (mode 5, `a[3]` on a local array):

```text
==47165==ERROR: AddressSanitizer: stack-buffer-overflow on address 0x00016f809bfc
SUMMARY: AddressSanitizer: stack-buffer-overflow ub.cpp:11 in main
```

### How to Read an ASan Report

```text
1. First line   -> WHAT went wrong: heap-buffer-overflow, heap-use-after-free, double-free, ...
2. READ/WRITE   -> what kind of access, and how many bytes
3. First stack  -> WHERE the bad access happened (look for your own file and line)
4. Second stack -> the history of that memory: where it was allocated, or where it was freed
```

The second stack is often the useful one. "Freed by" tells you which code destroyed the object you later used.

### UBSan Reports

UBSan prints one line per problem, then the program continues:

```text
ub.cpp:10:41: runtime error: signed integer overflow: 2147483647 + 2 cannot be represented in type 'int'
ub.cpp:11:66: runtime error: index 3 out of bounds for type 'int[3]'
```

### Other Sanitizers

| Flag | Catches | Notes |
|---|---|---|
| `-fsanitize=address` | Out-of-bounds, use-after-free, double free, use-after-return | The one to always use |
| `-fsanitize=undefined` | Signed overflow, bad shifts, null dereference, invalid enum/bool values, ... | Combine with ASan |
| `-fsanitize=thread` | Data races between threads | Can't be combined with ASan |
| `-fsanitize=memory` | Reads of uninitialized memory | Clang on Linux only |
| `-fsanitize=leak` | Memory leaks | Included in ASan on Linux; **not supported on macOS** |

On macOS, find leaks with Apple's tools instead: `leaks --atExit -- ./main` from the terminal, or the **Leaks** instrument in Xcode Instruments.

## 6. A Workflow for Learning C++

1. Always compile with `-Wall -Wextra`, and fix every warning.
2. During development, build with `-g -O0 -fsanitize=address,undefined`.
3. When a program "works", run it again with sanitizers and edge-case inputs (empty containers, size 1, maximum values).
4. When a program crashes, read the sanitizer report before guessing.
5. Prefer constructs that make UB impossible: `std::vector` and `.at()` over raw arrays, smart pointers over `new`/`delete`, references over pointers where null isn't valid, range-based `for` over index arithmetic.

> In Unreal Engine, the debugger and the crash reporter's callstack are the everyday tools. Unreal also has `check()` (asserts in development builds and crashes with a message) and `ensure()` (logs a callstack once and continues). AddressSanitizer can be enabled for Unreal builds too, though it generally needs an engine built from source.
