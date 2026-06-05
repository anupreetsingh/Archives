# What is Class Inheritance in Python OOP?

- Inheritance allows one class (child/subclass) to use the attributes and methods of another class (parent/superclass).
- It helps in code reuse and models "is-a" relationships. Example: A Dog is an Animal.

## Parent and Child Classes

```python
class Animal:
    def __init__(self, species):
        # Instance variable
        self.species = species

    def make_sound(self):
        # Parent method (can be inherited or overridden)
        print("Some generic animal sound")


class Dog(Animal):
    def __init__(self, name, age):
        # Call the parent constructor using super()
        super().__init__("Dog")
        self.name = name
        self.age = age

    # Method overriding: child changes parent behavior
    def make_sound(self):
        print("Woof!")

    # Child's own method
    def fetch(self):
        print(f"{self.name} is fetching the ball!")
```

## Object Creation and Method Calls

```python
dog1 = Dog("Buddy", 3)

print(dog1.species)   # Inherited from Animal → Dog
dog1.make_sound()     # Overridden method → Woof!
dog1.fetch()          # Defined only in Dog
```

## Multiple Inheritance Example

```python
class GuardAnimal:
    def guard(self):
        print("Guarding the house!")


class WorkingDog(Dog, GuardAnimal):
    def work(self):
        print(f"{self.name} is working hard!")


wdog = WorkingDog("Rex", 4)
wdog.make_sound()   # From Dog → Woof!
wdog.guard()        # From GuardAnimal
wdog.work()         # Own method
```

## Polymorphism Example

```python
class Cat(Animal):
    def __init__(self, name):
        super().__init__("Cat")
        self.name = name

    def make_sound(self):
        print("Meow!")


animals = [Dog("Buddy", 3), Cat("Luna")]

# Both Dog and Cat share the same interface (make_sound),
# but behave differently → polymorphism
for animal in animals:
    animal.make_sound()
```

## Class Attributes and Inheritance

```python
class A:
    kind = "generic"   # Class attribute

class B(A):
    pass

print(A.kind)  # generic
print(B.kind)  # generic (inherited, not copied)

# If parent class attribute changes, child sees it too
A.kind = "updated"
print(B.kind)  # updated

# If child defines its own attribute, it hides the parent's
B.kind = "special"
print(A.kind)  # updated
print(B.kind)  # special
```

## Key Notes about Inheritance

1. A parent class defines shared behavior.
2. A child class inherits from it and can add/override features.
3. `super()` calls parent methods from the child.
4. Class attributes are shared via lookup, not copied.
5. Python supports single, multiple inheritance, and polymorphism.
6. Encourages code reuse, clean design, and extensibility.

## Related Concepts

- Abstract Base Classes (ABC) define methods without implementation that children must implement.
- Composition is an alternative to inheritance: instead of "is-a", it models "has-a".
