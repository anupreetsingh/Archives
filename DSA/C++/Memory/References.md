# References in C++

A **reference** is another name for an existing object: an **alias**. Once a reference is bound to an object, every use of the reference is a use of that object.

```cpp
int score = 42;
int& r = score;      // r is another name for score

r = 50;              // writes to score
std::cout << score;  // 50
&r == &score         // true: same object, same address
```

Conceptually:

```text
 score ─┐
        ├──> [ 50 ]     one object, two names
 r ─────┘
```

That picture is exactly how Python names work: `b = a` makes two names for one object (see [Name Rebinding](<../../Python/Fundamentals/Name Rebinding.md>)). A C++ reference gives you that "two names, one object" behavior on purpose, inside a language where plain assignment otherwise **copies**.

---

## 1. The Three Rules of References

### Rule 1: A Reference Must Be Initialized

There's no such thing as an unbound reference:

```cpp
int& r;     // ERROR: declaration of reference variable 'r' requires an initializer
```

### Rule 2: A Reference Can't Be Reseated

Once bound, a reference refers to the **same object** for its whole life. Assigning to it assigns **through** it, to the object it refers to. It does not rebind the reference:

```cpp
int score = 42;
int other = 7;
int& r = score;

r = other;       // NOT "make r refer to other". This means score = other

std::cout << score;     // 7: score's VALUE changed
&r == &score            // still true: r still refers to score

other = 100;
std::cout << score;     // 7: r was never connected to other
```

This is the opposite of Python, where `r = other` would **rebind** the name `r`. There is no syntax in C++ to make a reference refer to a different object. If you need that, use a pointer.

### Rule 3: A Reference Can't Be Null

A reference always refers to an object. There's no null reference, and nothing to check:

```cpp
int& r = nullptr;    // ERROR: non-const lvalue reference to type 'int' cannot bind to a temporary of type 'std::nullptr_t'
```

The only way to get a "null reference" is to dereference a null pointer (`int& r = *p;` with `p == nullptr`), and that's undefined behavior at the moment you do it, not a valid state. In correct code, a function taking `Player&` never needs `if (target)`.

> Rules 2 and 3 are the key differences from pointers. If "no object" is a valid state, or the referent needs to change, you need a **pointer**. If it's always exactly one object, use a **reference**.

## 2. References as Function Parameters

This is where references are used most. As covered in [Functions](<../Fundamentals/Functions.md#passing-arguments>), C++ copies arguments by default. A reference parameter makes the parameter an alias for the caller's variable instead:

```cpp
void swap_vals(int& a, int& b) {
    int t = a;
    a = b;
    b = t;
}

int x = 1, y = 2;
swap_vals(x, y);    // x = 2, y = 1. The call site looks the same as pass-by-value
```

### `const&`: Read Without Copying

A `const` reference gives read-only access without copying. It's the default for passing anything larger than a few bytes:

```cpp
// Copies the whole vector (maybe millions of elements) on every call:
double average(std::vector<int> values);

// No copy, and the function can't modify the caller's vector:
double average(const std::vector<int>& values);
```

This matches Python's behavior (no copy), with a compiler-enforced promise not to mutate on top.

### `const&` Can Bind to Temporaries

A non-const reference must refer to an existing, named object (an **lvalue**). A `const` reference can also bind to a **temporary**, and it extends that temporary's lifetime to match the reference:

```cpp
void shout(std::string& s);         // needs a real, modifiable string
void print(const std::string& s);   // accepts anything that converts to a string

shout("hello");     // ERROR: no matching function. "hello" would have to become a temporary std::string,
                    //        and a non-const reference can't bind to a temporary
print("hello");     // OK: a temporary std::string is created and bound to s for the call

int& r = 5;         // ERROR: non-const lvalue reference to type 'int' cannot bind to a temporary
const int& cr = 5;  // OK

const std::string& name = std::string("temp") + "!";   // OK: the temporary lives as long as `name`
```

That's why `const T&` is the natural parameter type for read-only inputs: callers can pass variables, literals, or expression results.

## 3. References in Range-Based `for`

Covered in [Control Flow](<../Fundamentals/Control Flow.md#copy-vs-reference-the-important-part>). The loop variable is a fresh variable for each element, so the choice is the same as for parameters:

```cpp
for (auto x : items)          // copy of each element
for (auto& x : items)         // alias for each element: can modify
for (const auto& x : items)   // alias, read-only: no copies, no accidental changes
```

### `auto` Drops References

`auto` deduces the plain type and makes a copy, even when the initializer is a reference:

```cpp
std::vector<int> v = {1, 2, 3};

auto a = v[0];     // int: a COPY of v[0]
a = 99;            // v[0] is still 1

auto& b = v[0];    // int&: an alias for v[0]
b = 99;            // v[0] is now 99
```

## 4. Returning References

A function can return a reference to an object that **outlives the call**. That avoids a copy and lets the caller modify the original:

```cpp
class Inventory {
public:
    Item& at(std::size_t i) { return items_[i]; }               // caller can modify the item
    const Item& at(std::size_t i) const { return items_[i]; }   // read-only version for const Inventories
private:
    std::vector<Item> items_;
};

inv.at(0).count += 1;   // modifies the item stored inside inv
```

`std::vector::operator[]`, `std::map::operator[]`, and `std::string::operator[]` all return references. That's why `v[0] = 5;` writes into the vector.

Never return a reference to a **local** variable (see [Functions](<../Fundamentals/Functions.md#never-return-a-reference-to-a-local>)):

```cpp
const std::string& make_name() {
    std::string name = "Ava";
    return name;        // warning: reference to stack memory associated with local variable 'name' returned
}                       // name is destroyed here; the caller gets a dangling reference
```

## 5. Dangling References

The "can't be null" guarantee doesn't mean "always valid". A reference can still outlive its object, and then it's **dangling**, exactly like a dangling pointer:

```cpp
std::vector<int> v = {1, 2, 3};
int& first = v[0];

v.push_back(4);       // may reallocate: all elements move to a new, bigger block of heap memory
first = 10;           // UB if a reallocation happened: first still refers to the OLD block, which was freed
```

This is the same capacity growth as in [List Structure](<../../Python/Fundamentals/List Structure.md#resizing-a-list>), with one difference. Python only copies **references**, and the element objects stay put. A C++ vector **moves the elements themselves**, so anything that referred to the old location is invalidated. See iterator invalidation in [STL Containers](<../Standard Library/STL Containers.md>).

## 6. How References Work Underneath

The standard doesn't say how references are implemented. In practice, the compiler treats a reference as a pointer that it dereferences for you automatically:

```cpp
void reset(int& x) { x = 0; }       // what you write
void reset(int* x) { *x = 0; }      // roughly what the compiler generates
reset(score);                       // ...called as reset(&score)
```

So a reference parameter costs the same as a pointer parameter (8 bytes, one indirection), and `const T&` is cheap however large `T` is. A local reference such as `int& r = score;` often compiles to nothing at all: the compiler just uses `score` directly.

## 7. Lvalues, Rvalues, and `&&` (Preview)

Every expression is either:

- an **lvalue**: something with a name and an address that persists (`score`, `v[0]`, `*p`), or
- an **rvalue**: a temporary value about to disappear (`5`, `a + b`, `std::string("temp")`, a function's returned value).

`T&` binds to lvalues. `const T&` binds to both. C++11 added a third kind, the **rvalue reference** `T&&`, which binds **only** to temporaries. Since a temporary is about to be destroyed anyway, a function receiving `T&&` can safely **steal** its contents instead of copying them:

```cpp
void take(std::string&& s);   // only accepts temporaries

take(std::string("big text"));   // OK
std::string name = "Ava";
take(name);                      // ERROR: name is an lvalue
take(std::move(name));           // OK: std::move casts it to an rvalue: "I'm done with name, you may steal it"
```

This is the foundation of **move semantics**, covered in [Copy and Move Semantics](<../OOP/Copy and Move Semantics.md>).

## Pointers vs References

| | Reference `T&` | Pointer `T*` |
|---|---|---|
| Must be initialized | Yes | No (but always should be) |
| Can be null | **No** | Yes (`nullptr`) |
| Can be re-pointed to another object | **No**: assignment writes through it | Yes: `p = &other;` |
| Syntax to use the object | Same as the object: `r.health` | Dereference: `*p`, `p->health` |
| Syntax at the call site | `f(x)` | `f(&x)` |
| Arithmetic / arrays | No | Yes: `p + 1`, `p[i]` |
| Can refer to another reference/pointer | No references-to-references | Yes: `T**` |
| Can be stored in containers | Not directly | Yes |
| Typical use | Parameters and return values that always refer to an object | Optional objects, re-pointable links, arrays, C APIs, non-owning members |

Guideline: **use a reference when you can, a pointer when you must**. Use a pointer when "nothing" is a valid value or when what it refers to has to change over time.

```cpp
void apply_damage(Player& target, int amount);       // there is always a target
void set_target(Player* target);                     // nullptr = "clear the target"

class Guard {
    Player* current_target_ = nullptr;               // changes over time and may be empty: pointer
};
```

> Unreal follows the same split: `const FVector& Location` for inputs, `FHitResult& OutHit` for outputs, and `AActor*` wherever "no actor" is possible.
