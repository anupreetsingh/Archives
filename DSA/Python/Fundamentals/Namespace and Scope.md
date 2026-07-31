# Namespace and Scope Resolution in Python

This note explains how Python stores names, where those names are visible, and how Python decides which object a name refers to.

As we remember names and bound to some object in python. Read more about it in [Name Rebinding](<Name Rebinding.md>)

A **namespace** is the actual name-to-object mapping.

A **scope** is the region of code associated with a namespace level.

## How Scope is Defined

### Other Languages

In many languages, such as C, C++, C#, Java, Rust, Go, and modern JavaScript with `let`/`const`, `{}` often define block scope, and indentation is mostly for readability.

Example:

```cpp
#include <iostream>

int main() {
    if (true) {
        int x = 10;
    }
    std::cout << x << std::endl; // Error: x is not available here because the {} created a block scope. 
}
```

### Python

In Python, indentation creates code blocks but not a separate scope.

Names assigned inside blocks like `if`, `for`, `while`, `try`, and `with` still belong to the surrounding scope.

Python creates scopes and namespaces mainly through:

- **Functions**: function calls create local and enclosing scopes.
- **Modules**: each `.py` file has its own global scope.
- **Classes**: the class body runs in its own namespace, but that class namespace is not an enclosing function scope in the LEGB rule.
- **Comprehensions**: Variables used in comprehensions are local to them while the comprehension is active and are deleted afterwards.

Example:

```python
if True:
    x = 10

for i in range(3):
    value = i

print(x)      # Output: 10
print(i)      # Output: 2
print(value)  # Output: 2
```

The `if` block and `for` loop have their indented code blocks, but they do not create separate scopes. Since this code runs at module level, `x`, `i`, and `value` all belong to the module's global scope.

## When Python Classifies Names

Python needs to decide how a plain name should be resolved before the program actually runs. In particular, it needs to know whether that name belongs to the local scope, an enclosing function scope, or the global/built-in lookup path.

People often say Python is interpreted, but CPython still has a compilation step. Python source code is first compiled into **bytecode**, and that bytecode is stored inside a **code object**. The Python Virtual Machine, or **PVM**, then executes that bytecode from top to bottom. This execution phase is what people usually mean when they say Python is interpreted.

A **code object** stores the compiled bytecode plus metadata about that block of code. Some of that metadata describes how names are handled. For example, it records whether a name should be loaded from the function's local namespace, from an enclosing function scope, or through a global lookup.

This compile-time name classification is part of Python's **static code analysis**.

For example, if a name is assigned anywhere inside a function body, and there is no `global` or `nonlocal` declaration for it, Python treats that name as local to that function.

If a name is referenced but not assigned inside that function, Python does not immediately check whether the name actually exists. Instead, the compiler records the lookup strategy for that name as part of the code object. Depending on the surrounding code, the name may be looked up in an enclosing function scope, or through the global namespace and then the built-in namespace.

Whether the name actually exists at the moment it is used is checked later, at runtime, as the bytecode executes from top to bottom.

### Modules

Before a module runs, Python compiles the module source into a code object. During compilation, Python also compiles nested function bodies and class bodies into their own code objects and records how plain names should be treated inside each one.

Then Python executes the module code from top to bottom. Names assigned at the top level become names in that module's global namespace.

```python
def a():
    x = 10
    return x
```

When the module code reaches `def a():`, Python binds the name `a` in the module's global namespace. The body of `a()` does not run yet, but the code object for `a()` has already been compiled and its names have already been classified.

### Functions

During compilation, Python analyzes the function body and records which names are local, which names come from enclosing function scopes, and which names should be treated as global.

When Python later executes the `def` statement, it creates a function object from that already-compiled code object and binds the function name in the surrounding namespace. The function body itself does not run yet. Later, when the function is called, Python uses the function's code object and executes the body from top to bottom.

This is why assignment affects the whole function scope, even if the assignment appears after a read:

```python
def outer():
    count = 10

    def increment():
        count += 1

    increment()

outer()
# UnboundLocalError
```

Since `increment()` assigns to `count`, Python classifies `count` as local to `increment()` before `increment()` runs. When execution reaches `count += 1`, Python tries to read the local `count` before any local value has been bound.

### Classes

Class bodies are also compiled into code objects, but a `class` statement behaves differently from a `def` statement. When Python executes a `class` statement, it executes the class body immediately. That execution builds the class namespace. Names assigned directly in the class body become class attributes.

```python
class Student:
    school = "ABC"

    def show(self):
        print(self.school)
```

While the class body runs, `school` and `show` are added to the class namespace. After the class body finishes, Python uses that namespace to create the `Student` class object.

Class bodies do not behave exactly like function bodies for plain-name assignment. Assignment in a class body creates or updates a name in the class namespace, but it does not make earlier reads of that same name behave like unbound local variable reads.

```python
x = 10

class Example:
    print(x)
    x = 20

# Output:
# 10
```

Here, `print(x)` can find the global `x` because the class attribute `x` has not been created yet. This is different from a function, where assigning to `x` anywhere in the function would make `x` local for the whole function.

## Name Lookup Patterns

Name lookup means how Python decides which object a name refers to.

Python has two important lookup patterns:

- **Plain name lookup**: `name`
  - Uses the LEGB rule.
- **Attribute lookup**: `object.attribute`
  - Uses dot notation and searches inside an object, class, module, or package namespace.

## Plain Name Lookup

A plain name is any name used directly, without dot notation.

Variables, functions, classes, modules, and built-ins can all be plain names.

Example:

```python
count = 10

def show():
    return count

class Student:
    pass

import math

print(count)      # Resolves the plain name `print`, then resolve the plain name `count` 
                  # and calls the function object that `print` maps to in the namespace.
show()            # Resolves the plain name `show` to a function object and calls it.
Student()         # Resolves the plain name `Student` to a class object and calls it to make an instance.
```

### Scope Resolution using LEGB Rule

When Python resolves a plain name, it first checks the namespace associated with the current scope. If the name is not found there, Python searches outward through namespaces using the **LEGB rule**:

```text
Local -> Enclosing -> Global -> Built-in
```

- **Local**: names created inside the function currently running.
- **Enclosing**: names in outer functions when functions are nested.
- **Global**: names in the current module. Each Python file is a module with its own global namespace.
- **Built-in**: names Python provides automatically, like `print`, `len`, and `dict`.

Example:

```python
global_name = "global"

def outer():
    enclosing_name = "enclosing"

    def show():
        local_name = "local"

        print(local_name)       # local
        print(enclosing_name)   # enclosing
        print(global_name)      # global
        print(len("hello"))     # 5  

    show()

outer()
```

In this example:

- `local_name` is found in the local namespace of `show()`.
- `enclosing_name` is not found in `show()`, so Python moves outward and finds it in `outer()`.
- `global_name` is not found in `show()` or `outer()`, so Python finds it in the module's global namespace.
- `len` is not found in the local, enclosing, or global namespaces, so Python finds it in the built-in namespace.

### Using variable from an outer namespace

You can either use the object that an outer name already refers to, or assign to that name inside the inner scope.

#### Using object tied to an outer name

Reading an enclosing variable is automatic. If that variable refers to a mutable object, you can also mutate that object.

```python
def outer():
    state = {"count": 10, "history": []}

    def inner():
        print(state["count"])
        state["count"] += 1
        state["history"].append("updated")

    inner()
    print(state)

outer()
# Output:
# 10
# {'count': 11, 'history': ['updated']}
```

Here, `inner()` does not assign to the plain name `state`, so Python searches the enclosing function and finds `state` in `outer()`.

Then `state["count"] += 1` and `state["history"].append("updated")` mutate the dictionary that `state` refers to. The name `state` still points to the same dictionary object. Only the contents of that dictionary changed.

#### Function parameters are local names

The most accurate way to describe Python argument passing is:

```text
Python passes object references by value.
```

When a function is called, each parameter name is created in the function's local namespace. That local name receives a copy of the reference to the same object that the caller passed.

This means there are two separate names, but they can refer to the same object.

Example with a mutable object:

```python
def foo(s):
    s.add(3)

x = {1, 2}
foo(x)

print(x)
# Output: {1, 2, 3}
```

During the call, both `x` and `s` refer to the same set object:

```text
x ---\
      v
    {1, 2}
      ^
s ---/
```

So `s.add(3)` mutates the set object itself. Since `x` also refers to that same set, the change is visible after the function returns.

Rebinding the parameter is different:

```python
def foo(s):
    s = {100}

x = {1, 2}
foo(x)

print(x)
# Output: {1, 2}
```

Before rebinding, both names refer to the same set:

```text
x ---\
      v
    {1, 2}
      ^
s ---/
```

After `s = {100}`, only the local name `s` is changed:

```text
x ----> {1, 2}

s ----> {100}
```

The original set was not mutated. The function only made its local parameter name `s` refer to a different set object.

#### Rebinding the object tied to a name

Assigning to the plain name itself is different. If you assign to a name inside a function, Python treats that name as local to that function unless told otherwise with `nonlocal` or `global`.

This is true for normal assignment and augmented assignment:

```python
def outer():
    count = 10

    def replace():
        count = 20
        print(count)

    def increment():
        count += 1

    replace()
    print(count)
    increment()

outer()
# Output:
# 20
# 10
# UnboundLocalError
```

Python determines how names are classified in a function before the function body runs: local, free/nonlocal, global, or built-in lookup. If a function is just referencing / accessing a name then it uses LEGB for resolution of the name. If the function is using an assignment there are two scenarios

1. Using nonlocal or global: tells it that the name beind rebound is an enclosing or global name
2. Otherwise it assumes that it is a local name being bound or rebound in successive steps from top to bottom of the local scope.

In `replace()`, `count = 20` is a normal assignment. So it binds the name `count` to `20` as a local name, then when `print(count)` refers `count` resolves to a local name

In `increment()`, `count += 1` is equivalent to `count = count + 1` so it is an assignment. Before the function begins execution just seeing this assignment to the name `count`, python has determined that count is a local name but when execution reaches this line it tries to actually bind a value to local `count` it sees that value being bound is also `count` but count is local and hasn't been bound yet(going from top to bottom) so it gives an `unboundLocalError` which implies a name that is local is being referred / accessed before it has been bound to an object in the line of the execution.

---

#### `global`

Use `global` when you want assignment inside a function to affect a name in the module's global namespace. This also fixes the `count += 1` error for module-level names.

```python
count = 10

def update():
    global count
    count += 1

update()
print(count)
# Output: 11
```

Without `global`, `count += 1` would make `count` local to `update()`, then try to read that local `count` before it has been assigned.

Use `global` for module-level names only.

---

#### `nonlocal`

Use `nonlocal` when you want assignment inside a nested function to affect a name in an enclosing function. This fixes the `count += 1` error for enclosing function names.

```python
def outer():
    count = 10

    def update():
        nonlocal count
        count += 1

    update()
    print(count)

outer()
# Output: 11
```

Without `nonlocal`, `count += 1` would make `count` local to `update()`, then try to read that local `count` before it has been assigned.

`nonlocal` does not look in the global namespace. It only works with names from enclosing function scopes.

## Attribute Lookup

Attribute lookup happens when a name is accessed through another object using dot notation.

Examples:

```python
student.name
Student.school
math.sqrt
os.environ
```

In each example, Python first resolves the plain name before the dot, such as `student`, `Student`, `math`, or `os`. Then it looks for the attribute name after the dot, such as `name`, `school`, `sqrt`, or `environ`, inside that object's namespace or through that object's attribute lookup rules.

### Classes

A class body creates a namespace for the class object. Names assigned directly inside the class body become class attributes.

However, the class namespace is not the same as an enclosing function scope in LEGB. A method does not automatically find class attributes as plain names.

```python
x = "global"

class Student:
    x = "class"

    def show(self):
        print(x)

s1 = Student()
s1.show()
# Output: global
```

Inside `show()`, plain `x` is resolved using LEGB:

```text
show local -> enclosing functions -> module global -> built-in
```

Python does not search `Student`'s class namespace as the enclosing scope for the method. To access the class attribute, use attribute lookup:

```python
class Student:
    x = "class"

    def show(self):
        print(Student.x)
        print(self.x)
```

#### Class Attributes

A class attribute belongs to the class object itself.

```python
class Student:
    school = "ABC School"

print(Student.school)
# Output: ABC School
```

The class `Student` has its own namespace, and `school` is stored inside that class namespace.

Class attributes can be accessed through:

```python
Student.school
```

They can also be accessed through an instance:

```python
s1 = Student()
print(s1.school)
# Output: ABC School
```

When Python sees `s1.school`, it first looks for `school` on the instance. If it does not find it there, it then looks on the class.

#### Instance Attributes

An instance attribute belongs to one specific object. Each instance has its own namespace for instance attributes.

```python
class Student:
    school = "ABC School"

    def __init__(self, name):
        self.name = name

s1 = Student("Aman")
s2 = Student("Riya")

print(s1.name)
print(s2.name)
# Output:
# Aman
# Riya
```

Here:

- `school` is shared by the class.
- `name` is separate for each instance.

#### Accessing Instance Attributes Across Methods

Instance attributes are stored on the object, not inside one specific method.

Once an instance attribute is created, other instance methods can access it through `self`.

```python
class Student:
    def __init__(self, name, marks):
        self.name = name
        self.marks = marks

    def show_name(self):
        print(self.name)

    def show_result(self):
        print(self.name, self.marks)

s1 = Student("Aman", 95)

s1.show_name()
s1.show_result()
# Output:
# Aman
# Aman 95
```

Here, `self.name` and `self.marks` are created inside `__init__()`, but they are accessible inside `show_name()` and `show_result()` because all three methods are working with the same object through `self`.

##### Nested Functions Inside Methods

A nested function inside a method can also access instance attributes through `self`, but the important part is that `self` is the enclosing-scope name.

```python
class Solution:
    def combine(self):
        self.combinations = []

        def backtrack():
            self.combinations.append([])

        backtrack()
        return self.combinations
```

Inside `backtrack()`, Python resolves `self.combinations` in two steps:

1. Resolve the plain name `self` using LEGB.
2. Look up the attribute `combinations` on the object that `self` refers to.

So `backtrack()` can access `self.combinations` because `self` is available from the enclosing method scope.

The attribute name `combinations` itself is not a variable from the enclosing scope. It is stored on the instance object and is found through attribute lookup.

Instance attributes can also be created or changed in methods other than `__init__()`.

```python
class Student:
    def set_grade(self, grade):
        self.grade = grade

    def show_grade(self):
        print(self.grade)

s1 = Student()
s1.set_grade("A")
s1.show_grade()
# Output: A
```

This works because `set_grade()` creates `grade` on the object `s1`, and `show_grade()` later reads `grade` from the same object.

But if `show_grade()` is called before `set_grade()`, Python will not find `grade` on the object.

```python
s1 = Student()
s1.show_grade()
# AttributeError
```

Instance attributes are:

- Accessible Inside instance methods using `self.attribute`.
- Accessible Outside the class using `object.attribute`.
- Accessible Inside class methods or static methods only if an instance object is available.
- Not directly accessible in the class body because the class body runs before any object exists.

#### Overwriting Class Attribute

```python
class Student:
    school = "ABC School"

s1 = Student()
s2 = Student()

s1.school = "XYZ School"

print(s1.school)
print(s2.school)
print(Student.school)
# Output:
# XYZ School
# ABC School
# ABC School
```

`s1.school = "XYZ School"` does not modify the class attribute. It creates an instance attribute named `school` on `s1`.

Now lookup works like this:

```text
s1.school -> found on s1 instance
s2.school -> not found on s2 instance, so Python checks class level Student.school
Student.school -> found directly on the class
```

To modify the class attribute itself, assign through the class:

```python
Student.school = "New School"
```

#### Accessing Class Attributes Inside Methods

Inside an instance method, class attributes can be accessed using `self` or the class name.

```python
class Student:
    school = "ABC School"

    def show_school(self):
        print(self.school)    # Output: ABC School
        print(Student.school) # Output: ABC School
```

`self.school` first looks for an instance attribute named `school`. Since this object does not have its own `school` attribute, Python looks at the class and finds `Student.school`.

```python
class Student:
    school = "ABC School"

    @classmethod
    def show_school(cls):
        print(cls.school)
```

`cls` refers to the class that called the method. This is especially useful with inheritance.

## Not part of Python namespaces

Some things are related to names, but are not Python namespaces.

### Keywords

Keywords like `if`, `and`, `for`, `def`, and `class` are part of Python's grammar. They are different from built-in names like `print` because keywords cannot be shadowed as variable names.

```python
if = "hello"  # SyntaxError
print = "hello"  # allowed, but bad idea
```

### Environment variables

They are OS-level key-value settings available to a running program:

```python
import os  # os is a standard library module in Python

print(os.environ["HOME"])
```

Here, `os` is added to the current module's global namespace and maps to the `os` module.
Then `os.environ` gives access to OS-level environment variables. A `.env` file is a common way to define environment variables, but Python does not automatically load `.env` files unless a tool or library loads them.
`"HOME"` is a key inside that environment-variable mapping.
