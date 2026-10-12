# Classes and Objects in C++

- A **class** is a blueprint for creating objects. It defines what data each object holds (**data members**) and what it can do (**member functions**).
- An **object** is an instance of a class, holding actual values.

That's the same as Python (see [OOP-Self](<../../Python/OOP/OOP-Self .md>)). The differences are in the details: C++ fixes every member and its type when the class is defined, controls access in the compiler, and gives you an implicit `this` instead of an explicit `self`.

## Defining a Class

The `Dog` class from the Python note, in C++:

```cpp
#include <iostream>
#include <string>

class Dog {
public:                                   // everything below is accessible from outside the class
    Dog(std::string name, int age)        // CONSTRUCTOR: same name as the class, no return type
        : name_(name), age_(age) {}       // member initializer list: initializes the data members

    void speak() const {                  // member function (Python: instance method)
        std::cout << "My name is " << name_ << " and I am " << age_ << " years old.\n";
    }

    void birthday() {                     // modifies the object, so it's not const
        age_ += 1;
        std::cout << "Happy Birthday " << name_ << "! You are now " << age_ << ".\n";
    }

private:                                  // accessible only from inside Dog's member functions
    std::string name_;                    // data members (Python: instance attributes)
    int age_;
};                                        // the semicolon after a class definition is required
```

```cpp
Dog dog1("Buddy", 3);
Dog dog2("Luna", 5);

dog1.speak();      // My name is Buddy and I am 3 years old.
dog2.speak();      // My name is Luna and I am 5 years old.

dog1.birthday();   // Happy Birthday Buddy! You are now 4.
dog2.birthday();   // Happy Birthday Luna! You are now 6.
```

Here, compared with Python:

- **No `self` parameter.** Member functions receive the object implicitly, as `this` (below).
- **No `self.` prefix.** Inside a member function, `age_` already means "this object's `age_`", because the class scope is searched before the global scope (see [Scope and Namespaces](<../Fundamentals/Scope and Namespaces.md#4-class-scope>)).
- **Data members are declared up front**, with types, in the class body. In Python, `__init__` creates attributes by assigning `self.name = name`, and any method can add new ones later. In C++, the member list is fixed at compile time, so every `Dog` has exactly the same layout (see [Object Memory Layout](<../Memory/Object Memory Layout.md>)).
- **The trailing `_`** on `name_` and `age_` is a common convention for marking private data members, so a member can't be confused with a parameter called `name`. Unreal doesn't use it; Unreal members are PascalCase like `Health`.

## `this`: C++'s `self`

Inside every non-static member function, the keyword `this` is a **pointer** to the object the function was called on:

```text
dog1.speak()   is compiled roughly as   Dog::speak(&dog1)   with this = &dog1
```

That's the same translation Python does when it turns `dog1.speak()` into `Dog.speak(dog1)` (see [Why is `self` needed?](<../../Python/OOP/OOP-Self .md#why-is-self-needed>)). C++ just hides the parameter.

```cpp
class Counter {
public:
    void inc() {
        value++;            // implicit: this->value
        this->value++;      // explicit: same member
        (*this).value++;    // dereference this, then use .
    }
    int value = 0;
};

Counter c;
c.inc();                    // c.value == 3
```

You need to write `this->` explicitly only when a parameter or local has the same name as a member, or to return the object itself:

```cpp
class Player {
public:
    void set_health(int health) { this->health = health; }   // this->health is the member, health is the parameter
    Player& heal() { health = 100; return *this; }           // return a reference to this object, to chain calls
private:
    int health = 100;
};

player.heal().set_health(50);   // chaining
```

In a `const` member function, `this` is a pointer to `const`, which is how the compiler stops `const` functions from modifying members (see [Const Correctness](<../Memory/Const Correctness.md#4-const-member-functions>)).

## Access Specifiers

C++ enforces access in the compiler, unlike Python's naming conventions (`_name`, `__name`):

| Specifier | Accessible from | Python analogue |
|---|---|---|
| `public:` | Anywhere | Normal attribute |
| `protected:` | This class and classes derived from it | `_name` (by convention only) |
| `private:` | Only this class's own member functions (and friends) | `__name` (name mangling, still reachable) |

```cpp
Dog d("Buddy", 3);
d.name_;           // ERROR: 'name_' is a private member of 'Dog'
```

There's no way around `private` short of changing the class. Python's `__balance` can still be reached as `obj._BankAccount__balance`.

The split follows **interface vs. implementation**. The `public` section is the interface: what outside code is allowed to use. The `private` section is the implementation: how the class does its job. This is also why the `public` section is conventionally listed first, so a reader sees how the class is meant to be used before its internals. Data often stays private so that only the class's own member functions can change it, which lets the class keep its invariants (rules that must always hold, like "`age_` is never negative").

### `class` vs `struct`

`class` and `struct` are the **same thing** in C++, except for the default access:

```cpp
struct Point {      // members are PUBLIC by default
    int x = 0;
    int y = 0;
};

class Point2 {      // members are PRIVATE by default
    int x = 0;      // private!
    int y = 0;
};
```

By convention, `struct` is used for plain bundles of public data with little behavior (like a Python `dataclass` or `namedtuple`), and `class` for types with invariants, private state, and member functions.

```cpp
Point p{3, 4};               // aggregate initialization: members in declaration order
Point q;                     // uses the default member initializers: {0, 0}
Point r{.x = 1, .y = 2};     // C++20 designated initializers
```

> Unreal uses `struct` with `USTRUCT()` for value types like `FHitResult`, and `class` with `UCLASS()` for objects like `AActor`.

## Constructors

A **constructor** runs when an object is created and initializes its members. It's the C++ counterpart of `__init__`, with the same name as the class and no return type.

### The Member Initializer List

Members should be initialized in the **initializer list**, after the `:`, rather than assigned in the body:

```cpp
class Dog {
public:
    Dog(std::string name, int age)
        : name_(name), age_(age)      // initialize: each member is constructed directly with its value
    {
        // body: runs AFTER all members are already initialized
    }

    // Avoid: members are default-constructed first, then assigned over
    // Dog(std::string name, int age) { name_ = name; age_ = age; }
private:
    std::string name_;
    int age_;
};
```

The initializer list is **required** for members that can't be assigned after construction: `const` members, reference members, and members of class types with no default constructor.

Members are always initialized in the order they're **declared in the class**, not the order you list them in the initializer list:

```cpp
struct A {
    int x;
    int y;
    A(int v) : y(v), x(y) {}   // x is initialized FIRST, from y, which is still uninitialized
};
// warning: field 'y' will be initialized after field 'x' [-Wreorder-ctor]
// warning: field 'y' is uninitialized when used here
```

### Default Member Initializers

You can give members default values right where they're declared:

```cpp
class Player {
public:
    Player() = default;                         // constructor uses all the defaults below
    explicit Player(std::string name) : name_(name) {}   // constructor sets name_ from the argument; the rest use their defaults

private:
    std::string name_ = "Unnamed";
    int health_ = 100;
    float speed_ = 4.5f;
};
```

### Overloaded Constructors

Python has one `__init__` (with defaults or `@classmethod` factories as alternatives). C++ constructors can be **overloaded** like any function (see [Functions](<../Fundamentals/Functions.md#function-overloading>)):

```cpp
class Color {
public:
    Color() : r_(0), g_(0), b_(0) {}                  // default: black
    Color(int r, int g, int b) : r_(r), g_(g), b_(b) {}
    explicit Color(int gray) : Color(gray, gray, gray) {}   // delegating constructor: reuses another one
private:
    int r_, g_, b_;
};

Color black;
Color teal(0, 128, 128);
Color mid_gray(128);
```

### The Default Constructor

A **default constructor** takes no arguments. If you write **no** constructors at all, the compiler generates one for you. Once you write **any** constructor, it doesn't, so this fails:

```cpp
Dog d;   // ERROR: no matching constructor for initialization of 'Dog'
         // Dog only has Dog(std::string, int)
```

Add `Dog() = default;` (or a real default constructor) if objects should be creatable without arguments. Containers like `std::vector<Dog> v(10);` need one too.

### `explicit`

A constructor that can be called with one argument doubles as an **implicit conversion** (see [Type Conversion](<../Fundamentals/Type Conversion.md#4-conversions-for-your-own-types>)). Mark it `explicit` unless you really want that:

```cpp
class Player {
public:
    explicit Player(std::string name);
};

void greet(const Player& p);
greet(std::string("Ava"));   // ERROR with explicit. Without it, a temporary Player would be built silently
```

The full lifecycle (constructors, destructors, and the order they run in) is covered in [Object Lifetime and RAII](<Object Lifetime and RAII.md>).

## Defining Member Functions Outside the Class

In real projects, the class definition goes in a header and the member function bodies go in a `.cpp` file (see [Headers and Source Files](<../Fundamentals/Headers and Source Files.md#a-typical-header--source-pair>)). The `ClassName::` prefix says which class a function belongs to:

```cpp
// Dog.h
#pragma once
#include <string>

class Dog {
public:
    Dog(std::string name, int age);
    void speak() const;
    void birthday();
private:
    std::string name_;
    int age_;
};
```

```cpp
// Dog.cpp
#include "Dog.h"
#include <iostream>

Dog::Dog(std::string name, int age) : name_(name), age_(age) {}

void Dog::speak() const {         // const must be repeated here
    std::cout << "My name is " << name_ << "\n";
}

void Dog::birthday() {
    age_ += 1;
}
```

Member functions defined **inside** the class body are implicitly `inline`, so they can stay in the header. That's fine for one-line getters.

## Static Members: Class Attributes and Static Methods

A `static` data member is **one variable shared by the whole class**, like a Python class attribute (see [Class Attributes vs. Instance Attributes](<../../Python/OOP/OOP-Self .md#class-attributes-vs-instance-attributes>)). A `static` member function has no `this`, like a Python `@staticmethod`:

```cpp
class Dog {
public:
    Dog(std::string name, int age) : name_(name), age_(age) { ++count_; }

    static int count() { return count_; }                   // like @classmethod/@staticmethod: no object needed
    static bool is_valid_age(int age) { return age >= 0; }  // utility function: like @staticmethod

private:
    std::string name_;
    int age_;
    static inline int count_ = 0;   // ONE counter shared by every Dog (inline: C++17, can be initialized here)
};

Dog dog1("Buddy", 3);
Dog dog2("Luna", 5);

Dog::count()            // 2: called through the class, with ::
Dog::is_valid_age(-1)   // false
```

| Python | C++ |
|---|---|
| `wheels = 4` in the class body | `static inline int wheels = 4;` |
| `Car.wheels` | `Car::wheels` |
| `@staticmethod def is_positive(n)` | `static bool is_positive(int n)` |
| `@classmethod def from_string(cls, s)` (factory) | `static Car from_string(const std::string& s)` |

One difference from Python: assigning to a static member **through an object** doesn't create a per-object copy. There is only ever the one shared variable.

## Creating Objects: Stack vs Heap

In Python, every object is created on the heap and handled through references. In C++, you choose (see [Stack and Heap](<../Memory/Stack and Heap.md>)):

```cpp
Dog a("Buddy", 3);                         // on the stack: destroyed at the end of the scope
a.speak();

Dog* b = new Dog("Rex", 2);                // on the heap, via a raw pointer: you must delete it
b->speak();
delete b;

auto c = std::make_unique<Dog>("Max", 1);  // on the heap, owned by a smart pointer: deleted automatically
c->speak();
```

Prefer the stack. Use a smart pointer when the object must outlive its scope, or must be handled polymorphically (see [Smart Pointers](<../Memory/Smart Pointers.md>)).

> Unreal objects are the exception to all of this: `UObject`s are created with `NewObject<T>()`, actors with `GetWorld()->SpawnActor<T>()`, and components with `CreateDefaultSubobject<T>()` in constructors. They're never plain stack objects, and the engine's garbage collector manages their lifetime.

## Objects Are Values

Remember [value semantics](<../Fundamentals/What is C++.md#3-value-semantics>): assigning one object to another **copies** it, member by member:

```cpp
Dog a("Buddy", 3);
Dog b = a;           // b is a separate Dog, a copy of a
b.birthday();        // only b ages; a is still 3
```

In Python, `b = a` would make `b` another name for the same dog. What gets copied, and how to control it, is covered in [Copy and Move Semantics](<Copy and Move Semantics.md>).

## `friend`

A class can grant a specific function or class access to its private members by declaring it a `friend`:

```cpp
class Vault {
    friend class Auditor;            // Auditor's member functions can read Vault's private members
    friend void inspect(const Vault& v);
private:
    int gold_ = 1000;
};
```

Use it sparingly. Its most common legitimate use is `operator<<` for printing (see [Operator Overloading](<Operator Overloading.md>)).

## Summary of Key Points

1. Members are declared with types in the class body. An object's layout is fixed at compile time.
2. `this` is a pointer to the current object, passed implicitly. You rarely need to write it.
3. `public` / `protected` / `private` are enforced by the compiler. `struct` defaults to public, `class` to private.
4. Initialize members in the constructor's initializer list, in declaration order.
5. Writing any constructor removes the compiler-generated default constructor.
6. Mark single-argument constructors `explicit`, and read-only member functions `const`.
7. `static` members are shared by the whole class, like Python class attributes.
8. Objects are values: assignment copies.
