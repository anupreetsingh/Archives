# Language Patterns: Python & Programming Concepts

A reference covering four key language dimensions:

1. **Static vs Dynamic** typing  
2. **Weak vs Strong** typing  
3. **Procedural vs OOP**  
4. **Compiled vs Interpreted**

---

## 1. Static vs Dynamic Typing

- **Static typing:** Variable types are checked at *compile* time. Type errors are caught before running.  
  *Examples: Java, C++, Rust.*

- **Dynamic typing:** Variable types are checked at *run* time. Type errors appear when the code executes.  
  *Examples: Python, JavaScript, Ruby.*

**Python is dynamically typed:** names are bound to *objects* that have types; the names themselves don't have fixed types.

```python
x = 3          # x -> int object
x = "three"    # now x -> str object (legal in Python)
```

Type errors happen only when the operation is executed:

```python
def add(a, b):
    return a + b

print(add(2, 5))        # OK: 7
# print(add(2, "5"))   # TypeError at runtime: unsupported operand types
```

### Type Hints (PEP 484)

Python supports *optional* static type hints to catch issues earlier via tools like **mypy** or **pyright**. Hints do **not** enforce types at runtime by default.

```python
from typing import List

def total(xs: List[int]) -> int:
    return sum(xs)

print(total([1, 2, 3]))     # 6
# print(total([1, "2", 3])) # mypy would flag this; runtime will fail when summing
```

Optional runtime enforcement (if you need it) can be done manually:

```python
def total_ints_runtime(xs):
    if not all(isinstance(x, int) for x in xs):
        raise TypeError("expected all ints")
    return sum(xs)
```

### Dynamic Features in Action

```python
class Bag: pass

b = Bag()
b.new_attr = 42            # Add attributes at runtime (dynamic)
print(b.new_attr)          # 42
```

**Mini-exercises:**

1. Run mypy/pyright on this file and see what it flags.  
2. Add a type hint to `add` and intentionally pass mixed types.

---

## 2. Weak vs Strong Typing

- **Strong typing:** The language resists implicit type coercions. You must convert explicitly (e.g. Python, Java).

- **Weak typing:** The language performs many implicit coercions (e.g. JavaScript: `"2" == 2` is `true`, `"2" + 2` → `"22"`).

**Python is strongly typed:** it won't silently coerce unlike types.

```python
# Strong typing example:
# print(2 + "5")          # TypeError (no implicit coercion)

# Explicit conversion is required:
print(2 + int("5"))       # 7
print(str(2) + "5")       # "25"
```

**Duck typing** (still strong): operations depend on what an object *can do*, not its declared type, but invalid operations still raise errors.

```python
class Adder:
    def __init__(self, x): self.x = x
    def add(self, y):      return self.x + y  # works if x and y support +

print(Adder(10).add(5))     # 15
# print(Adder(10).add("5")) # TypeError at runtime
```

**Mini-exercises:**

1. Try adding `bytes` to `str`, `list` to `int`, etc., and note the errors.  
2. Contrast with JavaScript in a REPL to see weak-typing coercions.

---

## 3. Procedural vs Object-Oriented (OOP)

- **Procedural:** Organize as functions + data passed between them.  
  *Examples: C, Pascal, Fortran, BASIC, COBOL.*

- **OOP:** Bundle data + behavior into objects; use encapsulation, inheritance, and polymorphism to model domains.  
  *Examples: C++, Java, Python, C#, Ruby, Smalltalk.*

**Python is multi-paradigm:** it supports both procedural and OOP.

### Example: Tiny Order System

Compute totals with tax and discount — first procedurally, then in OOP style.

#### Procedural Style

```python
from dataclasses import dataclass

@dataclass
class LineItem:
    name: str
    qty: int
    unit_price: float

def subtotal(items):
    return sum(i.qty * i.unit_price for i in items)

def apply_discount(amount, pct):
    return amount * (1 - pct)

def apply_tax(amount, rate):
    return amount * (1 + rate)

items = [LineItem("pen", 3, 1.50), LineItem("notebook", 2, 4.00)]
amount = subtotal(items)              # data flows through functions
amount = apply_discount(amount, 0.10) # 10% off
amount = apply_tax(amount, 0.06)      # 6% tax
print(round(amount, 2))
```

| | |
|---|---|
| **Pros** | Simple for small scripts and pipelines; easy to trace flow. |
| **Cons** | Data and behavior are separate; grows messy as rules/features increase; harder to extend without modifying many functions. |

#### OOP Style

```python
class Order:
    def __init__(self, items):
        self.items = items
        self._discount_pct = 0.0
        self._tax_rate = 0.0

    # Encapsulation: data + behavior together
    @property
    def subtotal(self):
        return sum(i.qty * i.unit_price for i in self.items)

    def set_discount(self, pct: float):
        self._discount_pct = pct
        return self

    def set_tax(self, rate: float):
        self._tax_rate = rate
        return self

    def total(self):
        amt = self.subtotal
        amt *= (1 - self._discount_pct)
        amt *= (1 + self._tax_rate)
        return amt

order = Order(items).set_discount(0.10).set_tax(0.06)
print(round(order.total(), 2))
```

| | |
|---|---|
| **Pros** | Natural modeling; extensible (subclass/compose); polymorphism supports "open for extension, closed for modification". |
| **Cons** | More boilerplate for simple tasks; poorly designed hierarchies become rigid. |

### Polymorphism: Different Pricing Strategies

```python
class PricingStrategy:
    def compute(self, order: Order) -> float: raise NotImplementedError

class FlatDiscount(PricingStrategy):
    def __init__(self, pct): self.pct = pct
    def compute(self, order: Order) -> float:
        return order.subtotal * (1 - self.pct)

class TieredDiscount(PricingStrategy):
    def compute(self, order: Order) -> float:
        st = order.subtotal
        if st > 100: return st * 0.80
        if st > 50:  return st * 0.90
        return st

def total_with_tax(strategy: PricingStrategy, order: Order, tax_rate: float) -> float:
    pre_tax = strategy.compute(order)   # polymorphic call
    return pre_tax * (1 + tax_rate)

print(round(total_with_tax(FlatDiscount(0.1), order, 0.06), 2))
print(round(total_with_tax(TieredDiscount(), order, 0.06), 2))
```

### Composition Over Inheritance (Pythonic Tip)

Prefer injecting behavior (functions/objects) instead of deep class trees.

```python
def percent_off(pct):
    return lambda amount: amount * (1 - pct)

def compute_total(items, discount_fn, tax_rate):
    st = subtotal(items)
    st = discount_fn(st)
    return st * (1 + tax_rate)

print(round(compute_total(items, percent_off(0.1), 0.06), 2))
```

**Mini-exercises:**

1. Add a "Buy X Get Y" discount without editing existing functions/classes.  
2. Add a new tax strategy (e.g. compound city + state) and plug it in.  
3. Convert the procedural pipeline to use `functools.reduce` for practice.

---

## 4. Compiled vs Interpreted Languages

### Compiled Languages

Source code is fully translated into machine code *before* execution by a compiler. The resulting binary runs directly on the CPU without needing the compiler again at runtime.

**Examples:** C, C++, Go, Rust, Swift

**Flow:**

```
source code (.c)
     ↓
compiler → machine code (.exe / .out)
     ↓
CPU executes directly
```

| Pros | Cons |
|------|------|
| Very fast execution (already machine code) | Must recompile after each code change |
| Can be optimized heavily by the compiler | Harder to debug during execution |
| Fewer runtime errors if type-checked at compile time | Platform-dependent binaries (Windows vs Linux builds) |

### Interpreted Languages

Source code is *read and executed line by line* by an interpreter at runtime. No separate compilation step is needed.

**Examples:** Python, JavaScript, Ruby, PHP

**Flow:**

```
source code (.py)
     ↓
interpreter (Python VM)
     ↓
executes line by line
```

| Pros | Cons |
|------|------|
| Easy to test and debug quickly | Slower execution (each line interpreted every run) |
| Cross-platform (same script runs everywhere) | Errors only show up when the code path executes |
| Great for scripting, prototyping, and education | |

### Python's Hybrid Nature

Technically, Python is both *compiled and interpreted*:

1. The `.py` source is first compiled into **bytecode** (`.pyc`).  
2. That bytecode is then interpreted by the **Python Virtual Machine (PVM)**.

So Python is a *hybrid* language — compiled to bytecode, then interpreted.

You can see bytecode in `__pycache__`:

```python
>>> import example
# creates: __pycache__/example.cpython-311.pyc
```

**Demonstration:**

```python
def greet():
    print("hello")

greet()

# When you run this:
# 1. Python compiles greet() into bytecode instructions.
# 2. The interpreter executes those instructions one by one.

import dis
print("\nDisassembled bytecode for greet():")
dis.dis(greet)  # Show the bytecode that Python actually runs
```

### Just-In-Time (JIT) Compilation

Some environments (e.g. Java's JVM, **PyPy** for Python) use JIT compilers. They compile frequently used parts of code *at runtime* into native machine code for better performance.

| Environment | JIT |
|-------------|-----|
| Java | HotSpot JIT in JVM |
| C# | .NET CLR |
| JavaScript | V8 engine (Chrome/Node) |
| Python | PyPy |

### Summary Table

| Type | How it runs | Examples | Speed | Flexibility |
|------|-------------|----------|--------|-------------|
| **Compiled** | Translated to machine code before run | C, C++, Rust, Go | Fast | Less |
| **Interpreted** | Executed line-by-line at runtime | Python, JS, Ruby | Slow | High |
| **Hybrid** | Compiled to bytecode + interpreted | Java, Python, C# | Medium | High |
| **JIT** | Compiled dynamically at runtime | Java, JS (V8), PyPy | Fast | High |

**Mini-exercises:**

1. Run `python -m py_compile your_script.py` and check the `__pycache__` folder.  
2. Use `dis.dis` on a few of your functions to inspect bytecode.  
3. Research how PyPy improves Python speed via JIT.  
4. Compare Python (interpreted) vs C (compiled) in startup time for the same logic.

**Takeaway:**

- **Compiled:** high speed, strict, less flexible.  
- **Interpreted:** slower but flexible and interactive.  
- **Python:** compiles to bytecode, then interprets it.  
- **JIT** (e.g. PyPy, JVM) blurs the line further for performance.

---

## Quick Cheatsheet

### Python

- **Dynamic typing:** Types checked at runtime; names can rebind to any object.  
- **Strong typing:** No silent coercions; convert explicitly (`int(...)`, `str(...)`).  
- **Type hints:** Optional static hints to catch issues early with type checkers.

### Language Paradigms

| Paradigm | Languages |
|----------|-----------|
| **Procedural** | C, Pascal, Fortran, BASIC, COBOL |
| **OOP** | C++, Java, Python, C#, Ruby, Smalltalk |
| **Multi-paradigm** | Python, JavaScript, Go, Rust |

### Java vs JavaScript

| | Java | JavaScript |
|---|------|------------|
| **Execution** | Compiled to bytecode, runs on JVM | Interpreted/JIT, runs in browsers + Node.js |
| **Typing** | Statically typed | Dynamically typed |
| **Paradigm** | OOP-focused | Multi-paradigm (functional + OOP via prototypes) |
| **Typical use** | Enterprise apps, Android, backend | Web development |

### Takeaway

- Use **procedural** for small, linear scripts and pipelines.  
- Use **OOP** when modeling complex domains or needing extensibility.  
- **Java** = enterprise/OOP heavyweight; **JavaScript** = lightweight, flexible, web.
