# Program Structure in C++

This note walks through the smallest complete C++ program line by line, then covers the pieces every program uses: `main()`, statements, blocks, comments, and console input/output.

## Hello World, Line by Line

```cpp
// hello.cpp
#include <iostream>

int main() {
    std::cout << "Hello, World!" << "\n";
    return 0;
}
```

Here:

- `// hello.cpp` is a comment. The compiler ignores it.
- `#include <iostream>` is a **preprocessor directive**. Before compilation, it pastes in the contents of the standard `<iostream>` header, which declares `std::cout`. See [Preprocessor](<Preprocessor.md>).
- `int main()` defines a function named `main` that returns an `int`. Every C++ program starts running at `main()`.
- `{ ... }` is the function body. Braces mark blocks of code.
- `std::cout` is the standard output stream (the console). `std::` means "the `cout` that lives in the `std` namespace". See [Scope and Namespaces](<Scope and Namespaces.md>).
- `<<` sends the value on its right into the stream on its left.
- `"\n"` is a newline character.
- `;` ends a statement.
- `return 0;` ends `main()` and hands `0` back to the operating system as the program's **exit code**.

Build and run it:

```bash
clang++ -std=c++20 -Wall -Wextra hello.cpp -o hello   # compile + link into an executable named "hello"
./hello                                                # run it
# Hello, World!
```

Unlike `python hello.py`, there are two separate steps: **build** an executable once, then **run** it as many times as you like. [Compilation and Linking](<Compilation and Linking.md>) explains what `clang++` does in between.

## `main()` and the Exit Code

`main()` is the program's entry point. The operating system calls it, and when it returns, the program ends.

Its return value is the **exit code**. By convention, `0` means success and anything else means failure.

```cpp
int main() {
    return 3;
}
```

```bash
./app
echo $?   # prints the last command's exit code
# 3
```

`main()` is special in one more way: if execution reaches its closing `}` without a `return`, it behaves as if it returned `0`. No other function gets this treatment.

### Command-Line Arguments

`main()` can also receive the command-line arguments, much like Python's `sys.argv`:

```cpp
#include <iostream>

int main(int argc, char* argv[]) {
    // argc: argument count, including the program name itself
    // argv: array of C-style strings (see Arrays and Strings)
    std::cout << argc << "\n";      // ./app a b  ->  3
    std::cout << argv[0] << "\n";   // ./app
    std::cout << argv[1] << "\n";   // a
}
```

| Python | C++ |
|---|---|
| `sys.argv` | `argv` |
| `len(sys.argv)` | `argc` |
| `sys.argv[0]` | `argv[0]` |

## Statements, Blocks, and Whitespace

In Python, a newline ends a statement and indentation defines blocks. In C++:

- A **semicolon** ends a statement.
- **Braces** `{ }` define a block.
- Whitespace and indentation mean nothing to the compiler. They are only for humans.

```cpp
// These two are identical to the compiler:
if (x > 5) { std::cout << "big"; } else { std::cout << "small"; }

if (x > 5) {
    std::cout << "big";
} else {
    std::cout << "small";
}
```

A missing semicolon is the most common first compile error:

```cpp
int x = 5      // error: expected ';' at end of declaration
int y = 6;
```

Blocks also create **scope**: a variable declared inside `{ }` stops existing at the closing brace. This is a big difference from Python, where `if`/`for` blocks don't create a scope. See [Scope and Namespaces](<Scope and Namespaces.md>).

## Comments

```cpp
// Single-line comment, like Python's #

/* Multi-line comment.
   Everything until the closing marker is ignored.
   Cannot be nested. */

int x = 5; /* can also sit inside a line */ int y = 6;
```

C++ has no docstrings. Documentation comments are a convention (`///` or `/** */`) that tools such as Doxygen read.

## Console Output

### `std::cout`

`cout` stands for **character output**: every value you send it is turned into characters and written to standard output, which is usually the console.

`<<` can be chained. Each `<<` sends one more value to the stream, left to right:

```cpp
#include <iostream>
#include <string>

int main() {
    std::string name = "Ava";
    int score = 92;

    std::cout << name << " scored " << score << "\n";   // Ava scored 92
}
```

Unlike Python's `print()`, `std::cout` adds **no spaces and no newline** on its own. You add them yourself.

### `"\n"` vs `std::endl`

Both produce a newline. `std::endl` also **flushes** the stream, which forces buffered output to be written immediately. Flushing on every line is slow in loops, so prefer `"\n"` and use `std::endl` only when you need output to appear right away (for example, just before a crash you are debugging).

```cpp
std::cout << "line" << "\n";       // newline only
std::cout << "line" << std::endl;  // newline + flush
```

### `std::cerr`

`std::cerr` writes to the standard error stream, the equivalent of Python's `print(..., file=sys.stderr)`. It is unbuffered, so messages appear immediately.

```cpp
std::cerr << "Error: file not found\n";
```

### `std::format` and `std::print`

C++20 added `std::format`, which works like Python's f-strings / `str.format()` with `{}` placeholders. C++23 added `std::print` and `std::println`, which format and print in one step.

```cpp
#include <format>
#include <print>     // C++23
#include <iostream>
#include <string>

int main() {
    std::string name = "Ava";
    int score = 92;

    std::cout << std::format("{} scored {} ({:.1f}%)\n", name, score, 91.456);
    // Ava scored 92 (91.5%)

    std::println("{} scored {}", name, score);   // println adds the newline for you
    // Ava scored 92
}
```

The format spec after `:` follows the same mini-language as Python: `{:.1f}`, `{:>8}`, `{:08b}`, and so on.

> `std::print` needs `-std=c++23` and a recent standard library. Unreal Engine has its own logging (`UE_LOG`) and string formatting (`FString::Printf`), so you will mostly see `std::cout` in plain C++ practice programs.

## Console Input

`cin` stands for **character input**: it reads characters from standard input, which is usually the keyboard, and converts them into your variable's type.

`std::cin >>` reads one whitespace-separated value and converts it to the variable's type:

```cpp
#include <iostream>
#include <string>

int main() {
    int age;
    std::cout << "Age: ";
    std::cin >> age;          // reads "42" and stores the int 42

    std::string first_name;
    std::cin >> first_name;   // reads one word only: "Ava Smith" gives "Ava"
}
```

To read a whole line including spaces, like Python's `input()`, use `std::getline`:

```cpp
std::string full_name;
std::getline(std::cin, full_name);   // "Ava Smith"
```

### The `>>` then `getline` Trap

`std::cin >> age` stops reading right before the newline you typed. The next `std::getline` sees that leftover newline and immediately returns an empty string:

```cpp
int n;
std::string line;

std::cin >> n;               // input: "42\nhello world\n"  ->  n = 42, "\n" left in the buffer
std::getline(std::cin, line); // line = ""  (consumed only the leftover "\n")
std::getline(std::cin, line); // line = "hello world"
```

Fix it by discarding the rest of the line after `>>`:

```cpp
std::cin >> n;
std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');  // needs #include <limits>
std::getline(std::cin, line);  // "hello world"
```

## Typical File Layout

Bigger programs split code across many files (see [Headers and Source Files](<Headers and Source Files.md>)), but a single-file program usually reads top to bottom like this:

```cpp
// 1. Includes
#include <iostream>
#include <string>

// 2. Declarations and helper definitions
int square(int x);           // declared only: the body comes later

int cube(int x) {            // declared and defined in one step
    return x * square(x);    // OK: square was declared above
}

// 3. Entry point
int main() {
    std::cout << cube(4) << "\n";  // 64
}

// 4. Definitions of functions declared earlier
int square(int x) {
    return x * x;
}
```

A function must be **declared before it is used**. `cube` can call `square` even though `square`'s body appears at the bottom of the file, because the declaration `int square(int x);` above it already told the compiler its name, parameter, and return type. Delete that declaration line and the call inside `cube` fails to compile. Python doesn't need this because it looks names up at runtime. C++ resolves them at compile time while reading the file from top to bottom. See [Functions](<Functions.md>).
