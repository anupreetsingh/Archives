# Control Flow in C++

Control flow statements decide which code runs and how many times. The ideas are the same as Python's `if`/`for`/`while` (see [if_else](<../../Python/Fundamentals/if_else.md>) and [Iterables](<../../Python/Fundamentals/Iterables.md>)). The syntax differs in three consistent ways:

- The condition goes in **parentheses**: `if (x > 5)`, not `if x > 5:`.
- The body goes in **braces** `{ }`. Indentation means nothing to the compiler.
- `{ }` blocks create a **scope**: variables declared inside them disappear at the closing brace.

---

## 1. `if` / `else if` / `else`

```cpp
int x = 10;

// C++ checks the first condition
if (x > 5) {
    std::cout << "x is greater than 5\n";
}
// If the first condition is false, it checks this one (Python's elif)
else if (x == 5) {
    std::cout << "x is equal to 5\n";
}
// If none of the above conditions are true, this block runs
else {
    std::cout << "x is less than 5\n";
}
```

### Braces Are Optional for One Statement, but Always Use Them

```cpp
if (x > 5)
    std::cout << "big\n";       // only THIS line belongs to the if
    std::cout << "always runs\n"; // indentation lies: this runs no matter what
```

Python would treat both indented lines as the body. C++ attaches only the first statement. Always using braces avoids this whole class of bug.

### What Counts as a Condition

The condition must be convertible to `bool`. Numbers and pointers convert (zero/null is `false`), but strings and containers don't (see [Type Conversion](<Type Conversion.md#boolean-conversion>)):

```cpp
if (lives) { }               // OK: nonzero int is true
if (target) { }              // OK: non-null pointer is true
if (!name.empty()) { }       // strings: check .empty(), not if (name)
if (!items.empty()) { }      // same for containers
```

### Short-Circuit Evaluation

Works exactly like Python's (see [if_else](<../../Python/Fundamentals/if_else.md#short-circuit-evaluation>)):

```cpp
std::string s = "hi";
std::size_t i = 2;

if (i == s.size() || s[i] != 'x') {
    std::cout << "No out-of-bounds read, thanks to short-circuiting with ||\n";
}
```

**Explanation:** `i == s.size()` is true, so `s[i]` is never evaluated. In Python the unguarded version would raise `IndexError`. In C++ it would silently read past the end of the string's characters, which is undefined behavior. The guard matters more in C++.

### `if` With an Initializer (C++17)

You can declare a variable that only exists for the `if`/`else` chain:

```cpp
std::map<std::string, int> scores = {{"Ava", 90}};

if (auto it = scores.find("Ava"); it != scores.end()) {
    std::cout << "found " << it->second << "\n";   // found 90
}
// `it` doesn't exist here
```

This keeps helper variables from leaking into the surrounding scope. It plays a similar role to Python's walrus operator in `if (n := len(items)) > 0:`.

## 2. `switch`

`switch` jumps to the `case` label that matches an **integer-like** value: integers, `char`s, or enums.

```cpp
int day = 2;

switch (day) {
    case 1:
        std::cout << "Mon\n";
        break;                    // leave the switch
    case 2:
        std::cout << "Tue\n";
        break;
    case 6:
    case 7:                       // several labels can share one body
        std::cout << "Weekend\n";
        break;
    default:                      // like a final else
        std::cout << "Other\n";
        break;
}
```

It's similar to Python's `match` statement, but much more limited.

### Fallthrough: Forgetting `break`

A `case` label is only an **entry point**. Without `break`, execution keeps going straight into the next case's code:

```cpp
int day = 2;

switch (day) {
    case 1: std::cout << "Mon\n";
    case 2: std::cout << "Tue\n";      // enters here...
    case 3: std::cout << "Wed\n";      // ...falls through...
    default: std::cout << "default\n"; // ...and through
}
// Output:
// Tue
// Wed
// default
```

If you fall through on purpose, mark it with `[[fallthrough]];` so readers and the compiler's `-Wimplicit-fallthrough` warning know it's intentional.

### Limits of `switch`

```cpp
std::string cmd = "go";
switch (cmd) { }                  // ERROR: statement requires expression of integer type

switch (x) {
    case 1:
        int y = 5;                // ERROR when another case follows:
        break;                    // "jump bypasses variable initialization"
    case 2:
        break;
}
```

- Only integers, characters, and enums can be switched on. Strings need an `if`/`else if` chain or a `std::map` lookup.
- `case` values must be compile-time constants.
- Declaring a variable inside a case needs its own braces: `case 1: { int y = 5; break; }`.

`switch` pairs naturally with `enum class` (see [Enums and Type Aliases](<Enums and Type Aliases.md>)), and with `-Wall` the compiler warns if you forget to handle one of the enum's values.

## 3. `while` and `do`-`while`

`while` is the same as Python's:

```cpp
int i = 0;
while (i < 3) {
    std::cout << i << "\n";
    i++;                       // don't forget to increment!
}
```

`do`-`while` checks the condition **after** the body, so the body always runs at least once:

```cpp
int i = 10;
do {
    std::cout << "runs once even though i < 5 is false\n";
} while (i < 5);               // note the semicolon after the condition
```

A typical use is "ask, then repeat if the answer is invalid".

## 4. The Classic `for` Loop

Python's `for` always iterates over an iterable. C++'s classic `for` is a counter loop with three parts:

```cpp
for (initialization; condition; update) {
    body
}
```

```cpp
for (int i = 0; i < 5; ++i) {
    std::cout << i << " ";      // 0 1 2 3 4
}
// i doesn't exist here: it was scoped to the loop
```

The order of execution is:

```text
initialization (once)
  -> condition? false -> exit
     true -> body -> update -> condition? -> ...
```

Equivalents of Python's `range()` forms (see [Iterables](<../../Python/Fundamentals/Iterables.md#range>)):

| Python | C++ |
|---|---|
| `for i in range(5)` | `for (int i = 0; i < 5; ++i)` |
| `for i in range(2, 6)` | `for (int i = 2; i < 6; ++i)` |
| `for i in range(2, 10, 2)` | `for (int i = 2; i < 10; i += 2)` |
| `for i in range(5, 0, -1)` | `for (int i = 5; i > 0; --i)` |
| `for i in range(n // 2)` (reverse a list in place) | `for (int i = 0; i < n / 2; ++i)` |

Any part can hold more than one expression (using the comma operator), or be left empty:

```cpp
for (int i = 0, j = 4; i < j; ++i, --j) {   // two pointers moving toward each other
    std::cout << i << j << " ";             // 04 13
}

for (;;) {        // no condition: an infinite loop, like Python's while True:
    if (done) break;
}
```

### Loop Variable Types and `.size()`

Since `.size()` returns an unsigned `size_t`, comparing it with a signed `int` triggers a warning, and counting **down** with an unsigned index is a trap:

```cpp
for (std::size_t i = 0; i < v.size(); ++i) { }   // fine: both unsigned

for (std::size_t i = v.size() - 1; i >= 0; --i) { }
// INFINITE LOOP: an unsigned i is ALWAYS >= 0. After 0, --i wraps around to 18446744073709551615

for (int i = static_cast<int>(v.size()) - 1; i >= 0; --i) { }   // correct reverse loop
```

## 5. Range-Based `for` (C++11)

This is the closest thing to Python's `for x in iterable:`. It works on arrays, every standard container, strings, and any type that has `begin()` and `end()` (see [Iterators and Algorithms](<../Standard Library/Iterators and Algorithms.md>)):

```cpp
std::vector<int> nums = {10, 20, 30};

for (int n : nums) {
    std::cout << n << "\n";
}
```

### Copy vs Reference: the Important Part

In Python the loop variable is bound to each element object. In C++ it's a **new variable**, and by default each element is **copied** into it:

```cpp
std::vector<int> nums = {10, 20, 30};

for (int n : nums) {
    n *= 2;                 // doubles the COPY; nums is unchanged
}
// nums = {10, 20, 30}

for (int& n : nums) {
    n *= 2;                 // n is a reference to the real element
}
// nums = {20, 40, 60}
```

Pick the form by what you need:

| Form | Use when |
|---|---|
| `for (int n : nums)` | Small, cheap types (`int`, `double`, `char`) and you're only reading |
| `for (auto& x : items)` | You want to **modify** the elements |
| `for (const auto& x : items)` | You're only reading, and elements are expensive to copy (`std::string`, structs, containers) |

`const auto&` is the safe default for reading anything bigger than a number. Copying every `std::string` in a loop just to print it is a common performance mistake. See [References](<../Memory/References.md>).

### Structured Bindings in Loops (C++17)

Looping over a map gives you key/value **pairs**. Structured bindings unpack them, like Python's `for name, score in scores.items():`

```cpp
std::map<std::string, int> scores = {{"Ava", 90}, {"Leo", 85}};

for (const auto& [name, score] : scores) {
    std::cout << name << ": " << score << "\n";
}
// Ava: 90
// Leo: 85
```

### No Built-In `enumerate` or `zip`

Python's `enumerate()` and `zip()` (see [Iterables](<../../Python/Fundamentals/Iterables.md#enumerating-an-iterable-with-enumerate>)) only arrived in C++23, as `std::views::enumerate` and `std::views::zip` (see [Iterators and Algorithms](<../Standard Library/Iterators and Algorithms.md#ranges-and-views-c20>)). Library support is still catching up: the libc++ that ships with Xcode on this Mac has `zip` but not yet `enumerate`. The usual approach is the classic index loop:

```cpp
std::vector<std::string> names = {"Alice", "Bob"};
std::vector<int> marks = {85, 92};

for (std::size_t i = 0; i < names.size(); ++i) {
    std::cout << "Index " << i << ": " << names[i] << " " << marks[i] << "\n";
}
```

### Don't Change the Container's Size While Looping Over It

Adding or removing elements from a `std::vector` while a range-based `for` is looping over it can reallocate its storage out from under the loop (undefined behavior). Python's equivalent at least behaves predictably. See iterator invalidation in [STL Containers](<../Standard Library/STL Containers.md>).

## 6. `break` and `continue`

Same as Python:

```cpp
for (int k = 0; k < 5; ++k) {
    if (k % 2 == 1) continue;   // skip odd numbers
    std::cout << k;             // 024
}
```

`break` exits only the **innermost** loop (or `switch`), exactly like Python (see [Nested Loops and Flow Control](<../../Python/Fundamentals/Iterables.md#nested-loops-and-flow-control-in-python>)):

```cpp
for (int r = 0; r < 3; ++r) {
    for (int c = 0; c < 3; ++c) {
        if (c == 1) break;       // leaves the inner loop only
        std::cout << r << c << " ";
    }
}
// Output: 00 10 20
```

C++ has no `for ... else` like Python's. To exit two loops at once, use a flag, `return` from a helper function, or (rarely) `goto`.

### Loop Variables Modified in the Body

Unlike Python's `for i in range(...)`, where reassigning `i` in the body has no effect on the next iteration, a C++ counter loop uses the **same** `i` variable throughout. Changing it in the body changes the loop:

```cpp
for (int i = 0; i < 10; ++i) {
    i += 2;            // this really skips iterations: i takes 0, 3, 6, 9
}
```

## 7. Block Scope

Every `{ }` creates a new scope. This is the example from Python's [Namespace and Scope](<../../Python/Fundamentals/Namespace and Scope.md#other-languages>) note, compiled:

```cpp
int main() {
    if (true) {
        int x = 10;
    }
    return x;   // error: use of undeclared identifier 'x'
}
```

Variables declared in a `for` header, an `if` initializer, or a loop body are destroyed when that block ends, and if they're objects with destructors, the destructors run right then. That's the foundation of RAII (see [Object Lifetime and RAII](<../OOP/Object Lifetime and RAII.md>)).

## 8. `goto`

C++ still has `goto label;`, which jumps to `label:` in the same function. Avoid it, except as an occasional way out of deeply nested loops. A helper function with an early `return` is almost always clearer.
