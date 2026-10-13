# Enums and Type Aliases in C++

## Enumerations

An **enumeration** is a type whose values are a fixed set of named constants. Python gets this from the `enum` module (`class Color(Enum): ...`). In C++, it's built into the language and costs nothing at runtime: each enumerator is just an integer.

C++ has two kinds. Prefer the second.

### Plain `enum` (C-Style, Unscoped)

```cpp
enum Color { Red, Green, Blue };   // Red = 0, Green = 1, Blue = 2

Color c = Green;
int ci = c;                        // 1: converts to int silently
bool same = (c == 1);              // true: compares with plain ints

enum Light { Red, Off };           // ERROR: redefinition of enumerator 'Red'
```

The enumerators leak into the **surrounding** scope (you write `Red`, not `Color::Red`), so two enums can't both have a `Red`. They also convert to `int` implicitly, so nonsense like `Green + 7` compiles.

### `enum class` (Scoped, C++11)

```cpp
enum class AlertState { Idle, Suspicious, Alerted };

AlertState s = AlertState::Suspicious;   // must qualify with the enum's name, using ::

int i = s;                               // ERROR: no implicit conversion to int
if (s == 0) { }                          // ERROR: can't compare with an int
std::cout << s;                          // ERROR: no operator<< for AlertState

int raw = static_cast<int>(s);           // 1: explicit conversion is allowed
AlertState back = static_cast<AlertState>(0);   // AlertState::Idle
```

| | `enum` | `enum class` |
|---|---|---|
| Access | `Red` | `Color::Red` |
| Name clashes between enums | Yes | No, each enum is its own scope |
| Converts to `int` implicitly | Yes | No, needs `static_cast` |
| Use in new code | Rarely | **Default choice** |

Despite the keyword, an `enum class` isn't a class: it can't have member functions, constructors, or base classes. [Kinds of Classes](<../OOP/Kinds of Classes.md#7-enum-class-not-a-class>) compares it with real classes.

> Unreal uses `enum class` with the `UENUM()` macro and an `E` prefix, for example `enum class EGuardState : uint8 { Patrolling, Investigating, Chasing };`.

### Choosing Values and the Underlying Type

Enumerators count up from `0` by default, and you can set any of them explicitly:

```cpp
enum class HttpStatus { Ok = 200, NotFound = 404, ServerError = 500 };

enum class AlertState { Idle, Suspicious = 5, Alerted };   // Idle = 0, Suspicious = 5, Alerted = 6
```

You can also pick the integer type that stores the enum, which controls its size:

```cpp
#include <cstdint>

enum class AlertState : std::uint8_t { Idle, Suspicious, Alerted };

sizeof(AlertState)   // 1 byte, instead of the default 4 (int)
```

Small enums matter when you store thousands of them (one per tile, per AI agent, ...). See [Object Memory Layout](<../Memory/Object Memory Layout.md>).

> Printing a `std::uint8_t`, or an enum whose underlying type is `std::uint8_t`, after converting it with `std::to_underlying` shows a **character**, not a number, because `uint8_t` is an alias for `unsigned char`. Cast to `int` to print the number: `static_cast<int>(s)`.

### Enums and `switch`

`switch` and `enum class` work well together. With `-Wall`, the compiler warns when you forget a value:

```cpp
const char* to_string(AlertState s) {
    switch (s) {
        case AlertState::Idle:       return "Idle";
        case AlertState::Suspicious: return "Suspicious";
        case AlertState::Alerted:    return "Alerted";
    }
    return "?";
}

std::cout << to_string(AlertState::Suspicious);   // Suspicious
```

```text
// if you remove one case:
warning: enumeration value 'Alerted' not handled in switch [-Wswitch]
```

Adding a `default:` silences that warning, so leave `default` out of enum switches if you want the compiler to remind you when someone adds a new enumerator.

C++ has no built-in way to get an enumerator's name as a string (Python has `s.name`), so you write a `to_string` like the one above. Unreal's `UENUM` reflection generates this for you.

### Enums as Bit Flags

Give each enumerator its own bit to combine them, as in the bit-flag example in [Operators](<Operators.md#bit-flags>):

```cpp
enum Ability : unsigned {
    None      = 0,
    CanJump   = 1u << 0,
    CanCrouch = 1u << 1,
    CanClimb  = 1u << 2,
};

unsigned abilities = CanJump | CanClimb;
bool climbs = abilities & CanClimb;   // true
```

A plain `enum` is convenient here because it converts to integers. An `enum class` needs its own `operator|` and `operator&` definitions (Unreal's `ENUM_CLASS_FLAGS` macro writes them for you).

## Type Aliases

A **type alias** gives an existing type another name. It doesn't create a new type; both names mean exactly the same thing.

### `using` (Modern)

```cpp
using Score = int;
using NameToScores = std::map<std::string, std::vector<int>>;

Score s = 90;            // just an int
NameToScores grades;     // just a std::map<std::string, std::vector<int>>
grades["Ava"].push_back(90);
```

This is like a Python type alias (`Grid = list[list[int]]`), except that C++ uses the type for real checking.

Aliases are handy for:

- **Shortening** long template types, as above.
- **Documenting intent**: `using EntityId = std::uint32_t;` says what a number means.
- **Changing a type in one place**: switch `using Real = float;` to `double` and recompile.

### `typedef` (Older Syntax)

`typedef` does the same job with the name at the end, in C's declaration order:

```cpp
typedef unsigned long ulong_t;    // same as: using ulong_t = unsigned long;
typedef int (*Callback)(int);     // pointer-to-function: hard to read
using Callback = int (*)(int);    // same thing, easier to read
```

Prefer `using`. It reads left to right, and unlike `typedef` it also works for **alias templates**:

```cpp
template <typename T>
using Grid = std::vector<std::vector<T>>;

Grid<int> board;       // std::vector<std::vector<int>>
Grid<char> maze;       // std::vector<std::vector<char>>
```

See [Templates](<Templates.md>).

### Aliases Are Not New Types

Since an alias is only another name, it gives no type safety:

```cpp
using Meters = double;
using Seconds = double;

Meters distance = 100.0;
Seconds time = distance;   // compiles fine: both are just double
```

For a type the compiler will actually keep separate, define a small `struct` or class (see [Type Conversion](<Type Conversion.md#4-conversions-for-your-own-types>)). For a fixed set of values, use `enum class`.
