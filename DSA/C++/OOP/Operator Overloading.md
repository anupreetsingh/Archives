# Operator Overloading in C++

- Operator overloading lets your own classes work with normal C++ syntax: `a + b`, `a == b`, `std::cout << a`, `a[i]`, `a(x)`.
- It's the C++ counterpart of Python's dunder methods (see [OOP-dunder methods](<../../Python/OOP/OOP-dunder methods.md>)). Instead of `__add__`, you write a function named `operator+`.

**KEY IDEA:** `a + b` on a class type is just a function call the compiler writes for you. It becomes either `a.operator+(b)` (a member function) or `operator+(a, b)` (a free function).

---

## Python Dunder → C++ Operator

| Python | C++ | Example trigger |
|---|---|---|
| `__init__` | Constructor `Book(...)` | `Book b("C++", 250);` |
| `__del__` | Destructor `~Book()` | End of scope (see [Object Lifetime and RAII](<Object Lifetime and RAII.md>)) |
| `__str__` / `__repr__` | `operator<<(std::ostream&, const Book&)` | `std::cout << b` |
| `__len__` | A `size()` member function (by convention, not an operator) | `b.size()` |
| `__eq__` | `operator==` | `a == b` |
| `__ne__` | `operator!=` (generated from `==` in C++20) | `a != b` |
| `__lt__` `__le__` `__gt__` `__ge__` | `operator<` ... or all of them at once with `operator<=>` | `a < b` |
| `__add__` / `__iadd__` | `operator+` / `operator+=` | `a + b`, `a += b` |
| `__sub__` `__mul__` `__truediv__` | `operator-` `operator*` `operator/` | |
| `__neg__` | Unary `operator-()` | `-a` |
| `__getitem__` / `__setitem__` | `operator[]` returning a reference (one function does both) | `a[i]`, `a[i] = x` |
| `__contains__` | A `contains()` member function (by convention) | `a.contains(x)` |
| `__call__` | `operator()` | `a(x)` |
| `__bool__` | `explicit operator bool()` | `if (a)` |
| `__iter__` / `__next__` | `begin()` / `end()` returning iterators | `for (auto x : a)` (see [Iterators and Algorithms](<../Standard Library/Iterators and Algorithms.md>)) |
| `__hash__` | A specialization of `std::hash<T>` | Using `T` as an `unordered_map` key |
| `__enter__` / `__exit__` | Constructor / destructor (RAII) | Declaring a local object |
| `__copy__` / `__deepcopy__` | Copy constructor / copy assignment | `Book b2 = b1;` (see [Copy and Move Semantics](<Copy and Move Semantics.md>)) |

One difference to keep in mind: Python gives every class a default `__eq__` (identity) and `__str__`. C++ gives you **nothing** by default. Comparing or printing a class without the operator is a compile error:

```cpp
struct P { int x; };
P a{1}, b{1};

a == b;            // ERROR: invalid operands to binary expression ('P' and 'P')
std::cout << a;    // ERROR: invalid operands to binary expression ('ostream' and 'P')
```

## Basic Example

The `Book` class from the Python note:

```cpp
#include <iostream>
#include <string>

class Book {
public:
    Book(std::string title, int pages) : title_(title), pages_(pages) {}   // __init__

    int size() const { return pages_; }                                    // __len__ (by convention)

    bool operator==(const Book& other) const {                             // __eq__
        return title_ == other.title_ && pages_ == other.pages_;
    }

    friend std::ostream& operator<<(std::ostream& os, const Book& b) {     // __str__
        return os << b.title_ << " has " << b.pages_ << " pages";
    }

private:
    std::string title_;
    int pages_;
};

Book book1("C++ Basics", 250);
Book book2("C++ Basics", 250);

std::cout << book1 << "\n";          // C++ Basics has 250 pages
std::cout << book1.size() << "\n";   // 250
std::cout << (book1 == book2);       // 1
std::cout << (book1 != book2);       // 0: != is generated from == automatically (C++20)
```

## Member vs Non-Member Operators

An operator can be written two ways:

```cpp
struct Vec2 {
    float x, y;

    Vec2 operator-() const { return {-x, -y}; }                 // MEMBER: the left operand is *this
};

Vec2 operator*(Vec2 v, float s) { return {v.x * s, v.y * s}; }  // NON-MEMBER: both operands are parameters
```

| Write it as a | When |
|---|---|
| **Member** | It modifies the left operand (`+=`, `-=`, `=`, `++`), or it must be a member (`=`, `[]`, `()`, `->`) |
| **Non-member** (often a `friend`) | Symmetric binary operators (`+`, `*`, `==`), and anything where your class is on the **right** (`<<`, `2.0f * v`) |

A member operator only works when your class is on the **left**. `v * 2.0f` can be a member, but `2.0f * v` can't, because `float` has no member to add. That's the same reason Python has `__rmul__`.

### `friend` Functions

A `friend` declared inside the class is a non-member function that can still read private members. That's what `operator<<` above uses: its left operand is a `std::ostream`, so it can't be a member of `Book`, but it needs `title_` and `pages_`.

## 1. Printing: `operator<<`

This is C++'s `__str__`. It takes the stream by reference, writes to it, and **returns** the stream so calls can chain:

```cpp
friend std::ostream& operator<<(std::ostream& os, const Book& b) {
    return os << b.title_ << " has " << b.pages_ << " pages";
}

std::cout << book1 << " / " << book2 << "\n";
// works because each << returns the stream: ((std::cout << book1) << " / ") << book2 ...
```

It works with any output stream: `std::cout`, `std::cerr`, files (`std::ofstream`), and string streams.

## 2. Comparison: `operator==` and `operator<=>`

### Defaulted Comparisons (C++20)

Most of the time you want "compare every member in order". C++20 can generate that for you:

```cpp
struct Vec2 {
    float x = 0, y = 0;
    bool operator==(const Vec2&) const = default;   // compares x, then y
};

struct Version {
    int major, minor, patch;
    auto operator<=>(const Version&) const = default;   // generates <, <=, >, >=, and == / !=
};

Version v1{1, 2, 3};
Version v2{1, 10, 0};

v1 < v2                      // true: compares major, then minor (2 < 10), lexicographically like Python tuples
v1 == Version{1, 2, 3}       // true
```

`<=>` is the **three-way comparison** ("spaceship") operator. `a <=> b` tells you whether `a` is less than, equal to, or greater than `b` in one call, and the compiler rewrites `a < b` into `(a <=> b) < 0`. The effect is the same as Python's `@functools.total_ordering`, from a single declaration.

Defining `<` (or `<=>`) is what lets a type be sorted and used as a key in ordered containers:

```cpp
std::vector<Version> vs = {{2, 0, 0}, {1, 2, 3}, {1, 10, 0}};
std::sort(vs.begin(), vs.end());          // uses operator<: {1,2,3}, {1,10,0}, {2,0,0}
std::set<Version> unique_versions(vs.begin(), vs.end());   // std::set needs operator<
```

### Custom Comparisons

Write the body yourself when equality means something other than "all members are equal":

```cpp
struct Student {
    std::string name;
    int grade;

    bool operator==(const Student& other) const { return grade == other.grade; }   // compare by grade only
    auto operator<=>(const Student& other) const { return grade <=> other.grade; }
};
```

> To use a type as a key in `std::unordered_map` / `std::unordered_set` (hash tables), it needs `operator==` **and** a hash function, which is the `__eq__` + `__hash__` pair from [Hash Maps Structure](<../../Python/Fundamentals/Hash Maps Structure.md#hashing-a-key>). Without a hash, you get errors like `call to implicitly-deleted default constructor of 'std::hash<P>'`. See [STL Containers](<../Standard Library/STL Containers.md>).

## 3. Arithmetic: Implement `+=`, Then `+` in Terms of It

The idiomatic pattern is to write the compound assignment as a member that modifies `*this`, then write the binary operator as a non-member that copies and reuses it:

```cpp
struct Vec2 {
    float x = 0, y = 0;

    Vec2& operator+=(const Vec2& o) {      // modifies the left operand and returns it by reference
        x += o.x;
        y += o.y;
        return *this;
    }

    friend Vec2 operator+(Vec2 a, const Vec2& b) {   // a is taken BY VALUE: it's already a copy
        a += b;                                      // reuse +=
        return a;                                    // return a new Vec2
    }

    friend Vec2 operator*(Vec2 v, float s) { return {v.x * s, v.y * s}; }
    friend Vec2 operator*(float s, Vec2 v) { return v * s; }   // the "__rmul__" version

    Vec2 operator-() const { return {-x, -y}; }      // unary minus
};

Vec2 a{1, 2}, c{3, 4};
Vec2 s = a + c;      // {4, 6}
Vec2 m = 2.0f * a;   // {2, 4}
Vec2 n = -a;         // {-1, -2}
a += c;              // a = {4, 6}
```

Operators should behave like their built-in versions: `+` shouldn't modify its operands, `+=` should return `*this`, and `==` should be symmetric.

> Unreal's `FVector`, `FRotator`, and `FQuat` overload these operators, which is why vector math like `Location + Direction * Speed * DeltaTime` reads naturally in gameplay code.

## 4. Indexing: `operator[]`

Python needs two dunders, `__getitem__` and `__setitem__`. C++ needs one, because `operator[]` returns a **reference**, which can be both read and assigned through (see [References](<../Memory/References.md#4-returning-references>)). Provide a `const` overload for read-only access:

```cpp
class Grid {
public:
    Grid(int w, int h) : w_(w), cells_(w * h, 0) {}

    int& operator[](std::size_t i) { return cells_[i]; }               // read and write
    const int& operator[](std::size_t i) const { return cells_[i]; }   // read-only, for const Grids

    int& operator()(int r, int c) { return cells_[r * w_ + c]; }       // 2D access with ()

private:
    int w_;
    std::vector<int> cells_;
};

Grid g(3, 3);
g[0] = 1;          // write through the returned reference
g(1, 2) = 7;       // row 1, column 2 -> flat index 1 * 3 + 2 = 5
g[5];              // 7
```

(C++23 also allows a multi-argument `operator[]`, as in `g[1, 2]`.)

## 5. Calling: `operator()` (Function Objects)

A class with `operator()` can be called like a function, like Python's `__call__`. Such objects are called **function objects** or **functors**, and unlike plain functions they can carry state:

```cpp
struct MultiplyBy {
    int factor;
    int operator()(int x) const { return x * factor; }
};

MultiplyBy triple{3};
triple(5);    // 15

std::vector<int> nums = {1, 2, 3};
std::transform(nums.begin(), nums.end(), nums.begin(), triple);   // {3, 6, 9}
```

Lambdas are a shorthand that makes the compiler write one of these classes for you (see [Lambdas](<../Fundamentals/Lambdas.md>)).

## 6. Truthiness: `explicit operator bool`

Like Python's `__bool__`, this lets an object be used in a condition. Mark it `explicit`, so the object converts to `bool` only in conditions, and not silently in arithmetic like `grid + 1`:

```cpp
explicit operator bool() const { return !cells_.empty(); }

if (g) { }          // OK: conditions allow explicit bool conversion
int x = g + 1;      // ERROR, thanks to explicit
```

Smart pointers, streams, and `std::optional` all use this, which is why `if (ptr)`, `if (file)`, and `if (maybe_value)` work.

## 7. Increment and Decrement

Prefix and postfix forms are distinguished by a dummy `int` parameter:

```cpp
struct Counter {
    int n = 0;
    Counter& operator++() { ++n; return *this; }        // prefix ++c: returns the updated object
    Counter operator++(int) {                            // postfix c++: the int is just a marker
        Counter old = *this;                             // keep a copy of the old value...
        ++n;
        return old;                                      // ...and return it
    }
};
```

The copy in the postfix version is why `++it` is preferred for iterators (see [Operators](<../Fundamentals/Operators.md#2-increment--decrement-operators>)).

## Rules and Restrictions

- You can't invent new operators (no `**`), or change an operator's precedence or the number of operands it takes.
- At least one operand must be a class or enum type. You can't redefine `int + int`.
- These can't be overloaded: `::`, `.`, `.*`, `?:`, `sizeof`, `typeid`.
- Overloading `&&`, `||`, or `,` loses their short-circuit and sequencing behavior, so don't.
- Only overload an operator when its meaning is obvious. `Vec2 + Vec2` is obvious. `Player + Weapon` isn't, so give it a named function like `player.equip(weapon)`.

## Why They Matter

- They make your types feel like built-in types: vector math, string concatenation, and smart pointers all rely on them.
- They let your types work with the standard library: `<` (or `<=>`) for sorting and `std::map`/`std::set`, `==` + `std::hash` for `std::unordered_map`, `<<` for printing, `()` for algorithms.
- Unlike Python's dunders, there's no runtime lookup: an overloaded operator is an ordinary function call that's resolved at compile time and usually inlined.
