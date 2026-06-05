# 🧩 Composition

- A design principle where one class "has" or "uses" another class.
- Instead of inheriting behavior (like inheritance), it delegates work to components (objects from other classes).
- Encourages modularity, reusability, and flexibility.

📌 **KEY IDEA: "Has-a" relationship** — Example: Car HAS an Engine (not Car IS an Engine)

---

## ✅ Basic Example

```python
class Engine:
    def start(self):
        print("Engine starting...")

class Car:
    def __init__(self):
        # Car *contains* an Engine
        self.engine = Engine()

    def drive(self):
        # Car delegates the work to Engine
        self.engine.start()
        print("Car is moving!")

c = Car()
c.drive()
# Output:
# Engine starting...
# Car is moving!
```

## 🎯 Benefits

- More flexible than inheritance
- Swap out components easily (e.g., ElectricEngine, DieselEngine)
- Reduces tight coupling
- Follows "Composition over Inheritance" principle

## ⚡ Flexible Example

```python
class ElectricEngine:
    def start(self):
        print("Electric engine humming...")

class DieselEngine:
    def start(self):
        print("Diesel engine roaring...")

class Car:
    def __init__(self, engine):
        # Car can have any kind of Engine
        self.engine = engine

    def drive(self):
        self.engine.start()
        print("Car is moving!")

# Now we can plug in different engines
tesla = Car(ElectricEngine())
bmw = Car(DieselEngine())

tesla.drive()
bmw.drive()
# Output:
# Electric engine humming...
# Car is moving!
# Diesel engine roaring...
# Car is moving!
```
