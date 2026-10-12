# Object Oriented Programming in C++ — Four Pillars

These are the same four principles as in Python (see [OOP Four Pillars](<../../Python/OOP/OOP Four Pillars.md>)). C++ supports all four, but with stricter, compiler-enforced versions of each, and with one major difference: **runtime polymorphism is opt-in**, through the `virtual` keyword.

Better to be familiar with [Classes and Objects](<Classes and Objects.md>) first.

## 1. Encapsulation

- Restricting direct access to data members, and providing controlled access through member functions.
- Hides implementation details, so the class can maintain its **invariants** (rules that must always hold, like "balance is never negative").

```cpp
class BankAccount {
public:
    BankAccount(std::string owner, double balance = 0)
        : owner_(owner), balance_(balance) {}

    void deposit(double amount) {
        if (amount > 0) balance_ += amount;
    }

    bool withdraw(double amount) {
        if (amount <= balance_) {
            balance_ -= amount;
            return true;
        }
        std::cout << "Insufficient funds!\n";
        return false;
    }

    double balance() const { return balance_; }   // getter: read-only access

private:
    std::string owner_;
    double balance_;                              // private: only BankAccount's member functions can touch it
};

// Example:
BankAccount acc("Alice", 100);
acc.deposit(50);
std::cout << acc.balance();   // 150
// acc.balance_;              // ❌ ERROR: 'balance_' is a private member of 'BankAccount'
```

The Python version used `__balance`, which is only name-mangled and still reachable as `acc._BankAccount__balance`. C++'s `private` is enforced at compile time, and there's no way around it from outside the class.

| Access | Who can use it |
|---|---|
| `public` | Everyone |
| `protected` | The class itself and its derived classes |
| `private` | Only the class itself (and its `friend`s) |

## 2. Abstraction

- Exposing only what the user of a type needs, and hiding how it's done.
- In Python, it's achieved with `ABC` and `@abstractmethod`. In C++, with **pure virtual functions**.

A **pure virtual function** is declared with `= 0`. It has no implementation in the base class, and every concrete derived class must provide one. A class with at least one pure virtual function is an **abstract class**, and can't be instantiated:

```cpp
class Vehicle {
public:
    virtual ~Vehicle() = default;               // virtual destructor: required for base classes (section 4.4)
    virtual std::string move() const = 0;       // pure virtual: "every Vehicle can move, but there's no generic way"
};

class Car : public Vehicle {
public:
    std::string move() const override { return "Driving on the road 🚗"; }
};

class Boat : public Vehicle {
public:
    std::string move() const override { return "Sailing on water ⛵"; }
};

// Example:
std::vector<std::unique_ptr<Vehicle>> vehicles;
vehicles.push_back(std::make_unique<Car>());
vehicles.push_back(std::make_unique<Boat>());

for (const auto& v : vehicles) {
    std::cout << v->move() << "\n";
}
// Driving on the road 🚗
// Sailing on water ⛵

Vehicle v;   // ❌ ERROR: variable type 'Vehicle' is an abstract class
             //    note: unimplemented pure virtual method 'move' in 'Vehicle'
```

| Python | C++ |
|---|---|
| `class Vehicle(ABC):` | A class with at least one pure virtual function |
| `@abstractmethod def move(self): pass` | `virtual std::string move() const = 0;` |
| `TypeError: Can't instantiate abstract class` at runtime | Compile error |

A class made **only** of pure virtual functions is an **interface**, the C++ counterpart of a Python `Protocol` or a Java `interface`.

> Unreal uses the same idea through `UINTERFACE` for things like `IInteractable`, and with `UCLASS(Abstract)` for base classes that must not be placed in a level directly.

## 3. Inheritance

- A child (derived) class gets the data members and member functions of its parent (base) class.
- Promotes code reuse. The child can add new members, or **override** virtual functions.

```cpp
class Animal {
public:
    explicit Animal(std::string name) : name_(name) {}
    virtual ~Animal() = default;

    std::string eat() const { return name_ + " is eating."; }

protected:                     // accessible in Animal AND in classes derived from it
    std::string name_;
};

class Dog : public Animal {    // Dog inherits from Animal
public:
    explicit Dog(std::string name) : Animal(name) {}   // call the base constructor in the initializer list

    std::string bark() const { return name_ + " is barking! 🐶"; }   // can use the protected name_
};

// Example:
Dog dog("Buddy");
std::cout << dog.eat();    // Buddy is eating.       (inherited)
std::cout << dog.bark();   // Buddy is barking! 🐶   (child-specific)
```

Things that differ from Python:

- **Calling the base constructor**: Python calls `super().__init__(name)` inside `__init__`. C++ calls `Animal(name)` in the **initializer list**, and the base is always constructed **before** the derived class's members and body (see the construction order in [Object Lifetime and RAII](<Object Lifetime and RAII.md#2-the-order-of-construction-and-destruction>)). If you don't call one, the base's default constructor is used, and it's a compile error if there isn't one.
- **Inheriting constructors**: `using Animal::Animal;` inside `Dog` reuses all of `Animal`'s constructors.
- **Calling the base version** of a member function: Python writes `super().speak()`, C++ writes `Animal::speak()`.

### `public` Inheritance

`class Dog : public Animal` keeps `Animal`'s public members public in `Dog`, which means "a Dog **is an** Animal". That's almost always what you want. (`private` and `protected` inheritance exist but are rare. Note that `class Dog : Animal` without a keyword defaults to `private` inheritance, so always write `public`.)

> In Unreal, every gameplay class inherits from an engine class: `class AGuardCharacter : public ACharacter`. When you override an engine function like `BeginPlay`, you must call `Super::BeginPlay();`. `Super` is a type alias Unreal defines for the base class.

### Multiple Inheritance

C++ allows a class to inherit from several bases, as Python does:

```cpp
class Amphibious : public Car, public Boat { };
```

Unlike Python, there's no MRO (method resolution order) linearization. If both bases provide the same member, using it is an ambiguity error until you qualify it (`Car::move()`). The **diamond problem** (two bases that share a common base) needs `virtual` inheritance to avoid two copies of that shared base. In practice, keep multiple inheritance to "one real base class + interfaces", which is also the rule Unreal follows.

## 4. Polymorphism

**Definition:**

- "Many forms" → the same call behaves differently depending on the object.
- One interface represents multiple underlying forms.

Like the Python note says, there are two kinds:

1. **Compile-time polymorphism** → function overloading (and templates). The compiler chooses.
2. **Run-time polymorphism** → overriding `virtual` functions. The choice is made at runtime, based on the object's actual type.

### 4.1 Function Overloading (Compile-time Polymorphism)

The Python note had to show this in Java, because Python doesn't support it. C++ does:

```cpp
class MathOps {
public:
    int add(int a, int b) { return a + b; }
    int add(int a, int b, int c) { return a + b + c; }
    double add(double a, double b) { return a + b; }
};

MathOps m;
m.add(2, 3);        // 5     add(int, int)
m.add(2, 3, 4);     // 9     add(int, int, int)
m.add(2.5, 1.5);    // 4.0   add(double, double)
```

The compiler picks the overload from the argument types, at compile time, and each overload is a separate function in the compiled program (see [Functions](<../Fundamentals/Functions.md#function-overloading>)). Templates are the other form of compile-time polymorphism (see [Templates](<../Fundamentals/Templates.md>)).

### 4.2 Method Overriding (Run-time Polymorphism)

A derived class **overrides** a base class's `virtual` function by providing its own version with the same signature. When the function is called through a **base-class pointer or reference**, the version of the object's **actual** type runs:

```cpp
class Animal {
public:
    virtual ~Animal() = default;
    virtual std::string speak() const { return "Some generic sound"; }   // virtual: may be overridden
};

class Dog : public Animal {
public:
    std::string speak() const override { return "Woof!"; }               // override: replaces it for Dogs
};

class Cat : public Animal {
public:
    std::string speak() const override { return "Meow!"; }
};

std::vector<std::unique_ptr<Animal>> animals;
animals.push_back(std::make_unique<Dog>());
animals.push_back(std::make_unique<Cat>());

for (const auto& a : animals) {
    std::cout << a->speak() << "\n";   // Woof! / Meow!
}
// → Same interface (speak), different behavior at runtime.
```

#### Without `virtual`, There's No Override

In Python, **every** method can be overridden, and the lookup always uses the object's actual class. In C++, a member function is only dispatched at runtime if it's declared `virtual`. A non-virtual function with the same name in a derived class merely **hides** the base version, and which one runs depends on the **static type** of the pointer or reference:

```cpp
class Animal {
public:
    virtual std::string speak() const { return "Some generic sound"; }
    std::string describe() const { return "I am an animal"; }    // NOT virtual
};

class Dog : public Animal {
public:
    std::string speak() const override { return "Woof!"; }
    std::string describe() const { return "I am a dog"; }        // hides, doesn't override
};

Dog dog;
Animal& ref = dog;

ref.speak();      // "Woof!"           virtual: uses the object's real type (Dog)
ref.describe();   // "I am an animal"  non-virtual: uses the reference's type (Animal)
dog.describe();   // "I am a dog"      called on a Dog directly
```

How `virtual` does this under the hood (vtables), and what it costs, is the subject of [Virtual Functions and Vtables](<Virtual Functions and Vtables.md>).

#### `override` and `final`

Always mark overrides with `override`. It makes the compiler check that a matching virtual function actually exists in the base, which catches silent typos:

```cpp
class Base {
public:
    virtual void update(float dt);
};

class Derived : public Base {
public:
    void update(double dt) override;   // ❌ ERROR: non-virtual member function marked 'override'
                                       //    hides virtual member function
                                       //    (double ≠ float, so without override this would be a NEW function)
};
```

`final` stops further overriding, or further inheritance:

```cpp
class Mid : public Base { void update(float dt) override final; };   // classes below Mid can't override update
class Sealed final { };                                              // no class can inherit from Sealed
```

### 4.3 Polymorphism Needs Pointers or References: Object Slicing

Runtime polymorphism only works through a **pointer** or **reference** to the base class. Copying a derived object into a base-class **value** keeps only the base part. This is called **slicing**:

```cpp
void by_value(Animal a) {          // takes an Animal BY VALUE
    std::cout << a.speak();
}

Dog dog;
by_value(dog);                     // "Some generic sound": the Dog parts were sliced off when copying

Animal sliced = dog;               // same thing
sliced.speak();                    // "Some generic sound"
```

```text
Dog object:        [ Animal part | Dog part ]
                         |
Animal a = dog:    [ Animal part ]            <- a genuine Animal now, with Animal's vtable
```

This can't happen in Python, where variables are references, so nothing is ever copied into a smaller "box". In C++, pass polymorphic objects by `const Base&`, `Base*`, or a smart pointer, and store them in containers as `std::unique_ptr<Base>`, never as `std::vector<Base>`.

### 4.4 Virtual Destructors

If an object is deleted through a pointer to its base class, the base class **must** have a `virtual` destructor. Otherwise only the base destructor runs:

```cpp
struct Base {
    ~Base() { std::cout << "~Base\n"; }              // NOT virtual
};

struct Derived : Base {
    std::string big = std::string(100, 'x');         // owns heap memory
    ~Derived() { std::cout << "~Derived\n"; }
};

Base* b = new Derived();
delete b;     // prints only "~Base". ~Derived never runs, so `big`'s buffer leaks (formally: UB)
```

```cpp
struct Base {
    virtual ~Base() = default;                       // fix: virtual destructor
};
// delete b;  ->  ~Derived, then ~Base
```

Rule: **any class meant to be used as a polymorphic base gets `virtual ~Base() = default;`**. Clang warns when it can spot the problem, if the base has other virtual functions: `delete called on non-final 'Base' that has virtual functions but non-virtual destructor`.

> AddressSanitizer reports the leaked buffer on Linux, through its built-in leak checker. On macOS, leak detection isn't supported, so rely on the compiler warning, or on Apple's `leaks --atExit -- ./program` (see [Undefined Behavior and Sanitizers](<../Memory/Undefined Behavior and Sanitizers.md#other-sanitizers>)).

### 4.5 Downcasting with `dynamic_cast`

Going from a base pointer back to a derived pointer is a **downcast**. `dynamic_cast` checks at runtime whether the object really is of that type, and returns `nullptr` if not. It plays the role of Python's `isinstance()` check:

```cpp
Animal* p = &dog;

if (Dog* d = dynamic_cast<Dog*>(p)) {   // like: if isinstance(p, Dog):
    d->bark();                          // safe: p really points to a Dog
}

Animal* pc = &cat;
dynamic_cast<Dog*>(pc)                  // nullptr: a Cat isn't a Dog
```

`dynamic_cast` only works on polymorphic types (classes with at least one virtual function). `static_cast<Dog*>(p)` also compiles, but performs **no check**: if `p` isn't really a `Dog`, using the result is UB.

Needing lots of downcasts usually means a virtual function is missing from the base class.

> In Unreal, use `Cast<AGuardCharacter>(Actor)` instead. It's checked like `dynamic_cast` (returning `nullptr` on failure), but uses Unreal's reflection system.

## Summary

| Pillar | Python | C++ |
|---|---|---|
| Encapsulation | `_name` / `__name` conventions | `public` / `protected` / `private`, enforced by the compiler |
| Abstraction | `ABC`, `@abstractmethod`, error at instantiation | Pure virtual `= 0`, abstract classes, compile error |
| Inheritance | `class Dog(Animal)`, `super().__init__()` | `class Dog : public Animal`, base constructor in the initializer list |
| Compile-time polymorphism | Not supported (rebinding replaces methods) | Function overloading, templates |
| Run-time polymorphism | Every method, always | Only `virtual` functions, only through pointers/references |
| Type check | `isinstance(obj, Dog)` | `dynamic_cast<Dog*>(ptr) != nullptr` |
