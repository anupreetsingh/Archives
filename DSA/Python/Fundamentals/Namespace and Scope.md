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

You can either just read the value of a variable from an outer scope

#### Reading

Reading an enclosing variable is automatic.

```python
def outer():
    count = 10

    def inner():
        print(count)

    inner()

outer()
# Output: 10
```

#### Assigning

Here, `inner()` does not define `count`, so Python searches the enclosing function and finds it in `outer()`.

Assigning is different. If you assign to a name inside a function, Python treats that name as local to that function unless told otherwise.

Example:

```python
def outer():
    count = 10

    def inner():
        count = 20
        print(count)

    inner()
    print(count) 

outer()
# Output:
# 20
# 10
```

The assignment `count = 20` creates a new local variable inside `inner()`. It does not modify `outer()`'s `count`.

This can cause `UnboundLocalError`:

```python
count = 10

def update():
    print(count)
    count = 20

update()
# UnboundLocalError
```

Because `count = 20` appears inside `update()`, Python treats `count` as local for the whole function. So `print(count)` tries to read the local `count` before it has been assigned.

---

#### `global`

Use `global` when you want assignment inside a function to affect a name in the module's global namespace.

```python
count = 10

def update():
    global count
    count = 20

update()
print(count)
# Output: 20
```

Without `global`, `count = 20` would create a local variable inside `update()`.

Use `global` for module-level names only.

---

#### `nonlocal`

Use `nonlocal` when you want assignment inside a nested function to affect a name in an enclosing function.

```python
def outer():
    count = 10

    def update():
        nonlocal count
        count = 20

    update()
    print(count)

outer()
# Output: 20
```

Without `nonlocal`, `count = 20` would create a local variable inside `update()`.

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
