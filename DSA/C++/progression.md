# C++ Notes — Reading Order

These notes cover the C++ fundamentals needed before (and alongside) Unreal Engine C++: what C++ is, how code becomes a program, the core language, memory, classes, and the standard library. They follow the same style as the Python notes, and link back to them wherever an idea carries over. A link like "see [Operators](<../Python/Fundamentals/Operators.md>)" means "you already know this part; only the differences are explained here".

Read them in the order below. Each stage leans on the ones before it, and links usually point **backward** to notes you've already read. A forward link means a topic is previewed and covered properly later, so you can skip it on a first pass.

```text
C++/
├── progression.md          <- you are here
├── Fundamentals/           what C++ is, how it's built, the core language, generic code
├── Memory/                 where objects live, pointers, references, const, UB, layout, ownership
├── OOP/                    classes, lifetime, copying/moving, inheritance, virtual dispatch, operators
├── Standard Library/       containers, iterators, algorithms
└── Concurrency/            threads, data races, mutexes, locks, deadlock
```

---

## Stage 1: What C++ Is and How Code Becomes a Program

Start here. These explain the build model that every later "why" depends on: why headers exist, why declarations come before use, and what linker errors mean.

| # | Note | What it answers | Compare with (Python) |
|---|---|---|---|
| 1 | [What is C++](<Fundamentals/What is C++.md>) | Compiled vs interpreted, static typing, value semantics, zero-overhead, UB, standards | [Namespace and Scope](<../Python/Fundamentals/Namespace and Scope.md#when-python-classifies-names>) (bytecode + PVM) |
| 2 | [Program Structure](<Fundamentals/Program Structure.md>) | `main()`, statements and braces, `std::cout` / `std::cin`, building and running | |
| 3 | [Compilation and Linking](<Fundamentals/Compilation and Linking.md>) | Preprocess → compile → assemble → link, object files, symbols, name mangling, linker errors | |
| 4 | [Preprocessor](<Fundamentals/Preprocessor.md>) | `#include`, `#define`, macros, conditional compilation | [Modules and Packages](<../Python/Fundamentals/Modules and Packages.md>) (contrast with `import`) |
| 5 | [Headers and Source Files](<Fundamentals/Headers and Source Files.md>) | Declarations vs definitions, the ODR, `.h`/`.cpp` split, header guards, `inline`, forward declarations | [Modules and Packages](<../Python/Fundamentals/Modules and Packages.md>) |

**Checkpoint:** you can explain why `#include "Player.h"` is enough to *call* `Player` functions, and which file has to be in the build for the program to *link*.

## Stage 2: The Core Language

Mostly familiar ideas with C++ syntax, but watch for the traps that are flagged throughout: integer division, unsigned `.size()`, no chained comparisons, `switch` fallthrough, copying in range-`for`, and no keyword arguments.

| # | Note | What it answers | Compare with (Python) |
|---|---|---|---|
| 6 | [Variables and Types](<Fundamentals/Variables and Types.md>) | Variables as boxes, fundamental types and sizes, bits and ranges, signed/unsigned and two's complement, overflow, initialization forms, `auto`, literals | [Name Rebinding](<../Python/Fundamentals/Name Rebinding.md>) |
| 7 | [Type Conversion](<Fundamentals/Type Conversion.md>) | Implicit promotions and their traps, `static_cast` and friends, number ↔ string | [Type Conversion](<../Python/Fundamentals/Type Conversion.md>) |
| 8 | [Operators](<Fundamentals/Operators.md>) | What differs from Python's operators: `/` and `%`, `++`, no chaining, `&&`, `?:`, `::`, `->`, precedence | [Operators](<../Python/Fundamentals/Operators.md>) |
| 9 | [Control Flow](<Fundamentals/Control Flow.md>) | `if`, `switch`, loops, range-based `for` (copy vs reference), block scope | [if_else](<../Python/Fundamentals/if_else.md>), [Iterables](<../Python/Fundamentals/Iterables.md>) |
| 10 | [Functions](<Fundamentals/Functions.md>) | Declarations, pass by value / reference / pointer, returning multiple values, defaults, overloading | [Function Syntax](<../Python/Fundamentals/Function Syntax.md>), [Name Rebinding](<../Python/Fundamentals/Name Rebinding.md>) |
| 11 | [Scope and Namespaces](<Fundamentals/Scope and Namespaces.md>) | Block scope, shadowing, namespaces, `using`, storage duration, linkage, the meanings of `static` | [Namespace and Scope](<../Python/Fundamentals/Namespace and Scope.md>) |
| 12 | [Enums and Type Aliases](<Fundamentals/Enums and Type Aliases.md>) | `enum` vs `enum class`, underlying types, `using` aliases | |

**Checkpoint:** you can predict the output of `-7 / 2`, `-7 % 2`, `(5 < 3) < 2`, and a range-`for` that modifies its loop variable.

## Stage 3: Memory

The part of C++ that Python hides completely, and the source of most C++ bugs. **This stage matters most for Unreal.** Read it slowly, and run the examples under AddressSanitizer.

| # | Note | What it answers | Compare with (Python) |
|---|---|---|---|
| 13 | [Stack and Heap](<Memory/Stack and Heap.md>) | Process memory layout, stack frames, `new`/`delete`, what goes wrong with manual memory | [Stack and queue](<../Python/Fundamentals/Stack and queue.md>) |
| 14 | [Pointers](<Memory/Pointers.md>) | `&` and `*`, `nullptr`, `->`, dangling pointers, arrays and pointer arithmetic, `const` with pointers | |
| 15 | [References](<Memory/References.md>) | Aliases that can't be null or reseated, `const&`, dangling references, lvalues/rvalues, pointers vs references | [Name Rebinding](<../Python/Fundamentals/Name Rebinding.md>) |
| 16 | [Const Correctness](<Memory/Const Correctness.md>) | `const` variables, parameters, member functions, `mutable`, `constexpr` | |
| 17 | [Arrays and Strings](<Fundamentals/Arrays and Strings.md>) | C arrays and decay, `std::array`, `std::span`, C strings, `std::string`, `std::string_view` | [Array vs List](<../Python/Fundamentals/Array vs List.md>), [String Manipulation](<../Python/Fundamentals/String Manipulation.md>) |
| 18 | [Undefined Behavior and Sanitizers](<Memory/Undefined Behavior and Sanitizers.md>) | What UB is, why "it works" means nothing, warnings, reading ASan/UBSan reports | |

**Checkpoint:** you can say, for any variable, where it lives, when it dies, and whether a pointer or reference to it can dangle.

## Stage 4: Classes and Object Lifetime

Classes in C++ are built around **lifetime**: constructors, destructors, copying, and moving. RAII (note 20) is the most important idea in all of these notes.

| # | Note | What it answers | Compare with (Python) |
|---|---|---|---|
| 19 | [Classes and Objects](<OOP/Classes and Objects.md>) | Members, `this`, access control, constructors and initializer lists, static members | [OOP-Self](<../Python/OOP/OOP-Self .md>) |
| 20 | [Object Lifetime and RAII](<OOP/Object Lifetime and RAII.md>) | Destructors, construction/destruction order, RAII, stack unwinding | `with` statements, `__del__` |
| 21 | [Copy and Move Semantics](<OOP/Copy and Move Semantics.md>) | Shallow vs deep copy, rule of three/five/zero, rvalue references, `std::move`, `noexcept` | [List Manipulation](<../Python/Fundamentals/List Manipulation.md#listcopy>) (shallow copies) |
| 22 | [std move](<Memory/std move.md>) | Condensed `std::move` reference: lvalues/rvalues, `T&` vs `const T&` vs `T&&`, the stack-object → heap-buffer layout of `string`/`vector`, copy vs move, what `std::move` doesn't guarantee | Assignment sharing vs `copy.deepcopy` |
| 23 | [Smart Pointers](<Memory/Smart Pointers.md>) | `unique_ptr`, `shared_ptr`, `weak_ptr`, cycles, choosing an ownership model | CPython's reference counting and `gc` |
| 24 | [OOP Four Pillars](<OOP/OOP Four Pillars.md>) | Encapsulation, abstract classes, inheritance, overloading vs overriding, slicing, virtual destructors, `dynamic_cast` | [OOP Four Pillars](<../Python/OOP/OOP Four Pillars.md>) |
| 25 | [Virtual Functions and Vtables](<OOP/Virtual Functions and Vtables.md>) | Static vs dynamic dispatch, how dynamic dispatch works (vptr → vtable → function), its costs, virtual calls in constructors | |
| 26 | [Object Memory Layout](<Memory/Object Memory Layout.md>) | Alignment, padding, member ordering, unions, the vptr's cost, cache lines, AoS vs SoA | [List Structure](<../Python/Fundamentals/List Structure.md>) |
| 27 | [Operator Overloading](<OOP/Operator Overloading.md>) | `operator==`, `<=>`, `<<`, `+=`/`+`, `[]`, `()`, `explicit operator bool` | [OOP-dunder methods](<../Python/OOP/OOP-dunder methods.md>) |
| 28 | [Function Specifiers](<OOP/Function Specifiers.md>) | Where `static`, `virtual`, `const`, `&`/`&&`, `noexcept`, `override`, `= 0`/`= default`/`= delete` go, what each zone means, and which keywords an out-of-class definition repeats | `@staticmethod`, `@abstractmethod`, `@typing.override` |

**Checkpoint:** you can explain what breaks when a class with a raw owning pointer is copied, the order in which `Car : Vehicle` with two members is constructed and destroyed, and which keywords in `virtual int get() const noexcept override;` the `.cpp` definition repeats.

## Stage 5: Generic Code and the Standard Library

| # | Note | What it answers | Compare with (Python) |
|---|---|---|---|
| 29 | [Templates](<Fundamentals/Templates.md>) | Function and class templates, instantiation, why they live in headers, concepts | Duck typing |
| 30 | [Kinds of Classes](<OOP/Kinds of Classes.md>) | Class vs class template vs instantiation, what a function template is and how it differs from a class template, members of templates (`virtual` or not), abstract/`final`/nested classes, why `enum class` isn't a class | `Generic[T]`, `ABC`, `Enum` |
| 31 | [STL Containers](<Standard Library/STL Containers.md>) | `vector` (size/capacity, invalidation), `unordered_map`, `map`, sets, `deque`, `stack`/`queue`, `priority_queue` | [List Structure](<../Python/Fundamentals/List Structure.md>), [Hash Maps Structure](<../Python/Fundamentals/Hash Maps Structure.md>), [Heap Data Structure](<../Python/Fundamentals/Heap Data Structure.md>), [Collections](<../Python/Standard Library/Collections.md>) |
| 32 | [Lambdas](<Fundamentals/Lambdas.md>) | Syntax, captures by copy/reference, `mutable`, dangling captures, `std::function` | [Lambda In-line Function](<../Python/Fundamentals/Lambda In-line Fucntion.md>) |
| 33 | [Iterators and Algorithms](<Standard Library/Iterators and Algorithms.md>) | `begin`/`end`, how range-`for` works, `<algorithm>`, remove-erase, ranges and views | [Iterables](<../Python/Fundamentals/Iterables.md>), [Bisect](<../Python/Standard Library/Bisect.md>) |
| 34 | [Error Handling](<Fundamentals/Error Handling.md>) | Exceptions, RAII instead of `finally`, `noexcept`, `optional`, `expected`, `assert`, and how Unreal does it without exceptions | `try` / `except` |

**Checkpoint:** you can say which of `Box`, `Box<int>`, and `enum class AlertState` are types and which are classes, why a member function template can't be `virtual`, and why `std::vector<std::unique_ptr<Enemy>>` is a common choice, and why `ages["Leo"]` on a missing key doesn't raise an error.

## Stage 6: Concurrency

Threads share memory, so this stage builds directly on Stage 3 (UB) and Stage 4 (RAII). Read it before any code that runs work off the game thread.

| # | Note | What it answers | Compare with (Python) |
|---|---|---|---|
| 35 | [Mutexes and Locks](<Concurrency/Mutexes and Locks.md>) | `std::thread`, data races vs race conditions, ThreadSanitizer, `std::mutex`, `lock_guard` / `scoped_lock` / `unique_lock`, critical-section scope, deadlock and how to avoid it, `atomic`, Unreal's `FCriticalSection` / `FScopeLock` | `threading.Lock`, `with lock:`, the GIL |

**Checkpoint:** you can explain why `++counter` from two threads loses updates, why a value read after a `lock_guard`'s scope ends is unprotected, and why two threads calling `transfer(x, y)` and `transfer(y, x)` can hang.

---

## How This Lines Up With the Learning Plan

| Plan milestone | Read first |
|---|---|
| **0.2a Warm-up drills** (references, `const`, virtual dispatch, virtual-destructor leak) | Stages 1–3, plus notes 19, 20, 24, 25 |
| **0.2b RAII wrapper** (file handle, destructor on every exit path) | 20 [Object Lifetime and RAII](<OOP/Object Lifetime and RAII.md>), 34 [Error Handling](<Fundamentals/Error Handling.md>) (stack unwinding) |
| **0.2c Dynamic array** (ctor/dtor → copy → move → rule of five → template) | 21 [Copy and Move Semantics](<OOP/Copy and Move Semantics.md>), 22 [std move](<Memory/std move.md>), 29 [Templates](<Fundamentals/Templates.md>), and the `vector` section of 31 [STL Containers](<Standard Library/STL Containers.md>) |
| **0.5 Project tour** (`Build.cs`, modules, generated headers) | 3 [Compilation and Linking](<Fundamentals/Compilation and Linking.md>), 4 [Preprocessor](<Fundamentals/Preprocessor.md>), 5 [Headers and Source Files](<Fundamentals/Headers and Source Files.md>) |

The notes explain the concepts, but deliberately don't solve the 0.2b and 0.2c exercises. The examples use different classes (`ScopedTimer`, `Texture`) from the ones the plan asks you to build.

Your 0.1 diagnostic flagged these topics. If you're short on time, these notes are the priority:

- Pointers vs references: [References](<Memory/References.md>) (the three rules, and the comparison table at the end)
- `const` with pointers and `const` member functions: [Const Correctness](<Memory/Const Correctness.md>)
- `virtual` and vtables: [Virtual Functions and Vtables](<OOP/Virtual Functions and Vtables.md>)
- Destructors and RAII: [Object Lifetime and RAII](<OOP/Object Lifetime and RAII.md>)
- Move semantics and templates (not yet seen in the diagnostic): [Copy and Move Semantics](<OOP/Copy and Move Semantics.md>), [Templates](<Fundamentals/Templates.md>)

> **macOS note for 0.2a:** AddressSanitizer's leak detection isn't supported with Apple's toolchain (`detect_leaks is not supported on this platform`), so ASan alone won't report the virtual-destructor leak. Clang's `-Wall` does warn about it when the base class has virtual functions, and `leaks --atExit -- ./program` or Xcode Instruments' Leaks tool will find the leaked memory. Details in [OOP Four Pillars](<OOP/OOP Four Pillars.md#44-virtual-destructors>).

## How to Practice While Reading

The outputs, compiler error messages, and sizes quoted in these notes come from real runs with Apple Clang 21 on an Apple Silicon Mac, so they should match what you see. Most snippets are fragments: wrap them in `main()` with the right `#include`s to run them. To experiment:

```bash
clang++ -std=c++20 -Wall -Wextra -g -O0 -fsanitize=address,undefined scratch.cpp -o scratch && ./scratch
```

- Predict the output **before** running, especially for the trap examples.
- Change one thing and run again: remove a `&`, a `const`, a `virtual`, or a `std::move`, and see what changes.
- Use `-std=c++23` for the few C++23 examples (`std::print`, `std::expected`, `std::views::zip`).
