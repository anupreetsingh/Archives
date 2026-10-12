# Headers and Source Files in C++

C++ code is usually split into two kinds of files:

- **Header files** (`.h`, `.hpp`): **declarations**, which describe what exists. "There is a function `add` that takes two ints and returns an int."
- **Source files** (`.cpp`): **definitions**, which provide the actual code and storage. "Here is the body of `add`."

Better to be familiar with [Compilation and Linking](<Compilation and Linking.md>) first, because this split exists entirely because of how the build works:

```text
Each .cpp is compiled ALONE.
  -> To call a function from another file, the compiler needs its declaration in THIS translation unit.
  -> #include "math_utils.h" pastes that declaration in.
  -> The linker later connects the call to the one definition, which lives in math_utils.cpp.
```

In Python, `import math_utils` runs that module and gives you the real function object (see [Modules and Packages](<../../Python/Fundamentals/Modules and Packages.md>)). In C++, `#include "math_utils.h"` only gives you a **promise** that the function exists somewhere. The linker has to find it.

## Declarations vs Definitions

A **declaration** introduces a name and its type. A **definition** is a declaration that also provides the full thing: a function's body, a variable's storage, or a class's full layout.

```cpp
// Declarations: "this exists, and here is its type"
int add(int a, int b);         // function declaration (also called a prototype)
extern int high_score;         // variable declaration: storage lives elsewhere
class Weapon;                  // class declaration (forward declaration): layout unknown here

// Definitions: "here it is"
int add(int a, int b) {        // function definition: has a body
    return a + b;
}
int high_score = 0;            // variable definition: allocates storage
class Weapon {                 // class definition: full member list, so the size is known
    int damage;
};
```

Every definition is also a declaration, but not the other way round.

You can **declare** something as many times as you like, as long as the declarations agree. Defining it is subject to the One Definition Rule.

## The One Definition Rule (ODR)

The **One Definition Rule** says:

1. Within a single translation unit, each variable, function, class, and so on may be **defined** at most once.
2. Across the **whole program**, each non-inline function and variable that is used must be defined **exactly once**.
3. Classes, inline functions, inline variables, and templates **may** be defined in multiple translation units, as long as every definition is token-for-token identical. In practice, that means they live in a header that each `.cpp` includes.

Breaking rule 1, usually by including the same header twice, gives a compile error. That's what header guards prevent.

Breaking rule 2 produces linker errors:

```text
zero definitions   -> Undefined symbols / undefined reference / unresolved external symbol
two+ definitions   -> duplicate symbol / multiple definition
```

## What Goes Where

| Put in the header (`.h`) | Put in the source file (`.cpp`) |
|---|---|
| Function declarations | Function definitions (bodies) |
| Class definitions (member list) | Member function definitions (bodies) |
| `extern` variable declarations | Global variable definitions |
| `inline` functions, `constexpr` functions and constants | Helper functions used only by this file |
| Templates (full definitions, see [Templates](<Templates.md>)) | `static` / anonymous-namespace internals |
| Type aliases, enums | |

### A Typical Header / Source Pair

```cpp
// Player.h
#pragma once
#include <string>

class Player {
public:
    Player(std::string name, int health);   // constructor declaration

    void take_damage(int amount);           // member function declarations
    bool is_alive() const;

private:
    std::string name_;
    int health_;
};
```

```cpp
// Player.cpp
#include "Player.h"        // own header first

Player::Player(std::string name, int health)      // Player:: says "this belongs to class Player"
    : name_(name), health_(health) {}

void Player::take_damage(int amount) {
    health_ -= amount;
}

bool Player::is_alive() const {
    return health_ > 0;
}
```

```cpp
// main.cpp
#include "Player.h"        // gives main.cpp the class layout + member declarations

int main() {
    Player p("Ava", 100);
    p.take_damage(30);
}
```

Here:

- `main.cpp` needs the **class definition** from `Player.h` to know how big a `Player` is (so it can make room for one) and which members exist.
- `main.cpp` does **not** need the bodies of `take_damage` and `is_alive`. Those calls become undefined symbols in `main.o`, and the linker matches them to `Player.o`.
- `Player::take_damage` uses the **scope resolution operator** `::` to say which class the function belongs to. See [Classes and Objects](<../OOP/Classes and Objects.md>).

> Every Unreal class follows this exact pattern: `MyCharacter.h` holds the class definition with its `UCLASS()`/`UPROPERTY()` markup, and `MyCharacter.cpp` holds the member function bodies.

## Header Guards and `#pragma once`

Since `#include` is plain text pasting (see [Preprocessor](<Preprocessor.md#1-include>)), a header can easily end up pasted twice into one translation unit:

```cpp
// vec2.h
struct Vec2 { float x; float y; };
```

```cpp
// shapes.h
#include "vec2.h"
struct Circle { Vec2 center; float radius; };
```

```cpp
// main.cpp
#include "vec2.h"
#include "shapes.h"    // pastes vec2.h a second time, through shapes.h
```

```text
./vec2.h:1:8: error: redefinition of 'Vec2'
main.cpp:1:10: note: './vec2.h' included multiple times, additional include site here
```

### Classic Header Guards

Wrap the whole header in a conditional block keyed on a unique macro name:

```cpp
// vec2.h
#ifndef VEC2_H          // first time: VEC2_H isn't defined, so keep this block
#define VEC2_H          // ...and define it, so any second paste sees it

struct Vec2 { float x; float y; };

#endif                  // second time: VEC2_H is defined, so everything above is deleted
```

### `#pragma once`

One line does the same thing, and every major compiler supports it:

```cpp
// vec2.h
#pragma once

struct Vec2 { float x; float y; };
```

> Header guards only protect against double inclusion **within one translation unit**. Each `.cpp` file still gets its own copy of the header. That's fine for declarations and class definitions, but not for ordinary function or variable definitions, as the next section shows.

## Definitions in Headers: Why They Break, and `inline`

Suppose a header contains a function **definition** and a variable **definition**, and two `.cpp` files include it:

```cpp
// util.h
#pragma once
int helper() { return 1; }   // definition
int counter = 0;             // definition
```

```text
a.cpp includes util.h  ->  a.o defines helper() and counter
b.cpp includes util.h  ->  b.o defines helper() and counter

linker:
duplicate symbol '_counter' in: a.o, b.o
duplicate symbol 'helper()' in: a.o, b.o
```

`#pragma once` doesn't help, because each `.cpp` is a separate translation unit. There are three fixes.

**Fix 1: declare in the header, define in one `.cpp`.** This is the standard approach.

```cpp
// util.h
#pragma once
int helper();                // declaration
extern int counter;          // declaration only: "counter is defined in some other file"
```

```cpp
// util.cpp
#include "util.h"
int helper() { return 1; }   // the one definition
int counter = 0;             // the one definition
```

**Fix 2: mark it `inline`.** `inline` tells the linker "identical definitions of this may appear in many object files; keep one and discard the rest."

```cpp
// util.h
#pragma once
inline int helper() { return 1; }   // OK in a header
inline int counter = 0;             // C++17 inline variable: OK in a header, one shared variable
```

Despite the name, `inline` today is mostly about the ODR, not about inlining. The optimizer decides on its own whether to inline a call.

**Fix 3: things that are implicitly inline.** These can be defined in headers freely:

- Member functions defined **inside** the class body
- `constexpr` functions
- Templates (see [Templates](<Templates.md#3-why-templates-live-in-headers>))
- `constexpr` variables at namespace scope (they have internal linkage, so each TU gets its own private copy)

```cpp
// Vec2.h
#pragma once

struct Vec2 {
    float x, y;
    float length_squared() const { return x * x + y * y; }   // implicitly inline: fine in a header
};

constexpr float Gravity = 9.81f;                              // fine in a header
```

## Forward Declarations

Including a header pulls in its text **and** everything it includes, which slows compilation and couples files together. Often a header doesn't need the full class definition at all.

A **forward declaration** (`class Weapon;`) tells the compiler "a class called `Weapon` exists" without describing it. This makes `Weapon` an **incomplete type**. You can use an incomplete type only in ways that don't need its size or members:

```cpp
// Player.h
#pragma once
class Weapon;                  // forward declaration instead of #include "Weapon.h"

class Player {
    Weapon* weapon_;           // OK: a pointer is always 8 bytes, whatever it points to
    void equip(Weapon& w);     // OK: declaring a function with a reference parameter
    // Weapon spare_;          // ERROR: field has incomplete type 'Weapon'
                               //        (the compiler needs sizeof(Weapon) to lay out Player)
};
```

Then the `.cpp`, which actually calls `Weapon`'s member functions, includes the full header:

```cpp
// Player.cpp
#include "Player.h"
#include "Weapon.h"            // full definition needed here to call weapon_->fire()

void Player::equip(Weapon& w) {
    weapon_ = &w;
}
```

| You can, with only a forward declaration | You need the full definition to |
|---|---|
| Declare pointers and references to it | Create an object of it (local, member, or `new`) |
| Declare functions taking/returning it by pointer or reference | Access its members or call its member functions |
| Declare functions taking/returning it by value (declaration only) | Inherit from it |
| | Use `sizeof` on it |

> Unreal Engine coding standards ask for exactly this: forward-declare `class UCameraComponent;` in your header and `#include` its header only in the `.cpp`. It keeps the ripple effect of header changes, and so rebuild times, down.

### Breaking Circular Includes

Forward declarations also fix headers that need each other:

```cpp
// A.h                         // B.h
#pragma once                   #pragma once
#include "B.h"                 #include "A.h"
class A { B* b; };             class B { A* a; };
```

Whichever header is included first, `#pragma once` stops the cycle, but one class ends up being parsed before the other exists, so you get an "unknown type name" error. Since both only hold **pointers**, replace the includes with forward declarations:

```cpp
// A.h                         // B.h
#pragma once                   #pragma once
class B;                       class A;
class A { B* b; };             class B { A* a; };
```

## Include Order Convention

A common convention, used in Unreal as well, is:

```cpp
// Player.cpp
#include "Player.h"        // 1. this file's own header FIRST

#include "Weapon.h"        // 2. other headers from this project
#include "Inventory.h"

#include <string>          // 3. standard library headers
#include <vector>
```

Putting the own header first proves that `Player.h` compiles **on its own**. If `Player.h` forgot to include `<string>` but happened to work because some earlier include pulled it in, this order exposes the bug.

> In Unreal headers, `#include "MyActor.generated.h"` must be the **last** include in `MyActor.h`. UnrealHeaderTool enforces this.

## Summary

```text
.h    -> declarations + class definitions + inline/constexpr/template code
.cpp  -> definitions of everything else, compiled once
#include the full header only where you need size or members; forward-declare otherwise
Linker error "undefined"  -> missing definition (or missing .cpp/library in the build)
Linker error "duplicate"  -> a non-inline definition in a header, or defined in two .cpp files
```
