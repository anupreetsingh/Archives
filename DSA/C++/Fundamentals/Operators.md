# Operators in C++

Operators are special symbols that perform operations on **operands** and produce a result, just like in Python (see [Operators](<../../Python/Fundamentals/Operators.md>)).

```cpp
int x = 5 + 3;
// '+' is the operator
// 5 and 3 are operands
// The result, 8, is used to initialize x
```

Most of the symbols are the same as Python's. The differences that matter are:

- The **types** of the operands decide what the operator does (`7 / 2` is `3`, not `3.5`).
- C++ has operators Python doesn't (`++`, `--`, `?:`, `::`, `->`, `*` and `&` for pointers).
- Python has operators C++ doesn't (`**`, `//`, `is`, `in`, `not`/`and`/`or` as words, `:=`, `@`).
- Logical operators always return `bool`, and comparisons can't be chained.

---

## 1. Arithmetic Operators

```cpp
int a = 10;
int b = 3;

std::cout << a + b;   // Addition ➕ : 13
std::cout << a - b;   // Subtraction ➖ : 7
std::cout << a * b;   // Multiplication ✖️ : 30
std::cout << a / b;   // Division ➗ : 3   <- int / int is INTEGER division
std::cout << a % b;   // Modulus 🧮 : 1   the remainder (integers only)
```

### `/` Depends on the Operand Types

C++ has no separate `//` operator. The **same** `/` does integer division when both operands are integers, and floating-point division when at least one is floating point:

```cpp
7 / 2       // 3     int / int
7 / 2.0     // 3.5   int / double: 7 is converted to 7.0 first (see Type Conversion)
7.0 / 2     // 3.5
```

| Python | C++ |
|---|---|
| `7 / 2` → `3.5` | `7 / 2.0` or `static_cast<double>(7) / 2` → `3.5` |
| `7 // 2` → `3` | `7 / 2` → `3` (for non-negative values) |

### Negative Numbers: Truncation, Not Floor

Python's `//` rounds **down** toward negative infinity. C++'s integer `/` **truncates toward zero**. The two agree for positive numbers and differ for negatives:

```cpp
-10 / 3     // -3   (C++: -3.33 truncated toward zero)
-10 % 3     // -1   (the remainder takes the sign of the left operand)
10 % -3     //  1
```

```python
-10 // 3    # -4    (Python: -3.33 floored)
-10 % 3     #  2    (Python: the remainder takes the sign of the right operand)
```

Both languages keep the identity `(a / b) * b + a % b == a`. They just pick a different quotient. To get Python's always-non-negative modulo in C++ (common in hashing and wrap-around indexing, see [Modular Arithmetic](<../../Python/Fundamentals/Modular Arithmetic.md>)):

```cpp
int python_mod = ((a % m) + m) % m;   // ((-10 % 3) + 3) % 3 = 2
```

### No `**`: Use `std::pow`

C++ has no exponentiation operator. `^` is **XOR**, not power:

```cpp
#include <cmath>

std::pow(2, 10)   // 1024.0, returns a double
2 ^ 10            // 8 (binary 0010 XOR 1010). Clang even warns: did you mean '1 << 10' (1024)?
1 << 10           // 1024: powers of two as an integer, using a shift
```

### `%` Is Integer-Only

```cpp
7.5 % 2.0              // ERROR: invalid operands to binary expression
std::fmod(7.5, 2.0)    // 1.5, from <cmath>
```

### Division by Zero

```cpp
int z = 0;
10 / z;        // UNDEFINED BEHAVIOR for integers (often a crash). No ZeroDivisionError
1.0 / 0.0;     // inf: IEEE floating point defines this
```

### Unary Arithmetic Operators

Same as Python:

```cpp
int x = 5;

std::cout << +x;   // 5
std::cout << -x;   // -5
```

## 2. Increment / Decrement Operators

Python doesn't have these (see [Operators](<../../Python/Fundamentals/Operators.md#7-increment--decrement-operators>)), but C++ uses them constantly. `++` adds 1 and `--` subtracts 1, and each comes in two forms:

```cpp
int i = 5;

int a = i++;   // POSTFIX: a = 5 (the old value), then i becomes 6
int b = ++i;   // PREFIX:  i becomes 7 first, then b = 7 (the new value)

i--;           // i = 6
--i;           // i = 5
```

| Form | Changes the variable | Expression's value |
|---|---|---|
| `++i` (prefix) | `i = i + 1` | The **new** value |
| `i++` (postfix) | `i = i + 1` | The **old** value (a copy kept from before) |

On its own line, `i++;` and `++i;` do the same thing. Many C++ programmers write `++i` by habit, because for class types like iterators, postfix has to make a copy of the old value and prefix doesn't.

> Never modify the same variable twice in one expression, as in `i++ + i++` or `arr[i] = i++`. Depending on the case, the result is unspecified or outright undefined behavior.

## 3. Comparison Operators

```cpp
int a = 5;
int b = 7;

std::cout << (a == b);   // 0 (false)  Equal to
std::cout << (a != b);   // 1 (true)   Not equal to
std::cout << (a < b);    // 1          Less than
std::cout << (a > b);    // 0          Greater than
std::cout << (a <= b);   // 1          Less than or equal to
std::cout << (a >= b);   // 0          Greater than or equal to
```

They return a `bool`, which `std::cout` prints as `1`/`0` unless you send it `std::boolalpha` first.

The parentheses around each comparison are needed because `<<` binds tighter than `==` (see [Precedence](#14-operator-precedence)).

### No Chaining

Python reads `a < b < c` as `a < b and b < c`. **C++ does not.** It evaluates left to right, turns the first comparison into a `bool`, then compares that `bool` (as `0` or `1`) with `c`:

```cpp
int a = 5, b = 3, c = 2;

(a < b) < c     // (5 < 3) < 2  ->  false < 2  ->  0 < 2  ->  true!
a < b && b < c  // what you actually meant: false
```

`a < b < c` is legal C++, but compilers flag it. Recent Clang rejects it outright with `error: chained comparison 'X < Y < Z' does not behave the same as a mathematical expression`.

### Comparing Floating-Point Values

The same caution as in Python applies. Don't compare computed floating-point values with `==`. Compare against a tolerance:

```cpp
#include <cmath>

bool nearly_equal = std::abs((0.1 + 0.2) - 0.3) < 1e-9;   // true
```

> Unreal provides `FMath::IsNearlyEqual(A, B)` for exactly this.

### Comparing Strings

`==` and `<` work on `std::string` and compare contents, like Python. They do **not** work as you'd expect on C-style string literals, which are compared as addresses (see [Arrays and Strings](<Arrays and Strings.md>)):

```cpp
std::string s = "apple";
s == "apple"         // true: std::string compares contents
s < "banana"         // true: lexicographic, like Python

const char* p = "apple";
p == "apple"         // compares two ADDRESSES, not text: don't do this
```

### Comparing Your Own Types

In Python, `==` calls `__eq__`. In C++, `==` on a class calls a function named `operator==`, which you can define. Unlike Python, there is **no default**: if a class doesn't define (or default) `operator==`, comparing two objects of it is a compile error.

| Python dunder | C++ operator function |
|---|---|
| `__eq__` | `operator==` |
| `__ne__` | `operator!=` (generated from `==` in C++20) |
| `__lt__`, `__gt__`, `__le__`, `__ge__` | `operator<`, `operator>`, `operator<=`, `operator>=` (or all four at once from `operator<=>` in C++20) |

```cpp
struct Student {
    std::string name;
    int grade;

    bool operator==(const Student& other) const {
        return grade == other.grade;   // compare by grade only
    }
};

Student a{"Ava", 90};
Student b{"Leo", 90};
std::cout << (a == b);   // 1, because Student::operator== compares just grades
```

Full details in [Operator Overloading](<../OOP/Operator Overloading.md>).

## 4. Identity: There Is No `is`

Python's `is` checks whether two names refer to the same object (see [Operators](<../../Python/Fundamentals/Operators.md#3-identity-operators>)). C++ variables are separate objects by default (see [What is C++](<What is C++.md#3-value-semantics>)), so "is it the same object?" means "is it at the same **address**?". You check that with the address-of operator `&`:

```cpp
int p = 1;
int q = 1;
int& r = p;          // r is a reference: another name for p (see References)

p == q               // true:  same value
&p == &q             // false: different objects in memory (like Python's `is`)
&p == &r             // true:  r refers to the same object as p
```

The C++ counterpart of Python's `x is None` is a pointer null check: `ptr == nullptr`. See [Pointers](<../Memory/Pointers.md>).

## 5. Membership: There Is No `in`

C++ has no `in` operator. Containers provide member functions instead, and the cost depends on the container, exactly as in Python's table (see [Operators](<../../Python/Fundamentals/Operators.md#4-membership-operators>)):

```cpp
std::set<int> s = {1, 2, 3};
std::map<std::string, int> m = {{"a", 1}};
std::vector<int> v = {4, 5, 6};
std::string name = "Manpreet";

s.contains(2)                                  // true    O(log n)      C++20
m.contains("b")                                // false   checks keys   C++20
s.count(3)                                     // 1       pre-C++20 way: 0 or 1 for sets/maps
std::find(v.begin(), v.end(), 5) != v.end()    // true    O(n) scan     needs <algorithm>
name.find("preet") != std::string::npos        // true    substring search
name.contains("Man")                           // true    C++23
```

| Python | C++ |
|---|---|
| `x in some_set` / `k in some_dict` | `s.contains(x)` / `m.contains(k)` |
| `x in some_list` | `std::find(v.begin(), v.end(), x) != v.end()` |
| `"sub" in text` | `text.find("sub") != std::string::npos` (or `text.contains("sub")` in C++23) |

Containers are covered in [STL Containers](<../Standard Library/STL Containers.md>).

## 6. Logical Operators

```cpp
bool x = true;
bool y = false;

x && y    // AND : false
x || y    // OR  : true
!x        // NOT : false
```

| Python | C++ | Also allowed in C++ |
|---|---|---|
| `and` | `&&` | `and` |
| `or` | `\|\|` | `or` |
| `not` | `!` | `not` |

The word forms are real C++ keywords but rarely used. Stick with the symbols.

### Short-Circuit: Same as Python

`&&` stops at the first `false` operand and `||` stops at the first `true` one. The guard pattern from [if_else](<../../Python/Fundamentals/if_else.md#short-circuit-evaluation>) works the same way, and it's even more important in C++, because indexing out of range doesn't raise an error:

```cpp
if (i < v.size() && v[i] == target) {   // i is a size_t index; v[i] is never evaluated when i is out of range
}

if (ptr != nullptr && ptr->health > 0) {  // never dereferences a null pointer
}
```

### They Always Return `bool`

In Python, `and`/`or` return **one of the operands** (`0 or "backup"` gives `"backup"`). In C++, `&&`, `||`, and `!` always produce `true` or `false`:

```cpp
std::cout << std::boolalpha;
std::cout << (0 && 1);   // false
std::cout << (5 || 0);   // true   (not 5)
std::cout << !5;         // false
```

So the Python idiom `name = user_input or "default"` has no direct C++ equivalent. Use the ternary operator instead (section 10).

## 7. Assignment Operators

```cpp
int x = 5;    // Assignment 📦 : x = 5
x += 3;       // Add and assign ➕ : x = 8
x -= 2;       // Subtract and assign ➖ : x = 6
x *= 2;       // Multiply and assign ✖️ : x = 12
x /= 4;       // Divide and assign ➗ : x = 3   (integer division, since x is an int)
x %= 2;       // Modulus and assign 🧮 : x = 1
```

Bitwise compound assignments work just like Python's: `&=`, `|=`, `^=`, `<<=`, `>>=`.

Chained assignment works because assignment is right-associative: `a = (b = (c = 10))`.

```cpp
int a, b, c;
a = b = c = 10;   // all three are 10
```

### Assignment Is an Expression

Python needed the walrus operator `:=` to assign inside an expression. In C++, **every** assignment is already an expression whose value is the assigned variable:

```cpp
int y = 0;
int r = (y = 5) + 1;    // y = 5, r = 6

while ((line_count = read_next()) > 0) {   // assign-and-test, like Python's walrus
}
```

That makes this classic bug possible:

```cpp
if (x = 5) {    // ASSIGNS 5 to x, then tests 5 -> always true!
}
if (x == 5) {   // what was meant
}
```

`-Wall` warns: `using the result of an assignment as a condition without parentheses`.

## 8. Bitwise Operators

Same symbols and same meaning as Python (see [Operators](<../../Python/Fundamentals/Operators.md#8-bitwise-operators>)):

```cpp
int a = 5;   // 0101
int b = 3;   // 0011

a & b    // 1    AND
a | b    // 7    OR
a ^ b    // 6    XOR
~a       // -6   NOT (flips all 32 bits: one's complement, so ~a == -a - 1)
a << 1   // 10   left shift  = multiply by 2^n
a >> 1   // 2    right shift = divide by 2^n, rounding down
```

The differences come from C++ integers having a **fixed width** (see [Variables and Types](<Variables and Types.md#signed-and-unsigned-integers>) for how the bits encode negatives, and [Overflow](<Variables and Types.md#overflow-the-big-difference-from-python>)):

```cpp
1 << 30          // 1073741824: fine, fits in a 32-bit int
1 << 40          // UNDEFINED BEHAVIOR: shift count >= width of type (int is 32 bits)
1LL << 40        // 1099511627776: make the left operand 64-bit first
1u << 31         // 2147483648: unsigned, fits

-16 >> 2         // -4: right-shifting a negative value keeps the sign (defined since C++20)
```

Python's integers have no width, so `1 << 40` just works there.

### Small Types Get Promoted First

Operands narrower than `int` are promoted to `int` before any bitwise operation (see [Type Conversion](<Type Conversion.md#integral-promotion>)):

```cpp
unsigned char uc = 5;   // 00000101
~uc                     // -6, not 250: uc is promoted to a 32-bit int, then all 32 bits are flipped
static_cast<unsigned char>(~uc)   // 250 (11111010): truncate back to 8 bits
```

### Bit Flags

The most common use in game code is packing several on/off flags into one integer:

```cpp
constexpr unsigned CanJump   = 1u << 0;   // 0001
constexpr unsigned CanCrouch = 1u << 1;   // 0010
constexpr unsigned IsHidden  = 1u << 2;   // 0100

unsigned state = 0;
state |= CanJump | IsHidden;      // turn bits on:      0101
state &= ~IsHidden;               // turn a bit off:    0001
bool can_jump = state & CanJump;  // test a bit:        true
state ^= CanCrouch;               // toggle a bit:      0011
```

This is the same mask technique as the file-permission example in [Modules and Packages](<../../Python/Fundamentals/Modules and Packages.md#4-file-permissions>).

## 9. Precedence Trap: `&` vs `==`

In Python, bitwise operators bind **tighter** than comparisons, so `x & 1 == 0` means `(x & 1) == 0`. In C++, it's the **opposite**:

```cpp
int x = 6;

bool even = x & 1 == 0;     // parsed as x & (1 == 0)  ->  6 & 0  ->  0  ->  false. Wrong!
bool even = (x & 1) == 0;   // true. Always parenthesize bitwise operations in comparisons
```

Clang warns: `& has lower precedence than ==; == will be evaluated first`.

## 10. Ternary Operator

Python's `value_if_true if condition else value_if_false` (see [Operators](<../../Python/Fundamentals/Operators.md#9-ternary-expression>)) is written in C++ with `?` and `:`, with the condition **first**:

```cpp
condition ? value_if_true : value_if_false
```

```cpp
int a = 10;
int b = 3;

int max_value = a > b ? a : b;   // 10

std::string label = lives > 0 ? "alive" : "dead";
std::string name = input.empty() ? "default" : input;   // Python's: input or "default"
```

Both branches must have compatible types, since the result has one static type.

## 11. Scope Resolution Operator `::`

Python doesn't have this; it uses dot notation for everything (see [Operators](<../../Python/Fundamentals/Operators.md#10-scope-resolution-operator>)). C++ separates two ideas:

- `.` (and `->`) reaches into an **object**: a value that exists at runtime.
- `::` reaches into a **scope**: a namespace or class, resolved at compile time.

```cpp
std::cout                         // cout inside namespace std
std::string::npos                 // npos, a static member of class std::string
Player::take_damage               // the member function take_damage of class Player
Color::Red                        // the enumerator Red inside enum class Color
::global_count                    // leading :: = the global scope, skipping any local name that shadows it

void Player::take_damage(int amount) { ... }   // defining a member function outside the class
```

See [Scope and Namespaces](<Scope and Namespaces.md>) and [Classes and Objects](<../OOP/Classes and Objects.md>).

## 12. Member Access and Pointer Operators

These operators deal with objects and memory addresses. They're introduced here and explained properly in [Pointers](<../Memory/Pointers.md>):

| Operator | Name | Example | Meaning |
|---|---|---|---|
| `.` | Member access | `player.health` | Member of an object |
| `->` | Member access through pointer | `ptr->health` | Same as `(*ptr).health` |
| `&x` | Address-of | `&player` | The memory address of `player` |
| `*p` | Dereference | `*ptr` | The object that `ptr` points to |
| `[]` | Subscript | `arr[2]` | Element at index 2 (no bounds check for raw arrays and `vector::operator[]`) |
| `()` | Function call | `f(1, 2)` | Call `f` |

```cpp
Player player;
Player* ptr = &player;   // ptr holds player's address

player.health = 100;     // through the object
ptr->health = 90;        // through the pointer
(*ptr).health = 80;      // same thing, spelled out
```

> `&` and `*` each have **two** unrelated meanings depending on context. In an expression, `&x` is address-of and `a & b` is bitwise AND; `*p` is dereference and `a * b` is multiplication. In a declaration, `int& r` declares a reference and `int* p` declares a pointer.

## 13. Other Operators

```cpp
sizeof(int)               // 4: size in bytes, computed at compile time
sizeof x                  // parentheses optional for variables

static_cast<double>(x)    // cast operators (see Type Conversion)

new Player()              // allocate an object on the heap; delete frees it (see Stack and Heap)

int r = (a++, b++, a + b);   // comma operator: evaluate left to right, result is the last one
                             // rarely used, except in for-loop headers: for (i = 0, j = n; i < j; ++i, --j)
```

## 14. Operator Precedence

From highest to lowest, the ones you'll use most often:

| Precedence | Operators | Notes |
|---|---|---|
| 1 | `::` | Scope resolution |
| 2 | `a++` `a--` `f()` `a[i]` `.` `->` | Postfix |
| 3 | `++a` `--a` `+a` `-a` `!` `~` `*p` `&x` `sizeof` `new` `delete` casts | Unary (right-to-left) |
| 4 | `*` `/` `%` | Multiplicative |
| 5 | `+` `-` | Additive |
| 6 | `<<` `>>` | Shift, and stream insertion/extraction |
| 7 | `<=>` | Three-way comparison (C++20) |
| 8 | `<` `<=` `>` `>=` | Relational |
| 9 | `==` `!=` | Equality |
| 10 | `&` | Bitwise AND: **below** `==` (section 9) |
| 11 | `^` | Bitwise XOR |
| 12 | `\|` | Bitwise OR |
| 13 | `&&` | Logical AND |
| 14 | `\|\|` | Logical OR |
| 15 | `?:` `=` `+=` `-=` ... | Ternary and assignment (right-to-left) |
| 16 | `,` | Comma |

Two consequences you'll hit early:

```cpp
std::cout << a == b;     // ERROR: parsed as (std::cout << a) == b
std::cout << (a == b);   // OK

*ptr++                   // *(ptr++): moves the POINTER, not the value it points to
(*ptr)++                 // increments the value ptr points to
```

When in doubt, add parentheses. They cost nothing.

## 15. Operator Overloading

For class types, almost every operator (`+`, `==`, `<`, `[]`, `()`, `<<`, ...) can be given a meaning by writing a function called `operator+`, `operator==`, and so on. That's how `std::string` supports `+` and `==`, and how `std::cout` supports `<<`. It's the C++ counterpart of Python's dunder methods:

| Python | C++ |
|---|---|
| `__add__` | `operator+` |
| `__eq__` | `operator==` |
| `__lt__` | `operator<` |
| `__getitem__` | `operator[]` |
| `__call__` | `operator()` |
| `__str__` / `__repr__` | `operator<<` with `std::ostream` |
| `__contains__` | (no operator; a `contains()` member function by convention) |
| `__bool__` | `explicit operator bool()` |

Operators that can **not** be overloaded: `::`, `.`, `.*`, `?:`, and `sizeof`.

See [Operator Overloading](<../OOP/Operator Overloading.md>).
