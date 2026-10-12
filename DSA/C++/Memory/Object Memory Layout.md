# Object Memory Layout in C++

In Python, you never see how an object is laid out in memory: every object carries a header (reference count, type pointer), and attributes usually live in a per-object `__dict__`. A Python list holds references to objects scattered around the heap (see [List Structure](<../../Python/Fundamentals/List Structure.md#internal-storage>)).

In C++, an object is **exactly its data members, laid out in order, plus any padding the CPU's alignment rules require**, plus a vptr if the class has virtual functions. Nothing else. This note shows how to predict that layout, and why it matters for performance.

---

## 1. Members Are Stored In Order, Inline

```cpp
struct Vec3 {
    float x, y, z;
};

struct Particle {
    Vec3 pos;      // the whole Vec3 is stored INSIDE Particle, not referenced
    Vec3 vel;
    float life;
    bool alive;
};
```

```text
Particle (32 bytes):
offset:  0         12        24     28  29     32
         [  pos   ][  vel   ][life][a ][ pad  ]
          x  y  z   x  y  z
```

- No header, no type pointer, no per-object dictionary.
- Member objects (`pos`, `vel`) are embedded directly. A Python attribute would be a reference to a separate object.
- Members appear in **declaration order** (within the same access level), at increasing addresses.

## 2. Alignment

Each type has an **alignment** requirement: its address must be a multiple of that number. CPUs load aligned values in one operation. Misaligned access is slower on some CPUs and invalid on others.

```cpp
alignof(char)     // 1: any address
alignof(int)      // 4: address must be a multiple of 4
alignof(double)   // 8: address must be a multiple of 8
alignof(Vec3)     // 4: a struct's alignment is its strictest member's alignment
```

For fundamental types, alignment is usually equal to the size.

## 3. Padding

To keep every member aligned, the compiler inserts unused **padding** bytes between members, and at the end of the struct:

```cpp
struct Bad {
    char a;   // 1 byte
    int b;    // 4 bytes, must start at a multiple of 4
    char c;   // 1 byte
};

sizeof(Bad)   // 12, not 6!
```

```text
Bad (12 bytes):
offset:  0    1    2    3    4    5    6    7    8    9    10   11
         [ a ][ pad pad pad ][      b           ][ c ][ pad pad pad ]
```

- 3 bytes of padding after `a`, so that `b` starts at offset 4.
- 3 bytes of **tail padding** after `c`, so the total size (12) is a multiple of the alignment (4). That way, in an **array** of `Bad`, every element's `b` is still aligned.

Clang can print the exact layout:

```bash
clang++ -std=c++20 -Xclang -fdump-record-layouts -c layout.cpp -o /dev/null
```

```text
         0 | struct Bad
         0 |   char a
         4 |   int b
         8 |   char c
           | [sizeof=12, dsize=12, align=4]
```

`offsetof(Bad, b)` (from `<cstddef>`) gives the same information in code: `4`.

### Ordering Members to Reduce Padding

The compiler **won't** reorder your members for you, because declaration order is part of the language's guarantees. Ordering them from largest to smallest alignment usually removes most padding:

```cpp
struct Good {
    int b;    // offset 0
    char a;   // offset 4
    char c;   // offset 5
};            // 2 bytes tail padding

sizeof(Good)  // 8 (was 12)
```

```cpp
struct Mixed       { char flag; double value; int id; };   // sizeof = 24: offsets 0, 8, 16, with 7 + 4 bytes padding
struct MixedSorted { double value; int id; char flag; };   // sizeof = 16
```

A 33% saving per object adds up across a million of them: less memory, and more objects per cache line.

## 4. Special Cases

```cpp
struct Empty {};
sizeof(Empty)       // 1: every object needs a distinct address, so even an empty class takes a byte

struct WithVirt {
    virtual ~WithVirt() = default;
    int hp;
};
sizeof(WithVirt)    // 16: 8-byte vptr first, then the 4-byte hp, then 4 bytes tail padding
```

The hidden **vptr** is the price of virtual functions (see [Virtual Functions and Vtables](<../OOP/Virtual Functions and Vtables.md#the-costs-of-virtual>)).

### Booleans and Bit-Fields

A `bool` takes a whole byte. To pack several flags into one byte, use **bit-fields** or bit masks (see [Operators](<../Fundamentals/Operators.md#bit-flags>)):

```cpp
struct Flags { bool a, b, c, d; };     // sizeof = 4: one byte each

struct Bits {
    unsigned a : 1;                    // `: 1` means "use only 1 bit"
    unsigned b : 1;
    unsigned c : 1;
    unsigned d : 1;
};                                     // sizeof = 4 here (the size of the underlying unsigned); 28 bits unused
                                       // with more fields, all of them still fit in those 4 bytes
```

> Unreal uses bit-field booleans all over its engine classes, for example `uint8 bCanBeDamaged : 1;`.

## 5. Inheritance Layout

A derived object contains a complete base-class object at its start, followed by the derived members:

```cpp
struct Animal { int legs; };           // 4 bytes
struct Dog : Animal { int tricks; };   // 8 bytes

// Dog:  [ legs (the Animal part) | tricks ]
```

That's why a `Dog*` can be converted to an `Animal*` for free: with single inheritance, the address doesn't even change. It's also why slicing happens: copying into an `Animal` keeps only the first part (see [OOP Four Pillars](<../OOP/OOP Four Pillars.md#43-polymorphism-needs-pointers-or-references-object-slicing>)).

## 6. Arrays Are Contiguous

An array or `std::vector` of objects stores the objects themselves, back to back, with each element exactly `sizeof(T)` bytes after the previous one:

```cpp
Particle ps[3];
// address of ps[1] - address of ps[0] = 32 = sizeof(Particle)

std::vector<Vec3> v(3);
// address of v[2] - address of v[0] = 24 = 2 * sizeof(Vec3)
```

```text
C++ std::vector<Vec3>:
  heap: [ x y z ][ x y z ][ x y z ]                     one block, 36 bytes, values inline

Python list of Vec3 objects:
  list: [ ptr ][ ptr ][ ptr ]                           array of references
           |      |      |
           v      v      v
        object  object  object    each with a header and its own attribute storage, scattered on the heap
```

This difference is one of the biggest reasons C++ is fast at bulk data processing.

## 7. Why Layout Matters: The Cache

The CPU doesn't read memory one byte at a time. It loads whole **cache lines**: 64 bytes on most x86 CPUs, and 128 bytes on Apple Silicon (`sysctl -n hw.cachelinesize` prints `128`). Reading a value that's already in cache takes about 1 ns. Going to main memory takes about 100 ns.

So:

- **Contiguous arrays are fast to iterate**: every cache line loaded brings in the next several elements too, and the hardware prefetcher sees the pattern and loads ahead.
- **Pointer chasing is slow**: a linked list, or a `std::vector<std::unique_ptr<T>>` with each object allocated separately on the heap, can mean a cache miss on every element (see [Linked List](<../../Python/Fundamentals/Linked List.md>)).
- **Smaller objects are faster**: removing padding or unused members means more objects per cache line.

### Array of Structs vs Struct of Arrays

If a hot loop only uses **some** members, the others are still loaded into cache alongside them:

```cpp
// Array of Structs (AoS): natural, object-oriented
struct Particle { Vec3 pos; Vec3 vel; float life; bool alive; };   // 32 bytes
std::vector<Particle> particles;

for (auto& p : particles) {
    p.pos.x += p.vel.x * dt;   // uses pos and vel, but every 32-byte element is loaded, life and alive included
}
```

```cpp
// Struct of Arrays (SoA): one array per member
struct Particles {
    std::vector<Vec3> pos;
    std::vector<Vec3> vel;
    std::vector<float> life;
    std::vector<char> alive;   // (not std::vector<bool>, which is a bit-packed special case)
};

for (std::size_t i = 0; i < ps.pos.size(); ++i) {
    ps.pos[i].x += ps.vel[i].x * dt;   // reads only the pos and vel arrays: every byte loaded is used
}
```

AoS is simpler and is the right default. SoA is a targeted optimization for hot loops over large numbers of objects. It's the core idea behind "data-oriented design" and ECS architectures, such as Unreal's Mass framework.

### False Sharing (Multithreading)

If two threads write to different variables that happen to sit in the **same cache line**, the CPU cores keep stealing that line from each other, which is very slow even though they never touch the same variable. The fix is to align per-thread data to separate cache lines with `alignas`:

```cpp
struct alignas(128) PerThreadCounter {   // 128 on Apple Silicon. Or use std::hardware_destructive_interference_size from <new>
    long count = 0;
};
```

## 8. `sizeof` Is a Compile-Time Constant

Since the layout is fixed at compile time, so is `sizeof`. Adding a member to a class changes its size, and therefore requires recompiling **every** file that creates or embeds that class. That's another reason headers trigger large rebuilds (see [Compilation and Linking](<../Fundamentals/Compilation and Linking.md#incremental-builds>)), and why forward declarations, which only need a pointer's fixed 8 bytes, help (see [Headers and Source Files](<../Fundamentals/Headers and Source Files.md#forward-declarations>)).

## Summary

```text
Object        = members in declaration order + padding (+ vptr if any virtual functions)
Alignment     = each member at a multiple of its alignment; struct alignment = strictest member's
sizeof        = always a multiple of the alignment (tail padding), so arrays stay aligned
Reduce size   -> order members from largest to smallest alignment; bit-fields for flags; avoid needless virtual
Arrays        -> elements stored inline and contiguously: cache-friendly
Performance   -> contiguous > pointer chasing; small objects > padded ones; SoA for hot loops over a few members
```
