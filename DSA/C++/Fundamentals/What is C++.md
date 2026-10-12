# What is C++

C++ is a **compiled**, **statically typed**, general-purpose programming language. Bjarne Stroustrup started it in 1979 as "C with Classes" on top of the C language, and almost every valid C program is still valid C++.

→ You write source code, a **compiler** turns it into native machine code for a specific CPU and operating system, and the CPU runs that machine code directly. There is no interpreter or virtual machine involved at runtime.

**Example:**

```cpp
#include <iostream>

int main() {
    int x = 5 + 3;          // x is an int, and that can never change
    std::cout << x << "\n"; // 8
    return 0;
}
```

The full anatomy of this program is in [Program Structure](<Program Structure.md>).

---

## 1. How Python Runs vs How C++ Runs

Recall from [Namespace and Scope](<../../Python/Fundamentals/Namespace and Scope.md#when-python-classifies-names>) that CPython compiles `.py` source into **bytecode**, and the **PVM** then executes that bytecode every time the program runs.

C++ does all of its translation **before** the program runs:

```text
Python:
  app.py --(CPython compiles)--> bytecode --(PVM interprets, at runtime)--> CPU

C++:
  app.cpp --(compiler + linker, ahead of time)--> native executable --(runs directly)--> CPU
```

That difference drives most of what feels different about C++:

| | Python | C++ |
|---|---|---|
| When translation happens | Every run, plus at runtime | Once, before you run it (a separate **build** step) |
| What you ship | Source or bytecode + an interpreter | A native executable (or library) |
| Type errors found | At runtime, when the line executes | At compile time, before the program ever runs |
| Speed | Each operation goes through the interpreter | Operations are plain CPU instructions |
| Portability | Same `.py` runs anywhere Python is installed | Must recompile for each CPU/OS combination |
| Memory management | Reference counting + garbage collector | You (or your objects' destructors) decide when memory is freed |

The build step itself is covered in [Compilation and Linking](<Compilation and Linking.md>).

## 2. Static Typing

Every variable, parameter, and return value in C++ has a type that is known at **compile time** and never changes.

```cpp
int count = 10;
count = 20;        // fine, still an int
count = "hello";   // compile error: cannot assign a string literal to an int
```

In Python, a name can be rebound to an object of any type (see [Name Rebinding](<../../Python/Fundamentals/Name Rebinding.md>)). In C++, the type belongs to the **variable**, not just to the value inside it.

This is what lets the compiler:

- Reject type mistakes before the program runs.
- Know exactly how many bytes each variable needs.
- Generate the exact machine instruction for each operation (an `int + int` becomes a single integer-add instruction).

`auto` lets the compiler infer the type for you, but the type is still fixed once inferred:

```cpp
auto total = 0;    // inferred as int
total = 3.7;       // compiles (with a warning), but 3.7 is converted to int 3 (see Type Conversion)
```

More in [Variables and Types](<Variables and Types.md>).

## 3. Value Semantics

In Python, assignment binds another name to the **same object**:

```python
a = [1, 2, 3]
b = a          # b and a refer to the same list object
b.append(4)
print(a)       # [1, 2, 3, 4]
```

In C++, a variable **is** the object: a named region of memory that holds the value itself. Assignment and initialization **copy** the value into another region of memory.

```cpp
std::vector<int> a = {1, 2, 3};
std::vector<int> b = a;   // b is a brand-new vector; all three elements are copied
b.push_back(4);

// a is still {1, 2, 3}
// b is     {1, 2, 3, 4}
```

When you **want** two names to refer to the same object in C++, you have to say so explicitly with a **reference** or a **pointer**. See [References](<../Memory/References.md>) and [Pointers](<../Memory/Pointers.md>).

## 4. Zero-Overhead Abstraction

C++'s design principle, in Stroustrup's words:

> What you don't use, you don't pay for. And further: what you do use, you couldn't hand code any better.

- A `class` with no virtual functions takes exactly as much memory as its data members (see [Object Memory Layout](<../Memory/Object Memory Layout.md>)).
- A `std::vector<int>` stores its integers in one contiguous block, with no per-element overhead. A Python list stores references to separately allocated `int` objects (see [List Structure](<../../Python/Fundamentals/List Structure.md#internal-storage>)).
- [Templates](<Templates.md>) generate specialized code at compile time, so generic code runs as fast as hand-written code for each type.

The cost of this is that C++ trusts you. It does not check array bounds, it does not stop you from using memory you already freed, and it does not initialize variables for you unless you ask.

## 5. Deterministic Resource Management

C++ has **no garbage collector**. An object is destroyed at a precisely known point:

- A local variable is destroyed when execution leaves the `{ }` block it was declared in.
- An object you allocated yourself with `new` is destroyed when you `delete` it.

```cpp
void work() {
    std::string name = "Ava";  // memory for the string is acquired here
    // ... use name ...
}                              // name is destroyed here, and its memory is released, guaranteed
```

Classes can run code when they are destroyed (a **destructor**), which is how C++ cleans up files, locks, and memory automatically. This pattern is called **RAII** and plays the role Python's `with` blocks play. See [Object Lifetime and RAII](<../OOP/Object Lifetime and RAII.md>).

## 6. Undefined Behavior

The C++ standard says certain operations have **undefined behavior** (UB): the language makes no promise at all about what happens. The program might crash, print garbage, or appear to work today and break after a compiler upgrade.

Python raises an exception for many of these. C++ usually does not check:

```cpp
int arr[3] = {1, 2, 3};
std::cout << arr[5];      // UB: no IndexError; reads whatever memory happens to be there

int x;                    // declared but not initialized
std::cout << x;           // UB: reading an uninitialized variable
```

```python
arr = [1, 2, 3]
print(arr[5])             # IndexError: list index out of range
```

UB is the single biggest source of C++ bugs, and the tools that catch it are covered in [Undefined Behavior and Sanitizers](<../Memory/Undefined Behavior and Sanitizers.md>).

## 7. Multi-Paradigm

C++ does not force one style. The same program can mix:

- **Procedural**: free functions and plain data, like C.
- **Object-oriented**: classes, inheritance, and virtual functions (see [OOP Four Pillars](<../OOP/OOP Four Pillars.md>)).
- **Generic**: templates that work for any type (see [Templates](<Templates.md>)).
- **Functional bits**: lambdas, and algorithms that take functions as arguments (see [Lambdas](<Lambdas.md>)).

## 8. C++ Standards

C++ is defined by an ISO standard, not by one implementation. A new version ships every three years:

| Standard | Notable additions |
|---|---|
| C++98 / C++03 | The first standard: classes, templates, the STL containers |
| **C++11** | The "modern C++" turning point: `auto`, lambdas, range-based `for`, move semantics, `nullptr`, smart pointers, `constexpr` |
| C++14 | Smaller fixes: generic lambdas, `std::make_unique` |
| C++17 | Structured bindings, `if` with initializer, `std::optional`, `std::string_view`, inline variables |
| C++20 | Concepts, ranges, modules, `<=>` (spaceship operator), `std::format` |
| C++23 | `std::print`, `std::expected` |

You pick the version when compiling:

```bash
clang++ -std=c++20 main.cpp -o main
```

> Recent Unreal Engine 5 releases compile game code as C++20, so these notes use C++20 features where they matter and point out when something is newer.

## 9. Compilers and Standard Libraries

A **compiler** implements the language, and a **standard library implementation** provides `std::vector`, `std::string`, and the rest:

| Platform | Compiler | Command | Standard library |
|---|---|---|---|
| macOS | Apple Clang (comes with Xcode / Command Line Tools) | `clang++` | libc++ |
| Linux | GCC or Clang | `g++` / `clang++` | libstdc++ (GCC) |
| Windows | MSVC (comes with Visual Studio) | `cl.exe` | MSVC STL |

On macOS, `g++` is just an alias for Apple Clang. Run `g++ --version` and it prints `Apple clang version ...`.

Because the standard only fixes behavior and not implementation, some details differ between platforms, such as the size of `long` (8 bytes on macOS/Linux, 4 on Windows) or `sizeof(std::string)`. These notes mark those details as implementation-defined and give the values you'll see on an Apple Silicon Mac.

## 10. Where C++ Is Used

C++ is the default choice when performance, latency, or control over memory matters:

- Game engines and games: Unreal Engine, Unity’s native engine components, Source 2, CryEngine, and many AAA games are built largely in C++.

- Browsers, operating systems, databases, and compilers: Chromium/Google Chrome, parts of Windows and macOS, MySQL, MongoDB, LLVM, and Clang all make extensive use of C++.

-*Low-latency trading systems: High-frequency trading platforms, exchange connectivity systems, market-data processors, and order-execution engines at firms such as Jane Street, Citadel Securities, and Hudson River Trading commonly use C++.

- High-performance cores behind higher-level languages: CPython itself is written mainly in C, while libraries such as NumPy, PyTorch, TensorFlow, and OpenCV perform much of their computationally intensive work in C or C++ behind Python interfaces.

Languages like Rust can often match C++ in performance, but C++ has decades of accumulated ecosystem, tooling, libraries, platform support, and production code that companies cannot simply discard for the sake of adopting a newer language.
