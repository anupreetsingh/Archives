# Design Patterns

## 🟦 Singleton Pattern

- Ensures only ONE instance of a class exists.
- Provides a global access point to that instance.
- Useful for shared resources: Logger, Config, DB Connection.

```python
class Singleton:
    _instance = None  # Class-level storage

    def __new__(cls, *args, **kwargs):
        if cls._instance is None:  # First time creation
            cls._instance = super().__new__(cls)
        return cls._instance

# Test Singleton
s1 = Singleton()
s2 = Singleton()
print(s1 is s2)  # True → both point to the same object
```

## 🟩 Observer Pattern

- Defines "one-to-many" relationship.
- When Subject changes → all Observers are notified.
- Use case: Event systems, News feed, GUIs.

```python
# Subject (Publisher)
class Subject:
    def __init__(self):
        self._observers = []

    def attach(self, observer):
        self._observers.append(observer)

    def notify(self, message):
        for observer in self._observers:
            observer.update(message)

# Observer Interface
class Observer:
    def update(self, message):
        raise NotImplementedError

# Concrete Observers
class EmailNotifier(Observer):
    def update(self, message):
        print(f"📧 Email: {message}")

class SMSNotifier(Observer):
    def update(self, message):
        print(f"📱 SMS: {message}")

# Usage
subject = Subject()
subject.attach(EmailNotifier())
subject.attach(SMSNotifier())

subject.notify("New article published!")
# 📧 Email: New article published!
# 📱 SMS: New article published!
```

## 🟥 Factory Pattern

- Centralizes object creation to a Factory class.
- Lets a method/class decide which concrete object to return.
- Use case: Plugging in new object types without changing client code.

```python
# Product classes
class Dog:
    def speak(self): return "Woof!"

class Cat:
    def speak(self): return "Meow!"

# Factory class
class AnimalFactory:
    @staticmethod
    def create_animal(animal_type):
        if animal_type == "dog":
            return Dog()
        elif animal_type == "cat":
            return Cat()
        else:
            raise ValueError("Unknown animal type")

# Usage
animal1 = AnimalFactory.create_animal("dog")
animal2 = AnimalFactory.create_animal("cat")

print(animal1.speak())  # Woof!
print(animal2.speak())  # Meow!
```
