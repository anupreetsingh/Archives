# Object Lifetime and RAII in C++

Every C++ object has a precise **lifetime**: it begins when its constructor finishes and ends when its destructor runs. Unlike Python, where an object lives "until nothing references it and the runtime gets around to freeing it", C++ tells you **exactly** when each object dies.

**RAII** (Resource Acquisition Is Initialization) builds on that precision: tie every resource (memory, a file, a lock, a network connection) to an object, acquire the resource in the constructor, and release it in the destructor. Then the resource is released automatically, on every path out of the scope, with no cleanup code to forget.

It's the most important idiom in C++, and it plays the role that `with` statements and `try`/`finally` play in Python.

---

## 1. Destructors

A **destructor** is a member function that runs automatically when an object's lifetime ends. Its name is the class name with a `~` in front. It takes no parameters and returns nothing:

```cpp
class Tracer {
public:
    explicit Tracer(std::string name) : name_(name) {
        std::cout << "construct " << name_ << "\n";
    }

    ~Tracer() {                                   // destructor
        std::cout << "destroy " << name_ << "\n";
    }

private:
    std::string name_;
};
```

You never call a destructor yourself. It runs:

| Object kind | Destructor runs |
|---|---|
| Local (automatic) variable | When execution leaves its enclosing `{ }` block, by **any** route |
| Heap object (`new`) | When you `delete` it, or when its owning smart pointer is destroyed |
| Data member | When the object containing it is destroyed |
| Temporary | At the end of the full expression (usually the `;`) that created it |
| Global / `static` | After `main()` returns |

Python has `__del__`, but it runs only when the reference count drops to zero (or later, when the cycle collector gets to it), and it isn't guaranteed to run at all at interpreter shutdown. That's why Python code doesn't rely on `__del__` for cleanup and uses `with` instead. C++ destructors are **deterministic**, so C++ code relies on them for everything.

## 2. The Order of Construction and Destruction

Here's the flow for an object built from a base class and member objects:

```text
Construction (outside in, top down):
  1. base class          (its members first, then its constructor body)
  2. data members        (in DECLARATION order)
  3. the constructor body

Destruction (exactly the reverse):
  1. the destructor body
  2. data members        (in REVERSE declaration order)
  3. base class          (its destructor body, then its members)
```

The reasoning: while a constructor body runs, it can rely on every member and the base already being fully built. While a destructor body runs, they're all still alive.

```cpp
struct Vehicle {
    Tracer t{"Vehicle (base member)"};
    Vehicle()  { std::cout << "Vehicle ctor body\n"; }
    ~Vehicle() { std::cout << "Vehicle dtor body\n"; }
};

struct Car : Vehicle {              // Car inherits from Vehicle (see OOP Four Pillars)
    Engine engine;                  // each holds a Tracer member that prints
    Wheel wheel;
    Car()  { std::cout << "Car ctor body\n"; }
    ~Car() { std::cout << "Car dtor body\n"; }
};

{
    Car c;
    std::cout << "-- using car --\n";
}
```

```text
construct Vehicle (base member)
Vehicle ctor body                 <- base fully built first
construct Engine (member)
construct Wheel (member)          <- members in declaration order
Car ctor body                     <- body last
-- using car --
Car dtor body                     <- body first
destroy Wheel (member)
destroy Engine (member)           <- members in reverse order
Vehicle dtor body
destroy Vehicle (base member)     <- base last
```

Local variables in one scope follow the same rule: destroyed in **reverse order of declaration**. Later objects may depend on earlier ones, so they have to go first.

## 3. The Problem RAII Solves

Here's resource cleanup done by hand, with a C file handle:

```cpp
#include <cstdio>

bool save_scores(const char* path, const std::vector<int>& scores) {
    std::FILE* f = std::fopen(path, "w");    // acquire
    if (!f) return false;

    if (scores.empty()) {
        return false;                        // BUG: f is never closed (leaked file handle)
    }

    for (int s : scores) {
        write_score(f, s);                   // BUG: if this throws, fclose never runs either
    }

    std::fclose(f);                          // release, but only on this one path
    return true;
}
```

Every `return` and every possible exception is an exit that skips the cleanup. With several resources and several exits, the cleanup code grows combinatorially, and it only takes one forgotten path to leak.

Python solves this with a context manager:

```python
with open(path, "w") as f:       # __enter__ acquires
    for s in scores:
        f.write(f"{s}\n")
# __exit__ closes f here, even on return or exception
```

## 4. RAII: Let the Destructor Do It

In C++, the object **is** the context manager. Its constructor is `__enter__`, its destructor is `__exit__`, and its scope is the `with` block:

```cpp
#include <fstream>

bool save_scores(const std::string& path, const std::vector<int>& scores) {
    std::ofstream out(path);                 // constructor opens the file
    if (!out) return false;

    if (scores.empty()) {
        return false;                        // out's destructor closes the file
    }

    for (int s : scores) {
        out << s << "\n";                    // an exception here? out's destructor still closes the file
    }

    return true;                             // out's destructor closes the file
}
```

`std::ofstream` is an RAII class: its destructor closes the file. Every exit path is covered without a single line of cleanup code.

| Python | C++ |
|---|---|
| `with open(p) as f:` | `std::ofstream f(p);` |
| The `with` block | The enclosing `{ }` scope |
| `__enter__` | Constructor |
| `__exit__` | Destructor |
| `try: ... finally: cleanup()` | A local object whose destructor does `cleanup()` |

The difference is that in C++ you don't have to **remember** to write `with`. Simply declaring the object as a local variable gets you the guarantee.

### Your Own RAII Class

The pattern is always the same: acquire in the constructor, release in the destructor. A tiny example that measures how long a scope takes:

```cpp
#include <chrono>

class ScopedTimer {
public:
    explicit ScopedTimer(std::string label)
        : label_(label), start_(std::chrono::steady_clock::now()) {}   // "acquire": record the start time

    ~ScopedTimer() {                                                    // "release": report on scope exit
        auto elapsed = std::chrono::steady_clock::now() - start_;
        auto us = std::chrono::duration_cast<std::chrono::microseconds>(elapsed).count();
        std::cout << label_ << " took " << us << " us\n";
    }

private:
    std::string label_;
    std::chrono::steady_clock::time_point start_;
};

void update_ai() {
    ScopedTimer timer("update_ai");    // starts timing
    // ... any number of returns, or an exception ...
}                                      // always prints "update_ai took N us"
```

> Milestone 0.2b of the learning plan has you build an RAII wrapper around a file handle, then use breakpoints to watch its destructor run on early returns and exceptions.

### RAII Everywhere in the Standard Library

| Resource | RAII type | Released by its destructor |
|---|---|---|
| Heap memory for a sequence | `std::vector`, `std::string` | Frees the buffer |
| A single heap object | `std::unique_ptr`, `std::shared_ptr` | `delete`s the object (see [Smart Pointers](<../Memory/Smart Pointers.md>)) |
| A file | `std::ifstream`, `std::ofstream` | Closes the file |
| A mutex lock | `std::lock_guard`, `std::scoped_lock`, `std::unique_lock` | Unlocks the mutex |
| A thread | `std::jthread` (C++20) | Requests stop and joins the thread |

```cpp
std::mutex m;

void add_score(int s) {
    std::lock_guard<std::mutex> lock(m);   // locks m
    scores.push_back(s);                   // even if this throws...
}                                          // ...m is unlocked here
```

Python's equivalent is `with lock:`.

## 5. RAII and Exceptions: Stack Unwinding

When an exception is thrown, C++ performs **stack unwinding**: it exits each scope between the `throw` and the matching `catch`, and runs the destructor of every fully constructed local object along the way:

```cpp
void thrower() {
    Tracer guard("guard");
    throw std::runtime_error("boom");
}

try {
    thrower();
} catch (const std::exception& e) {
    std::cout << "caught " << e.what() << "\n";
}
```

```text
construct guard
destroy guard          <- destructor runs during unwinding, BEFORE the catch block
caught boom
```

This is why RAII and exceptions are designed to work together: any code can throw, and nothing leaks, as long as every resource is owned by an object. See [Error Handling](<../Fundamentals/Error Handling.md>).

Two rules follow from this:

- **Destructors must not throw.** Destructors are `noexcept` by default, so an exception escaping a destructor calls `std::terminate` and the program dies. (Even if one were allowed to throw, throwing while another exception is already unwinding the stack would terminate anyway.)
- If an exception is **never caught**, the program terminates, and whether the stack is unwound first is implementation-defined. Catch exceptions somewhere (at least in `main`) if you rely on destructors running.

## 6. Lifetime Pitfalls

### Unnamed Temporaries Die Immediately

An object created without a name is a temporary, destroyed at the end of the statement:

```cpp
Tracer("temp");                       // construct temp / destroy temp: right here, at the ;
std::cout << "after temp statement\n";
```

This matters for RAII guards:

```cpp
std::lock_guard<std::mutex>{m};       // BUG: locks and IMMEDIATELY unlocks; nothing below is protected
std::lock_guard<std::mutex> lock(m);  // correct: the named object lives until the end of the scope
```

### Dangling References to Destroyed Objects

An object's lifetime doesn't stretch because something still refers to it. Python's reference counting keeps the object alive; C++ doesn't:

```cpp
const std::string& name_ref = get_player().name();   // get_player() returns a temporary Player by value
                                                      // the temporary dies at the ;
std::cout << name_ref;                                // UB: name_ref refers into a destroyed object
```

Keep the owner alive (`Player p = get_player(); const auto& name = p.name();`), or copy the value out.

## 7. Special Member Functions

The constructor, destructor, and the copy and move operations are called **special member functions**. The compiler can generate them for you, and you can control that explicitly:

```cpp
class Player {
public:
    Player() = default;                          // ask for the compiler-generated version
    ~Player() = default;

    Player(const Player&) = delete;              // forbid copying: any attempt is a compile error
    Player& operator=(const Player&) = delete;
};
```

Once a class owns a resource through a destructor, its copy and move behavior needs thought: copying a `Tracer` is harmless, but copying an object that owns a raw file handle would make **two** destructors close the same handle. That's the subject of [Copy and Move Semantics](<Copy and Move Semantics.md>).

## Summary

```text
Constructor   -> acquire the resource, establish the object's invariants
Destructor    -> release the resource; runs automatically, deterministically, and must not throw
Scope exit    -> by fallthrough, return, break, or exception: destructors always run (reverse order)
RAII          -> own every resource with an object, so cleanup can't be forgotten
Python `with` -> what C++ gets just by declaring a local object
```
