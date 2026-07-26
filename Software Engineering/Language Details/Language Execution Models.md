# Language Execution Models

Language execution models describe how source code becomes a running program.

## Compiled vs Interpreted Languages

### Compiled Languages

Source code is fully translated into machine code *before* execution by a compiler. The resulting binary runs directly on the CPU without needing the compiler again at runtime.

**Examples:** C, C++, Go, Rust, Swift

**Flow:**

```text
source code (.c)
     |
compiler -> machine code (.exe / .out)
     |
CPU executes directly
```

| Pros | Cons |
|------|------|
| Very fast execution because the program is already machine code | Must recompile after each code change |
| Can be optimized heavily by the compiler | Harder to debug during execution |
| Fewer runtime errors if type-checked at compile time | Platform-dependent binaries, such as Windows vs Linux builds |

### Interpreted Languages

Source code is *read and executed line by line* by an interpreter at runtime. No separate compilation step is needed.

**Examples:** Python, JavaScript, Ruby, PHP

**Flow:**

```text
source code (.py)
     |
interpreter (Python VM)
     |
executes line by line
```

| Pros | Cons |
|------|------|
| Easy to test and debug quickly | Slower execution because each line is interpreted while running |
| Cross-platform because the same script can run anywhere the interpreter exists | Errors only show up when the code path executes |
| Good for scripting, prototyping, and education | |

### Bytecode and Virtual Machines

Some languages use an intermediate representation called **bytecode**.

Bytecode is lower-level than source code but not the same as native CPU machine code. A virtual machine/runtime executes that bytecode.

### Python's Hybrid Nature

Technically, Python is both *compiled and interpreted*:

1. The `.py` source is first compiled into **bytecode** (`.pyc`).
2. That bytecode is then interpreted by the **Python Virtual Machine (PVM)**.

So Python is a *hybrid* language: compiled to bytecode, then interpreted.

You can see bytecode in `__pycache__`:

```python
>>> import example
# creates: __pycache__/example.cpython-311.pyc
```

Example:

```python
def greet():
    print("hello")

greet()

# When you run this:
# 1. Python compiles greet() into bytecode instructions.
# 2. The interpreter executes those instructions one by one.

import dis
print("\nDisassembled bytecode for greet():")
dis.dis(greet)
```

### Just-In-Time Compilation

Some environments use **JIT (Just-In-Time) compilation**. They compile frequently used parts of code *at runtime* into native machine code for better performance.

| Environment | JIT |
|-------------|-----|
| Java | HotSpot JIT in JVM |
| C# | .NET CLR |
| JavaScript | V8 engine in Chrome/Node |
| Python | PyPy |

### Summary Table

| Type | How it runs | Examples | Speed | Flexibility |
|------|-------------|----------|-------|-------------|
| **Compiled** | Translated to machine code before running | C, C++, Rust, Go | Fast | Less |
| **Interpreted** | Executed by an interpreter at runtime | Python, JavaScript, Ruby | Slower | High |
| **Hybrid** | Compiled to bytecode, then executed by a VM/runtime | Java, Python, C# | Medium | High |
| **JIT** | Compiled dynamically at runtime | Java, JavaScript, PyPy | Fast after warmup | High |

### Takeaway

- **Compiled:** source code becomes native machine code before execution.
- **Interpreted:** source code is executed by an interpreter at runtime.
- **Hybrid:** source code is compiled to bytecode, then run by a VM/runtime.
- **JIT:** frequently used code is compiled into native machine code while the program is running.
