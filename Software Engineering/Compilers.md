# Compilers

## What a Compiler Does

A **compiler** converts high level language (C, C++, Swift, Julia or Rust) to machine code so that CPU can directly read and execute them.

```text
source code (.c, .cpp, .cxx)
        ↓
     compiler
        ↓
machine code for a target architecture (x86, ARM, PowerPC, RISC-V)
```

Which machine code comes out depends on the CPU **architecture** you are compiling for. The same C++ file produces different assembly on x86 than on ARM.

## Compilers Optimizations

The compiler tries to understand the intent of your code and performs various optimizations to fulfill the same intent using less compute or memory.

Examples of these Optimizations, also called **transformations** or **passes**, could be:

### Constant Folding

Consider:

```c
x = 5;
while (x < 10) {
    x = x + 1;
}
```

The loop exits as soon as the condition fails, so what you actually want is for `x` to end up as `10`. Run literally, this burns CPU cycles setting `x`, checking the condition, incrementing, and checking again. A compiler that understands the intent can simply set `x` to its final value.

This is **constant folding**: computing at compile time what would otherwise be computed at runtime. It makes your program faster without you having to write better code in the first place.

### Function Inlining

When you call a function, the machine normally has to set up a call and jump into it. With **function inlining**, if the function is short enough the compiler takes the body of that function and plops it directly where the call was. In many cases this greatly improves performance.

### Loop Unrolling

Every iteration of a loop costs more than the work inside it: the counter has to be incremented, the condition checked, and the branch taken back to the top. For a short loop that bookkeeping can dominate the actual work.

With **loop unrolling** the compiler writes out several iterations back to back, so that overhead is paid once for a group of iterations instead of once per iteration.

```c
for (int i = 0; i < 8; i++) {
    a[i] = a[i] * 2;
}
```

unrolled by a factor of four becomes the equivalent of:

```c
for (int i = 0; i < 8; i += 4) {
    a[i]     = a[i]     * 2;
    a[i + 1] = a[i + 1] * 2;
    a[i + 2] = a[i + 2] * 2;
    a[i + 3] = a[i + 3] * 2;
}
```

Two checks and two branches instead of eight. The independent statements also sit next to each other, which gives the CPU more it can work on at once.

The trade-off is size: unrolled code is longer, so the compiler weighs the speed gain against the extra instructions, which is part of why optimizing for binary size can turn it off.

## The Problem with Monolithic Compilers

Almost all compilers written in the past were **monolithic**. Each one had its own private set of optimizations and transformations, and only that compiler could ever use them.

If you wanted to introduce a new transformation — insert some instrumentation, or apply a brand new idea for making programs faster in a very specific domain — you were largely out of luck. You had to fight the monolithic nature of the compiler to somehow integrate your passes into it.

## LLVM

The LLVM (Low Level Virtual Machine) compiler architecture introduces a modular approach that builds an architecture-agnostic **LLVM IR** (Intermediate Representation) for applying optimizations. The LLVM IR is then converted to a CPU architecture specific representation.

### The Pipeline

The IR lives in `.ll` files and sits in the middle of the journey from source to binary:

```text
.c file
   ↓
LLVM IR (.ll)  ⇄  transformation passes (run over and over)
   ↓
LLVM backend (per architecture)
   ↓
assembly → machine code
```

The IR is not static. Code is converted to LLVM IR, run through a series of transformations, and comes back out as LLVM IR again, over and over, until LLVM decides it is done optimizing. Only then does a **backend** kick in and turn that platform-generic LLVM code into platform-specific assembly.

So whether you compile for x86, ARM, RISC-V or PowerPC, in most cases you have the *same* LLVM IR, optimized the same way. Only the backend differs.

### Reuse Across Languages

This is why LLVM matters: optimizations are written once and reused across compilers and across languages. Languages built on LLVM include:

- **C/C++** via **clang** (GCC and other C compilers do not use LLVM; clang does)
- **Swift**
- **Rust**
- **Julia**

Julia shows that LLVM is not restricted to ahead-of-time compilation — it can be **just-in-time** compiled as well, which is what enables Julia's powerful in-language features and frameworks like Flux and Zygote.

### Multiple Intermediate Representations

Sometimes it is important to understand the *high level* code in order to apply meaningful, language-specific optimizations.

C is still relatively low level, so going from C to LLVM IR decomposes structure only a little. But in a high-level language like Swift you give up a lot of control to the compiler — you usually do not manage memory manually, you hand it to automatic reference counting. Because of that, the resulting LLVM IR is far longer and contains many more instructions to do almost the same thing you would see in C, and a lot of architectural information about the program is lost between the two stages.

So the Swift compiler team added a second IR:

```text
Swift
  ↓
SIL (Swift Intermediate Language)   ← higher-level optimizations
  ↓
LLVM IR                              ← LLVM's optimizations
  ↓
assembly
```

LLVM IR is roughly "assembly with types", sitting between C and assembly. SIL sits between Swift and LLVM IR — closer to Swift than LLVM IR is.

Each stage introduces its own optimization passes. Because SIL is higher level, it can understand higher-level details about the program and perform optimizations that produce better, more canonicalized LLVM IR, which LLVM can then optimize further. And crucially, the same optimizations already written for C instantly became applicable to Swift — the Swift team did not need to rewrite years of transformation work.

### Reading LLVM IR

LLVM IR is written in **SSA**, single static assignment, and works on an **infinite registers** concept. Registers appear as numbered labels — `%2`, `%3`, `%4`, `%5` and so on — each a label for a place something is stored.

Other labels have specific meaning: `%1` is the first parameter passed into the function, and its type is spelled out alongside it, for example a 32-bit integer. The function's return type is declared as a 32-bit integer too.

A typical unoptimized function body starts by allocating memory: two registers hold pointers to allocations of 32-bit integers, then the incoming argument is stored into one of them.

Several of those instructions are useless. There may be an entire register that is never used. This is not the best the code can be, because unoptimized IR is a blind translation from the C you wrote to the most obvious LLVM code that works. It does a lot of unnecessary operations.

### Canonicalization

Alongside optimization, LLVM performs **canonicalization**: rewriting equivalent operations into one standard form. For example:

```c
int f(int x) {
    if (x != 5) return 1;
    else return 0;
}
```

The canonicalized IR flips this around — it tests whether `x == 5`, returning `0`, and otherwise returns `1`, effectively swapping the true and false branches. Similarly, `x - 1` may be rewritten as `x + (-1)`.

By standardizing the format of certain operations, optimization passes only have to look for a few canonical forms instead of a wide range of equivalent ones, which makes their job much easier.

### Who Builds It

The primary maintainer of LLVM and of clang — notable as the only C compiler supporting Objective-C — has been Apple. Google now contributes heavily as well; Chris Lattner, creator of both LLVM and Swift, left Apple for Google's accelerator and Swift for TensorFlow work. Companies like IBM contribute optimization passes too, such as loop optimization and loop merging.

Google is also contributing **MLIR**, which further increases the amount of reuse possible across compilers, extending to things like TPUs and other accelerators.

### Going Further

Beyond reading IR, LLVM exposes a **pass manager** that lets you write your own transformation and optimization passes — for example, inserting instrumentation into functions entirely automatically. This is exactly the capability that monolithic compilers denied you, and it is the point of the whole modular design.
