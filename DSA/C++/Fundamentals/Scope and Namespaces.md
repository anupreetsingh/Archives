# Scope, Namespaces, and Lifetime in C++

This note explains where a C++ name is visible, how the compiler decides which declaration a name refers to, and how long the object behind a name actually exists.

It's the C++ counterpart of [Namespace and Scope](<../../Python/Fundamentals/Namespace and Scope.md>), but the two languages work very differently underneath:

- In Python, a **namespace** is a real runtime mapping from names to objects (a dict, essentially), and lookup walks those mappings using the LEGB rule while the program runs.
- In C++, names exist only at **compile time**. The compiler resolves every name to a specific declaration while compiling. At runtime there are no names left, only addresses.

So in C++, three separate ideas need to be kept apart:

| Idea | Question it answers | Decided |
|---|---|---|
| **Scope** | Where in the source code can this name be used? | At compile time |
| **Namespace** | Which named group does this name belong to, to avoid clashes? | At compile time |
| **Storage duration / lifetime** | When is the object created and destroyed in memory? | At runtime |

---

## 1. Block Scope

Every pair of braces `{ }` creates a new scope. A name declared inside it is visible from its declaration to the closing brace, and nowhere else:

```cpp
int main() {
    int a = 1;              // visible until main's closing }

    if (a > 0) {
        int b = 2;          // visible only inside this if-block
        std::cout << a + b; // OK: a is visible from the enclosing block
    }

    std::cout << b;         // ERROR: use of undeclared identifier 'b'

    for (int i = 0; i < 3; ++i) { }
    std::cout << i;         // ERROR: i was scoped to the for statement
}
```

This is the main difference from Python, where `if`, `for`, and `while` blocks don't create scopes and loop variables survive the loop (see [How Scope is Defined](<../../Python/Fundamentals/Namespace and Scope.md#how-scope-is-defined>)).

C++ also checks names **in order**. A name can't be used above the line where it's declared, even in the same scope:

```cpp
int main() {
    std::cout << x;   // ERROR: x hasn't been declared YET
    int x = 5;
}
```

This is unlike Python, where the compiler classifies names per function before running it, which leads to `UnboundLocalError` instead.

### Shadowing

An inner scope can declare a name that already exists in an outer scope. The inner one **shadows** the outer one until the inner block ends:

```cpp
int value = 1;                  // global

int main() {
    int value = 2;              // shadows the global
    {
        int value = 3;          // shadows main's value
        std::cout << value;     // 3
        std::cout << ::value;   // 1: a leading :: means "look in the global scope"
    }
    std::cout << value;         // 2: main's value is visible again
}
```

There's no way to reach the shadowed `value = 2` from inside the inner block. That's legal but confusing, and `-Wshadow` warns about it (`declaration shadows a local variable`).

### No `global` or `nonlocal` Needed

Python needs `global` and `nonlocal` to **assign** to an outer name (see [`global`](<../../Python/Fundamentals/Namespace and Scope.md#global>)), because assignment creates a new local. In C++, **declaration and assignment are different statements**, so there's no ambiguity:

```cpp
int count = 0;              // global

void increment() {
    count += 1;             // no declaration here, so this is the global count
}

void reset() {
    int count = 0;          // `int` makes this a NEW local that shadows the global
}
```

`type name = value;` declares, and `name = value;` assigns to whatever `name` already refers to.

## 2. Name Lookup, Simplified

When the compiler meets an **unqualified** name (one without `::`), it searches outward from the point of use and stops at the first scope that declares that name:

```text
current block -> enclosing blocks -> function parameters -> class members (inside member functions)
              -> enclosing namespaces -> global scope
```

It's roughly Python's LEGB walk, with two differences: it happens entirely at compile time, and C++ has no "built-in" scope. The standard library lives in an ordinary namespace, `std`, that you have to name or import.

A **qualified** name, such as `std::cout` or `Player::take_damage`, skips the search and looks directly in the named scope.

## 3. Namespaces

A **namespace** is a named scope used to group related declarations so that names from different libraries don't collide.

```cpp
namespace physics {
    double gravity = 9.81;
    double fall_speed(double t) { return gravity * t; }   // inside the namespace, gravity is found directly
}

namespace audio {
    double gravity = 0.5;   // different variable; no clash with physics::gravity
}

int main() {
    std::cout << physics::gravity;        // 9.81
    std::cout << audio::gravity;          // 0.5
    std::cout << physics::fall_speed(2);  // 19.62
}
```

Conceptually, `::` builds a path, much like Python's dotted module paths:

| Python | C++ |
|---|---|
| `math.sqrt` | `std::sqrt` |
| `shapes.three_d.volume` | `shapes::three_d::volume` |
| A module is a namespace, created by a file | A namespace is declared explicitly with `namespace name { }` and can span many files |

Namespaces are **open**: you can add to the same namespace from any number of files, so a whole library can live in one namespace across hundreds of headers.

### Nested Namespaces

```cpp
namespace game::ai {          // C++17 shorthand for namespace game { namespace ai { ... } }
    int max_guards = 4;
}

std::cout << game::ai::max_guards;   // 4

namespace ga = game::ai;      // namespace alias, like Python's `import numpy as np`
std::cout << ga::max_guards;  // 4
```

### `using` Declarations and Directives

Writing `std::` everywhere gets verbose. There are two ways to bring names in, which correspond closely to Python's two import styles (see [Modules and Packages](<../../Python/Fundamentals/Modules and Packages.md#modules>)):

```cpp
using std::cout;              // using DECLARATION: brings in ONE name. Like `from std import cout`
cout << "hi";

using namespace std;          // using DIRECTIVE: brings in EVERY name in std. Like `from std import *`
string s = "hi";
```

The same advice applies as Python's "avoid `from module import *`". `std` contains thousands of names, and common words like `count`, `distance`, `size`, `max`, and `swap` are among them:

```cpp
#include <iostream>
using namespace std;

int count = 0;           // our global variable

int main() {
    cout << count;       // ERROR: reference to 'count' is ambiguous
}                        // (::count, or std::count from <algorithm>, pulled in through <iostream>)
```

Two namespaces that both declare `gravity` break the same way once both are `using namespace`d.

Guidelines:

- **Never** put `using namespace` in a header. Every file that includes it inherits the pollution.
- In a `.cpp` file, prefer `using std::cout;`-style declarations, or a `using namespace` inside a single function body.
- Just writing `std::` is the norm in professional code.

> Unreal Engine code mostly doesn't use namespaces for gameplay classes. It avoids clashes with **prefixes** instead: `AActor`, `UObject`, `FVector`, `TArray`, `EMovementMode`.

### Anonymous Namespaces: File-Private Names

A namespace with no name makes its contents visible **only inside the current `.cpp` file** (translation unit). It's the C++ way of making a helper "private to this module", similar in spirit to Python's leading-underscore `_helper` convention, but enforced by the linker:

```cpp
// player.cpp
namespace {
    int hidden_counter = 0;                     // other .cpp files can't see or link to this
    int clamp_health(int h) { return h < 0 ? 0 : h; }
}

void Player::take_damage(int amount) {
    health_ = clamp_health(health_ - amount);   // usable anywhere in this file
}
```

Two different `.cpp` files can each have their own `clamp_health` in an anonymous namespace without a duplicate-symbol error. The older way to get the same effect is the `static` keyword on a namespace-scope function or variable (see section 5).

## 4. Class Scope

A class body is also a scope. Members are reached through an object with `.`/`->`, or through the class name with `::`:

```cpp
class Player {
public:
    static int player_count;      // one shared variable for the whole class
    int health = 100;             // one per object
    void take_damage(int amount);
};

int Player::player_count = 0;     // define the static member (in one .cpp file)

void Player::take_damage(int amount) {   // Player:: puts this definition inside class scope...
    health -= amount;                     // ...so `health` is found as a member, without this->
}

Player p;
p.health;                         // instance member, through an object
Player::player_count;             // static member, through the class: like a Python class attribute
```

Inside a member function, lookup checks the class's members **before** globals, which is why `health` above needs no `self.`-style prefix. This differs from Python, where a method body must write `self.health` because the class namespace isn't part of LEGB (see [Classes](<../../Python/Fundamentals/Namespace and Scope.md#classes-1>)). Details in [Classes and Objects](<../OOP/Classes and Objects.md>).

## 5. Storage Duration: How Long a Variable Lives

**Scope** is about where a **name** is visible. **Lifetime** is about when the **object** exists in memory. They often line up for locals, but they're separate ideas. Every object has one of four storage durations:

| Storage duration | Created | Destroyed | Examples |
|---|---|---|---|
| **Automatic** | When execution reaches the declaration | When its block ends | Ordinary local variables and parameters |
| **Static** | Before `main()` (globals), or the first time execution reaches it (static locals) | After `main()` returns | Globals, namespace variables, `static` locals, `static` class members |
| **Dynamic** | When you call `new` (or a container/smart pointer does it for you) | When you call `delete` (or its owner does) | Heap objects. See [Stack and Heap](<../Memory/Stack and Heap.md>) |
| **Thread** | When a thread starts | When it ends | `thread_local` variables |

A small type that prints when it's created and destroyed shows the order:

```cpp
struct Noisy {
    std::string name;
    Noisy(std::string n) : name(n) { std::cout << "create " << name << "\n"; }
    ~Noisy() { std::cout << "destroy " << name << "\n"; }   // destructor: runs when the object dies
};

Noisy global_obj("global");          // static storage duration

void f() {
    static Noisy s("static-local");  // static: created on the FIRST call only
    Noisy l("local");                // automatic: created and destroyed on every call
}

int main() {
    std::cout << "main start\n";
    f();
    f();
    {
        Noisy a("a");
        Noisy b("b");
    }                                // b, then a, destroyed here
    std::cout << "main end\n";
}
```

```text
create global          <- before main() even starts
main start
create static-local    <- first call to f()
create local
destroy local          <- end of first f()
create local           <- second call: the static is NOT created again
destroy local
create a
create b
destroy b              <- reverse order of creation
destroy a
main end
destroy static-local   <- after main() returns
destroy global
```

Things to notice:

- Automatic objects are destroyed in **reverse order** of creation when their block ends. This is guaranteed, and it's what [RAII](<../OOP/Object Lifetime and RAII.md>) is built on.
- A `static` local keeps its **scope** (only `f` can name `s`) but has a **static lifetime** (it outlives every call to `f`).
- Python's lifetime is fuzzy by comparison: an object lives until its reference count drops to zero or the garbage collector reclaims it.

## 6. Linkage: Visible Across Files?

**Linkage** decides whether a name declared in one `.cpp` file refers to the same entity as the same name in another `.cpp` file. It's what the linker uses (see [Compilation and Linking](<Compilation and Linking.md#4-linking>)):

| Linkage | Meaning | How you get it |
|---|---|---|
| **External** | Shared across the whole program: every file means the same entity | Default for non-`const` globals and functions |
| **Internal** | Private to one translation unit | `static` at namespace scope, anonymous namespace, `const`/`constexpr` globals |
| **None** | Only within its own scope | Local variables |

```cpp
// a.cpp
int shared_score = 0;          // external linkage: the definition
static int private_score = 0;  // internal linkage: invisible outside a.cpp

// b.cpp
extern int shared_score;       // declaration: refers to a.cpp's variable
extern int private_score;      // declares an external one... which nobody defines -> linker error if used
```

### The Many Meanings of `static`

`static` is one keyword with three unrelated jobs, depending on where it appears:

```cpp
static int helper() { ... }         // 1. at namespace scope: INTERNAL LINKAGE (file-private)

void f() {
    static int calls = 0;           // 2. inside a function: STATIC STORAGE DURATION (persists across calls)
}

class Player {
    static int player_count;        // 3. inside a class: ONE SHARED MEMBER for the whole class, not per object
    static Player* create();        //    a static member function: called as Player::create(), no `this`
};
```

Meaning 3 corresponds to Python's class attributes and `@staticmethod`s (see [OOP-Self](<../../Python/OOP/OOP-Self .md#class-attributes-vs-instance-attributes>)).

## Summary

```text
Scope          -> where a NAME is visible. { } blocks, functions, classes, namespaces. Compile time.
Namespace      -> a named scope for grouping and avoiding clashes. Reached with ::
Lifetime       -> when the OBJECT exists. Automatic, static, dynamic, or thread. Runtime.
Linkage        -> whether the same name in two .cpp files means the same thing. Link time.

Lookup         -> innermost scope outward, first match wins, all at compile time
::name         -> global scope;   ns::name -> inside namespace ns;   Class::name -> inside class
using std::x   -> import one name (fine in .cpp files)
using namespace std -> import everything (never in headers)
```
