# Lambdas in C++

## What Is a Lambda?

A **lambda** is an unnamed function you write inline, right where it's needed, usually to pass to another function. It's the same idea as Python's `lambda` (see [Lambda In-line Function](<../../Python/Fundamentals/Lambda In-line Fucntion.md>)), with two big differences:

- A C++ lambda can contain **any number of statements**, not just one expression.
- You must say explicitly **which outside variables it uses, and how**: by copy or by reference. That's the **capture list**.

## Syntax

```text
[captures] (parameters) -> return_type { body }
    |           |              |            |
    |           |              |            +-- any statements; use return like a normal function
    |           |              +-- optional: usually deduced from the return statements
    |           +-- like a normal function's parameter list (can be omitted if empty)
    +-- which surrounding variables the body may use, and how
```

## Basic Example

```cpp
auto add = [](int a, int b) { return a + b; };

std::cout << add(2, 3);   // 5
```

```python
add = lambda a, b: a + b
print(add(2, 3))          # 5
```

The type of a lambda has no name you can write (the compiler invents one), so you store it in `auto`.

A multi-statement body is fine:

```cpp
auto classify = [](int n) {
    if (n < 0) return "negative";
    if (n == 0) return "zero";
    return "positive";
};
```

## Use With Algorithms for Custom Sort Keys

The most common use is customizing a standard algorithm (see [Iterators and Algorithms](<../Standard Library/Iterators and Algorithms.md>)).

Python's `sorted()` takes a **key** function that maps each element to a value to sort by. C++'s `std::sort` takes a **comparator**: a function of **two** elements that returns `true` if the first should come before the second:

```cpp
#include <algorithm>

std::vector<std::string> words = {"banana", "kiwi", "apple", "fig"};

std::sort(words.begin(), words.end(),
          [](const std::string& a, const std::string& b) { return a.size() < b.size(); });
// {"fig", "kiwi", "apple", "banana"}
```

```python
words.sort(key=len)
```

To sort descending, flip the comparison (`a.size() > b.size()`). Python's `reverse=True` has no direct equivalent.

## Use With `transform`, `count_if`, and `accumulate`

These play the roles of Python's `map()`, `filter()`, and `reduce()`:

```cpp
#include <algorithm>
#include <numeric>

std::vector<int> v = {1, 5, 2, 8, 3};

// map(lambda n: n * n, v)
std::vector<int> squares(v.size());
std::transform(v.begin(), v.end(), squares.begin(), [](int n) { return n * n; });
// {1, 25, 4, 64, 9}

// len(list(filter(lambda n: n > 3, v)))
auto count = std::count_if(v.begin(), v.end(), [](int n) { return n > 3; });
// 2

// reduce(lambda a, b: a * b, v, 1)
int product = std::accumulate(v.begin(), v.end(), 1, [](int a, int b) { return a * b; });
// 240
```

## Captures: Using Outside Variables

A Python lambda can read any variable in its enclosing scopes, looked up when it runs (see [Namespace and Scope](<../../Python/Fundamentals/Namespace and Scope.md#scope-resolution-using-legb-rule>)). A C++ lambda sees **nothing** from the surrounding scope unless it's listed in the capture list:

```cpp
int threshold = 3;

auto big = [](int x) { return x > threshold; };            // ERROR: 'threshold' cannot be implicitly captured
auto big = [threshold](int x) { return x > threshold; };   // OK: captures a copy of threshold
```

(Globals and `static` variables don't need capturing, because they don't live in a function's stack frame.)

### By Copy vs By Reference

```cpp
int x = 1;

auto by_value = [x]() { return x; };    // copies x NOW, when the lambda is created
auto by_ref = [&x]() { return x; };     // refers to x itself

x = 100;

by_value();   // 1:   it kept its own copy from creation time
by_ref();     // 100: it reads the current x
```

| Capture | Meaning |
|---|---|
| `[]` | Capture nothing |
| `[x]` | Copy `x` |
| `[&x]` | Reference to `x` |
| `[x, &y]` | Copy `x`, reference `y` |
| `[=]` | Copy everything used in the body (avoid: hides what's captured) |
| `[&]` | Reference everything used in the body (avoid: easy to dangle) |
| `[this]` | Access the enclosing object's members (copies the `this` pointer) |
| `[p = std::move(ptr)]` | Init-capture: create a new member `p`, here by moving a `unique_ptr` in |

By-reference capture is how a lambda modifies an outside variable, the job Python's `nonlocal` does (see [`nonlocal`](<../../Python/Fundamentals/Namespace and Scope.md#nonlocal>)):

```cpp
int total = 0;
std::for_each(v.begin(), v.end(), [&total](int x) { total += x; });
// total = 19
```

### `mutable`: Modifying Your Own Copies

Copied captures are `const` inside the lambda by default. `mutable` lets the lambda change its **own copies**. The originals are never affected:

```cpp
int x = 1;
auto f = [x]() { x = 2; };            // ERROR: cannot assign to a variable captured by copy in a non-mutable lambda

auto make_counter() {
    int count = 0;
    return [count]() mutable { return ++count; };   // the lambda owns a private count that persists between calls
}

auto counter = make_counter();
counter();   // 1
counter();   // 2
counter();   // 3
```

That's the C++ version of a Python closure that keeps state.

### The Dangling Capture Trap

A by-reference capture doesn't keep the variable alive. Python closures keep captured variables alive as long as the closure exists. C++ lambdas don't:

```cpp
std::function<int()> make_bad() {
    int local = 42;
    return [&local]() { return local; };   // captures a reference to a local...
}                                          // ...which is destroyed here

auto f = make_bad();
f();    // UB: reads a dead stack variable (dangling reference, see References)
```

Rule: if a lambda may outlive the current scope (it's returned, stored, or run later, such as a callback or a delegate), capture **by copy** or by move, never by reference.

> This rule matters a lot in Unreal. Timers, async tasks, and delegates run lambdas later. Capturing `this` or locals by reference into them is a classic crash. Unreal provides `CreateWeakLambda` and `FTimerDelegate::CreateUObject` to tie callbacks safely to an object's lifetime.

## How Lambdas Work: Compiler-Generated Function Objects

A lambda is shorthand for a small class with an `operator()` (see [Operator Overloading](<../OOP/Operator Overloading.md#5-calling-operator-function-objects>)). Each capture becomes a data member:

```cpp
int threshold = 3;
auto big = [threshold](int x) { return x > threshold; };

// roughly what the compiler generates:
struct __Lambda_1 {
    int threshold;                                            // the capture, copied in
    bool operator()(int x) const { return x > threshold; }    // const: why copies can't be modified without mutable
};
__Lambda_1 big{threshold};
```

So a lambda is just an object whose size is its captures:

```cpp
sizeof(add)        // 1: no captures (an empty class still takes 1 byte)
sizeof(by_value)   // 4: one captured int
```

Calling it is an ordinary, inlinable function call, which is why passing a lambda to `std::sort` is as fast as hand-writing the comparison.

A lambda with **no captures** can also convert to a plain function pointer (see [Pointers](<../Memory/Pointers.md#9-function-pointers>)):

```cpp
int (*fp)(int, int) = [](int a, int b) { return a * b; };
fp(6, 7);   // 42
```

## Generic Lambdas

`auto` parameters make the lambda a template (see [Templates](<Templates.md#abbreviated-templates-auto-parameters-c20>)), so one lambda works for many types, much like an untyped Python lambda:

```cpp
auto generic = [](auto a, auto b) { return a + b; };

generic(1, 2);                                   // 3
generic(std::string("a"), std::string("b"));     // "ab"
```

## Storing Lambdas: `auto` vs `std::function`

Every lambda has its own unique type. To store lambdas of **different** types in one variable or container (Python's "lambdas as dictionary values"), use `std::function<Signature>`, which can hold anything callable with that signature:

```cpp
#include <functional>
#include <map>

std::map<std::string, std::function<double(double, double)>> ops = {
    {"add", [](double a, double b) { return a + b; }},
    {"mul", [](double a, double b) { return a * b; }},
};

ops["mul"](3, 4);   // 12
```

| | `auto` | `std::function<R(Args...)>` |
|---|---|---|
| Holds | One lambda, its exact type | Any callable with that signature |
| Size | The captures only | Fixed (32 bytes with libc++), plus a possible heap allocation for big captures |
| Call cost | Direct, inlinable | Indirect, like a virtual call |
| Use for | Local variables, template parameters | Struct members, containers, callbacks stored for later |

### Recursive Lambdas

A lambda can't refer to itself by name, since it's still being defined. Two ways around that:

```cpp
// C++23: an explicit object parameter ("deducing this")
auto fact = [](this auto self, int n) -> int { return n <= 1 ? 1 : n * self(n - 1); };
fact(5);    // 120

// Pre-C++23: capture a std::function by reference
std::function<int(int)> fib = [&fib](int n) { return n < 2 ? n : fib(n - 1) + fib(n - 2); };
fib(10);    // 55
```

For anything non-trivial, a named function is usually clearer.

## Limitations of Lambdas

- Writing out the capture list takes a little thought. Python's automatic closures are simpler, but less explicit.
- By-reference captures can dangle if the lambda outlives the scope.
- Long lambdas inline in a function call hurt readability. Give them a name (`auto by_length = [...]`) or make them a function.

## When to Use a Lambda

- ✔ Custom comparisons and predicates for algorithms (`std::sort`, `std::find_if`, `std::count_if`, `std::erase_if`)
- ✔ Short callbacks (event handlers, Unreal delegates via `BindLambda`, timers)
- ✔ Small, local helpers that need to capture some local state
- ✘ Long, reused logic: write a named function instead
