# Python Names

In Python, functions are objects, just like integers, strings, lists, and classes are objects.

Python works by binding names to objects.

Assignment operator binds a name data type/ datastructure to a name.
A function definition binds a function object to a name.
A class definition binds a class object to a name.

```python
x = 10
first_name = "Alice"

def greet():
    return "Hello"
```

Here:

- `x` is a name bound to an integer object.
- `first_name` is a name bound to a string object.
- `greet` is a name bound to a function object.

Because of pythons nature of binding names to objects we can't just declare a variable, we need to atleast put a bind a placeholder `None` value to it.

```Python
x = None # Accepted
x  # NameError : name 'x' is not defined
```

## Rebinding/Overwriting

When an assignment, function definition, or class definition runs, Python binds a name to an object. If another assignment or definition later uses the same name in the same scope, Python rebinds that name to the new object.

The old object is not changed; the name simply stops pointing to it.

> This behavior is also why true method overloading does not exist in Python. If multiple methods are defined with the same name, only the later method remains bound to that name.

For Example:

```python
# Variable name rebinding
x = 10
x = 15

print(x)  # Output: 15
# The name x now points to the integer object 15 instead of 10.


# Function name rebinding
def greet(): # Initially binds the function object to the name "greet"
    return "Hello"

def greet(): 
    return "Hi"

print(greet())  # Output: Hi
# The name greet now points to the second function object.
# The first function object is no longer accessible through the name greet.


# Method name rebinding inside a class
class MathOps:
    def add(self, a, b):
        return a + b

    def add(self, a, b, c):
        return a + b + c

m = MathOps()

print(m.add(2, 3))  # TypeError, because the first add() was overwritten; 
print(m.add(2, 3, 4))    # Output: 9
# The name add inside MathOps now points to the second method definition.


# Class name rebinding
class Car:
    def drive(self):
        return "Ford"

old_car = Car()

class Car:
    def drive(self):
        return "Ferrari"

new_car = Car()

print(old_car.drive())  # Output: Ford
print(new_car.drive())  # Output: Ferrari

# The first Car class still exists because old_car was created from it.
# But the name Car now points to the second class definition.
```

In all of these examples, the old object may still exist if something else references it, but the name now points to the newer object.

> Not to be confused with method overriding, where a child class defines a method with the same name as a parent class method, and the behavior depends on which class's instance calls the method.
