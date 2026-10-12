# Stack and Heap in C++

In Python you never decide **where** an object lives. Every object is allocated by the interpreter on its private heap, and reference counting plus the garbage collector decide when it's freed.

In C++ you make that choice for every object, and the choice decides how fast it is to create, how long it lives, and who is responsible for destroying it. This note covers the two main places objects live: the **stack** and the **heap**.

## The Process Memory Layout

When a C++ program runs, the OS gives it one virtual address space, divided into regions:

```text
 high addresses
 +-------------------------+
 |         Stack           |  local variables, parameters, return addresses
 |           |             |  grows DOWN as functions are called
 |           v             |
 |                         |
 |           ^             |
 |           |             |
 |          Heap           |  objects created with new / by containers
 |                         |  grows UP as memory is requested
 +-------------------------+
 |  Static data (.data/.bss)|  globals, static variables
 +-------------------------+
 |  Code (.text)           |  the compiled machine code of your functions
 +-------------------------+
 low addresses
```

Printing addresses from a real program on an Apple Silicon Mac shows the same layout:

```cpp
int global_var = 1;

void inner() { int c = 3; std::cout << "inner local c: " << &c << "\n"; }
void outer() { int b = 2; std::cout << "outer local b: " << &b << "\n"; inner(); }

int main() {
    int a = 1;
    std::cout << "main local a:  " << &a << "\n";
    outer();

    int* h1 = new int(42);
    int* h2 = new int(43);
    std::cout << "heap h1:       " << h1 << "\n";
    std::cout << "heap h2:       " << h2 << "\n";
    std::cout << "global:        " << &global_var << "\n";
    std::cout << "code (main):   " << (void*)&main << "\n";

    delete h1;
    delete h2;
}
```

```text
main local a:  0x16ee09fc4    <- stack (high addresses)
outer local b: 0x16ee09e9c    <- lower: outer's frame sits below main's
inner local c: 0x16ee09e7c    <- lower still: the stack grows down
heap h1:       0x10102db60    <- heap
heap h2:       0x10102db70    <- higher: 16 bytes later
global:        0x100ffc000    <- static data
code (main):   0x100ff47d0    <- code (lowest)
```

`&x` gives the address of `x`. That's the **address-of** operator, explained in [Pointers](<Pointers.md>).

## The Stack

### How It Works

The stack is a contiguous block of memory reserved for each thread (8 MB for the main thread on macOS and Linux by default, 1 MB on Windows). The CPU keeps a **stack pointer** register pointing at its current top.

Each function call pushes a **stack frame** holding that call's parameters, local variables, and return address. When the function returns, its frame is popped by simply moving the stack pointer back:

```text
main() calls outer(), which calls inner():

 +----------------------+  <- top of the stack region
 | main's frame         |
 |   a = 1              |
 +----------------------+
 | outer's frame        |
 |   b = 2              |
 |   return address     |
 +----------------------+
 | inner's frame        |
 |   c = 3              |
 |   return address     |
 +----------------------+  <- stack pointer

inner() returns  ->  stack pointer moves back up; c's memory is now free for the next call
```

It's the same LIFO discipline as the stack data structure in [Stack and queue](<../../Python/Fundamentals/Stack and queue.md>), implemented in hardware.

### Automatic Variables Live on the Stack

Every ordinary local variable is an **automatic** object (see [Scope and Namespaces](<../Fundamentals/Scope and Namespaces.md#5-storage-duration-how-long-a-variable-lives>)). It lives in its function's frame and is destroyed automatically when its block ends:

```cpp
void spawn_wave() {
    int count = 5;                   // on the stack
    Enemy boss("Warden", 500);       // the whole Enemy object is on the stack, not just a reference to it
    std::string label = "Wave 1";    // the std::string object is on the stack
}                                    // label, boss, count destroyed here, in reverse order
```

Unlike Python, `Enemy boss(...)` doesn't create a reference to an object stored elsewhere. The object's bytes sit directly in the stack frame.

### Why the Stack Is Fast

- **Allocation is one instruction**: moving the stack pointer reserves room for every local in the frame at once.
- **Freeing is free**: returning moves the pointer back, with nothing to search or bookkeep.
- **Cache-friendly**: the top of the stack is almost always in the CPU cache, because it was just used.

### The Limits of the Stack

1. **Fixed, small size.** A big local array or very deep recursion runs out of room, which is a **stack overflow**:

    ```cpp
    void recurse() {
        char buffer[1024];   // 1 KB per frame
        recurse();           // no base case
    }
    // After roughly 7,000-8,000 calls (about 8 MB), the program crashes:
    // Segmentation fault, exit code 139 (128 + signal 11, SIGSEGV)
    ```

    Python raises a catchable `RecursionError` long before this. C++ just crashes. (See [Recursion](<../../Python/Traversal and Search Algorithms/Recursion.md>).)

2. **Lifetime is tied to the scope.** A stack object can't outlive the function that created it. Returning a pointer or reference to a local is a classic bug:

    ```cpp
    int* make_number() {
        int n = 42;
        return &n;            // warning: address of stack memory associated with local variable 'n' returned
    }                         // n's frame is popped here

    int* p = make_number();   // p points into a dead frame: a dangling pointer
    std::cout << *p;          // UB: may print 42, garbage, or crash, depending on what reused that memory
    ```

3. **Size must be known at compile time** (for plain arrays). `int arr[n];` with a runtime `n` isn't standard C++.

## The Heap

### How It Works

The **heap** (the standard calls it the **free store**) is a large pool of memory managed by an **allocator** (`malloc` underneath). You ask for a block of a given size, and the allocator finds a free region, records that it's in use, and returns its address. The block stays reserved **until you explicitly give it back**.

### `new` and `delete`

`new` allocates memory on the heap, constructs an object there, and returns a **pointer** to it. `delete` destroys the object and releases the memory:

```cpp
Enemy* boss = new Enemy("Warden", 500);   // allocate + construct on the heap; boss holds its address
boss->take_damage(50);                     // use it through the pointer
delete boss;                               // destroy + free. You MUST do this exactly once
boss = nullptr;                            // optional: so a later mistaken use fails loudly
```

```text
stack                         heap
 boss: [0x10102db60] -------> [ Enemy "Warden", health 500 ]
 (8-byte pointer)              (the actual object)
```

Arrays use a separate pair, `new[]` and `delete[]`, and mixing up the forms is undefined behavior:

```cpp
int* scores = new int[5]{1, 2, 3, 4, 5};   // array of 5 ints on the heap
std::cout << scores[4];                    // 5
delete[] scores;                           // must be delete[], not delete
```

### When You Need the Heap

- **The object must outlive the current function.** For example, a function creates an enemy that the level keeps afterwards.
- **The size is only known at runtime.** For example, "load N tiles from a file".
- **The object is too large for the stack.** For example, a 100 MB buffer.
- **Polymorphism through base pointers.** Storing different derived classes in one container (see [Virtual Functions and Vtables](<../OOP/Virtual Functions and Vtables.md>)).

### Why the Heap Is Slower

- The allocator has to search for a suitable free block, and may have to lock if several threads allocate at once.
- Every allocation carries bookkeeping overhead. On this Mac, two consecutive `new int` calls returned addresses 16 bytes apart for a 4-byte `int`.
- Objects allocated at different times end up scattered in memory, which hurts cache locality (see [Object Memory Layout](<Object Memory Layout.md>)).

Game engines often allocate up front and reuse memory (pools, arenas) to avoid allocating in the middle of a frame.

## How the Stack and Heap Grow

The two regions grow in completely different ways, and that difference decides what can live where.

### The Stack: Contiguous Frames With Fixed Layouts

The compiler decides each function's frame layout **before the program runs**: how big the frame is, and the fixed offset of every local inside it. The generated machine code reaches each local as "stack pointer + offset", so those offsets are baked into the instructions.

Frames are packed back to back, with each new call's frame placed directly below its caller's:

```text
 +------------------------------+
 | main's frame                 |
 +------------------------------+
 | outer's frame                |
 |   int score     (offset 0)   |
 |   int cells[4]  (offset 4)   |
 |   int hp        (offset 20)  |
 +------------------------------+
 | inner's frame                |  <- directly below, no gap
 +------------------------------+  <- stack pointer
```

So a frame can't change size while its function runs. If `cells` needed more room, it would overwrite `hp`, and below that the next call's frame, and every offset baked into the code would be wrong. That's why everything on the stack must have a size known at compile time, and why a `std::vector` keeps only its fixed 24-byte header (pointer, size, capacity) on the stack, and its elements on the heap.

The same rigid structure is what makes the stack so cheap. Growing means moving the stack pointer down by a frame's size, and shrinking means moving it back up. Frames are always freed in exact reverse order (LIFO), so the free space is always one unbroken region below the stack pointer. **The stack can never have holes.**

### The Heap: Blocks Anywhere, Freed in Any Order

The heap has no fixed layout. The allocator hands out blocks of whatever size is asked for at runtime, and they can be freed in **any order**. Freeing a block in the middle leaves a hole:

```text
allocate A, B, C, D:   [ A ][ B ][ C ][ D ]
free B and D:          [ A ][    ][ C ][    ]   <- holes
allocate E (small):    [ A ][ E ][ C ][    ]   <- E reuses B's hole
allocate F (large):    [ A ][ E ][ C ][    ]  ...  [ F ]   <- too big for any hole, goes elsewhere
```

To manage this, the allocator keeps track of which regions are in use and which are free, groups blocks by size, and searches for a suitable hole on every allocation. That bookkeeping is part of why `new` is slower than a stack allocation, and why a 4-byte `new int` used 16 bytes earlier.

Holes cause **fragmentation**. Plenty of memory can be free in total while no single hole is big enough for a large request, and objects allocated one after another end up scattered, which hurts cache performance. This is a big reason game engines use pools and arenas: same-sized blocks in one region can't leave unusable holes.

When no free region is big enough, the allocator asks the OS for more pages (`mmap` on macOS and Linux). Those pages can come from anywhere in the address space, so heap addresses don't simply increase over time.

### Why the Diagram Shows Them Growing Toward Each Other

The memory layout diagram at the top is the classic picture: one stack at the top growing down, one heap at the bottom growing up. Placing them at opposite ends meant either one could use the shared gap without anyone deciding the split in advance. Modern programs are messier. Each thread gets its own fixed-size stack, and the heap is many separate regions requested from the OS. The picture is still the right mental model for how each region behaves.

### At the Limit

- **Stack:** the stack has a fixed maximum size (8 MB for the main thread here). Below it sits a **guard page** that the program isn't allowed to touch. Going past the limit hits the guard page and the program crashes with a stack overflow (`SIGSEGV`, as in the recursion example above).
- **Heap:** the allocator asks the OS for more memory. It only fails when the system really can't provide it, and then `new` throws `std::bad_alloc` (`malloc` returns `nullptr`).

## What Goes Wrong With Manual `new`/`delete`

Python's reference counting makes all of these impossible. In C++ with raw `new`/`delete`, each one is easy to write:

| Bug | What happens | Example |
|---|---|---|
| **Memory leak** | Forgot to `delete`; memory is never returned | `new` in a loop with no `delete` |
| **Dangling pointer / use-after-free** | Using an object after it was deleted | `delete p; p->update();` |
| **Double free** | `delete`-ing the same block twice; corrupts the allocator | Two pointers to one object, both deleted |
| **Mismatched delete** | `delete` on something from `new[]`, or vice versa | `delete scores;` instead of `delete[] scores;` |
| **Leak on early exit** | A `return` or exception skips the `delete` | See below |

The last one is the subtlest:

```cpp
void process() {
    Enemy* e = new Enemy("Grunt", 100);
    if (!e->is_valid()) {
        return;                // LEAK: the delete below never runs
    }
    risky_operation();         // if this throws an exception: also a leak
    delete e;
}
```

## The Modern Answer: Let Objects Own Memory

Modern C++ code almost never writes `new` and `delete` directly. Instead, a **stack object owns the heap memory** and frees it in its destructor, automatically, on every path out of the scope:

```cpp
#include <memory>
#include <vector>

void process() {
    auto e = std::make_unique<Enemy>("Grunt", 100);   // heap Enemy, owned by a stack object `e`
    if (!e->is_valid()) {
        return;                                       // e's destructor deletes the Enemy. No leak
    }
    risky_operation();                                // exception? e's destructor still runs. No leak
}                                                     // normal exit: deleted here

std::vector<int> scores(1'000'000);   // the vector object is on the stack;
                                      // its million ints are on the heap;
                                      // the vector frees them when it goes out of scope
```

This pattern is **RAII**, the most important idea in C++. It's covered in [Object Lifetime and RAII](<../OOP/Object Lifetime and RAII.md>) and [Smart Pointers](<Smart Pointers.md>).

Most standard types already work this way. A `std::vector` or `std::string` is a small object (24 bytes here) that can sit on the stack, holding a pointer to its data on the heap:

```text
stack                                   heap
 scores: [ data ptr | size | capacity ] ---> [ 0 ][ 0 ][ 0 ] ... (1,000,000 ints)
```

This is the same size/capacity idea described in [List Structure](<../../Python/Fundamentals/List Structure.md#size-vs-capacity>), except the heap block holds the `int` values themselves, not references to separate `int` objects.

> Unreal Engine adds a third model on top of these two: `UObject`s are created with `NewObject<T>()` (never plain `new`) and freed by Unreal's own **garbage collector**, which only knows an object is still in use if a `UPROPERTY()` pointer refers to it.

## Summary

| | Stack | Heap |
|---|---|---|
| Created by | Declaring a local variable | `new`, `std::make_unique`, containers |
| Destroyed | Automatically, at the end of the block | When `delete`d, or when the owning object is destroyed |
| Speed | Extremely fast (move a pointer) | Slower (allocator search + bookkeeping) |
| Size | Small and fixed (MBs) | Large (limited by system memory) |
| Lifetime | Can't outlive its function | As long as you want |
| Typical bugs | Stack overflow, returning addresses of locals | Leaks, use-after-free, double free |
| Python equivalent | None: Python objects are always on the heap | Every Python object |

Default to the stack. Use the heap through an owning object (`std::vector`, `std::string`, `std::unique_ptr`) when you need a runtime size, a longer lifetime, or polymorphism.
