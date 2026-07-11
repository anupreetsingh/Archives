# Software Design Techniques

Software design techniques are ways of organizing code and relationships between
objects. They help control coupling, reuse behavior, and make software easier to
change.

They are different from:

- **Language features**, such as classes, methods, and Python's inheritance syntax.
- **Design principles**, which provide guidance, such as "favor composition over
  inheritance."
- **Design patterns**, which are reusable arrangements of several classes or
  objects, such as Strategy and Factory.

## Organizing Relationships Between Objects

### Inheritance

Inheritance creates an **is-a** relationship. A subclass specializes a parent type
and inherits its behavior.

```python
class Vehicle:
    def move(self):
        print("The vehicle is moving")


class Car(Vehicle):
    pass
```

A `Car` is a `Vehicle`, so code expecting a `Vehicle` should also work correctly
with a `Car`. Inheritance is most appropriate when:

- The subclass represents a genuine subtype of the parent.
- The subclass can safely substitute for the parent.
- The shared behavior is fundamental to what the object is.

Inheritance creates a strong relationship between the classes. Changes to the
parent can affect every subclass, so inheritance should not be used only to avoid
duplicating code.

### Composition

Composition creates a **has-a** or **uses-a** relationship. An object contains
other objects and delegates work to them.

```python
class PetrolEngine:
    def start(self):
        print("Petrol engine starting")


class Car:
    def __init__(self):
        self.engine = PetrolEngine()

    def drive(self):
        self.engine.start()
        print("The car is moving")
```

A `Car` has an engine rather than being an engine. The engine's implementation can
be separated from the responsibilities of the car.

Composition is usually preferable when:

- One object needs the behavior of another but is not its subtype.
- A component may need to be replaced independently.
- Behavior should be assembled from smaller parts.
- Inheritance would expose unnecessary parent behavior or create tight coupling.

Composition and inheritance are not mutually exclusive. A design can use
inheritance to model genuine subtypes and composition to assemble their behavior.

#### Supplying Composed Dependencies

Once an object uses another object, the design must determine who creates and
supplies that dependency.

##### Internal Construction

In the previous example, `Car` constructs `PetrolEngine` itself:

```python
class Car:
    def __init__(self):
        self.engine = PetrolEngine()
```

This is simple, but it tightly couples `Car` to one engine implementation. Changing
the engine requires changing `Car`.

Internal construction is reasonable when the dependency is a private
implementation detail, is inexpensive to create, and is unlikely to vary.

##### Dependency Injection

Dependency injection means that a dependency is created outside an object and
supplied to it. Constructor injection passes the dependency when the object is
created:

```python
class ElectricEngine:
    def start(self):
        print("Electric engine starting")


class PetrolEngine:
    def start(self):
        print("Petrol engine starting")


class Car:
    def __init__(self, engine):
        self.engine = engine

    def drive(self):
        self.engine.start()
        print("The car is moving")


electric_car = Car(ElectricEngine())
petrol_car = Car(PetrolEngine())
```

`Car` now depends on the behavior it needs—an object with `start()`—instead of
constructing one specific engine. This makes engines replaceable and makes `Car`
easier to test with a test double.

Dependency injection is a technique used to implement flexible composition; it is
not an alternative to composition.

Other ways an object can obtain dependencies include:

- **Internal construction:** the object creates the dependency itself.
- **Dependency lookup or service locator:** the object requests the dependency
  from a shared registry. This allows replacement but hides the dependency.
- **Global or singleton access:** the object accesses shared state directly. This
  is convenient but creates hidden coupling and can make tests interfere with one
  another.

###### Factories and Dependency Injection

A factory centralizes construction when choosing or assembling objects is complex.
It can create the dependency and inject it into the composed object:

```python
def create_car(engine_type):
    if engine_type == "electric":
        engine = ElectricEngine()
    elif engine_type == "petrol":
        engine = PetrolEngine()
    else:
        raise ValueError("Unknown engine type")

    return Car(engine)
```

A factory is therefore usually complementary to dependency injection rather than
an alternative to it. The factory decides how the object graph is assembled; the
constructed objects receive their dependencies.

## Choosing a Technique

Use the relationship and expected change as the primary guides:

| Question | Likely technique |
| --- | --- |
| Is this object genuinely a specialized form of another type? | Inheritance |
| Does this object contain or use another object? | Composition |
| Should a composed component be replaceable or independently testable? | Dependency injection |
| Is construction or selection of components complex? | Factory |
| Is the dependency small, private, and unlikely to vary? | Internal construction may be sufficient |

"Favor composition over inheritance" does not mean inheritance is always wrong.
It means inheritance should model a valid subtype relationship, while composition
is generally safer when the goal is only behavior reuse or replaceable parts.

## Relationship to Design Patterns

Design techniques are building blocks used by design patterns. For example, the
Strategy pattern uses composition and dependency injection so that an object's
behavior can be replaced. The pattern describes the larger collaboration between
objects; composition and dependency injection are the techniques used to
implement it.
