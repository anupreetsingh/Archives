# What is `self` in Python OOP?

- In Object-Oriented Programming (OOP), a class is a blueprint for creating objects.
- An object is an instance of a class with actual values stored in it.
- `self` represents the current instance of the class.
- It is used to access instance variables and instance methods from within the class.

## Instance Variables and Methods

```python
class Dog:
    def __init__(self, name, age):
        # - This method is called the constructor or initializer.
        # - It runs automatically when you create a new object from the class.
        # - `name` and `age` are **parameters**, local to this method.

        # - `self.name` and `self.age` are **instance variables**.
        # - They are attached to the object being created and stored in the object's namespace.
        self.name = name
        self.age = age

    def speak(self):
        # - This is an **instance method**.
        # - The first parameter of the instance method must be `self`.
        # - When you call an instance method like: dog1.speak(), python automatically translates it to Dog.speak(dog1) puts dog1 in place of self on the back end
        # - Inside this method, we can access instance variables using `self.name`, `self.age`, etc. These will execute as dog1.name and dog1.age
        print(f"My name is {self.name} and I am {self.age} years old.")

    def birthday(self):
        # - We can also modify instance variables using `self`.
        self.age += 1
        print(f"Happy Birthday {self.name}! You are now {self.age}.")
```

## Object Creation and Method Calls

```python
# Creating two objects (instances) of the Dog class
dog1 = Dog("Buddy", 3)
dog2 = Dog("Luna", 5)

# Each object has its own separate `name` and `age` stored in backend of the object and accessed using `self` by invoking that objects backend
dog1.speak()  # My name is Buddy and I am 3 years old.
dog2.speak()  # My name is Luna and I am 5 years old.

# The instance variable `self.age` belongs to that specific object
dog1.birthday()  # Happy Birthday Buddy! You are now 4.
dog2.birthday()  # Happy Birthday Luna! You are now 6.
```

## Why is `self` needed?

- When you call `dog1.speak()`, Python internally does: `Dog.speak(dog1)`
- So the object is automatically passed as the first argument (`self`).
- This is how Python binds the method to the object.

## Summary of Key Points about `self`

1. `self` must be the first parameter in instance methods.
2. It gives access to instance variables and other instance methods.
3. Variables defined as `self.var` live as long as the object exists (object scope).
4. Do not omit `self` or use just `var = ...` if you want the value to persist across methods.
5. `self` is just a naming convention, but changing it is strongly discouraged.

## Related Concepts: classmethod and staticmethod

```python
class Example:
    class_variable = 0  # shared across all instances

    def __init__(self, value):
        self.value = value  # instance variable

    # -------------------------------------
    # Instance Method
    # -------------------------------------
    def instance_method(self):
        # Can access both instance and class variables
        return f"Instance value: {self.value}, class value: {Example.class_variable}"

    # -------------------------------------
    # Class Method — a factory method
    # -------------------------------------
    @classmethod
    def from_string(cls, data_str):
        # Parses input and returns an instance
        value = int(data_str)
        cls.class_variable += 1  # modifying class-level variable
        return cls(value)

    # -------------------------------------
    # Static Method — a utility function
    # -------------------------------------
    @staticmethod
    def is_positive(number):
        # Doesn't depend on instance or class — just a general-purpose check
        return number > 0
```

## Class Attributes vs. Instance Attributes

- **Instance Attributes**:
  - Defined using `self.attribute` inside methods (usually `__init__`).
  - Unique to each object/instance.
  - Example: `self.name`, `self.age` in the Dog class.

- **Class Attributes**:
  - Declared directly inside the class but outside methods.
  - Shared across all instances of the class.
  - Accessible via the class name (`<ClassName>.<attributeName>`) or instance (`instance.attribute`).
  - If modified through the class, the change is seen by all instances.

Example:

```python
class Car:
    wheels = 4  # class attribute (shared)

    def __init__(self, brand, color):
        self.brand = brand    # instance attribute (unique per object)
        self.color = color    # instance attribute (unique per object)

# Creating objects
car1 = Car("Toyota", "Red")
car2 = Car("Honda", "Blue")

print(car1.brand, car1.color, car1.wheels)  # Toyota Red 4
print(car2.brand, car2.color, car2.wheels)  # Honda Blue 4

# Change instance attribute -> affects only that object
car1.color = "Black"
print(car1.color)  # Black
print(car2.color)  # Blue

# Change class attribute -> affects all objects (if not overridden at instance level)
Car.wheels = 6
print(car1.wheels)  # 6
print(car2.wheels)  # 6
```

## Final Notes

- ✔ Use `self` when defining instance methods and accessing/modifying instance variables.
- ✔ Use `@staticmethod` when your method doesn't need access to the instance or class.
- ✔ Use `@classmethod` when your method needs to work with the class (e.g., factory methods).
- ✔ Use class attributes for values common to all instances, and instance attributes for data unique to each object.
