# Templates in C++

A **template** is a blueprint the compiler uses to generate functions or classes for whatever types you use it with. You write the code once with a placeholder type `T`, and the compiler produces a separate, fully typed version for each `T` that the program actually uses.

→ Python gets "works with any type" for free through **duck typing**: `def max_of(a, b): return a if a > b else b` works for anything that supports `>`, checked at runtime. Templates give C++ the same flexibility, checked at **compile time**, with no runtime cost.

Every standard container is a template: `std::vector<int>`, `std::map<std::string, int>`, `std::unique_ptr<Enemy>`. Unreal's `TArray<T>`, `TMap<K, V>`, and `TSubclassOf<T>` are templates too.

---

## 1. Function Templates

```cpp
template <typename T>          // "T is a placeholder for some type"
T max_of(T a, T b) {
    return a > b ? a : b;
}

max_of(3, 7);                                          // 7         T = int
max_of(2.5, 1.5);                                      // 2.5       T = double
max_of(std::string("apple"), std::string("banana"));   // "banana"  T = std::string
```

### Template Argument Deduction

The compiler works out `T` from the argument types. Every argument has to agree:

```cpp
max_of(3, 7.5);           // ERROR: no matching function for call to 'max_of'
                          // note: candidate template ignored: deduced conflicting types for parameter 'T' ('int' vs. 'double')

max_of<double>(3, 7.5);   // 7.5: state T explicitly, and 3 converts to 3.0
```

`typename` and `class` mean the same thing in a template parameter list: `template <class T>` is equivalent.

### Abbreviated Templates: `auto` Parameters (C++20)

`auto` in a parameter list makes a function template without the `template` line. Each `auto` parameter gets its own independent type:

```cpp
auto add(auto a, auto b) {
    return a + b;
}

add(1, 2.5);                    // 3.5: a is int, b is double
add(std::string("a"), "b");     // "ab"
```

This is the closest C++ comes to an untyped Python function. The difference is that the compiler still checks every call.

## 2. How Templates Work: Instantiation

A template isn't code yet, only a recipe. When the compiler sees `max_of(3, 7)`, it **instantiates** the template: it substitutes `int` for `T` and compiles the resulting function, as if you'd written it by hand.

```text
template <typename T> T max_of(T a, T b)
          |
          |  used with int, double, std::string
          v
int         max_of(int a, int b)                 -> compiled into machine code
double      max_of(double a, double b)           -> compiled into machine code
std::string max_of(std::string a, std::string b) -> compiled into machine code
```

`nm` on an object file shows a separate symbol for each instantiation:

```text
0000000000000000 T int twice<int>(int)
000000000000001c T double twice<double>(double)
```

Consequences:

- **No runtime cost.** Each instantiation is ordinary, fully typed code that can be optimized and inlined. Templates are C++'s main form of **compile-time polymorphism** (see [OOP Four Pillars](<../OOP/OOP Four Pillars.md#4-polymorphism>)), with none of a virtual call's overhead.
- **Type errors appear at instantiation.** If `T` doesn't support an operation the template uses, the error is reported when that instantiation is compiled:

    ```cpp
    struct P { int x; };
    max_of(P{1}, P{2});
    // error: invalid operands to binary expression ('P' and 'P')
    // note: in instantiation of function template specialization 'max_of<P>' requested here
    ```

    Python would raise a `TypeError` only once that line actually ran. C++ rejects the program before it runs.

- **Code bloat and build time.** Each instantiation is separate machine code, and every translation unit that uses a template compiles it again. Heavy template use is a major reason C++ builds are slow.

## 3. Why Templates Live in Headers

To instantiate `max_of<int>`, the compiler needs the template's full **definition**, not just a declaration. But each `.cpp` is compiled separately (see [Compilation and Linking](<Compilation and Linking.md>)). If the definition lives in some other `.cpp`, no translation unit has both the definition and the request for `<int>`:

```cpp
// twice.h
template <typename T> T twice(T x);                       // declaration only

// twice.cpp
#include "twice.h"
template <typename T> T twice(T x) { return x + x; }      // definition. But nobody here asks for twice<int>,
                                                          // so NO code is generated (nm twice.o shows nothing)
// main.cpp
#include "twice.h"
int main() { return twice(21); }                          // needs twice<int>, but only has the declaration
```

```text
Undefined symbols for architecture arm64:
  "int twice<int>(int)", referenced from:
      _main in main.o
```

The fix: **put the whole template definition in the header**. Templates are exempt from the One Definition Rule's "exactly once" requirement, so identical definitions in many translation units are fine (see [Headers and Source Files](<Headers and Source Files.md#the-one-definition-rule-odr>)).

This is why the standard library is mostly headers, and why `<vector>` or `<algorithm>` add so many lines to every translation unit that includes them.

## 4. Class Templates

```cpp
template <typename T>
class Box {
public:
    explicit Box(T value) : value_(value) {}
    const T& get() const { return value_; }
private:
    T value_;
};

Box<int> bi(42);              // a Box that holds an int
Box<std::string> bs("hi");    // a completely separate class that holds a std::string
Box b2(3.14);                 // C++17 class template argument deduction (CTAD): Box<double>
```

`Box<int>` and `Box<std::string>` are **different, unrelated types**. A `Box<int>` can't be assigned to a `Box<double>`, and they share no code at runtime.

[Kinds of Classes](<../OOP/Kinds of Classes.md>) compares class templates with normal classes, function templates, and the other constructs that use the word `class`.

> Milestone 0.2c ends by turning a hand-written dynamic array into a class template. That's exactly how `std::vector<T>` and Unreal's `TArray<T>` are built.

### Member Functions Defined Outside the Class

They need the template header again, and the class name with its parameter:

```cpp
template <typename T>
const T& Box<T>::get() const {
    return value_;
}
```

## 5. Non-Type Template Parameters

A template parameter can also be a **value** known at compile time, such as an integer:

```cpp
template <typename T, std::size_t N>
struct FixedStack {
    std::array<T, N> data{};      // N elements, stored inline: no heap allocation
    std::size_t size = 0;
    void push(const T& v) { data[size++] = v; }
};

FixedStack<int, 8> st;            // sizeof(st) = 8 ints + size_t = 40 bytes
```

`std::array<T, N>` (see [Arrays and Strings](<Arrays and Strings.md#2-stdarray-the-modern-fixed-size-array>)) is exactly this: the size is part of the type.

## 6. Concepts: Constraining `T` (C++20)

An unconstrained `T` accepts anything until something fails deep inside the template. A **concept** states the requirements up front, and makes error messages point at the call:

```cpp
#include <concepts>

template <std::integral T>        // T must be an integer type
T gcd(T a, T b) {
    while (b != 0) {
        T t = a % b;
        a = b;
        b = t;
    }
    return a;
}

gcd(48, 18);      // 6
gcd(1.5, 2.5);    // ERROR: no matching function for call to 'gcd'
                  // note: candidate template ignored: constraints not satisfied [with T = double]
                  // note: because 'double' does not satisfy 'integral'
```

Useful standard concepts: `std::integral`, `std::floating_point`, `std::same_as<U>`, `std::convertible_to<U>`, `std::totally_ordered`, `std::invocable<Args...>`.

You can define your own with a `requires` expression, which lists the operations `T` must support. It's the compile-time version of a Python `Protocol`:

```cpp
template <typename T>
concept Printable = requires(std::ostream& os, const T& v) {
    os << v;                      // "T must be printable with <<"
};

template <Printable T>
void show(const T& v) { std::cout << v << "\n"; }

show(42);        // OK
show("text");    // OK
```

## 7. Specialization

You can give a template a different implementation for one specific type:

```cpp
template <typename T>
void describe(const T&) { std::cout << "generic\n"; }

template <>                                   // full specialization for bool
void describe<bool>(const bool&) { std::cout << "bool specialization\n"; }

describe(5);      // generic
describe(true);   // bool specialization
```

The standard library uses this for `std::hash<T>`: to put your own type in an `std::unordered_map`, you specialize `std::hash` for it (see [STL Containers](<../Standard Library/STL Containers.md>)). It's also why `std::vector<bool>` behaves differently from every other `std::vector`: it's a specialization that packs bools into bits.

## 8. Compile-Time Checks: `static_assert`

`static_assert` checks a condition at compile time and stops the build with your message if it's false. It's commonly used inside templates:

```cpp
template <typename T>
void serialize(const T& value) {
    static_assert(std::is_trivially_copyable_v<T>, "serialize() needs a type that can be copied byte by byte");
    // ...
}

static_assert(sizeof(int) == 4, "this code assumes 4-byte ints");
```

## 9. Templates vs Virtual Functions

Both let one piece of code work with many types:

| | Templates | Virtual functions |
|---|---|---|
| Type decided | At compile time, per call site | At runtime, per object |
| Dispatch | Static | Dynamic |
| Call cost | Direct call, usually inlined | Indirect call through the vtable |
| Mixing types in one container | No: `std::vector<T>` holds one `T` | Yes: `std::vector<std::unique_ptr<Base>>` |
| Code size | One copy per type used | One copy |
| Errors | At compile time, at instantiation | At compile time, against the base interface |
| Python analogue | Duck typing, checked early | Subclass overriding a method |

Use templates for generic algorithms and containers where the type is known statically. Use virtual functions when objects of different types must be handled together at runtime, like a level full of different enemy types. See [Virtual Functions and Vtables](<../OOP/Virtual Functions and Vtables.md>).

## Summary

```text
template <typename T>   -> write once, the compiler generates a typed copy per T used
Deduction               -> T inferred from the arguments; they must agree, or specify <T> explicitly
auto parameters         -> shorthand for function templates (C++20)
Definition in headers   -> the compiler needs the full body wherever it instantiates
Concepts                -> state what T must support; clearer errors
Specialization          -> a custom implementation for one specific type
Cost                    -> zero at runtime; paid in compile time and binary size
```
