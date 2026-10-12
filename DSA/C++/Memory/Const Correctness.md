# Const Correctness in C++

`const` marks something as **read-only**. The compiler then rejects any code that tries to modify it.

Python has no real equivalent. `MAX_HEALTH = 100` in capitals is only a convention, and a tuple is immutable as a type, but you can't take a list and say "this function may look at it but not change it". In C++, you say exactly that, the compiler enforces it, and it costs nothing at runtime.

**Const correctness** means using `const` everywhere something shouldn't change, so the type system documents and checks your intent.

---

## 1. `const` Variables

```cpp
const int max_health = 100;
max_health = 50;         // ERROR: cannot assign to variable 'max_health' with const-qualified type 'const int'

const int k;             // ERROR: default initialization of an object of const type 'const int'
                         // a const must be initialized, since it can never be assigned later
```

A `const` object stays const all the way down. You can't call anything on it that would modify it:

```cpp
const std::vector<int> v = {1, 2, 3};
v.push_back(4);          // ERROR: no matching member function for call to 'push_back'
v[0] = 2;                // ERROR: cannot assign to return value because function 'operator[]' returns a const value
std::cout << v[0];       // OK: reading is fine
```

Compare Python, where "constant" containers stay mutable:

```python
SCORES = [1, 2, 3]
SCORES.append(4)         # nothing stops this
```

## 2. `const` With Pointers

Covered in detail in [Pointers](<Pointers.md#7-pointers-and-const>). Read the declaration **right to left**, and remember that `const` applies to whatever is immediately to its **left** (or to its right, if nothing is on its left):

```cpp
const int* p;          // pointer to const int:        can't change *p, can re-point p
int* const p;          // const pointer to int:        can change *p, can't re-point p
const int* const p;    // const pointer to const int:  can't change either
```

## 3. `const` References as Parameters

This is the most common use of `const` by far. A `const&` parameter avoids the copy **and** promises the caller that their object won't be modified:

```cpp
void print_roster(const std::vector<std::string>& names);   // reads names, guaranteed not to change them
void add_player(std::vector<std::string>& names);           // non-const: the signature says "I will modify this"
```

Just reading the signature tells the caller which arguments might change. Python can't express that.

Pass small types (`int`, `double`, `bool`, pointers) by value instead. Copying 8 bytes is cheaper than an indirection, and a by-value parameter is already a private copy:

```cpp
void set_volume(float level);                    // not const float& level
```

## 4. `const` Member Functions

A member function marked `const` after its parameter list promises not to modify the object it's called on:

```cpp
class Player {
public:
    int health() const { return health_; }           // const: only reads
    bool is_alive() const { return health_ > 0; }    // const: only reads
    void take_damage(int amount) { health_ -= amount; }  // non-const: modifies

private:
    int health_ = 100;
};
```

Inside a `const` member function, every data member is treated as `const`:

```cpp
int health() const {
    health_ = 0;     // ERROR: cannot assign to non-static data member within const member function 'health'
    return health_;
}
```

### Why It Matters: const Objects Can Only Call const Functions

Through a `const` object, a `const&`, or a `const*`, you can only call `const` member functions:

```cpp
void print_status(const Player& p) {
    std::cout << p.health();    // OK: health() is const
    p.take_damage(5);           // ERROR: 'this' argument to member function 'take_damage' has type
                                //        'const Player', but function is not marked const
}
```

This is where forgetting `const` bites. If `health()` weren't marked `const`, `print_status` couldn't call it, even though it doesn't modify anything. Every function that only reads should be `const`, so it's usable everywhere a `const Player&` is passed.

How it works: inside a member function, `this` is a pointer to the object. In a non-const member function `this` is a `Player*`, and in a `const` member function it's a `const Player*` (see [Classes and Objects](<../OOP/Classes and Objects.md>)).

### const Overloads

A class can provide both a `const` and a non-`const` version of the same function. The compiler picks one based on whether the object is `const`:

```cpp
class Inventory {
public:
    Item& at(std::size_t i)             { return items_[i]; }   // called on non-const Inventory: modifiable
    const Item& at(std::size_t i) const { return items_[i]; }   // called on const Inventory: read-only
private:
    std::vector<Item> items_;
};
```

This is how `std::vector::operator[]` gives you a modifiable element from a normal vector and a read-only one from a `const` vector.

### `mutable`: The Escape Hatch

A data member marked `mutable` can be modified even inside `const` member functions. Use it for internal bookkeeping that doesn't change the object's **observable** state, such as caches, counters, or mutexes:

```cpp
class Player {
public:
    int health() const {
        ++reads_;            // OK: reads_ is mutable
        return health_;
    }
private:
    int health_ = 100;
    mutable int reads_ = 0;  // how many times health() was called: doesn't change what a Player "is"
};
```

## 5. `constexpr`: Known at Compile Time

`const` means "won't change after initialization", but the value can still come from runtime input. `constexpr` means the value is **computed by the compiler**:

```cpp
int n = read_number();
const int a = n * 2;         // OK: const, but only known at runtime
constexpr int b = n * 2;     // ERROR: n isn't a compile-time constant

constexpr int grid = 16;     // OK
int cells[grid * grid];      // array sizes and template arguments need compile-time constants
```

`constexpr` functions can run at compile time when given constant arguments, and at runtime otherwise (see [Functions](<../Fundamentals/Functions.md#inline-and-constexpr-functions>)):

```cpp
constexpr int square(int x) { return x * x; }

constexpr int s = square(8);   // 64, computed during compilation: zero runtime cost
int arr[s];                    // 256 bytes
int r = square(n);             // fine at runtime too
```

C++20's `consteval` goes further and **requires** compile-time evaluation:

```cpp
consteval int cube(int x) { return x * x * x; }

constexpr int c = cube(3);     // 27
int r = cube(n);               // ERROR: call to consteval function 'cube' is not a constant expression
```

| Keyword | Meaning | Typical use |
|---|---|---|
| `const` | Can't be modified after initialization | Parameters, member functions, values computed at runtime |
| `constexpr` variable | Known at compile time (and therefore also const) | Named constants, array sizes, tuning values |
| `constexpr` function | **Can** run at compile time | Small pure helper functions |
| `consteval` function | **Must** run at compile time | Compile-time-only computations |

Prefer `constexpr` constants to `#define` macros: they have a type and a scope, and the debugger can see them (see [Preprocessor](<../Fundamentals/Preprocessor.md#2-define-object-like-macros>)).

## 6. `const` Return Values

Returning a `const&` hands out read-only access to something the object owns, without copying it:

```cpp
class Player {
public:
    const std::string& name() const { return name_; }   // caller can read name_, not change it
private:
    std::string name_;
};
```

Don't return `const` **by value** (`const std::string name() const;`). It adds nothing, and it blocks move semantics (see [Copy and Move Semantics](<../OOP/Copy and Move Semantics.md>)).

## 7. `const_cast`: Removing `const`

`const_cast` removes `const` from a pointer or reference. It exists mainly so you can call old APIs that take non-const pointers but don't actually modify anything:

```cpp
void legacy_print(char* text);         // old C API: never modifies text, but wasn't declared const

const std::string msg = "hello";
legacy_print(const_cast<char*>(msg.c_str()));   // OK as long as legacy_print really doesn't write
```

**Modifying** an object that was originally declared `const`, through a `const_cast`, is undefined behavior. The compiler may have placed it in read-only memory or assumed it never changes. If you find yourself writing `const_cast` in new code, the design usually needs a rethink.

## 8. Const Correctness Checklist

1. Mark every variable `const` (or `constexpr`) if it doesn't change after initialization.
2. Pass large inputs by `const&`, and small ones by value.
3. Mark every member function that doesn't modify the object as `const`.
4. Return `const&` to expose internal data read-only.
5. Use `const auto&` in range-based `for` loops that only read.
6. Start const from the beginning. Retrofitting it later forces changes through every caller, because one non-const function blocks all the const code that wants to call it.

> Unreal follows the same rules: `const` getters like `GetActorLocation() const`, `const FString&` inputs, and Blueprint-callable functions marked `UFUNCTION(BlueprintPure)` are typically `const`.
