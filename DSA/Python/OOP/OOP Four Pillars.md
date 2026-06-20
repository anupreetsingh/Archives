# Object Oriented Programming — Four Pillars

## 1. Encapsulation

- Restricting access to attributes and methods inside a class. And maybe providing controlled access using getters/setters methods.
- Hides implementation details (often with private attributes).

```python
class BankAccount:
    def __init__(self, owner, balance=0):
        self.owner = owner
        self.__balance = balance   # "__" makes it private (cannot access directly)

    def deposit(self, amount):
        self.__balance += amount

    def withdraw(self, amount):
        if amount <= self.__balance:
            self.__balance -= amount
        else:
            print("Insufficient funds!")

    def get_balance(self):
        return self.__balance

# Example:
acc = BankAccount("Alice", 100)
acc.deposit(50)
print(acc.get_balance())  # 150
# print(acc.__balance)    # ❌ ERROR: private attribute, balance for this Bank Account object isn't directly accessible but accessible through the getter method get_balance
```

## 2. Abstraction

- Hides complex implementation, only shows necessary detail required for context to the user at that point.
- Achieved with Abstract Base Classes in Python.

```python
from abc import ABC, abstractmethod

class Vehicle(ABC):
    @abstractmethod # Forces the subclass to implement that method
    def move(self):
        pass  # pass is a placeholder, it does nothing, it is only there to tell us that this method here is not doing, and the 

class Car(Vehicle):
    def move(self):
        return "Driving on the road 🚗"

class Boat(Vehicle):
    def move(self):
        return "Sailing on water ⛵"

# Example:
vehicles = [Car(), Boat()]
for v in vehicles:
    print(v.move())
```

## 3. Inheritance

- A child/sub class can use attributes and methods of parent/super/Base class.
- Promotes code reusability.
- The child class can add additional attributes/methods or override original attributes/methods from the super class by defining them again.

```python
class Animal:
    def __init__(self, name):
        self.name = name

    def eat(self):
        return f"{self.name} is eating."

class Dog(Animal):  # inherits from Animal
    def bark(self):
        return f"{self.name} is barking! 🐶"

# Example:
dog = Dog("Buddy")
print(dog.eat())   # inherited
print(dog.bark())  # child-specific
```

## 4. Polymorphism

**Definition:**

- "Many forms" → Same method name behaves differently depending on the object (or the way it's called).
- It allows one interface to represent multiple underlying forms.
- Promotes code flexibility and extensibility.

Two main types of Polymorphism:

1. Compile-time Polymorphism → Method Overloading
2. Run-time Polymorphism → Method Overriding

### 4.1 Method Overloading (Compile-time Polymorphism)

- Method overloading means multiple methods have the same name but different parameters.
- The correct version is chosen based on the arguments passed.
- This is common in Java/C++ and is considered compile-time polymorphism.
- Python does NOT support true method overloading.

Example in Java:

```java
class MathOps {
    int add(int a, int b) {
        return a + b;
    }

    int add(int a, int b, int c) {
        return a + b + c;
    }
}

// add(2, 3) chooses add(int, int)
// add(2, 3, 4) chooses add(int, int, int)
```

In Python, overloading is usually simulated using default arguments or variable arguments.

```python
class MathOps:
    # Simulated Overloading using default arguments
    def add(self, a, b=0, c=0):
        return a + b + c

m = MathOps()
print(m.add(2, 3))        # Output: 5
print(m.add(2, 3, 4))     # Output: 9

# Same method name 'add', behaves differently depending on arguments.
```

### 4.2 Method Overriding (Run-time Polymorphism)

- When a subclass defines a method with the same name and parameters as a method in its parent class.
- The subclass's method "overrides" the parent's version.
- Enables dynamic behavior — the method called depends on the object type at runtime.

```python
class Animal:
    def speak(self):
        return "Some generic sound"

class Dog(Animal):
    def speak(self):              # Overrides parent method
        return "Woof!"

class Cat(Animal):
    def speak(self):              # Overrides parent method
        return "Meow!"

animals = [Dog(), Cat()]
for a in animals:
    print(a.speak())              # Output: Woof! / Meow!

# → Same interface (speak), different behavior at runtime.
```

---

## Summary of Polymorphism

POLYMORPHISM → "Many forms" — one name, many behaviors.

- **OVERLOADING (Compile-time)**
  - Same method name, different parameters
  - Chosen at compile-time
  - Not truly supported in Python (simulated)
- **OVERRIDING (Run-time)**
  - Same method name, same parameters in subclass
  - Chosen at runtime based on object type
  - Fully supported in Python

Quick Comparison Table:

| Type | Binding Time | Same Parameters? | Python Support |
|------|--------------|------------------|----------------|
| Overloading | Compile-time | No | ⚠️ Simulated |
| Overriding | Run-time | Yes | ✔️ Yes |

---

## ✅ Summary

- **Encapsulation** → Hide internal details, provide methods.
- **Abstraction** → Focus on WHAT, not HOW.
- **Inheritance** → Reuse and extend existing code.
- **Polymorphism** → One interface, multiple implementations.
