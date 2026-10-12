# Type Conversion in C++

Type conversion means producing a value of one type from a value of another type.

Python rarely converts types for you: `2 + "5"` is a `TypeError`, you explicitly call constructor functions like `int()` and `str()` for type conversion. (see [Type Conversion](<../../Python/Fundamentals/Type Conversion.md>)).

C++ is the opposite in two ways:

1. Between **numeric types** (`char`, `int`, `double`, `bool`, ...), it converts **implicitly and silently**, all the time, sometimes losing data.
2. Between **numbers and text**, there is no implicit conversion at all, and `int("42")` is not even valid syntax. You call library functions.

```cpp
int a = 2 + 3.9;            // implicit: 2 + 3.9 = 5.9 (double), then 5.9 -> 5 (int). Compiles with a warning
std::string s = 5;          // ERROR: no viable conversion from 'int' to 'std::string'
int x = "42";               // ERROR: cannot initialize a variable of type 'int' with ... 'const char[3]'
```

---

## 1. Implicit Conversions

The compiler inserts these automatically whenever a value of one type is used where another is expected.

### Integral Promotion

Arithmetic is never done on anything smaller than `int`. `bool`, `char`, and `short` operands are **promoted** to `int` first:

```cpp
char a = 'a';   // 97
char b = 'b';   // 98
auto sum = a + b;           // int 195, not char
std::cout << sizeof(sum);   // 4

short s1 = 10, s2 = 20;
auto ss = s1 + s2;          // int, 4 bytes
```

This is why `c - 'a'` (see [Variables and Types](<Variables and Types.md#characters>)) gives you an `int` index directly.

### Usual Arithmetic Conversions

When a binary operator gets two **different** arithmetic types, the "smaller" operand is converted to the "larger" type, roughly in this order:

```text
int  ->  unsigned int  ->  long  ->  unsigned long  ->  long long  ->  ...  ->  float  ->  double  ->  long double
```

```cpp
7 / 2          // int / int       = 3     (integer division: see Operators)
7 / 2.0        // int / double    -> 7.0 / 2.0 = 3.5
7.0 / 2        // double / int    -> 7.0 / 2.0 = 3.5
```

The most common bug is converting **too late**:

```cpp
int total = 7, count = 2;

double avg1 = total / count;                        // 3.0  <- integer division happens FIRST, then 3 -> 3.0
double avg2 = static_cast<double>(total) / count;   // 3.5  <- convert one operand BEFORE dividing
```

### Signed / Unsigned Mixing

If one operand is `int` and the other is `unsigned int`, the `int` is converted to `unsigned`. A negative number becomes a huge positive one:

```cpp
int i = -1;
unsigned u = 1;

std::cout << (i < u);   // 0 (false)!  -1 becomes 4294967295, which is not < 1
std::cout << i + u;     // 0           4294967295 + 1 wraps around to 0
```

This matters constantly, because `.size()` on every standard container returns an **unsigned** `size_t`:

```cpp
std::vector<int> v;               // empty, so v.size() == 0

for (int i = 0; i <= v.size() - 1; ++i) {
    // v.size() - 1 is 0 - 1 in unsigned arithmetic = 18446744073709551615
    // so this loop runs "forever" and indexes out of bounds
}

for (int i = 0; i < static_cast<int>(v.size()); ++i) {   // safe: compare as signed
}

for (int x : v) {                                         // safest: no index at all (see Control Flow)
}
```

`-Wall -Wextra` warns about signed/unsigned comparisons (`-Wsign-compare`). Take those warnings seriously.

### Narrowing Conversions

A **narrowing** conversion goes to a type that may not be able to hold the value:

```cpp
int a = 3.99;       // 3: the fraction is truncated toward zero (not rounded)
int b = -3.99;      // -3

int x = 300;
char c = x;         // 44: 300 doesn't fit in 1 byte, so it keeps the low 8 bits (300 - 256)

double huge = 1e20;
int d = huge;       // UNDEFINED BEHAVIOR: the value doesn't fit in an int at all
```

Brace initialization turns narrowing into a compile error, which is one good reason to prefer it (see [Variables and Types](<Variables and Types.md#three-ways-to-initialize>)):

```cpp
int e{3.99};        // ERROR: type 'double' cannot be narrowed to 'int'
```

### Boolean Conversion

Any number or pointer converts to `bool`. Zero (or `nullptr`) is `false`, and anything else is `true`. This is C++'s much smaller version of Python's truthiness rules (see [Operators](<../../Python/Fundamentals/Operators.md#truthy-and-falsy-values>)):

```cpp
int lives = 3;
if (lives) { }           // true: nonzero

Player* target = nullptr;
if (!target) { }         // true: null pointer

std::string name = "";
if (name) { }            // ERROR: std::string does not convert to bool
if (name.empty()) { }    // true: empty string

std::vector<int> v;
if (v) { }               // ERROR: same. Use if (!v.empty())
```

Python's "empty containers are falsy" does **not** carry over. Containers and strings have to be checked with `.empty()`, so Python's `if name:` becomes `if (!name.empty())`.

## 2. Explicit Conversions (Casts)

When you want a conversion, or want to make an implicit one visible, you **cast**.

### `static_cast`: the Normal Choice

```cpp
double avg = static_cast<double>(total) / count;   // int -> double
int index = static_cast<int>(v.size());             // size_t -> int
char letter = static_cast<char>(65);                // 'A'
int code = static_cast<int>('A');                   // 65
int truncated = static_cast<int>(3.99);             // 3
```

`static_cast` performs conversions the compiler can check at compile time: between numeric types, between enums and integers, and up/down a class hierarchy (unchecked, see [OOP Four Pillars](<../OOP/OOP Four Pillars.md>)). It refuses nonsense like casting an `int*` to a `double*`.

### C-Style and Functional Casts

C++ inherited C's cast syntax, and it also allows a function-call style:

```cpp
int n = 65;
char a = (char)n;     // C-style cast
char b = char(n);     // functional cast: same meaning for simple types
```

Both are shorter, but a C-style cast will silently try `static_cast`, then `const_cast`, then `reinterpret_cast`, until something compiles. That means a typo can turn a safe conversion into a dangerous one with no warning. Prefer the named casts in real code. They're also easy to search for.

### The Other Named Casts

| Cast | What it does | When you'll see it |
|---|---|---|
| `static_cast<T>(x)` | Compile-time-checked conversion | Almost every deliberate conversion |
| `dynamic_cast<T*>(p)` | Runtime-checked downcast in a class hierarchy; returns `nullptr` if the object isn't really a `T` | Polymorphism, see [Virtual Functions and Vtables](<../OOP/Virtual Functions and Vtables.md>) |
| `const_cast<T>(x)` | Adds or removes `const` | Calling old APIs that forgot `const`. Writing through it to a truly const object is UB. See [Const Correctness](<../Memory/Const Correctness.md>) |
| `reinterpret_cast<T>(x)` | Reinterprets the raw bits as another type (e.g. pointer to integer) | Low-level code: serialization, hardware, hashing pointers |

> Unreal Engine has its own `Cast<T>(Object)` for `UObject` types. It does the job of `dynamic_cast`, using Unreal's reflection system instead of C++ RTTI, and returns `nullptr` on failure.

### Rounding Instead of Truncating

Casting a floating-point value to an integer always truncates toward zero. Use `<cmath>` for other rounding:

```cpp
#include <cmath>

static_cast<int>(3.99)    // 3     truncate toward zero (like Python's int(3.99))
static_cast<int>(-3.99)   // -3
std::round(3.5)           // 4.0   round half away from zero (Python's round(2.5) gives 2: banker's rounding)
std::lround(2.5)          // 3     round, returned as long
std::floor(-3.5)          // -4.0  toward negative infinity (like Python's math.floor)
std::ceil(3.1)            // 4.0
```

## 3. Text Conversions

Since there are no implicit conversions between numbers and strings, the standard library provides functions.

### Number to String: `std::to_string`

```cpp
#include <string>

std::string a = std::to_string(42);      // "42"
std::string b = std::to_string(3.14);    // "3.140000" (always 6 decimal places)

std::string msg = "Score: " + std::to_string(42);   // "Score: 42"
```

For control over the format, use `std::format` (C++20), which takes Python's format-spec mini-language (see [Program Structure](<Program Structure.md#stdformat-and-stdprint>)):

```cpp
#include <format>

std::string c = std::format("{:.2f}", 3.14159);   // "3.14"
std::string d = std::format("{:08b}", 42);        // "00101010"
std::string e = std::format("{:x}", 255);         // "ff"
```

> **Trap:** `std::string + int` doesn't compile, but `std::string += int` **does**, because `+=` has an overload that takes a single `char`, and the `int` silently converts to it:
>
> ```cpp
> std::string s = "score: ";
> s += 65;                     // "score: A"   <- appended the character with code 65!
> s += std::to_string(65);     // "score: A65"
> ```

### String to Number: `std::stoi` and Friends

| Python | C++ | Result type |
|---|---|---|
| `int("42")` | `std::stoi("42")` | `int` |
| `int("9000000000")` | `std::stoll("9000000000")` | `long long` |
| `float("3.14")` | `std::stod("3.14")` | `double` |
| `int("ff", 16)` | `std::stoi("ff", nullptr, 16)` | `int` |
| `int("101", 2)` | `std::stoi("101", nullptr, 2)` | `int` |

```cpp
std::stoi("42")         // 42
std::stoi("  42abc")    // 42: skips leading whitespace and stops at the first non-digit (Python would raise)
std::stod("3.14")       // 3.14
```

When the text can't be converted, they **throw exceptions**, much like Python's `ValueError`:

```cpp
try {
    std::stoi("abc");
} catch (const std::invalid_argument& e) {
    std::cout << e.what();   // stoi: no conversion
}

try {
    std::stoi("99999999999");
} catch (const std::out_of_range& e) {
    std::cout << e.what();   // stoi: out of range  (doesn't fit in an int)
}
```

Exceptions are covered in [Error Handling](<Error Handling.md>).

### `std::from_chars`: Fast and No Exceptions

`<charconv>` (C++17) provides `std::from_chars`, which doesn't throw and doesn't allocate. It reports what happened through its return value. It's the best choice for parsing lots of input, or in code built without exceptions (like Unreal):

```cpp
#include <charconv>

std::string text = "123xyz";
int value = 0;

auto [ptr, ec] = std::from_chars(text.data(), text.data() + text.size(), value);

// value = 123
// ec == std::errc()  -> success (std::errc::invalid_argument if nothing could be parsed)
// ptr points at "xyz": where parsing stopped
```

### Single Digits and Characters

Digit characters are consecutive in ASCII, so converting one digit doesn't need a library call:

```cpp
int d = '7' - '0';                       // 7
char c = static_cast<char>('0' + 7);     // '7'
```

This replaces Python's `int('7')` and `str(7)` for single digits, and shows up in nearly every string-parsing problem.

## 4. Conversions for Your Own Types

Classes can define their own conversions:

- A constructor that takes one argument acts as an implicit conversion **into** the class, unless it's marked `explicit`.
- A conversion operator such as `operator bool()` defines a conversion **out of** the class.

```cpp
class Meters {
public:
    explicit Meters(double value) : value_(value) {}   // explicit: no silent double -> Meters
    explicit operator double() const { return value_; }

private:
    double value_;
};

Meters m(5.0);                        // OK
Meters n = 5.0;                       // ERROR because the constructor is explicit
double raw = static_cast<double>(m);  // OK, explicit conversion out
```

Marking single-argument constructors `explicit` by default prevents surprise conversions. More in [Classes and Objects](<../OOP/Classes and Objects.md>) and [Operator Overloading](<../OOP/Operator Overloading.md>).

## Summary

| Situation | What C++ does |
|---|---|
| `char`/`short`/`bool` in arithmetic | Promoted to `int` |
| `int` op `double` | `int` converted to `double` |
| `int` op `unsigned` | `int` converted to `unsigned`: negative values become huge |
| `double` -> `int` | Truncates toward zero; UB if the value doesn't fit |
| Number / pointer -> `bool` | Zero / null is `false`, everything else `true` |
| `std::string` / container -> `bool` | Not allowed: use `.empty()` |
| Number <-> string | Never implicit: `std::to_string`, `std::format`, `std::stoi`, `std::from_chars` |
| Deliberate conversion | `static_cast<T>(x)` |
