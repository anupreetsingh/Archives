# Smart Pointers in C++

A **smart pointer** is a class that holds a pointer to a heap object and **owns** it: its destructor deletes the object automatically. It's RAII (see [Object Lifetime and RAII](<../OOP/Object Lifetime and RAII.md>)) applied to heap memory, and it removes the leak, double-free, and early-return bugs of manual `new`/`delete` (see [Stack and Heap](<Stack and Heap.md#what-goes-wrong-with-manual-newdelete>)).

All three live in `<memory>`:

| Smart pointer | Ownership | Python analogy |
|---|---|---|
| `std::unique_ptr<T>` | **Exactly one** owner. Move-only | None: Python never has single ownership |
| `std::shared_ptr<T>` | **Shared** among several owners, with a reference count | Every Python reference |
| `std::weak_ptr<T>` | **No** ownership: observes a `shared_ptr` object without keeping it alive | `weakref.ref` |

Raw pointers (`T*`) and references stay in use for **non-owning** access: looking at an object somebody else owns.

---

## 1. `std::unique_ptr`: Single Ownership

`std::unique_ptr<T>` is the default smart pointer. It owns one heap object, and deletes it when the `unique_ptr` is destroyed:

```cpp
#include <memory>

struct Enemy {
    std::string name;
    explicit Enemy(std::string n) : name(n) { std::cout << "spawn " << name << "\n"; }
    ~Enemy() { std::cout << "despawn " << name << "\n"; }
};

{
    auto a = std::make_unique<Enemy>("Grunt");   // allocate + construct, owned by a
    a->name;                                     // use it like a pointer: ->, *, and if (a)
    (*a).name;
}                                                // a is destroyed -> the Enemy is deleted
```

```text
spawn Grunt
despawn Grunt
```

Always create one with `std::make_unique<T>(constructor args...)` rather than `new`.

### Move-Only

Two `unique_ptr`s can never own the same object, so copying is forbidden:

```cpp
auto a = std::make_unique<Enemy>("Grunt");
auto b = a;              // ERROR: call to implicitly-deleted copy constructor of 'unique_ptr<Enemy>'
auto b = std::move(a);   // OK: ownership is TRANSFERRED to b, and a becomes null
```

```text
a is null, b is set
```

This is the move-only pattern from [Copy and Move Semantics](<../OOP/Copy and Move Semantics.md#move-only-and-non-copyable-types>). The type system tracks who owns the object at every point.

### Passing and Returning

```cpp
// Factory: return by value. Ownership moves to the caller automatically
std::unique_ptr<Enemy> spawn(const std::string& name) {
    return std::make_unique<Enemy>(name);
}

// Sink: takes ownership. The caller must std::move in, and the Enemy dies when this function ends
void take(std::unique_ptr<Enemy> e);

// Just using the object: take a reference (or a raw pointer if null is allowed). No ownership involved
void look(const Enemy& e);

auto boss = spawn("Warden");
look(*boss);              // pass the object itself
take(std::move(boss));    // hand over ownership; boss is null afterwards
```

| Parameter type | Meaning |
|---|---|
| `std::unique_ptr<T>` (by value) | "I take ownership" |
| `std::unique_ptr<T>&` | "I may reseat or reset your pointer" (rare) |
| `T&` / `const T&` | "I just use the object" (most functions) |
| `T*` | "I just use the object, which may be absent" |

Don't pass `const std::unique_ptr<T>&` just to read the object. Pass `const T&`, so the function works no matter how the object is owned.

### Other Operations

```cpp
auto c = spawn("Sniper");
c.reset();                      // delete the object now; c becomes null
c = spawn("Tank");              // assigning a new one deletes any old one first

Enemy* observer = c.get();      // a non-owning raw pointer. Don't delete it, and don't keep it past c's lifetime

std::unique_ptr<Enemy> adopt(new Enemy("Raw"));   // take ownership of an existing raw pointer (prefer make_unique)
Enemy* released = adopt.release();                // give up ownership WITHOUT deleting: now you must delete it

auto many = std::make_unique<int[]>(100);         // array form: uses delete[]
```

### Zero Overhead

A `unique_ptr` is exactly the size of a raw pointer, and compiles to the same code as a hand-written `new`/`delete`:

```cpp
sizeof(std::unique_ptr<Enemy>)   // 8
sizeof(Enemy*)                   // 8
```

### Containers of `unique_ptr`

This is the standard way to hold a collection of heap objects, often polymorphic ones (see [Virtual Functions and Vtables](<../OOP/Virtual Functions and Vtables.md>)):

```cpp
std::vector<std::unique_ptr<Enemy>> enemies;
enemies.push_back(std::make_unique<Enemy>("A"));
enemies.push_back(std::make_unique<Enemy>("B"));
// when the vector is destroyed, it destroys each unique_ptr, which deletes each Enemy
```

### Custom Deleters

A `unique_ptr` can call any cleanup function instead of `delete`, which turns any C-style resource into an RAII object:

```cpp
#include <cstdio>

std::unique_ptr<std::FILE, int (*)(std::FILE*)> file(std::fopen("log.txt", "w"), &std::fclose);
// fclose(file) runs automatically when `file` goes out of scope
```

## 2. `std::shared_ptr`: Shared Ownership

`std::shared_ptr<T>` lets several owners share one object. It keeps a **reference count** of how many `shared_ptr`s point at the object, and deletes the object when the count drops to zero.

That's exactly how CPython manages **every** object: `sys.getrefcount()` is the same idea as `use_count()`.

```cpp
{
    auto s1 = std::make_shared<Enemy>("Boss");   // count = 1
    {
        auto s2 = s1;                            // COPYING is allowed: count = 2
    }                                            // s2 destroyed: count = 1
    std::cout << s1.use_count();                 // 1
}                                                // s1 destroyed: count = 0 -> Boss deleted
```

```text
spawn Boss
count 1
count 2
count 1
despawn Boss
```

How it works:

```text
 s1: [ obj ptr | ctrl ptr ]──┐
                             ├──> control block: [ strong count = 2 | weak count | deleter ]
 s2: [ obj ptr | ctrl ptr ]──┘                         |
                                                        v
                                                    [ Enemy "Boss" ]
```

- A `shared_ptr` is **two** pointers: 16 bytes instead of 8.
- Copying one updates the count with an **atomic** operation, so it's thread-safe but noticeably slower than copying a raw pointer.
- `std::make_shared` allocates the object and the control block in **one** allocation. Prefer it over `std::shared_ptr<T>(new T)`.

Use `shared_ptr` only when ownership is genuinely shared and there's no single obvious owner, such as an asset loaded once and used by several systems, each of which may outlive the others. Most of the time, one object clearly owns another, so `unique_ptr` is the right choice.

## 3. Reference Cycles and `std::weak_ptr`

Pure reference counting has a known weakness: a **cycle** of objects that point at each other never reaches a count of zero:

```cpp
struct Leaky {
    std::shared_ptr<Leaky> other;
    ~Leaky() { std::cout << "~Leaky\n"; }
};

{
    auto x = std::make_shared<Leaky>();
    auto y = std::make_shared<Leaky>();
    x->other = y;     // y's count = 2
    y->other = x;     // x's count = 2
}                     // x and y (the locals) are destroyed: each count drops to 1, never 0
// (no destructors ran): both objects leak
```

Python has the same problem with reference counting alone, which is why CPython also runs a **cycle-detecting garbage collector** (the `gc` module). C++ has no such collector. You break cycles yourself, by making one direction of the link a `std::weak_ptr`:

```cpp
struct Node {
    std::shared_ptr<Node> next;   // owning: forward link
    std::weak_ptr<Node> prev;     // NON-owning: back link, doesn't increase the count
};

{
    auto n1 = std::make_shared<Node>();
    auto n2 = std::make_shared<Node>();
    n1->next = n2;
    n2->prev = n1;
}
// ~Node n1
// ~Node n2   <- both freed
```

### Using a `weak_ptr`

A `weak_ptr` might outlive the object it observes, so you can't use it directly. You ask it for a temporary `shared_ptr` with `lock()`, which is null if the object is already gone:

```cpp
auto boss = std::make_shared<Enemy>("Boss");
std::weak_ptr<Enemy> target = boss;      // doesn't change boss.use_count()

if (auto locked = target.lock()) {       // still alive? get a shared_ptr for the duration of this block
    std::cout << locked->name;           // Boss
}

boss.reset();                            // Boss is deleted
target.expired();                        // true
target.lock() == nullptr;                // true: safe to check, never dangles
```

This is the safe answer to "an AI guard remembers its target, but the target may be destroyed at any time". A raw pointer would dangle. A `weak_ptr` reports that the target is gone.

## 4. Choosing an Ownership Model

```text
Does this code OWN the object (responsible for destroying it)?
 ├─ No  -> T& (always present) or T* (may be absent). A raw pointer or reference; never delete through it
 └─ Yes -> Can the object live on the stack, or as a plain member?
            ├─ Yes -> do that. No pointer at all
            └─ No  -> Is there exactly one owner at a time?
                       ├─ Yes -> std::unique_ptr<T>         (the default)
                       └─ No  -> std::shared_ptr<T>         (genuinely shared)
                                  + std::weak_ptr<T> for back-links, caches, and observers
```

| Situation | Choice |
|---|---|
| A level owns its enemies | `std::vector<std::unique_ptr<Enemy>>` |
| A factory function creates an object | Return `std::unique_ptr<T>` (it converts to `shared_ptr` if the caller needs that) |
| A texture shared by many meshes, freed when the last one goes | `std::shared_ptr<Texture>` |
| A guard's current target, which may be destroyed | `std::weak_ptr<Player>` (or a raw pointer plus a lifetime guarantee) |
| A function that just uses an object | `T&` / `const T&` / `T*` |
| A tree's child links / parent link | `unique_ptr` children, raw `T*` parent |

> **Unreal Engine** has two separate systems:
>
> - For `UObject`s (actors, components, assets), Unreal's **garbage collector** owns everything. Members are raw `UObject*` or `TObjectPtr<T>` marked `UPROPERTY()` so the GC sees them, and `TWeakObjectPtr<T>` plays the role of `weak_ptr`. Never put a `UObject` in a `std::unique_ptr` or `TUniquePtr`.
> - For plain C++ objects, Unreal has its own versions of the standard smart pointers: `TUniquePtr`, `TSharedPtr`, `TSharedRef` (a never-null shared pointer), and `TWeakPtr`, created with `MakeUnique` / `MakeShared`.
