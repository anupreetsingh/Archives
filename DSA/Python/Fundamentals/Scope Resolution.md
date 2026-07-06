# Scope Resolution in Python

Scope resolution means how Python decides which name a variable, function, class, module, or attribute refers to.

Python does not have a C++-style scope resolution operator like `::`. Instead, Python resolves plain names using the **LEGB rule**(discussed in [Namespace and Modules](<Namespace and Modules.md#legb-rule>)) and resolves attributes using **dot notation**.

```text
Plain name lookup:   name
Attribute lookup:    object.attribute
```

## Functions(Enclosing Variables)

### Reading

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

Here, `inner()` does not define `count`, so Python searches the enclosing function and finds it in `outer()`.

### Assigning

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

## Classes

### Class Attributes

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

### Instance Attributes

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

### Accessing Instance Attributes Across Methods

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

### Overwriting Class Attribute

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

### Accessing Class Attributes Inside Methods

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
