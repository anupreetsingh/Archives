# The C++ Preprocessor

The **preprocessor** is the first stage of the build (see [Compilation and Linking](<Compilation and Linking.md#1-preprocessing>)). It reads each `.cpp` file, carries out every **directive** (a line starting with `#`), and hands the resulting plain text to the compiler.

→ It is a **text-substitution** tool. It does not understand C++ types, scopes, or syntax. It only copies, pastes, replaces, and deletes text.

```text
main.cpp + headers --(preprocessor: #include, #define, #if)--> translation unit --(compiler)--> ...
```

You can see its output with `-E`:

```bash
clang++ -E main.cpp          # print the preprocessed translation unit
clang++ -E -P main.cpp       # same, without the line-marker noise
```

Python has no equivalent stage. `import` is a runtime statement that runs a module and binds a name to a module object (see [Modules and Packages](<../../Python/Fundamentals/Modules and Packages.md#modules>)). `#include` is closer to literally copy-pasting another file's text into yours before compiling.

---

## 1. `#include`

`#include` replaces itself with the entire contents of the named file.

```cpp
#include <iostream>       // angle brackets: search the system / standard library include paths
#include "math_utils.h"   // quotes: search the current file's directory first, then the system paths
```

| Form | Use for | Searched in |
|---|---|---|
| `#include <name>` | Standard library and third-party headers | System include directories, plus directories added with `-I` |
| `#include "name"` | Your own project's headers | Directory of the current file first, then the same places as `<>` |

Because it is pure text pasting, including the same header twice pastes it twice, which can cause redefinition errors. **Header guards** prevent this. See [Headers and Source Files](<Headers and Source Files.md#header-guards-and-pragma-once>).

Standard C++ headers have no `.h` extension: `<iostream>`, `<vector>`, `<string>`. The C library headers are available in two forms, `<cmath>` (C++ style, names in `std::`) and `<math.h>` (C style). Prefer the `<c...>` versions in C++.

## 2. `#define`: Object-Like Macros

A **macro** is a name that the preprocessor replaces with other text before compiling. The word comes from Greek *makros*, "large": you write something short, and it **expands** into something longer, like a keyboard macro that replays a recorded sequence of keystrokes. There are two kinds: **object-like** macros (no parameters, this section) and **function-like** macros (with parameters, [section 3](#3-define-function-like-macros)).

`#define NAME replacement` tells the preprocessor to replace every later occurrence of the token `NAME` with `replacement`:

```cpp
#define PI 3.14159
#define MAX_PLAYERS 4

double area = PI * r * r;         // becomes: double area = 3.14159 * r * r;
int slots[MAX_PLAYERS];           // becomes: int slots[4];
```

Macros have **no type and no scope**.

- **No type**: `PI` is not a `double`. The preprocessor stores it as a sequence of tokens, `3.14159`, and pastes that text in. Types only exist in the compiler, which never sees the name `PI`. It sees the literal `3.14159` and gives *that* the type `double`, exactly as if you had typed it yourself. The replacement doesn't even have to be a value: `#define BEGIN {` is legal.
- **No scope**: once defined, `PI` is replaced everywhere below that line, in every file that includes it, until `#undef PI`. That's why macro names are conventionally ALL_CAPS: to make them hard to collide with by accident.

Because the replacement is pasted as text, an expression can regroup with its surroundings:

```cpp
#define TWO_PLUS_THREE 2 + 3
int x = TWO_PLUS_THREE * 10;      // becomes: int x = 2 + 3 * 10;  → 32, not 50
```

A string macro is just a replacement that includes the quote characters. The compiler then sees an ordinary string literal:

```cpp
#define GREETING "Hello"
std::cout << GREETING;            // becomes: std::cout << "Hello";
```

In modern C++, prefer a typed constant instead:

```cpp
constexpr double Pi = 3.14159;   // typed, scoped, visible to the debugger
constexpr int MaxPlayers = 4;
```

See [Const Correctness](<../Memory/Const Correctness.md>).

## 3. `#define`: Function-Like Macros

A macro can take parameters. The arguments are substituted as **raw text**, which causes classic bugs:

```cpp
#define SQUARE(x) x * x

int a = SQUARE(2 + 3);
// expands to: int a = 2 + 3 * 2 + 3;
// a = 11, not 25, because * binds tighter than +
```

Fix: parenthesize every parameter and the whole body:

```cpp
#define SQUARE(x) ((x) * (x))

int a = SQUARE(2 + 3);   // ((2 + 3) * (2 + 3)) = 25
```

Even then, an argument with a side effect is pasted, and therefore evaluated, more than once:

```cpp
int i = 3;
int r = SQUARE(i++);   // ((i++) * (i++)): i is modified twice. Undefined behavior.
                       // clang warns: multiple unsequenced modifications to 'i'
```

A real function has none of these problems, and the compiler inlines small functions anyway:

```cpp
constexpr int square(int x) { return x * x; }

square(2 + 3);   // 25: the argument is evaluated once, then passed
```

> Rule of thumb: use a function, `constexpr` variable, or template instead of a macro whenever you can. Use macros only for things functions can't do, such as the tricks below.

## 4. Stringizing `#` and Token Pasting `##`

Macros can do two things that ordinary C++ cannot:

```cpp
#define STRINGIFY(x) #x            // # turns the argument's text into a string literal
#define CONCAT(a, b) a##b          // ## glues two tokens into one token

std::cout << STRINGIFY(hello world);   // "hello world"

int CONCAT(health, Max) = 100;         // becomes: int healthMax = 100;
```

A practical use is a debug-print macro that shows the expression's source text along with its value:

```cpp
#define LOG_VALUE(v) std::cout << #v << " = " << (v) << "\n"

int score = 42;
LOG_VALUE(score * 2);   // score * 2 = 84
```

No function could do this, because a function receives only the value `84`, never the source text `score * 2`.

> Unreal Engine relies heavily on macros of this kind. `UE_LOG`, `check`, `UCLASS()`, `UPROPERTY()`, and `GENERATED_BODY()` are all macros. `GENERATED_BODY()` expands to code that UnrealHeaderTool wrote into your class's `.generated.h` file.

## 5. Conditional Compilation

`#if`, `#ifdef`, `#ifndef`, `#elif`, `#else`, and `#endif` keep or delete blocks of text **before** the compiler sees them. Deleted code is never compiled at all, so it can even contain code that wouldn't compile on this platform.

```cpp
#ifdef __APPLE__                 // "if the macro __APPLE__ is defined"
    std::cout << "macOS build\n";
#endif

#if defined(_WIN32)
    std::cout << "Windows\n";
#elif defined(__linux__)
    std::cout << "Linux\n";
#else
    std::cout << "Something else\n";
#endif
```

Common uses:

- **Platform-specific code**: `_WIN32`, `__APPLE__`, `__linux__` are defined by the compiler for the platform being built.
- **Debug-only code**: the standard `assert()` macro is removed when `NDEBUG` is defined (release builds usually define it).
- **Feature switches**: pass `-DENABLE_CHEATS` (or `-DENABLE_CHEATS=1`) on the command line to define a macro without editing code.

```cpp
#ifdef ENABLE_CHEATS
    give_infinite_ammo();
#endif
```

```bash
clang++ -DENABLE_CHEATS main.cpp -o game   # -D defines a macro for this build
```

> In Unreal, the same idea shows up as `#if WITH_EDITOR` (code that only exists in editor builds) and `#if UE_BUILD_SHIPPING`.

### Commenting Out Large Blocks

`/* */` comments cannot be nested, so `#if 0` is a common way to disable a block that already contains comments:

```cpp
#if 0
    old_code();   /* this inner comment is fine */
#endif
```

## 6. Predefined Macros

The compiler defines some macros automatically:

```cpp
std::cout << __FILE__ << "\n";     // macros.cpp           current file name
std::cout << __LINE__ << "\n";     // 19                   current line number
std::cout << __cplusplus << "\n";  // 202002               the C++ standard: 201703 = C++17, 202002 = C++20
std::cout << __func__ << "\n";     // main                 current function (technically a variable, not a macro)
```

`__FILE__` and `__LINE__` are expanded where the macro is **used**, which is why logging and assertion macros can report the caller's location. A function couldn't, because inside the function `__LINE__` would always be the function's own line.

`clang++ -E -dM -x c++ /dev/null` lists every macro the compiler predefines.

### Recognizing Them: Dunder Names

Most predefined macros are **dunder** names (**d**ouble **under**score), like Python's `__name__` and `__file__` (see [`__name__`](<../../Python/Fundamentals/Modules and Packages.md#__name__>)). The idea is the same in both languages: the tool running your code (the compiler here, the interpreter in Python) fills these names in with facts about the situation, and your code reads them but never assigns them.

The C++ rule behind this: names containing a double underscore, or starting with an underscore followed by a capital letter, are **reserved** for the compiler and standard library. So:

- `__APPLE__`, `__LINE__`, `__cplusplus` → almost certainly predefined.
- Some predefined macros use the single-underscore-plus-capital form instead, such as `_WIN32` and `_MSC_VER` (Microsoft's compiler).
- **Never** give your own macros or variables a reserved name. Use plain ALL_CAPS like `ENABLE_CHEATS`.

The difference is *when* they exist. Python's `__name__` is a real `str` variable at runtime. C++'s `__APPLE__` is a text rule that is gone before compilation, and an `#ifdef __APPLE__` block that doesn't match is deleted, not skipped.

> `#ifdef` treats an unknown name as "not defined" with no warning, so a typo like `#ifdef __APPEL__` silently deletes the block. If a platform block seems to be ignored, check the spelling against the `-dM` output.

## 7. Other Directives

| Directive | Purpose |
|---|---|
| `#undef NAME` | Stop the macro `NAME` from being defined from here on |
| `#error "message"` | Stop compilation with a message, e.g. inside an `#else` for unsupported platforms |
| `#pragma once` | Non-standard but universally supported: include this file at most once per translation unit |
| `#pragma ...` | Other compiler-specific instructions (warning control, packing, etc.) |

```cpp
#if !defined(__cplusplus) || __cplusplus < 202002L
    #error "This code needs C++20"
#endif
```

## Summary

```text
#include      -> paste a file's text here
#define       -> find-and-replace text (optionally with parameters)
#if / #ifdef  -> keep or delete blocks of text before compiling
#pragma once  -> paste this header at most once per translation unit
```

The preprocessor runs once per `.cpp` file and produces one translation unit. Everything after it, including type checking, scopes, and overloading, is the compiler's job.
