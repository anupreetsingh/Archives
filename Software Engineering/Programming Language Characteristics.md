# Programming Language Characteristics

A reference covering five key language dimensions:

1. **Static vs Dynamic** typing  
2. **Weak vs Strong** typing  
3. **Duck vs Nominal vs Structural** typing
4. **Procedural vs OOP vs Functional**  
5. **Compiled vs Interpreted**

---

## 1. Static vs Dynamic Typing

### Static typing

Variable types are checked at *compile* time. Once a variable type is declared explicitly or inferred by a compiler/type checker, it cannot be assigned to a different type in the same scope because that would lead to a type error before the program runs.
  
Type errors are caught by the compiler or a type checker before running.

Languages Examples: C++, TypeScript, Java, Rust, Go

Example:

```java
int count = 10;
count = "10"; // Type error: String cannot be assigned to int

var age = "24"; // inferred as String
age = 24;        // Type error: int cannot be assigned to String
```

Here, the errors are flagged before the program runs. The compiler/type checker rejects the assignments because "10" is not compatible with `int`, and 24 is not compatible with the inferred String type of age.

### Dynamic typing

Variable types are checked at *run* time. A name is not tied to any type and can be rebound to objects of different types in the same scope over time.

Type errors appear only when an operation is executed with an incompatible value.

Languages Examples: Python, JavaScript, Ruby

Example of Name rebinding:

```python
x = 3          # x -> int object
x = "three"    # now x -> str object (legal in Python)
```

Exampel of Runtime type error:

```python
value = "hello"
print(value.upper())  # OK: str has upper()

value = 42
# print(value.upper()) # AttributeError at runtime: int has no upper()
```

The name `value` is not permanently tied to one type. It can first refer to a `str` object and later be rebound to an `int` object, so an invalid method call like `value.upper()` is caught only when that line is executed at runtime.

---

## 2. Weak vs Strong Typing

### Weak typing

A weakly typed language performs more implicit type coercion. The language may automatically convert a value from one type to another during an operation.

Examples: JavaScript, PHP

Special Case: Because TypeScript adds static type checking on top of JavaScript, it catches many unsafe operations at compile time, such as `"5" - 2`, while still allowing JavaScript-valid coercions like `"5" + 2` as string concatenation; after compilation, the emitted JavaScript follows JavaScript's weak runtime coercion rules.

Example Usage

Implicit coercion:

```js
console.log("2" == 2);  // true
console.log("2" + 2);   // "22", converts to string because + applies to string concatenation and number arithmetic but the rules of implicit coercion in JS tell it to use concatentation in this scenario.
console.log("5" - 2);   // 3, converts to number because operator like -, *, / only really apply to numbers
```

### Strong typing

A strongly typed language resists implicit type coercion between incompatible types. If an operation needs a different type, the programmer usually has to convert the value explicitly.

Examples: Python, Java, Rust, Go

Example Usage

No implicit coercion:

```python
print(2 + "5") # unsupported operand types for +: int and str
```

Causes a TypeError at runtime because of strong typing

Explicit conversion:

```python
print(2 + int("5"))       # 7
print(str(2) + "5")       # "25"
```

---

## 3. Duck vs Nominal vs Structural Typing

This category is about **type compatibility**. It describes how a language decides whether a value is acceptable for an operation, function parameter, variable, or interface.

### Duck typing

Duck typing focuses on whether an object supports the operation being used, rather than whether it has a specific declared type.

It is named after the idea: "if it walks like a duck and quacks like a duck, then it is a duck."

Because Python and Ruby usually do not require declared parameter types, duck typing is common: an object is considered usable if it supports the operation being performed at runtime.

Example:

```python
def make_sound(obj):
    obj.speak()

class Dog:
    def speak(self):
        print("woof")

class Person:
    def speak(self):
        print("hello")

class Robot:
    def move(self):
        print("rolling")

make_sound(Dog())     # OK: Dog has speak()
make_sound(Person())  # OK: Person has speak()
# make_sound(Robot()) # AttributeError: Robot has no speak() method
```

The function does not require `obj` to be declared as a `Dog` or a `Person`. It only requires that `obj` supports the `.speak()` operation at runtime.

### Nominal typing

Nominal typing focuses on the declared or named type. A value is compatible because it belongs to a specific class, implements a specific interface, or is part of a declared inheritance relationship.

Nominal typing is common in languages such as Java, C#, and C++.

Example:

```java
interface Speaker {
    void speak();
}

class Dog implements Speaker {
    public void speak() {
        System.out.println("woof");
    }
}

void makeSound(Speaker obj) {
    obj.speak();
}

makeSound(new Dog()); 
```

Here, `makeSound` requires a value whose declared type is `Speaker` or a class that implements `Speaker`. The `Dog` object is accepted because `Dog` explicitly declares that it implements the named `Speaker` interface.

### Structural typing

Structural typing focuses on the shape or structure of a value. A value is compatible if it has the required fields or methods, even if it was not explicitly declared as a specific named type.

Structural typing is used by TypeScript and is also similar to how Go interfaces work.

Example:

```ts
type Speaker = {
    speak: () => void;
};

const dog = {
    speak() {
        console.log("woof");
    },
};

function makeSound(obj: Speaker) {
    obj.speak();
}

makeSound(dog); // OK: dog has the required speak method
```

The object `dog` does not have to explicitly declare that it is a `Speaker`. It is compatible because its structure matches the `Speaker` type.

---

## 4. Procedural vs Object-Oriented (OOP) vs Functional

This category is about **programming paradigms**. A programming paradigm describes the main style a language encourages for organizing data, behavior, and program flow.

The same idea can be written in different paradigms. Most modern languages are **multi-paradigm**, meaning they support more than one style. For example, Python can be written procedurally, with classes and objects, or in a more functional style.

### Procedural Programming

Procedural programming organizes code around **procedures**, also called functions or routines. The program is usually written as a sequence of steps that operate on data.

The main focus is:

- What steps should happen?
- In what order should they happen?
- What data should each function read or modify?

Procedural code often keeps data and behavior separate. The data may be stored in variables, dictionaries, structs, or records, while functions operate on that data.

**Examples:** C, Pascal, Bash, Python

Example Usage

```python
account = {"owner": "Maya", "balance": 100}

def deposit(account, amount):
    account["balance"] += amount

def withdraw(account, amount):
    if amount <= account["balance"]:
        account["balance"] -= amount
    else:
        print("Insufficient funds")

deposit(account, 50)
withdraw(account, 30)

print(account["balance"])  # 120
```

In this example, the account data is stored in a dictionary, and the functions receive that dictionary as an argument. The functions directly modify the account's balance.

Procedural programming is useful when the program is naturally a sequence of operations, such as scripts, small utilities, data processing tasks, and system-level code.

### Object-Oriented Programming

Object-oriented programming organizes code around **objects**. An object combines data and behavior into one unit.

The main focus is:

- What objects exist in the system?
- What data does each object store?
- What behavior does each object provide?

In OOP, related data and functions are bundled together in a **class**. The class defines the structure and behavior, while each object is a specific instance of that class.

**Examples:** Java, C++, C#, Python, Ruby

Example Usage

```python
class BankAccount:
    def __init__(self, owner, balance):
        self.owner = owner
        self.balance = balance

    def deposit(self, amount):
        self.balance += amount

    def withdraw(self, amount):
        if amount <= self.balance:
            self.balance -= amount
        else:
            print("Insufficient funds")

account = BankAccount("Maya", 100)

account.deposit(50)
account.withdraw(30)

print(account.balance)  # 120
```

Here, the account's data and behavior are grouped inside the `BankAccount` class. Instead of passing the account into separate functions, the account object knows how to deposit and withdraw from itself.

OOP is useful when a program has many related entities with their own state and behavior, such as users, accounts, orders, files, windows, game characters, or network connections.

Important OOP ideas include:

| Idea | Meaning |
|------|---------|
| **Class** | A blueprint for creating objects |
| **Object** | A specific instance of a class |
| **Encapsulation** | Keeping related data and behavior together |
| **Inheritance** | Creating a class from another class to reuse or extend behavior |
| **Polymorphism** | Using different object types through a shared interface |

### Functional Programming

Functional programming organizes code around **functions** and values. It emphasizes computing new values instead of changing existing state.

The main focus is:

- What value should this function return?
- Can this function avoid changing outside state?
- Can larger behavior be built by combining smaller functions?

Functional code often prefers **pure functions**. A pure function returns the same output for the same input and does not modify outside state.

**Examples:** Haskell, Elixir, Erlang, F#, Lisp, Clojure, Scala

Python, JavaScript, and many other languages also support functional programming features.

Example Usage

```python
def deposit(balance, amount):
    return balance + amount

def withdraw(balance, amount):
    if amount <= balance:
        return balance - amount
    return balance

balance = 100
balance = deposit(balance, 50)
balance = withdraw(balance, 30)

print(balance)  # 120
```

In this version, the functions do not modify an account object or dictionary. They receive values and return new values. The caller decides what to do with the returned result.

A more structured functional version might return a new account dictionary instead of mutating the original one:

```python
def deposit(account, amount):
    return {
        "owner": account["owner"],
        "balance": account["balance"] + amount,
    }

account = {"owner": "Maya", "balance": 100}
updated_account = deposit(account, 50)

print(account["balance"])          # 100
print(updated_account["balance"])  # 150
```

Functional programming is useful when predictable data transformation matters, such as mathematical computation, data pipelines, concurrency, testing, and programs where avoiding shared mutable state reduces bugs.

Important functional ideas include:

| Idea | Meaning |
|------|---------|
| **Pure function** | A function with no side effects and the same output for the same input |
| **Immutability** | Avoiding changes to existing values after they are created |
| **First-class functions** | Treating functions like normal values that can be passed around |
| **Higher-order function** | A function that takes another function as input or returns a function |
| **Composition** | Building larger behavior by combining smaller functions |

### Comparison

| Paradigm | Main organizing idea | Data and behavior | Common strength |
|----------|----------------------|-------------------|-----------------|
| **Procedural** | Step-by-step procedures | Data and functions are usually separate | Simple control flow and direct scripts |
| **Object-Oriented** | Objects with state and behavior | Data and behavior are bundled together | Modeling systems with related entities |
| **Functional** | Functions transforming values | Behavior is functions; data is often treated as immutable | Predictable transformations and fewer state bugs |

The paradigms are not mutually exclusive. A Python program might use procedural code for a simple script, OOP for modeling application entities, and functional techniques for transforming lists or processing data.

---

## 5. Compiled vs Interpreted Languages

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
