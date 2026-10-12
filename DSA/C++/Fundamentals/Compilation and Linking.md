# Compilation and Linking in C++

This note follows a C++ program from source files to a running executable: what each stage of the build takes in, what it produces, and which kinds of errors come from which stage.

As covered in [What is C++](<What is C++.md#1-how-python-runs-vs-how-c-runs>), all of this happens **before** the program runs, unlike CPython, which compiles to bytecode on the fly.

## The Pipeline at a Glance

```text
 main.cpp   math_utils.cpp        <- source files you write (+ the headers they #include)
    |             |
    v             v
 [1] Preprocessor                 pastes #includes, expands macros
    |             |
    v             v
 main.ii    math_utils.ii         <- "translation units": one big file per .cpp
    |             |
    v             v
 [2] Compiler                     parses, type-checks, optimizes, generates assembly
    |             |
    v             v
 main.s     math_utils.s          <- assembly for the target CPU
    |             |
    v             v
 [3] Assembler                    turns assembly text into machine-code bytes
    |             |
    v             v
 main.o     math_utils.o          <- object files: machine code + a symbol table
     \           /
      v         v
 [4] Linker                       stitches object files + libraries together, resolves symbols
          |
          v
         app                      <- the executable
          |
          v
 [5] OS loader (at run time)      loads app + any dynamic libraries into memory, jumps to main()
```

Each `.cpp` file goes through stages 1 to 3 **on its own**, completely independently of every other `.cpp` file. Only the linker in stage 4 ever sees them together. That one fact explains most of the rules in [Headers and Source Files](<Headers and Source Files.md>).

`clang++ main.cpp math_utils.cpp -o app` runs all four stages in one command. The sections below run them one at a time, using this two-file program:

```cpp
// math_utils.h
#pragma once
int add(int a, int b);              // declaration only: "add exists somewhere"
```

```cpp
// math_utils.cpp
#include "math_utils.h"
int add(int a, int b) {             // definition: the actual code
    return a + b;
}
```

```cpp
// main.cpp
#include <iostream>
#include "math_utils.h"

int main() {
    std::cout << add(2, 3) << "\n"; // 5
}
```

## 1. Preprocessing

The **preprocessor** handles every line that starts with `#`. It is a text-substitution tool and knows nothing about C++ types or syntax.

- `#include "math_utils.h"` is replaced by the full text of `math_utils.h`.
- `#include <iostream>` is replaced by the full text of the `<iostream>` header, plus everything **that** header includes, recursively.
- Macros defined with `#define` are expanded.

The output is a **translation unit** (TU): a single, self-contained stream of C++ code with no `#` directives left in it.

```bash
clang++ -std=c++20 -E main.cpp -o main.ii   # -E: stop after preprocessing
wc -l main.ii
# 83712 main.ii
```

The 5-line `main.cpp` became an 83,712-line translation unit, almost all of it from `<iostream>`. This is why heavy includes make C++ builds slow, and why Unreal Engine code tries hard to keep headers small.

Details in [Preprocessor](<Preprocessor.md>).

## 2. Compiling

The **compiler** proper takes one translation unit and:

1. Parses it and checks it against the language rules: syntax, types, and whether every name used has been declared.
2. Optimizes it (if optimization is on).
3. Generates **assembly** for the target CPU.

```bash
clang++ -std=c++20 -O1 -S math_utils.cpp -o math_utils.s   # -S: stop after generating assembly
```

On an Apple Silicon (arm64) Mac, the body of `add` becomes:

```text
__Z3addii:
    add  w0, w1, w0     ; w0 = b + a   (arguments arrive in registers w0 and w1)
    ret                 ; return; the result is left in w0
```

A whole C++ function compiled down to two CPU instructions. Nothing is looked up or interpreted at runtime.

**Compile errors** come from this stage. Since the compiler only sees one TU at a time, it can only check what has been **declared** inside that TU:

```cpp
// nodecl.cpp
int main() { return square(3); }      // square has not been declared yet at this point
int square(int x) { return x * x; }
```

```text
nodecl.cpp:1:20: error: use of undeclared identifier 'square'
```

The compiler reads top to bottom, and at line 1 it has never heard of `square`.

### Inside the Compiler: Front End, IR, Back End

"The compiler" is really three parts in a row, connected by an **intermediate representation (IR)**. With clang, the IR is **LLVM IR**:

```text
 translation unit (main.ii)
    |
    v
 FRONT END (clang)       lex -> parse -> AST -> semantic checks (types, overloads, templates)
    |
    v
 LLVM IR                 a simple, low-level, mostly CPU-independent language
    |
    v
 MIDDLE END (LLVM)       optimization passes on the IR: inlining, constant folding,
    |                    dead code removal, loop optimizations...
    v
 BACK END (LLVM)         instruction selection -> register allocation -> scheduling
    |                    for one specific CPU (arm64, x86-64, ...)
    v
 assembly (main.s)
```

1. **Front end**: understands C++. It turns the text into an **AST** (abstract syntax tree) and checks it against the language rules. Almost every error you see while learning (missing `;`, type mismatch, undeclared name) comes from here, before any IR exists.
2. **IR**: the front end lowers the AST into LLVM IR. The C++ features (classes, templates, overloading) are gone by now. What's left is functions, typed values, loads, stores, and calls.
3. **Middle end**: the optimizer rewrites the IR into faster IR that does the same thing. The passes don't care which CPU will run the result, so the same optimizer serves every language and target LLVM supports (C, C++, Rust, Swift...).
4. **Back end**: maps the IR onto one CPU's real instructions and registers, and writes the assembly.

This split is why one optimizer and one set of back ends can be shared by many languages: a new language only needs a front end that emits IR, and a new CPU only needs a back end that consumes it.

You can stop after the IR step with `-emit-llvm`:

```cpp
// sq.cpp
int square(int x) { return x * x; }
int main() { return square(4); }
```

```bash
clang++ -std=c++20 -S -emit-llvm -O0 sq.cpp -o sq_O0.ll   # IR, no optimization
clang++ -std=c++20 -S -emit-llvm -O2 sq.cpp -o sq_O2.ll   # IR after the middle end
```

At `-O0`, the IR is a literal translation: `x` is stored to memory and loaded back, and `main` really calls `square`:

```llvm
define i32 @_Z6squarei(i32 noundef %0) {
  %2 = alloca i32, align 4          ; reserve stack space for x
  store i32 %0, ptr %2, align 4     ; x = argument
  %3 = load i32, ptr %2, align 4    ; read x
  %4 = load i32, ptr %2, align 4    ; read x again
  %5 = mul nsw i32 %3, %4           ; x * x
  ret i32 %5
}

define i32 @main() {
  ...
  %2 = call i32 @_Z6squarei(i32 noundef 4)
  ret i32 %2
}
```

At `-O2`, the middle end has removed the pointless memory traffic, inlined `square` into `main`, and computed `4 * 4` at compile time:

```llvm
define i32 @_Z6squarei(i32 noundef %0) {
  %2 = mul nsw i32 %0, %0
  ret i32 %2
}

define noundef i32 @main() {
  ret i32 16                        ; the call is gone: the answer was worked out while compiling
}
```

`@_Z6squarei` is the mangled name of `square(int)` (see [Name Mangling](#name-mangling) below).

> The IR is **mostly**, not fully, CPU-independent. The top of every `.ll` file has a `target triple` (here `arm64-apple-macosx`) and a `target datalayout` (pointer size, alignment). The C++ front end has already made platform-specific decisions such as struct sizes and padding, so IR built for an arm64 Mac can't simply be compiled for x86-64 Windows. The **optimizations** are generic; the IR itself is not portable.

## 3. Assembling

The **assembler** converts the assembly text into machine-code bytes and packages them into an **object file** (`.o` on macOS/Linux, `.obj` on Windows).

```bash
clang++ -std=c++20 -c main.cpp -o main.o              # -c: compile + assemble, don't link
clang++ -std=c++20 -c math_utils.cpp -o math_utils.o
```

An object file is not runnable yet. Besides machine code, it contains a **symbol table** listing:

- Symbols this file **defines** (code or data that lives in this file)
- Symbols this file **uses but doesn't define** (to be found somewhere else later)

`nm` prints the symbol table:

```bash
nm main.o
#                  U __Z3addii      <- U = Undefined: main.o calls add but doesn't contain it
# 0000000000000000 T _main          <- T = defined in this file's code (Text) section

nm math_utils.o
# 0000000000000000 T __Z3addii      <- math_utils.o defines add
```

### Name Mangling

`__Z3addii` is the **mangled** name of `int add(int, int)`. The compiler encodes the parameter types into the symbol name:

```text
_Z  3add  i  i
|   |     |  |
|   |     |  +-- second parameter: int
|   |     +----- first parameter: int
|   +----------- name with length prefix: 3 characters, "add"
+--------------- "this is a mangled C++ name"

(macOS adds one extra leading underscore to every symbol, giving __Z3addii)
```

Mangling is what makes **function overloading** possible. If `math_utils.cpp` also defines `double add(double, double)`, it gets a different symbol, `__Z3adddd`, so both can coexist. Python has no overloading because a later `def` simply rebinds the same name (see [Name Rebinding](<../../Python/Fundamentals/Name Rebinding.md>)). In C++, the two functions really are two different symbols. See [Functions](<Functions.md>).

`c++filt` demangles names back into readable form:

```bash
nm math_utils.o | c++filt
# 0000000000000020 T add(double, double)
# 0000000000000000 T add(int, int)
```

## 4. Linking

The **linker** (`ld` on macOS) takes all the object files plus any libraries, and:

1. Collects all defined symbols from every input.
2. Matches every **undefined** symbol (`U`) to exactly one definition.
3. Patches each call site with the final address of its target.
4. Writes out the executable.

```bash
clang++ main.o math_utils.o -o app
./app
# 5
```

Conceptually:

```text
main.o                          math_utils.o
  _main:  ... call ???  ---------->  __Z3addii: add w0, w1, w0
          (needs __Z3addii)            (defines __Z3addii)

                   after linking: call 0x100000460  (the real address of add)
```

### Linker Errors

Linker errors look scarier than compile errors because they don't point at a line of code. There are two common kinds.

**Undefined symbol**: something was declared and used, but no object file defines it.

```bash
clang++ main.o -o app      # forgot math_utils.o
```

```text
Undefined symbols for architecture arm64:
  "add(int, int)", referenced from:
      _main in main.o
ld: symbol(s) not found for architecture arm64
```

Each compiler phrases it differently:

| Toolchain | Message |
|---|---|
| macOS `ld` | `Undefined symbols for architecture arm64` |
| GNU `ld` (Linux) | `undefined reference to 'add(int, int)'` |
| MSVC (Windows) | `LNK2019: unresolved external symbol` |

Typical causes: a `.cpp` file wasn't added to the build, a function was declared but never defined, the definition's signature differs from the declaration (`add(int, int)` vs `add(long, long)` are different symbols), or a library wasn't linked.

> In Unreal Engine, an unresolved external symbol very often means the module you are calling into isn't listed in your module's `Build.cs` dependencies, or the class you're using wasn't exported from its module with an `_API` macro.

**Duplicate symbol**: two object files both define the same symbol.

```cpp
// other.cpp
int add(int a, int b) { return a - b; }   // a second definition of add(int, int)
```

```text
duplicate symbol 'add(int, int)' in:
    math_utils.o
    other.o
ld: 1 duplicate symbols
```

This is the **One Definition Rule** being enforced. It usually happens when a function **definition** is put in a header that is included by more than one `.cpp` file. See [Headers and Source Files](<Headers and Source Files.md#the-one-definition-rule-odr>).

## 5. Loading and Running

The executable is a native binary for one CPU and OS:

```bash
file app
# app: Mach-O 64-bit executable arm64
```

When you run it, the OS **loader** maps it into memory, also loads any **dynamic libraries** it depends on, and jumps to `main()`:

```bash
otool -L app        # list dynamic library dependencies (ldd on Linux)
# /usr/lib/libc++.1.dylib       <- the C++ standard library implementation
# /usr/lib/libSystem.B.dylib    <- macOS's C library and system calls
```

## Static vs Dynamic Libraries

A **library** is a bundle of already-compiled object files that you link against instead of recompiling.

| | Static library | Dynamic (shared) library |
|---|---|---|
| File | `.a` (macOS/Linux), `.lib` (Windows) | `.dylib` (macOS), `.so` (Linux), `.dll` (Windows) |
| When it's joined to your code | At link time: the needed code is **copied into** your executable | At load/run time: the executable keeps a **reference** and the loader finds the library |
| Executable size | Larger | Smaller |
| Update the library | Must relink | Swap the library file (if compatible) |

> Unreal Editor builds each engine and game **module** as a separate dynamic library, which is how the editor can load your game module on top of an engine it didn't recompile.

## Incremental Builds

Because each `.cpp` compiles independently, a build tool only needs to recompile the `.cpp` files that changed, then relink:

```text
edit math_utils.cpp  ->  recompile math_utils.o only  ->  relink app
edit math_utils.h    ->  recompile EVERY .cpp that #includes it (directly or indirectly)  ->  relink
```

This is why changing a widely included header triggers long rebuilds, and why headers should include as little as possible (see forward declarations in [Headers and Source Files](<Headers and Source Files.md#forward-declarations>)).

## Useful Compiler Flags

```bash
clang++ -std=c++20 -Wall -Wextra -g -O0 main.cpp math_utils.cpp -o app
```

| Flag | Meaning |
|---|---|
| `-std=c++20` | Which C++ standard to compile against |
| `-o app` | Name of the output file |
| `-Wall -Wextra` | Turn on most useful warnings. Always use these when learning |
| `-Werror` | Treat every warning as an error |
| `-g` | Include debug info so a debugger can map machine code back to your source lines |
| `-O0` / `-O2` / `-O3` | Optimization level: none (easy to debug) through aggressive (fast) |
| `-E` / `-S` / `-c` | Stop after preprocessing / compiling to assembly / assembling |
| `-S -emit-llvm` | Stop after generating LLVM IR (a `.ll` file) instead of assembly |
| `-I path` | Add a directory to search for `#include` files |
| `-fsanitize=address,undefined` | Add runtime checks for memory errors and UB (see [Undefined Behavior and Sanitizers](<../Memory/Undefined Behavior and Sanitizers.md>)) |

**Debug** vs **Release** builds are just different flag sets. Debug is `-O0 -g`: slow but every variable is inspectable in a debugger. Release is `-O2` or `-O3`: fast, but the optimizer reorders and removes code, so stepping through it in a debugger is confusing.

> Unreal's **DebugGame** configuration compiles your game module unoptimized and the engine optimized, while **Development** optimizes both. That's why breakpoints in your own code behave better in DebugGame.

## Build Systems

Typing compiler commands by hand stops scaling after a few files. A **build system** tracks which files depend on which, and reruns only what's needed:

- **Make**: rules in a `Makefile`.
- **CMake**: the most common cross-platform C++ build generator. It reads a `CMakeLists.txt` and writes Makefiles, Ninja files, Xcode projects, or Visual Studio solutions.
- **UnrealBuildTool (UBT)**: Unreal Engine's own build system. It reads each module's `Build.cs` for dependencies and settings, then drives the platform compiler.

Unreal adds one extra stage **before** preprocessing: **UnrealHeaderTool (UHT)** scans your headers for macros like `UCLASS()` and `UPROPERTY()` and generates extra C++ code (the `*.generated.h` files) that the normal pipeline then compiles along with your code.

```text
MyActor.h --(UHT)--> MyActor.generated.h --> preprocessor --> compiler --> assembler --> linker
```
