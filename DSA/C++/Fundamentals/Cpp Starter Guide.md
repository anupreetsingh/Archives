# C++ Starter Notes (Programmer Edition)

These notes are designed to get you productive in C++ fast, with enough depth to avoid common beginner mistakes.

---

## 1) What C++ Is (and Why It Feels Different)

- Compiled, statically typed, multi-paradigm language.
- High performance and low-level control.
- You can write:
  - procedural code,
  - object-oriented code,
  - generic code (templates),
  - functional-style code.
- C++ gives you power; that means you are responsible for correctness, lifetime management, and undefined behavior.

---

## 2) Tooling Setup

### Compiler choices

- `g++` (GCC)
- `clang++` (Clang)
- MSVC (`cl`) on Windows

Use a modern standard: at least **C++17**, ideally **C++20**.

### Compile + run

```bash
g++ -std=c++20 -Wall -Wextra -Wpedantic -O2 main.cpp -o app
./app
```

Useful flags:

- `-g` for debug symbols
- `-fsanitize=address,undefined` for runtime bug detection

Example debug build:

```bash
g++ -std=c++20 -Wall -Wextra -Wpedantic -g -fsanitize=address,undefined main.cpp -o app
```

---

## 3) Basic Program Structure

```cpp
#include <iostream>

int main() {
    std::cout << "Hello, C++!\n";
    return 0;
}
```

- `#include` brings declarations from headers.
- Execution starts at `main`.
- `std::` means symbol comes from the standard namespace.

---

## 4) Variables, Types, and `const`

### Common types

- Integers: `int`, `long long`
- Floating point: `float`, `double`
- Characters: `char`
- Boolean: `bool`
- Strings: `std::string`

```cpp
int age = 25;
double pi = 3.14159;
bool ok = true;
std::string name = "Ada";
```

### `const`

Use `const` by default for values that should not change.

```cpp
const int maxRetries = 3;
```

### `auto`

Lets compiler infer the type.

```cpp
auto x = 42;          // int
auto y = 3.14;        // double
auto s = std::string("hi");
```

---

## 5) Input/Output

```cpp
#include <iostream>
#include <string>

int main() {
    std::string name;
    std::cout << "Name: ";
    std::getline(std::cin, name);
    std::cout << "Hello, " << name << "\n";
}
```

- `std::cin` for input
- `std::cout` for output
- Prefer `'\n'` over `std::endl` unless you explicitly need a flush.

---

## 6) Control Flow

```cpp
if (x > 0) {
    // ...
} else if (x == 0) {
    // ...
} else {
    // ...
}

for (int i = 0; i < 10; ++i) {}
while (condition) {}
```

Range-based loop:

```cpp
for (const auto& value : values) {
    // use value
}
```

---

## 7) Functions

```cpp
int add(int a, int b) {
    return a + b;
}
```

### Pass by value vs reference

- By value: copies argument.
- By reference: no copy.
- Use `const T&` for read-only large objects.

```cpp
void printName(const std::string& name) {
    std::cout << name << '\n';
}
```

---

## 8) References and Pointers

### Reference

Alias to an existing object; must be initialized.

```cpp
int a = 10;
int& ref = a;
ref = 20; // a becomes 20
```

### Pointer

Stores memory address.

```cpp
int a = 10;
int* p = &a;
std::cout << *p << '\n'; // dereference
```

Use raw pointers mainly for:

- non-owning references,
- low-level APIs.

For ownership, prefer smart pointers.

---

## 9) Memory Management + RAII

### Stack vs Heap

- Stack: automatic lifetime.
- Heap: dynamic lifetime (`new` / `delete`) — avoid manual management unless necessary.

### RAII (core C++ idea)

Resource Acquisition Is Initialization:

- tie resource lifetime to object lifetime.
- cleanup happens in destructor automatically.

Prefer:

- `std::vector`, `std::string`, etc.
- `std::unique_ptr` for exclusive ownership
- `std::shared_ptr` for shared ownership (use carefully)

```cpp
#include <memory>

auto p = std::make_unique<int>(42);
```

---

## 10) Classes and OOP Basics

```cpp
#include <string>

class User {
private:
    std::string name_;

public:
    explicit User(std::string name) : name_(std::move(name)) {}

    const std::string& name() const { return name_; }
    void setName(const std::string& n) { name_ = n; }
};
```

Key points:

- Access control: `public`, `private`, `protected`.
- Constructor initializes object.
- `this` pointer exists in member functions.
- `const` member function means it won't modify object state.

### Inheritance + polymorphism

Use virtual functions for runtime polymorphism.

```cpp
class Shape {
public:
    virtual double area() const = 0;
    virtual ~Shape() = default;
};
```

---

## 11) Standard Template Library (STL): Must Know

### Containers

- `std::vector<T>` dynamic array
- `std::array<T, N>` fixed-size array
- `std::string`
- `std::deque<T>`
- `std::list<T>`
- `std::set<T>`, `std::map<K, V>`
- `std::unordered_set<T>`, `std::unordered_map<K, V>`

### Algorithms (`<algorithm>`)

- `std::sort`
- `std::find`
- `std::binary_search`
- `std::reverse`
- `std::min`, `std::max`

```cpp
#include <algorithm>
#include <vector>

std::vector<int> v{4, 1, 3, 2};
std::sort(v.begin(), v.end());
```

### Iterators

Most algorithms work on iterator ranges `[begin, end)`.

---

## 12) Templates and Generic Programming

```cpp
template <typename T>
T maxValue(T a, T b) {
    return (a > b) ? a : b;
}
```

- Templates generate code for used types at compile time.
- Foundation of STL.

---

## 13) Error Handling

### Exceptions

```cpp
throw std::runtime_error("something went wrong");
```

Catch:

```cpp
try {
    // risky code
} catch (const std::exception& e) {
    std::cerr << e.what() << '\n';
}
```

Use exceptions for exceptional failures, not normal control flow.

---

## 14) File I/O

```cpp
#include <fstream>
#include <string>

std::ofstream out("data.txt");
out << "hello\n";
out.close();

std::ifstream in("data.txt");
std::string line;
while (std::getline(in, line)) {
    // process line
}
```

---

## 15) Common C++ Pitfalls

- Uninitialized variables.
- Dangling references/pointers.
- Returning reference to local variable.
- Out-of-bounds access.
- Double delete / memory leaks (if using raw `new/delete`).
- Integer overflow.
- Forgetting virtual destructor in polymorphic base classes.
- Undefined behavior (UB): code may compile and still be wrong.

---

## 16) Header/Source Organization

Typical structure:

- `*.h` / `*.hpp` for declarations
- `*.cpp` for definitions

Example:

```cpp
// math_utils.h
#pragma once
int add(int a, int b);

// math_utils.cpp
#include "math_utils.h"
int add(int a, int b) { return a + b; }
```

`#pragma once` prevents multiple inclusion.

---

## 17) Build Systems (What to Learn Early)

- Start with direct compiler commands.
- Then move to **CMake** for real projects.

Minimal `CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 3.20)
project(MyApp LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_executable(my_app main.cpp)
```

Build:

```bash
cmake -S . -B build
cmake --build build
./build/my_app
```

---

## 18) Debugging and Quality Habits

- Compile with warnings enabled.
- Treat warnings as serious.
- Use sanitizers while developing.
- Use debugger (`gdb`/`lldb`) to inspect runtime state.
- Write small tests for logic-heavy functions.
- Prefer standard library solutions before custom low-level code.

---

## 19) High-Value Modern C++ Concepts (After Basics)

- Move semantics (`std::move`)
- Rule of 0 / 3 / 5
- Lambdas
- `constexpr`
- `std::optional`, `std::variant`
- Concurrency basics: `std::thread`, mutexes
- Ranges (C++20)

---

## 20) 30-Day Learning Roadmap

### Week 1

- Syntax, types, functions, control flow, references/pointers.
- Solve 20 easy DSA problems in C++.

### Week 2

- STL containers + algorithms + iterators.
- Practice maps/sets/priority queues.

### Week 3

- OOP, classes, RAII, smart pointers, file I/O.
- Build a small CLI project.

### Week 4

- Templates, exceptions, CMake, debugging tools.
- Refactor project with better structure and tests.

---

## 21) Fast Cheatsheet

- Prefer `std::vector` over raw arrays.
- Prefer `const` correctness.
- Prefer `const T&` for read-only heavy params.
- Prefer RAII and smart pointers over manual `new/delete`.
- Use `-std=c++20 -Wall -Wextra -Wpedantic`.
- Use sanitizers for bug hunting.
- Lean on STL first.

---

## 22) Suggested Practice Projects

- CLI todo manager
- File parser (CSV log stats)
- Mini banking/account simulator
- LRU cache implementation
- Expression evaluator

Each project should include:

- modular files,
- input validation,
- error handling,
- simple tests,
- README with build/run steps.
