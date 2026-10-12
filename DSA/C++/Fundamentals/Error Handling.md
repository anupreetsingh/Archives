# Error Handling in C++

C++ has several ways to report that something went wrong, and choosing between them is part of designing a function:

| Mechanism | Use for | Python analogue |
|---|---|---|
| **Exceptions** (`throw` / `try` / `catch`) | Unexpected failures the immediate caller usually can't handle | `raise` / `try` / `except` |
| **`std::optional<T>`** | "There may be no result" when that's a normal outcome | Returning `None` |
| **`std::expected<T, E>`** (C++23) / error codes | Failures the caller is expected to check | Returning `(value, error)`, or a result object |
| **`assert`** / `static_assert` | Programmer mistakes that should never happen | `assert` |

And one category with no mechanism at all: **undefined behavior** (see [Undefined Behavior and Sanitizers](<../Memory/Undefined Behavior and Sanitizers.md>)). Out-of-bounds `[]`, null dereferences, and the like don't raise anything. They're bugs to prevent, not errors to catch.

---

## 1. Exceptions

### Throwing and Catching

The structure mirrors Python's `try` / `except`:

```cpp
#include <stdexcept>

double divide(double a, double b) {
    if (b == 0) {
        throw std::invalid_argument("division by zero");   // like: raise ValueError("division by zero")
    }
    return a / b;
}

try {
    divide(1, 0);
} catch (const std::invalid_argument& e) {                  // like: except ValueError as e:
    std::cout << "invalid_argument: " << e.what() << "\n";  // e.what() is like str(e)
}
// invalid_argument: division by zero
```

| Python | C++ |
|---|---|
| `raise ValueError("msg")` | `throw std::invalid_argument("msg");` |
| `except ValueError as e:` | `catch (const std::invalid_argument& e)` |
| `str(e)` | `e.what()` |
| `except Exception:` | `catch (const std::exception& e)` |
| `except:` (bare) | `catch (...)` |
| `raise` (re-raise) | `throw;` |
| `finally:` | **No `finally`**: use RAII (section 2) |
| `else:` | No equivalent: put the code at the end of the `try` block |

### Catch Order and Catch by `const&`

Handlers are tried **top to bottom**, and the first matching one wins. List specific types first:

```cpp
try {
    std::vector<int> v(3);
    v.at(5);                                      // throws std::out_of_range
} catch (const std::out_of_range& e) {            // specific first
    std::cout << "out_of_range: " << e.what();    // out_of_range: vector
} catch (const std::exception& e) {               // general fallback
    std::cout << "something else";
}
```

Always catch by **`const` reference**. Catching by value copies the exception, and **slices** a derived exception into its base type, losing its information (see [OOP Four Pillars](<../OOP/OOP Four Pillars.md#43-polymorphism-needs-pointers-or-references-object-slicing>)).

### The Standard Exception Hierarchy

All standard exceptions derive from `std::exception` (in `<exception>`; most of the derived types are in `<stdexcept>`):

```text
std::exception
 ├── std::logic_error            bugs that could have been prevented
 │    ├── std::invalid_argument     std::stoi("abc")
 │    ├── std::out_of_range         vector::at, map::at, string::at, std::stoi("99999999999")
 │    └── std::length_error
 ├── std::runtime_error          problems only detectable at runtime
 │    ├── std::range_error
 │    └── std::overflow_error
 ├── std::bad_alloc              new couldn't allocate memory (like MemoryError)
 └── std::bad_cast               dynamic_cast on a reference failed
```

### Custom Exceptions

Derive from a standard exception, as Python code derives from `Exception`:

```cpp
class SaveError : public std::runtime_error {
public:
    SaveError(const std::string& msg, int slot)
        : std::runtime_error(msg), slot_(slot) {}
    int slot() const { return slot_; }
private:
    int slot_;
};

try {
    throw SaveError("disk full", 2);
} catch (const SaveError& e) {
    std::cout << e.what() << " slot " << e.slot();   // disk full slot 2
}

try {
    throw SaveError("x", 1);
} catch (const std::exception& e) {                  // a base-class handler also catches it
    std::cout << "caught as base: " << e.what();
}
```

C++ can technically throw any type (`throw 42;` is caught by `catch (int code)`), but always throw classes derived from `std::exception`.

### Rethrowing

```cpp
try {
    load_level();
} catch (const std::exception& e) {
    log_error(e.what());
    throw;               // rethrow the SAME exception object. Not `throw e;`, which copies and may slice
}
```

### Uncaught Exceptions Terminate the Program

If no handler matches, the program ends, much like an uncaught Python exception, but without a traceback:

```text
libc++abi: terminating due to uncaught exception of type std::runtime_error: nobody catches me
exit code 134   (128 + SIGABRT)
```

## 2. No `finally`: RAII Does the Cleanup

When an exception propagates out of a scope, C++ **unwinds the stack** and runs the destructor of every local object on the way out (see [Object Lifetime and RAII](<../OOP/Object Lifetime and RAII.md#5-raii-and-exceptions-stack-unwinding>)). So instead of writing cleanup in a `finally` block, you put it in a destructor, once, and it runs on **every** exit path:

```python
f = open("save.dat", "w")
try:
    write_data(f)
finally:
    f.close()
```

```cpp
{
    std::ofstream f("save.dat");   // closes itself in its destructor
    write_data(f);                 // if this throws, f is still closed during unwinding
}
```

That's why exceptions and RAII are designed together. Code that owns resources only through RAII objects is automatically **exception-safe**: it can't leak, whatever throws.

## 3. `noexcept`

`noexcept` declares that a function never throws:

```cpp
void swap_buffers(Buffer& a, Buffer& b) noexcept;
```

- It's a promise to callers and to the optimizer. Containers use it to choose cheap moves over copies (see [Copy and Move Semantics](<../OOP/Copy and Move Semantics.md#7-why-noexcept-matters-on-move-operations>)).
- If a `noexcept` function does throw, the program calls `std::terminate` immediately. Clang warns when it can see this: `'f' has a non-throwing exception specification but can still throw`.
- Destructors are `noexcept` by default.

Mark move operations, swaps, and destructors `noexcept`. Don't mark ordinary functions that could fail.

### What Happens at the `noexcept` Boundary

Follow an exception thrown inside a function, for example `std::bad_alloc` from a `push_back` or `emplace_back` that has to reallocate and can't get memory:

```text
Without noexcept:  throw -> unwind out of the function (destructors run) -> caller's catch, if any -> ... -> main -> std::terminate
With noexcept:     throw -> reaches the function's edge -> std::terminate
```

Without `noexcept`, the exception reaches the caller, which can catch it and carry on. A `std::vector` is left unchanged when a reallocation fails, so the object is still usable afterwards. Only if nothing up the stack catches it does the program terminate.

With `noexcept`, the exception stops at the function's edge. A `try`/`catch` around the call never sees it, and whether destructors run before `std::terminate` is implementation-defined.

### Writing Code Inside a `noexcept` Function

`noexcept` also restricts the function's own body: it can never report an error by throwing. Every failure it can foresee needs a non-throwing design:

```cpp
std::int64_t min(std::int64_t t) noexcept {
    if (empty_window(t))
        throw std::runtime_error("empty window");   // std::terminate
    ...
}
```

The alternatives are to return a sentinel value (such as `std::numeric_limits<std::int64_t>::max()`), return a `std::optional` (section 4), or document a precondition the caller must check first ("call only when `count(t) > 0`").

Failures the code can't sensibly handle, such as running out of memory inside a container, are usually accepted as fatal in a `noexcept` function. When an interface is declared `noexcept`, that is the designer saying allocation failure is outside the expected workload and the program may crash if it happens. Unreal takes the same stance: exceptions are disabled (section 8), and running out of memory ends the program with a fatal error.

## 4. `std::optional`: "No Result" Isn't an Error

When "nothing found" is a normal outcome, throwing is wrong: exceptions are slow when thrown, and they force the caller into `try`/`catch` for something routine. Python returns `None` in this case. C++ returns a `std::optional<T>`, which holds either a `T` or nothing:

```cpp
#include <optional>

std::optional<int> find_index(const std::vector<int>& v, int target) {
    for (std::size_t i = 0; i < v.size(); ++i) {
        if (v[i] == target) return static_cast<int>(i);
    }
    return std::nullopt;                       // like: return None
}

if (auto idx = find_index({4, 5, 6}, 5)) {     // converts to true if it holds a value
    std::cout << *idx;                         // 1: * reads the value
}

auto none = find_index({4, 5, 6}, 9);
none.has_value()       // false
none.value_or(-1)      // -1: like Python's `result if result is not None else -1`
none.value()           // throws std::bad_optional_access
*none                  // UB: no check
```

The type itself says "may be empty", where an `int` return with `-1` meaning "not found" relies on documentation. That's the same benefit Python's `Optional[int]` type hint describes (see [Function Syntax](<../../Python/Fundamentals/Function Syntax.md#type-hints>)), except that C++ enforces it.

## 5. `std::expected`: Result or Error (C++23)

When the caller needs to know **why** something failed, and failure is common enough that it should be handled locally, `std::expected<T, E>` holds either a value `T` or an error `E`:

```cpp
#include <expected>

enum class ParseError { Empty, NotANumber };

std::expected<int, ParseError> parse(const std::string& s) {
    if (s.empty()) return std::unexpected(ParseError::Empty);
    try {
        return std::stoi(s);
    } catch (...) {
        return std::unexpected(ParseError::NotANumber);
    }
}

auto p = parse("42");
auto q = parse("abc");

*p                                     // 42
q.has_value()                          // false
q.error() == ParseError::NotANumber    // true
```

Before C++23, the same idea was written as an error code plus an output parameter (`bool parse(const std::string& s, int& out);`), or as a returned struct. That's still common in C APIs and in game code.

## 6. Assertions: Catching Bugs, Not Errors

An **assertion** checks something that must be true if the code is correct. A failure means a bug, so the right reaction is to stop immediately and fix the code, not to recover:

```cpp
#include <cassert>

void apply_damage(int health) {
    assert(health > 0 && "health must be positive");   // the string shows up in the failure message
}
```

```text
Assertion failed: (health > 0 && "health must be positive"), function main, file as.cpp, line 2.
exit code 134
```

`assert` is a macro (see [Preprocessor](<Preprocessor.md#5-conditional-compilation>)) that's compiled out entirely when `NDEBUG` is defined, as it usually is in release builds. So never put code with side effects inside an `assert`.

`static_assert(condition, "message")` checks at **compile time** instead (see [Templates](<Templates.md#8-compile-time-checks-static_assert>)).

## 7. Choosing a Mechanism

```text
Could this only happen because of a bug in MY code?
 ├─ Yes -> assert (or Unreal's check / ensure)
 └─ No  -> Is "no result" a normal, expected outcome?
            ├─ Yes -> std::optional<T>
            └─ No  -> Should the immediate caller handle it?
                       ├─ Yes -> std::expected<T, E> or an error code
                       └─ No  -> throw an exception (it travels up to whoever can handle it)
```

Python uses exceptions for almost everything, including normal control flow (`StopIteration`, `KeyError` in an EAFP-style `try`). C++ reserves exceptions for **exceptional** situations. Throwing is cheap to set up and costs nothing when nothing is thrown, but actually throwing is expensive (often microseconds).

## 8. Exceptions in Unreal Engine

> **Unreal Engine compiles with C++ exceptions disabled by default.** With exceptions off, `throw` doesn't even compile:
>
> ```text
> error: cannot use 'throw' with exceptions disabled
> ```
>
> Unreal code reports errors in other ways instead:
>
> - Return values: `bool` success flags, `nullptr` from `Cast<T>()` and `FindComponentByClass<T>()`, `TOptional<T>` (Unreal's `std::optional`), and `TValueOrError<T, E>` (similar to `std::expected`).
> - `check(Condition)`: like `assert`. Crashes with a message and callstack in development builds.
> - `ensure(Condition)`: logs an error with a callstack the first time it fails, then **continues**. Use it for "this shouldn't happen, but the game can carry on".
> - `UE_LOG(LogTemp, Error, TEXT("..."))`: log and handle locally.
>
> The RAII habits from this note still apply in full. Destructors are how Unreal code avoids leaks too, with or without exceptions.
