# Variables and Types in C++

## Variables Are Boxes, Not Name Tags

In Python, a name is a label bound to an object that lives somewhere else, and the same name can be rebound to any object of any type (see [Name Rebinding](<../../Python/Fundamentals/Name Rebinding.md>)).

In C++, a **variable** is a named region of memory with a fixed **type**. The type decides:

- How many bytes the variable occupies
- How those bytes are interpreted (as an integer, a floating-point number, a character, ...)
- Which operations are allowed on it

```cpp
int score = 10;   // reserve 4 bytes, interpret them as a signed integer, store 10
score = 15;       // overwrite those same 4 bytes with 15. No new object, no rebinding
```

Conceptually:

```text
Python:   score ──> int object 10        then   score ──> int object 15   (name moves to a new object)
C++:      score: [ 10 ]                  then   score: [ 15 ]             (same box, new contents)
```

## Declaration and Initialization

The basic form is `type name = value;`:

```cpp
int lives = 3;
double speed = 4.5;
char grade = 'A';
bool is_running = true;
```

Several variables of the same type can be declared together, though one per line is clearer:

```cpp
int x = 1, y = 2, z = 3;
```

Unlike Python, you **can** declare a variable without giving it a value:

```cpp
int count;            // declared, but NOT initialized
std::cout << count;   // undefined behavior: reads whatever bytes happened to be in that memory
```

```python
count                 # Python: NameError, the name doesn't exist at all
```

A local variable of a fundamental type that isn't initialized holds an **indeterminate** value, and reading it is undefined behavior (see [Undefined Behavior and Sanitizers](<../Memory/Undefined Behavior and Sanitizers.md>)). `-Wall` catches the obvious cases:

```text
warning: variable 'count' is uninitialized when used here [-Wuninitialized]
```

> **Always initialize variables.** Unreal's coding standard requires it too.

### Three Ways to Initialize

```cpp
int a = 5;     // copy initialization
int b(5);      // direct initialization
int c{5};      // brace (list) initialization, C++11
int d{};       // brace with nothing inside: value-initialized to zero
```

For a plain `int` the first three all store `5`. The difference is that **brace initialization refuses narrowing conversions**, which are conversions that can lose information:

```cpp
int a = 3.7;      // compiles (with a warning): a = 3, the .7 is silently dropped
int b{3.7};       // ERROR: type 'double' cannot be narrowed to 'int' in initializer list
char c{300};      // ERROR: constant expression evaluates to 300 which cannot be narrowed to type 'char'
long big = 5;
int e{big};       // ERROR: non-constant-expression cannot be narrowed from type 'long' to 'int'
```

Prefer `{}` when you want the compiler to catch accidental data loss. `int total{};` is also a clean way to say "start at zero".

> One gotcha: `Widget w();` with empty parentheses does **not** create an object. It declares a function named `w` that returns a `Widget` (the "most vexing parse"). `Widget w;` or `Widget w{};` creates the object.

## Fundamental Types

| Category | Type | Size (Apple Silicon Mac) | Range / precision | Notes |
|---|---|---|---|---|
| Boolean | `bool` | 1 byte | `true` / `false` | The smallest addressable unit is a byte, so even a bool takes one |
| Character | `char` | 1 byte | −128 … 127 | Enough for every ASCII character (see [Characters](#characters)) |
| Integer | `short` | 2 bytes | −32,768 … 32,767 | |
| | `int` | 4 bytes | ±2.1 billion | The default integer type |
| | `long` | 8 bytes | ±9.2 × 10^18 | **4 bytes on Windows**: the classic portability trap |
| | `long long` | 8 bytes | ±9.2 × 10^18 | Guaranteed at least 64 bits everywhere |
| Floating point | `float` | 4 bytes | ~7 significant digits | IEEE 754 **single** precision |
| | `double` | 8 bytes | ~15-16 digits | **Double** precision (twice float's bits); the same as Python's `float` |
| | `long double` | 8 bytes here (16 on x86 Linux) | | |
| No value | `void` | n/a | | "Nothing": a function that returns nothing, or a generic pointer `void*` |

Every range in that table comes from the type's bit count, explained in the next two sections.

The standard only guarantees **minimum** sizes (`int` at least 16 bits, `long` at least 32, `long long` at least 64). The exact sizes are implementation-defined. `int` was meant to be the machine's natural word size: 16 bits on 1970s machines and 32 since, and it stayed 32 on 64-bit CPUs for compatibility and to save memory. `sizeof` tells you the size in **bytes** on the machine you're compiling for:

```cpp
std::cout << sizeof(int) << "\n";      // 4
std::cout << sizeof(long) << "\n";     // 8 (macOS/Linux), 4 (Windows)
std::cout << sizeof(double) << "\n";   // 8
std::cout << sizeof(int*) << "\n";     // 8: a pointer holds a memory address, which is 64 bits on a 64-bit machine
std::cout << sizeof(score) << "\n";    // 4: works on variables too
```

### Bits, Bytes, and Ranges

Memory is addressed in **bytes** of 8 bits, so every type is a whole number of bytes: 8, 16, 32, or 64 bits. n bits give `2^n` distinct patterns, so the largest unsigned value is all 1s, `2^n - 1`, just as the largest 3-digit decimal number is `999 = 10^3 - 1`:

```text
8 bits:  11111111 = 255 = 2^8 - 1        unsigned range: 0 ... 2^n - 1
```

Python never shows you this, because its `int` grows as large as it needs. A C++ integer is a fixed-width bit pattern, and its range is fixed with it.

### Signed and Unsigned Integers

Each integer type comes in a **signed** (default) and an **unsigned** version:

```cpp
int a = -5;               // signed: negative and positive values
unsigned int b = 5;       // unsigned: zero and positive only, double the positive range
unsigned c = 5;           // "unsigned" alone means unsigned int
```

They are separate types (`f(int)` and `f(unsigned int)` are different overloads) with the **same size**. The bits are identical; only how they're read differs. Unsigned reads them as plain binary. Signed reads them as **two's complement**, which C++20 made mandatory:

#### Two's Complement: the Top Bit Weighs −2^(n−1)

In two's complement, the top bit has a **negative** weight and every other bit keeps its usual positive weight:

```text
weights:   -128  64  32  16   8   4   2   1

01111111 =        64+ 32+ 16+  8+  4+  2+  1 =  127    (max)
10000000 = -128                              = -128    (min)
11111111 = -128 + 127                        =   -1
11111011 = -128 + 64+32+16+8 + 2+1           =   -5    (the same bits are 251 unsigned)
```

So an n-bit signed type holds `-2^(n-1) ... 2^(n-1) - 1`. There's one more negative than positive because zero takes one of the non-negative patterns. It isn't "1 sign bit + 7 bits of magnitude": that scheme (sign-magnitude) would give −127 … 127 and waste a pattern on −0.

**Why it's built this way.** The goal is for the CPU's ordinary adder to work on negatives too. The adder adds column by column and drops any carry out of the top bit. So the pattern for −5 should be whatever gives 0 when added to 5.

**Step 1: flip every bit.** `5 + ~5` never carries, because every column is `1+0` or `0+1`. It always lands on all 1s:

```text
  00000101   (5)
+ 11111010   (~5)
= 11111111   all 1s: one step short of 0
```

**Step 2: add 1.** All 1s plus 1 carries through every column, and the carry out of the top is dropped:

```text
  11111111
+ 00000001
= 1 00000000   → 00000000 = 0 ✓
```

So `5 + (~5 + 1) = 0`, which makes `~5 + 1 = 11111011` the pattern for −5. In general, **−x is `~x + 1`: flip every bit, then add 1.** Flipping alone (one's complement) misses by one.

The same idea on a clock: 8 bits wrap after 255, like a clock with 256 positions. Flipping computes `255 - x`, and the +1 makes it `256 - x`, which is x steps back from 0. For 5, that's `256 - 5 = 251 = 11111011`.

What this buys:

- **One adder for everything:** subtraction is `a + ~b + 1`, and signed and unsigned addition are the same operation on the same bits.
- **One zero:** no wasted −0 pattern.
- **Easy sign check:** top bit 1 means negative. To read a negative pattern, negate it (flip, +1) to get the magnitude: `11110101` → `00001010` + 1 = `00001011` = 11, so the value is −11.
- **`~` is one's complement:** `~` flips the bits without the +1, so `~x == -x - 1` (`~5` is −6), the same as Python's `~`.

```cpp
#include <limits>

std::cout << std::numeric_limits<int>::max() << "\n";           // 2147483647   (2^31 - 1)
std::cout << std::numeric_limits<int>::min() << "\n";           // -2147483648  (-2^31)
std::cout << std::numeric_limits<unsigned int>::max() << "\n";  // 4294967295   (2^32 - 1)
std::cout << std::numeric_limits<long long>::max() << "\n";     // 9223372036854775807
```

Because the same bits mean different values, mixing signed and unsigned is a trap: in `-1 < 1u`, the −1 is converted to 4294967295 first. Converting to a smaller type keeps the low bits, so `char c = 300;` stores 44, and `char d = 200;` stores the pattern `11001000`, which reads as −56. See [Signed / Unsigned Mixing](<Type Conversion.md#signed--unsigned-mixing>) and [Narrowing Conversions](<Type Conversion.md#narrowing-conversions>).

### Overflow: the Big Difference from Python

Python's `int` grows to any size it needs. C++ integers have a fixed number of bits, so they **overflow**:

```cpp
unsigned char u = 255;
u++;                    // 0: unsigned overflow WRAPS AROUND (modulo 2^n). Well-defined.

int m = 2147483647;     // INT_MAX
m = m + 1;              // UNDEFINED BEHAVIOR: signed overflow is not defined to wrap
```

```python
m = 2147483647
m = m + 1               # 2147483648, Python just uses more memory
```

Unsigned wraparound is the 256-position clock from above: going past the top lands back at 0, and going below 0 lands at the top (`0u - 1` is 4294967295). Two's complement being mandatory does **not** make signed overflow wrap. It is still undefined behavior, `-fsanitize=undefined` reports it at runtime, and the optimizer assumes it never happens, so it may delete checks like `if (x + 1 < x)`. If you need values larger than about 2 billion, use `long long` / `int64_t` from the start.

> The modular arithmetic trick of taking `% MOD` after every step (see [Modular Arithmetic](<../../Python/Fundamentals/Modular Arithmetic.md>)) is often **required** in C++ to stay inside the type's range, not just to keep the numbers small.

### Fixed-Width Integer Types

Since `int` and `long` vary by platform, `<cstdint>` provides types with an exact size:

```cpp
#include <cstdint>

int8_t    a;   // exactly 8 bits, signed      uint8_t   e;   // exactly 8 bits, unsigned
int16_t   b;   // exactly 16 bits             uint16_t  f;
int32_t   c;   // exactly 32 bits             uint32_t  g;
int64_t   d;   // exactly 64 bits             uint64_t  h;

size_t    n;   // unsigned, big enough to hold the size of any object: what sizeof and .size() return
```

`size_t` is 64 bits on a 64-bit machine (`unsigned long` on macOS), because it must be able to count every byte of memory you could address. Being unsigned, it can't go below zero, which is the source of the `v.size() - 1` trap on an empty vector.

> Unreal Engine defines its own equivalents and uses them everywhere: `int32`, `uint8`, `int64`, `uint32`, and so on. Plain `int` is avoided in UE code partly for the portability reason above. Unreal also makes `TArray::Num()` a signed `int32`, so `Num() - 1` on an empty array is just −1, and `INDEX_NONE` can be −1 (all 1 bits).

### Floating-Point Types

`float` is 32 bits and `double` is 64 bits, in both C and C++, following the IEEE 754 standard's single- and double-precision formats. Python's `float` is the same thing as a C++ `double`.

Both store a number as a **binary** fraction, so a decimal is stored exactly only if, written as a fraction, its denominator is a power of 2: `0.5` (1/2), `0.25` (1/4), and `0.375` (3/8) are exact. Most decimals are not. `0.1` is 1/10, and the factor of 5 makes it repeat forever in binary (`0.000110011001100…`), the way 1/3 repeats in decimal. It is cut off and rounded to the nearest representable value. Of `0.0`, `0.1`, … `0.9`, only `0.0` and `0.5` are exact.

Those small rounding errors mean arithmetic results rarely land on the exact value you'd expect:

```cpp
std::cout << (0.1 + 0.2 == 0.3) << "\n";   // 0 (false), same as in Python
// 0.1 is stored as 0.1000000000000000055…, 0.2 as 0.2000000000000000111…
// their sum rounds to 0.3000000000000000444…, but 0.3 is stored as 0.2999999999999999888…
```

Some sums happen to come out equal (`0.1 + 0.1 == 0.2` is true), but there's no simple way to predict which. So **never compare floating-point results with `==`**. Check that they're within a small tolerance instead:

```cpp
#include <cmath>
std::abs((0.1 + 0.2) - 0.3) < 1e-9          // true
```

The Python equivalent is `math.isclose(a, b)`, and Unreal's is `FMath::IsNearlyEqual(A, B)`.

Whole numbers are exact only up to a limit: `2^53` for `double` and `2^24` (about 16.7 million) for `float`. Beyond that, the gap between neighbouring representable values is bigger than 1.

`float` has much less precision than `double`:

```cpp
std::cout << std::setprecision(20);   // needs #include <iomanip>
std::cout << 0.1f << "\n";            // 0.10000000149011611938   (float)
std::cout << 0.1  << "\n";            // 0.10000000000000000555   (double)

float big = 16777217.0f;              // 2^24 + 1
std::cout << big << "\n";             // 16777216: float can't represent every integer above 2^24
```

> Games use `float` heavily because it's half the size and fast on GPUs. Unreal's `FVector` used `float` in UE4 and switched to `double` in UE5 to support large worlds.

### Characters

A `char` is a 1-byte integer that is usually used to hold one character of text. One byte is enough because ASCII defines 128 characters (codes 0–127, which need 7 bits), and 8 bits give 256 patterns. A character literal like `'A'` is just its ASCII code:

```cpp
char ch = 'A';
std::cout << ch << "\n";        // A
std::cout << (int)ch << "\n";   // 65, see ASCII values
ch += 2;
std::cout << ch << "\n";        // C
```

Whether plain `char` is signed is up to the platform. It is signed on macOS and Windows (−128 … 127) but unsigned on ARM Linux (0 … 255), so use `signed char` / `unsigned char` (or `int8_t` / `uint8_t`) when you use it as a number. `std::cout` prints a `char` as a character, so cast to `int` to see its value.

This replaces Python's `ord()` and `chr()`. See [ASCII values](<../../Python/Fundamentals/ASCII values.md>). In C++, the conversion is just a cast:

| Python | C++ |
|---|---|
| `ord('A')` | `(int)'A'` or `static_cast<int>('A')` |
| `chr(65)` | `(char)65` or `static_cast<char>(65)` |
| `ord(c) - ord('a')` | `c - 'a'` (char arithmetic produces an `int`) |

> Single quotes and double quotes mean different things in C++. `'A'` is a `char` (one byte). `"A"` is a string literal (an array of two chars: `'A'` and a terminating `'\0'`). See [Arrays and Strings](<Arrays and Strings.md>).

### Booleans

`bool` holds `true` or `false`. It converts to `1` / `0`, and any number converts to `bool` (zero is `false`, everything else is `true`):

```cpp
bool flag = true;
std::cout << flag << "\n";                       // 1
std::cout << std::boolalpha << flag << "\n";     // true

bool from_int = 42;                              // true
int from_bool = true + true;                     // 2
```

## Literals

A **literal** is a value written directly in code. Its form decides its type:

```cpp
42          // int
42u         // unsigned int
42L         // long
42LL        // long long
42ull       // unsigned long long

0x2A        // 42 in hexadecimal
0b101010    // 42 in binary (C++14)
052         // 42 in OCTAL: a leading 0 means base 8, a classic trap
1'000'000   // digit separators (C++14), like Python's 1_000_000

3.14        // double
3.14f       // float
1e9         // double: 1,000,000,000.0

'a'         // char
"abc"       // string literal: const char[4], NOT std::string
true        // bool
nullptr     // the null pointer (see Pointers)
```

## `auto`: Type Deduction

`auto` asks the compiler to deduce the type from the initializer. The type is still fixed at compile time, so this is not Python-style dynamic typing:

```cpp
auto a = 5;       // int
auto b = 5.0;     // double
auto c = 5.0f;    // float
auto d = 5u;      // unsigned int
auto e = 5LL;     // long long
auto s = "hi";    // const char*  <- NOT std::string!
```

The last one surprises people coming from Python. A string literal is a C-style character array, not a `std::string`. If you want a `std::string`:

```cpp
#include <string>
using namespace std::string_literals;

auto t = std::string("hi");   // std::string
auto u = "hi"s;               // std::string, via the s suffix
```

`auto` is most useful when the type is long or obvious from the right-hand side:

```cpp
auto it = scores.begin();                        // instead of std::vector<int>::iterator it = ...
auto player = std::make_unique<Player>("Ava");   // instead of std::unique_ptr<Player> player = ...
```

> `auto` drops references and top-level `const`, so `auto x = some_reference;` makes a **copy**. Use `auto&` or `const auto&` when you want a reference. See [References](<../Memory/References.md>).

## Constants

`const` makes a variable read-only after initialization. `constexpr` additionally requires the value to be known at **compile time**:

```cpp
const int max_health = 100;        // can't be changed later
max_health = 50;                   // ERROR: cannot assign to variable with const-qualified type

constexpr int grid_size = 16;      // computed at compile time
int cells[grid_size * grid_size];  // OK: array sizes must be compile-time constants
```

Python's `ALL_CAPS = 100` is only a naming convention. C++ `const` is enforced by the compiler. Full details in [Const Correctness](<../Memory/Const Correctness.md>).

## Where Variables Live

Where a variable is declared decides how long it exists and who can see it:

```cpp
int global_count = 0;          // global: exists for the whole program

void f() {
    int local = 5;             // local: created when f() reaches this line, destroyed at the closing }
    static int calls = 0;      // static local: created once, keeps its value between calls
    calls++;
}
```

This is covered in [Scope and Namespaces](<Scope and Namespaces.md>) and, at the memory level, in [Stack and Heap](<../Memory/Stack and Heap.md>).
