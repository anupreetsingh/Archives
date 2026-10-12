# Functions in C++

A function in C++ is the same idea as in Python: a named block of code that takes **parameters** and can **return** a value. The vocabulary from [Function Syntax](<../../Python/Fundamentals/Function Syntax.md>) carries over directly:

- In a function definition, **parameters** name the values the function expects.
- In a function call, **arguments** provide the actual values for those parameters.

What's new in C++: every parameter and the return value have a **fixed type**, a function must be **declared before it's called**, arguments are **copied** by default, and several functions can share one name (**overloading**).

## Anatomy of a Function

```cpp
int add(int a, int b) {
    return a + b;
}
```

```text
 int        add      (int a, int b)       { return a + b; }
  |          |             |                     |
return     name       parameter list            body
 type              (each with its own type)
```

Python's type hints (see [Function Syntax](<../../Python/Fundamentals/Function Syntax.md#type-hints>)) are optional and not enforced. In C++ the types are **required** and **enforced** at compile time:

```python
def add(a: int, b: int) -> int:   # hints only: add("x", "y") still runs
    return a + b
```

```cpp
int add(int a, int b) { return a + b; }

add(2, 3);        // 5
add("x", "y");    // ERROR: no matching function for call to 'add'
```

### `void`: No Return Value

A function that returns nothing has the return type `void`. It's the equivalent of a Python function that implicitly returns `None`, except that there's no value at all to use:

```cpp
void greet(const std::string& name) {
    std::cout << "Hello, " << name << "\n";
    // return; is optional at the end
}

int x = greet("Ava");   // ERROR: cannot initialize a variable of type 'int' with an rvalue of type 'void'
```

A non-`void` function that reaches its closing `}` without a `return` is undefined behavior (`main` is the only exception). `-Wall` warns about it.

## Declare Before Use

The compiler reads each file top to bottom (see [Compilation and Linking](<Compilation and Linking.md#2-compiling>)), so a function must be declared before the first call:

```cpp
int square(int x);           // declaration (prototype): name, parameter types, return type, then ;

int main() {
    return square(3);        // OK: the compiler has seen the declaration
}

int square(int x) {          // definition, which can come later or live in another .cpp file
    return x * x;
}
```

Parameter **names** are optional in a declaration (`int square(int);`), since only the types matter there. This is exactly the declaration/definition split that headers are built on (see [Headers and Source Files](<Headers and Source Files.md>)).

Python doesn't need this because `def` runs at runtime, and names inside a function body are looked up only when it's called (see [Namespace and Scope](<../../Python/Fundamentals/Namespace and Scope.md#functions>)).

## Passing Arguments

This is the biggest difference from Python. In Python, every argument passes a reference to the caller's object (see [Namespace and Scope](<../../Python/Fundamentals/Namespace and Scope.md#function-parameters-are-local-names>)), so a function can mutate a list you pass in. In C++, **you choose** how each parameter receives its argument:

### 1. By Value: a Copy (the Default)

```cpp
void reset(int x) {
    x = 0;                   // changes the local copy only
}

void add_item(std::vector<int> v) {
    v.push_back(4);          // the whole vector was copied in; the caller's vector is untouched
}

int n = 7;
reset(n);                    // n is still 7

std::vector<int> nums = {1, 2, 3};
add_item(nums);              // nums.size() is still 3, and a full copy of nums was made
```

### 2. By Reference: `&`

A **reference** parameter is another name for the caller's variable. No copy is made, and changes are visible to the caller:

```cpp
void reset(int& x) {
    x = 0;                   // modifies the caller's variable
}

void add_item(std::vector<int>& v) {
    v.push_back(4);          // modifies the caller's vector
}

int n = 7;
reset(n);                    // n is now 0

std::vector<int> nums = {1, 2, 3};
add_item(nums);              // nums.size() is now 4
```

### 3. By `const` Reference: `const &`

No copy, and the function promises not to modify the argument. This is the standard way to pass anything bigger than a number when you only need to **read** it:

```cpp
void print_all(const std::vector<std::string>& names) {
    for (const auto& name : names) {
        std::cout << name << "\n";
    }
    names.push_back("x");    // ERROR: names is const
}
```

### 4. By Pointer: `*`

Passes an address. It works like a reference, but can be `nullptr`, so it's used when "no object" is a valid argument:

```cpp
void heal(Player* target) {
    if (target != nullptr) {
        target->health = 100;
    }
}

heal(&player);               // pass the address
heal(nullptr);               // "no target"
```

### Choosing a Parameter Type

| Parameter | Copies? | Can modify the caller's object? | Use for |
|---|---|---|---|
| `T x` | Yes | No | Small types: `int`, `double`, `bool`, `char`, small structs, pointers |
| `const T& x` | No | No | Reading anything larger: `std::string`, `std::vector`, your classes |
| `T& x` | No | **Yes** | "Out" or "in-out" parameters the function must change |
| `T* x` | No | Yes (unless `const T*`) | Optional objects (`nullptr` allowed), or C-style APIs |

The full story is in [References](<../Memory/References.md>) and [Pointers](<../Memory/Pointers.md>).

> Unreal uses this exact convention: `const FString& Name` for read-only inputs, `FHitResult& OutHit` (prefixed `Out`) for outputs, and `AActor* Target` for objects that may be null.

## Returning Values

`return` hands back one value, **by value** (a copy, which the compiler usually optimizes away):

```cpp
std::vector<int> make_range(int n) {
    std::vector<int> result;
    for (int i = 0; i < n; ++i) result.push_back(i);
    return result;           // not actually copied: return value optimization / move (see Copy and Move Semantics)
}
```

Returning a large object by value is cheap in modern C++, so don't contort code to avoid it.

### Returning Several Values

Python returns a tuple: `return lo, hi`. C++ returns one object, so you bundle the values in a `std::pair`, a `std::tuple`, or (clearest) a small `struct`, then unpack them with **structured bindings**:

```cpp
#include <utility>   // std::pair

std::pair<int, int> min_max(const std::vector<int>& v) {
    int lo = v[0], hi = v[0];
    for (int x : v) {
        lo = std::min(lo, x);
        hi = std::max(hi, x);
    }
    return {lo, hi};
}

auto [lo, hi] = min_max({5, 1, 9, 3});   // like Python's: lo, hi = min_max(...)
// lo = 1, hi = 9
```

```cpp
struct Stats {
    int min;
    int max;
    double mean;
};

Stats compute_stats(const std::vector<int>& v);   // named fields beat .first / .second
```

### Never Return a Reference to a Local

A local variable is destroyed when the function returns, so a reference or pointer to it is left dangling:

```cpp
int& bad() {
    int x = 5;
    return x;    // warning: reference to stack memory associated with local variable 'x' returned
}                // x is destroyed here; the caller gets a reference to dead memory (UB)
```

Return by value instead. See [Stack and Heap](<../Memory/Stack and Heap.md>).

### `auto` Return Type

The compiler can deduce the return type from the `return` statements:

```cpp
auto square(int x) { return x * x; }          // returns int

auto mul(int a, int b) -> int { return a * b; }   // trailing return type: same as int mul(int, int)
```

## Default Arguments

Like Python's default parameters, with the same rule that they must come **after** all required parameters:

```cpp
void connect(const std::string& host, int timeout = 5, bool verbose = false) {
    std::cout << host << " " << timeout << " " << verbose << "\n";
}

connect("localhost");             // localhost 5 0
connect("localhost", 10);         // localhost 10 0
connect("localhost", 10, true);   // localhost 10 1

void f(int a = 1, int b);         // ERROR: missing default argument on parameter 'b'
```

Two C++-specific rules:

- **Put defaults in the declaration only** (usually the header), not again in the definition.
- C++ evaluates a default argument on **every call**, so there's no Python-style shared mutable default problem.

### No Keyword Arguments

C++ has **no keyword arguments**. Arguments are matched to parameters purely by position, so you can't skip `timeout` to set `verbose`:

```cpp
connect("localhost", verbose = true);   // ERROR: there is no such syntax
connect("localhost", 5, true);          // you must spell out every earlier argument
```

When a function has many optional settings, a common C++20 pattern is to pass a struct with **designated initializers**, which reads like keyword arguments:

```cpp
struct ConnectOptions {
    int timeout = 5;
    bool verbose = false;
};

void connect(const std::string& host, ConnectOptions opts = {});

connect("localhost", {.verbose = true});   // timeout keeps its default of 5
```

Designated initializers must appear in the order the members are declared.

## Function Overloading

Several functions can share **one name** if their **parameter lists** differ. The compiler picks the best match from the argument types at each call site:

```cpp
int add(int a, int b)                 { return a + b; }
double add(double a, double b)        { return a + b; }
int add(int a, int b, int c)          { return a + b + c; }
std::string add(const std::string& a, const std::string& b) { return a + b; }

add(2, 3);                           // 5     calls add(int, int)
add(2.5, 1.0);                       // 3.5   calls add(double, double)
add(1, 2, 3);                        // 6     calls add(int, int, int)
add(std::string("a"), std::string("b")); // "ab"
```

Python can't do this: a second `def add` simply **rebinds** the name, and the first function is lost (see [Name Rebinding](<../../Python/Fundamentals/Name Rebinding.md#rebindingoverwriting>) and [OOP Four Pillars](<../../Python/OOP/OOP Four Pillars.md#41-method-overloading-compile-time-polymorphism>), which shows the Java version of this). In C++, each overload is a genuinely separate function with its own mangled symbol, such as `_Z3addii` and `_Z3adddd` (see [Name Mangling](<Compilation and Linking.md#name-mangling>)). This is **compile-time polymorphism**: the choice is made by the compiler, not at runtime.

Overloading rules:

```cpp
int f(int a);
double f(int a);       // ERROR: functions that differ only in their return type cannot be overloaded

add(2, 3.5);           // ERROR: call to 'add' is ambiguous
                       // int->double and double->int are equally good conversions, so neither wins
add(2.0, 3.5);         // OK: exact match for add(double, double)
```

## `static` Local Variables

A `static` local is initialized **once**, the first time execution reaches it, and keeps its value between calls:

```cpp
int counter() {
    static int calls = 0;   // runs only on the first call
    return ++calls;
}

counter();   // 1
counter();   // 2
counter();   // 3
```

It's like a global variable that only this function can see. In Python you'd use a closure, a function attribute, or a class for this. See [Scope and Namespaces](<Scope and Namespaces.md#5-storage-duration-how-long-a-variable-lives>).

## `inline` and `constexpr` Functions

- `inline` allows a function to be **defined in a header** that many `.cpp` files include, without duplicate-symbol linker errors (see [Headers and Source Files](<Headers and Source Files.md#definitions-in-headers-why-they-break-and-inline>)). Whether the call actually gets inlined is up to the optimizer.
- `constexpr` means the function **can** run at compile time when its arguments are compile-time constants. `constexpr` functions are implicitly `inline`.

```cpp
constexpr int cube(int x) { return x * x * x; }

constexpr int size = cube(4);   // computed by the compiler: size = 64, no runtime cost
int arr[cube(2)];               // OK: array size must be a compile-time constant

int n = read_number();
int r = cube(n);                // also fine: just runs at runtime
```

## `[[nodiscard]]`

Marks a function whose return value must not be silently ignored:

```cpp
[[nodiscard]] bool save_game();

save_game();   // warning: ignoring return value of function declared with 'nodiscard' attribute
```

Useful for functions that return errors or newly allocated results.

## Recursion

Recursion works just like in Python (see [Recursion](<../../Python/Traversal and Search Algorithms/Recursion.md>)), with two differences:

- There's no recursion limit like Python's default of 1000. The limit is the size of the **call stack** (typically 8 MB for the main thread on macOS/Linux, 1 MB on Windows). Exceeding it is a **stack overflow**, and the program crashes with a segmentation fault instead of raising `RecursionError`.
- Each call's frame is usually much smaller than a Python frame, so you can recurse far deeper before running out.

See [Stack and Heap](<../Memory/Stack and Heap.md>).

## Functions as Values

Functions can be passed to other functions, like Python's `key=len` arguments. The modern way is a **lambda** or `std::function`, covered in [Lambdas](<Lambdas.md>):

```cpp
#include <algorithm>

std::vector<std::string> words = {"banana", "kiwi", "apple"};

std::sort(words.begin(), words.end(),
          [](const std::string& a, const std::string& b) { return a.size() < b.size(); });
// like Python's words.sort(key=len)
// {"kiwi", "apple", "banana"}
```

## Summary

| Python | C++ |
|---|---|
| `def f(a, b):` | `ReturnType f(TypeA a, TypeB b) { }` |
| Type hints optional, not enforced | Types required and enforced |
| Can call any function defined by the time the call runs | Must be declared above the call (or in an included header) |
| Arguments always share the caller's object | By value (copy) unless you write `&`, `const &`, or `*` |
| `return a, b` | Return a `struct` / `std::pair`, unpack with `auto [a, b] = ...` |
| Keyword arguments | None: positional only (use a struct of options) |
| Later `def` with the same name replaces the earlier one | Overloading: same name, different parameter types, all coexist |
| `RecursionError` | Stack overflow crash |
