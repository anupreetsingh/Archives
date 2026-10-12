# Copy and Move Semantics in C++

In Python, `b = a` never copies anything. It binds a second name to the same object, and copying only happens when you ask for it (`a.copy()`, `copy.deepcopy(a)`; see the shallow copy notes in [List Manipulation](<../../Python/Fundamentals/List Manipulation.md#listcopy>)).

In C++, objects are values (see [What is C++](<../Fundamentals/What is C++.md#3-value-semantics>)), so `b = a` **does** copy. That raises two questions this note answers:

1. **What does "copy" mean** for a class that owns a resource, such as heap memory?
2. **How do you avoid copying** when the source is about to be thrown away anyway? (**Move semantics**)

Better to be familiar with [Object Lifetime and RAII](<Object Lifetime and RAII.md>) and [References](<../Memory/References.md>) first.

---

## 1. When Copies Happen

```cpp
Texture a(4, 4);

Texture b = a;            // copy CONSTRUCTION: b is a new object, built as a copy of a
Texture c(2, 2);
c = a;                    // copy ASSIGNMENT: c already exists; its old contents are replaced by a copy of a

void upload(Texture t);   // pass by value: the argument is copy-constructed into t
upload(a);

std::vector<Texture> v;
v.push_back(a);           // a copy of a is stored in the vector
```

Two different special member functions handle these:

| Operation | Special member | Signature |
|---|---|---|
| Create a new object from an existing one | **Copy constructor** | `Texture(const Texture& other)` |
| Overwrite an existing object with another's value | **Copy assignment operator** | `Texture& operator=(const Texture& other)` |

If you don't write them, the compiler generates them: it copies **each member**, one by one. For members like `int`, `std::string`, and `std::vector`, that's exactly right, because each of those types already knows how to copy itself.

## 2. The Shallow Copy Problem

The compiler-generated copy goes wrong when a class owns a resource through a **raw pointer**:

```cpp
class Texture {
public:
    Texture(int w, int h)
        : size_(static_cast<std::size_t>(w) * h),
          pixels_(new std::uint8_t[size_]{}) {}    // acquire a heap buffer

    ~Texture() { delete[] pixels_; }                // release it (RAII)

private:
    std::size_t size_;
    std::uint8_t* pixels_;                          // owning raw pointer
};

int main() {
    Texture a(4, 4);
    Texture b = a;       // compiler-generated copy: copies size_ and copies the POINTER pixels_
}                        // b's destructor deletes the buffer... then a's destructor deletes it AGAIN
```

```text
after Texture b = a:

 a: [ size_=16 | pixels_ ]──┐
                            ├──> [ 16 bytes on the heap ]   ONE buffer, TWO owners
 b: [ size_=16 | pixels_ ]──┘
```

```text
==50259==ERROR: AddressSanitizer: attempting double-free on 0x6020000000d0 in thread T0:
    #1 0x000100fd4a04 in Texture::~Texture() shallow.cpp:6
```

Copying the pointer gave two objects that both believe they own the buffer. Any write through `b` also changes `a`, and the second destructor frees memory that has already been freed. This is a **shallow copy**, the same idea as Python's shallow copy of a nested list, but here it crashes instead of just sharing data.

> This is the learning plan's check question "What breaks if you copy a class that owns heap memory but has no copy constructor?", and milestone 0.2c has you hit this crash on purpose.

## 3. Deep Copy: Writing the Copy Operations

The fix is a **deep copy**: give the new object its own buffer, and copy the contents:

```cpp
class Texture {
public:
    // ... constructor and destructor as before ...

    // Copy constructor: allocate a new buffer and copy the pixels into it
    Texture(const Texture& other)
        : size_(other.size_), pixels_(new std::uint8_t[other.size_]) {
        std::copy(other.pixels_, other.pixels_ + size_, pixels_);
    }

    // Copy assignment: replace our buffer with a copy of other's
    Texture& operator=(const Texture& other) {
        if (this == &other) return *this;                 // self-assignment: a = a must not free our own buffer

        std::uint8_t* fresh = new std::uint8_t[other.size_];   // allocate FIRST: if this throws, *this is untouched
        std::copy(other.pixels_, other.pixels_ + other.size_, fresh);

        delete[] pixels_;                                  // only now release the old buffer
        pixels_ = fresh;
        size_ = other.size_;
        return *this;                                      // returning *this allows a = b = c
    }
};
```

```cpp
Texture a(4, 4);
a.set(0, 255);
Texture b = a;      // copy ctor
b.set(0, 7);
// a[0] = 255, b[0] = 7: independent buffers
```

Assignment has more to do than the copy constructor, because the target already owns a buffer that has to be released, and `a = a` must still work.

## 4. Move Semantics: Stealing Instead of Copying

Copying is wasteful when the source is about to be destroyed anyway:

```cpp
Texture make_texture() {
    Texture t(1024, 1024);   // 1 MB of pixels
    return t;                // t is about to die...
}

Texture c(2, 2);
c = make_texture();          // ...so deep-copying 1 MB out of it, then freeing it, is pure waste
```

A **move** transfers the source's resources to the destination, then leaves the source empty but still valid, so its destructor has nothing to free:

```text
before move:                              after move:
 src: [ size_=16 | pixels_ ]──> buffer     src: [ size_=0 | nullptr ]
 dst: [ (being created)    ]               dst: [ size_=16 | pixels_ ]──> same buffer, no copy
```

### Move Constructor and Move Assignment

They take an **rvalue reference** `Texture&&`, which binds only to objects that are about to expire (see [References](<../Memory/References.md#7-lvalues-rvalues-and--preview>)):

```cpp
class Texture {
public:
    // Move constructor: take over other's buffer
    Texture(Texture&& other) noexcept
        : size_(other.size_), pixels_(other.pixels_) {
        other.pixels_ = nullptr;           // leave other empty, so its destructor deletes nothing
        other.size_ = 0;
    }

    // Move assignment: free our buffer, then take over other's
    Texture& operator=(Texture&& other) noexcept {
        if (this == &other) return *this;
        delete[] pixels_;
        pixels_ = other.pixels_;
        size_ = other.size_;
        other.pixels_ = nullptr;
        other.size_ = 0;
        return *this;
    }
};
```

A move costs a few pointer assignments, whatever the size of the buffer.

### Which One Gets Called?

The compiler picks the overload from the **value category** of the source:

| Source expression | Category | Calls |
|---|---|---|
| A named variable: `a` | lvalue | Copy |
| A function's returned temporary: `make_texture()` | rvalue | Move |
| `std::move(a)` | rvalue (xvalue) | Move |

Tracing a program that prints from each special member:

```text
Texture b = a;                    ->  copy ctor
c = a;                            ->  copy assign
Texture d = std::move(a);         ->  move ctor       a.size() = 0, d.size() = 16
c = make_texture();               ->  ctor, move assign
Texture e = make_texture();       ->  ctor            (no copy or move at all, see copy elision below)
```

## 5. `std::move` Doesn't Move Anything

`std::move(x)` is just a **cast** to an rvalue reference. It means "I'm done with `x`; you may steal from it". The actual moving happens in the move constructor or move assignment it causes to be selected:

```cpp
std::string s1 = "hello";
std::string s2 = std::move(s1);   // std::string's move constructor steals s1's buffer

// s1 == ""        (moved-from: valid but unspecified; empty in practice for std::string)
// s2 == "hello"

std::vector<int> big(1'000'000, 1);
std::vector<int> stolen = std::move(big);   // O(1): three pointers change hands
// big.size() == 0, stolen.size() == 1000000
```

After moving from an object, you may only **destroy it** or **assign it a new value**. Don't read its contents.

Common places to use `std::move`:

```cpp
names.push_back(std::move(name));          // insert into a container without copying, when name is no longer needed

Player::Player(std::string name)
    : name_(std::move(name)) {}            // "sink" parameter: take by value, then move into the member
```

> Don't write `return std::move(local);`. Returning a local already moves it (or elides it entirely), and the explicit `std::move` actually prevents copy elision.

## 6. Copy Elision: No Copy at All

In some cases the compiler constructs the object **directly in its final location**, skipping both the copy and the move:

```cpp
Texture make_texture() {
    return Texture(8, 8);      // returning a temporary: elision is GUARANTEED since C++17
}

Texture make_named() {
    Texture t(8, 8);
    return t;                  // returning a named local: elision is allowed and almost always done (NRVO)
}

Texture e = make_texture();    // prints only "ctor"
```

So returning large objects **by value** is efficient. Don't return pointers or use output parameters just to avoid a copy.

## 7. Why `noexcept` Matters on Move Operations

When a `std::vector` grows, it moves its elements into a new buffer (see [STL Containers](<../Standard Library/STL Containers.md>)). If a move constructor could throw halfway through, the vector couldn't restore its original state. So the vector **only moves elements if the move constructor is `noexcept`**, and **copies** them otherwise:

```text
vector growth, move constructor NOT noexcept:  copy   <- every element deep-copied on every reallocation
vector growth, move constructor noexcept:      move   <- cheap
```

Always mark move constructors and move assignment `noexcept`. They usually just swap pointers, so they can't throw.

## 8. The Rule of Three, Five, and Zero

The special member functions come as a package:

```text
~T()                          destructor
T(const T&)                   copy constructor
T& operator=(const T&)        copy assignment
T(T&&) noexcept               move constructor
T& operator=(T&&) noexcept    move assignment
```

**Rule of Three** (pre-C++11): if a class needs a custom destructor, copy constructor, **or** copy assignment, it almost certainly needs **all three**. A custom destructor means the class owns something, and the default member-by-member copy doesn't know that.

**Rule of Five** (C++11): the same, plus the two move operations. Declaring a destructor or a copy operation **suppresses** the compiler-generated move operations, so without writing them, every "move" silently falls back to a copy.

**Rule of Zero** (the goal): write classes that need **none** of the five, by holding resources in members that already manage themselves:

```cpp
class Texture {
public:
    Texture(int w, int h) : pixels_(static_cast<std::size_t>(w) * h) {}
    // no destructor, no copy or move operations written:
    // the compiler-generated versions copy/move/destroy the vector correctly
private:
    std::vector<std::uint8_t> pixels_;   // owns the buffer instead of a raw pointer
};
```

Only the small set of classes whose whole job is to manage a resource (like `std::vector` itself, or the RAII wrapper and dynamic array in milestones 0.2b and 0.2c) should write the five. Everything else should follow the rule of zero.

### Move-Only and Non-Copyable Types

Some resources shouldn't be duplicated at all, such as a unique file handle, a thread, or a lock. Delete the copy operations and keep the moves:

```cpp
class FileHandle {
public:
    FileHandle(const FileHandle&) = delete;              // copying is a compile error
    FileHandle& operator=(const FileHandle&) = delete;
    FileHandle(FileHandle&&) noexcept;                   // ownership can be transferred
    FileHandle& operator=(FileHandle&&) noexcept;
};
```

`std::unique_ptr`, `std::ofstream`, and `std::thread` all work this way (see [Smart Pointers](<../Memory/Smart Pointers.md>)).

> Unreal's `TArray`, `FString`, and `TMap` support move semantics too (Unreal uses `MoveTemp(x)` in place of `std::move(x)`). `UObject`s are never copied or moved like values at all: you always handle them through pointers.

## Summary

| Python | C++ |
|---|---|
| `b = a` shares the object | `b = a` copies the object (copy ctor / copy assignment) |
| `copy.copy(a)` / `copy.deepcopy(a)` | Whatever the class's copy constructor does: member-wise by default, deep if you write it |
| No equivalent | Move: transfer resources out of an object that's about to expire |
| Garbage collector frees memory | The destructor frees memory; copy and move must keep exactly one owner |

```text
Owns a resource through a raw pointer?  -> write all five (rule of five), with noexcept moves
Everything else                          -> write none of them (rule of zero), use vector/string/unique_ptr members
std::move(x)                             -> a cast: "x may be stolen from". Don't use x afterwards
Return by value                          -> cheap: copy elision or an automatic move
```
