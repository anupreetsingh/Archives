# Function Specifiers and Qualifiers in C++

- A function declaration can carry many extra keywords besides its return type, name, and parameters: `static`, `virtual`, `const`, `noexcept`, `override`, `= 0`, and more. Most of them have already appeared one at a time in earlier notes.
- This note puts them together. It explains **where** each keyword goes, **what kind of thing** it says about the function, and which ones must be repeated when the body is defined outside the class.

**KEY IDEA:** position tells you the job. Keywords **before the name** say what kind of function it is. Qualifiers **right after the parameter list** say what the function may do to the object it's called on. Keywords at the **end** connect the function to a base class or say where its body comes from.

---

## The Full Shape

Read a declaration left to right. Each zone is optional:

```text
[[nodiscard]]  virtual   int   get   (int i)   const & noexcept   override final   = 0  ;
      ①           ②       ③     ④      ④             ⑤                 ⑥            ⑦
```

| Zone | Contains | Answers |
|---|---|---|
| ① Attributes | `[[nodiscard]]`, `[[deprecated]]`, `[[maybe_unused]]` | What should the compiler warn about? |
| ② Before the name | `static`, `virtual`, `inline`, `constexpr`, `consteval`, `explicit`, `friend` | What kind of function is this? |
| ③④ Core | Return type, name, parameters | (see [Functions](<../Fundamentals/Functions.md#anatomy-of-a-function>)) |
| ⑤ After the `()` | `const`, `&` / `&&`, `noexcept` | What may it do to the object, and can it throw? |
| ⑥ Virt-specifiers | `override`, `final` | How does it relate to the base class's version? |
| ⑦ Body choice | `= 0`, `= default`, `= delete` | Where does the body come from, if anywhere? |

The order inside zones ⑤ and ⑥ is fixed. The compiler rejects other orders:

```cpp
int f() override const;    // ERROR: 'const' qualifier may not appear after the virtual specifier 'override'
int f() noexcept const;    // ERROR: expected ';' at end of declaration list
int f() const & noexcept override final;   // OK: cv, then ref, then noexcept, then override/final
```

## 1. Attributes: Hints for Warnings and Optimization

Attributes go in `[[double brackets]]` at the very front. They don't change what the function does. They tell the compiler what to warn about or what it may assume:

```cpp
[[nodiscard]] bool try_connect();          // warn if the caller ignores the result
[[deprecated("use load_v2")]] void load(); // warn on every call
void log([[maybe_unused]] int level);      // don't warn that 'level' is unused
```

See [`[[nodiscard]]`](<../Fundamentals/Functions.md#nodiscard>) for the most common one.

## 2. Before the Name: What Kind of Function It Is

These keywords describe the function as a whole: whether it belongs to an object, how calls are dispatched, how it links, and when it can run.

| Specifier | Meaning | Covered in |
|---|---|---|
| `static` (member) | Belongs to the class, not an object. Has no `this` | [Classes and Objects](<Classes and Objects.md#static-members-class-attributes-and-static-methods>) |
| `static` (free function) | Visible only inside this `.cpp` file (internal linkage) | [Scope and Namespaces](<../Fundamentals/Scope and Namespaces.md#the-many-meanings-of-static>) |
| `virtual` | Calls through a base pointer or reference run the derived version | [Virtual Functions and Vtables](<Virtual Functions and Vtables.md>) |
| `inline` | May be defined in several `.cpp` files without linker errors | [Headers and Source Files](<../Fundamentals/Headers and Source Files.md#definitions-in-headers-why-they-break-and-inline>) |
| `constexpr` | **Can** run at compile time | [Const Correctness](<../Memory/Const Correctness.md#5-constexpr-known-at-compile-time>) |
| `consteval` (C++20) | **Must** run at compile time | [Const Correctness](<../Memory/Const Correctness.md#5-constexpr-known-at-compile-time>) |
| `explicit` | A constructor or conversion operator that's never used for implicit conversions | [Classes and Objects](<Classes and Objects.md#explicit>) |
| `friend` | Not a member, but may access private members | [Classes and Objects](<Classes and Objects.md#friend>) |

```cpp
class Enemy {
public:
    explicit Enemy(int health);                     // Enemy e = 50;  won't compile
    static Enemy spawn_default();                   // Enemy::spawn_default(), no object needed
    virtual void update(float dt);                  // dispatched at runtime
    constexpr int max_health() const { return 100; }
    friend void debug_print(const Enemy& e);        // a free function with private access
};
```

Some combinations make no sense, and the compiler says so:

```cpp
virtual static void f();   // ERROR: 'virtual' can only appear on non-static member functions
```

A `static` function has no object, so there's no object type to dispatch on.

`inline` doesn't force a call to be inlined. It's a rule about **linking**: identical definitions in several `.cpp` files are allowed and get merged. Whether a call is actually replaced by the body is decided by the optimizer. Functions defined inside a class body and `constexpr` functions are implicitly `inline`.

## 3. After the Parameters: What It May Do to the Object

Every non-static member function receives a hidden parameter: the object it was called on, available as `this` (see [Classes and Objects](<Classes and Objects.md#this-cs-self>)). It's the C++ version of Python's explicit `self`. The object has no slot in the parameter list where you could write `const`, so its qualifiers go **after** the parameter list instead.

### `const`: Doesn't Modify the Object

```cpp
int health() const { return health_; }   // 'this' is a const Player*: members are read-only
```

A `const` member function can be called on `const` objects and through `const&`. Non-`const` ones can't:

```cpp
void show(const Player& p) {
    p.health();        // OK
    p.take_damage(5);  // ERROR: 'this' argument to member function 'take_damage' has type
                       //        'const Player', but function is not marked const
}
```

Details, including `const` overloads and `mutable`: [Const Correctness](<../Memory/Const Correctness.md#4-const-member-functions>).

### `&` and `&&`: Lvalue or Temporary Objects Only

**Ref-qualifiers** restrict a member function to objects that are lvalues (`&`) or temporaries (`&&`). Lvalues and rvalues are introduced in [References](<../Memory/References.md#7-lvalues-rvalues-and--preview>).

```cpp
struct A {
    int f() &;     // only on lvalues (named objects)
    int g() &&;    // only on temporaries
};

A a;
a.g();         // ERROR: 'this' argument to member function 'g' is an lvalue,
               //        but function has rvalue ref-qualifier
A{}.f();       // ERROR: 'this' argument to member function 'f' is an rvalue,
               //        but function has non-const lvalue ref-qualifier
```

Their main use is overloading on the object's value category, so a temporary can hand over its contents instead of copying them:

```cpp
class Buffer {
public:
    const std::string& data() const & { return data_; }    // named Buffer: return a reference
    std::string data() &&             { return std::move(data_); }   // temporary: move the string out
private:
    std::string data_;
};

Buffer b;
const std::string& r = b.data();         // lvalue: no copy
std::string s = load_buffer().data();    // temporary: string is moved, not copied
```

With only the `const&` version, `load_buffer().data()` would return a reference into a temporary `Buffer` that's destroyed at the end of the statement. Copying from it in time works, but `const auto& r = load_buffer().data();` would leave `r` dangling. The `&&` overload returns the string by value, which avoids both the copy and the dangling reference. `std::optional::value()` is overloaded this way.

If one overload of a name has a ref-qualifier, all of them must:

```cpp
struct A { int f() &; int f(); };
// ERROR: cannot overload a member function without a ref-qualifier with a member function with ref-qualifier '&'
```

### `noexcept`: Never Throws

```cpp
void swap(Buffer& other) noexcept;
```

`noexcept` is a promise to callers and the optimizer. If the function throws anyway, the program calls `std::terminate`. Unlike `const` and `&`, it isn't about the object. It sits in this zone because it describes how the function behaves when called. See [Error Handling](<../Fundamentals/Error Handling.md#3-noexcept>), and [Copy and Move Semantics](<Copy and Move Semantics.md#7-why-noexcept-matters-on-move-operations>) for why move operations need it.

`noexcept` also applies to free functions. `const` and the ref-qualifiers don't, since a free function has no object.

### Part of the Function's Type

Everything in zone ⑤ is part of the function's **type**, like the parameters are. That has two consequences:

- `const` and ref-qualifiers create **different overloads**. `at(i)` and `at(i) const` can coexist, and the compiler picks one based on the object.
- `noexcept` is part of the type (since C++17) but **can't** be used to overload. Two declarations that differ only in `noexcept` are an error:

```cpp
void f();
void f() noexcept;   // ERROR: exception specification in declaration does not match previous declaration
```

## 4. Virt-Specifiers: `override` and `final`

These come after zone ⑤ and only apply to virtual functions:

```cpp
class Guard : public Enemy {
public:
    void update(float dt) override;         // must override a base virtual function
    void on_alert() override final;         // overrides, and no class below Guard may override it again
};
```

`override` turns a silent mistake (a typo in the signature creates a new function) into a compile error. Details in [OOP Four Pillars](<OOP Four Pillars.md#override-and-final>).

```cpp
struct B { void f(float); };
struct D : B { void f(float) override; };   // ERROR: only virtual member functions can be marked 'override'
```

`override` and `final` are **contextual keywords**: they only have special meaning in this position. They were added in C++11, and placing them at the end meant existing code with variables named `override` or `final` kept compiling. `int final = 4;` is still legal.

## 5. Body Choice: `= 0`, `= default`, `= delete`

The last zone, just before the `;`, says where the body comes from:

| Form | Meaning | Applies to |
|---|---|---|
| `= 0` | Pure virtual: no body here, derived classes must override. The class becomes abstract | Virtual member functions |
| `= default` | The compiler generates the usual body | Special members (constructors, destructor, copy/move, `operator=`) and `==` / `<=>` |
| `= delete` | Calling it is a compile error | Any function |

```cpp
class Shape {
public:
    virtual ~Shape() = default;
    virtual double area() const = 0;               // see Abstraction in OOP Four Pillars
};

class FileHandle {
public:
    FileHandle(const FileHandle&) = delete;        // non-copyable
    bool operator==(const FileHandle&) const = default;
};
```

`= delete` also works on free functions, to block conversions you don't want:

```cpp
void set_volume(int level);
void set_volume(double) = delete;

set_volume(0.5);   // ERROR: call to deleted function 'set_volume'
```

Without the deleted overload, `0.5` would be silently converted to `0`.

Related notes: [Abstraction](<OOP Four Pillars.md#2-abstraction>) for `= 0`, [Move-Only and Non-Copyable Types](<Copy and Move Semantics.md#move-only-and-non-copyable-types>) for `= delete`, and [Defaulted Comparisons](<Operator Overloading.md#defaulted-comparisons-c20>) for `= default` on operators.

## 6. Trailing Return Types and `requires`

Two more pieces can appear after the qualifiers:

- A **trailing return type** `-> T` comes after `noexcept` and before `override` (see [`auto` Return Type](<../Fundamentals/Functions.md#auto-return-type>)).
- A **`requires` clause** constrains a template's member function (see [Concepts](<../Fundamentals/Templates.md#6-concepts-constraining-t-c20>)).

```cpp
auto area() const noexcept -> double override;

template <typename T>
struct Wrapper {
    void print() const requires std::integral<T>;   // only exists when T is an integer type
};
```

## 7. Declaring in the Class, Defining Outside It

When the body moves to a `.cpp` file (see [Defining Member Functions Outside the Class](<Classes and Objects.md#defining-member-functions-outside-the-class>)), some keywords must be repeated and others must not appear. The rule follows from the zones:

- **Zone ⑤ (`const`, `&`/`&&`, `noexcept`) is repeated.** It's part of the function's type, so the definition must match the declaration exactly.
- **Zones ② and ⑥ (`static`, `virtual`, `explicit`, `friend`, `override`, `final`) appear only in the class.** They describe the function's role in the class, and that was settled when the class was defined.

```cpp
// Enemy.h
class Enemy {
public:
    explicit Enemy(int health);
    static Enemy spawn_default();
    virtual int health() const noexcept;
};
```

```cpp
// Enemy.cpp
Enemy::Enemy(int health) : health_(health) {}       // no 'explicit'
Enemy Enemy::spawn_default() { return Enemy(100); } // no 'static'
int Enemy::health() const noexcept {                // const and noexcept repeated; no 'virtual'
    return health_;
}
```

The errors when this goes wrong:

```cpp
static Enemy Enemy::spawn_default() { ... }
// ERROR: 'static' can only be specified inside the class definition

virtual int Enemy::health() const noexcept { ... }
// ERROR: 'virtual' can only be specified inside the class definition

int Guard::health() const noexcept override { ... }
// ERROR: 'override' specifier is not allowed outside a class definition

int Enemy::health() noexcept { ... }      // const forgotten
// ERROR: out-of-line definition of 'health' does not match any declaration in 'Enemy'

int Enemy::health() const { ... }         // noexcept forgotten
// ERROR: 'health' is missing exception specification 'noexcept'
```

The remaining keywords follow their own rules:

- **`constexpr` / `consteval`** must be on every declaration. `non-constexpr declaration of 'f' follows constexpr declaration` is the error.
- **`inline`** may go on either the declaration or the definition.
- **`[[nodiscard]]`** on the declaration is enough, since that's what callers see.
- **`= default`** may go on the out-of-class definition instead: `Enemy::Enemy() = default;`. **`= delete`** must be on the first declaration.

## How Python Compares

Python expresses most of these with decorators, checked at runtime or only by type checkers. C++ checks them all at compile time:

| C++ | Python |
|---|---|
| `static` member function | `@staticmethod` |
| `virtual` | Nothing: every method is effectively virtual |
| `= 0` | `@abc.abstractmethod` |
| `override` | `@typing.override` (3.12+, type checkers only) |
| `final` | `@typing.final` (type checkers only) |
| `const` member function | No equivalent |
| `noexcept` | No equivalent |
| `= delete` | Raising `TypeError` from the method body |
| `[[deprecated]]` | `warnings.warn(..., DeprecationWarning)` or `@warnings.deprecated` (3.13+) |

> Unreal adds its own markers in the same positions. Macros like `UFUNCTION(BlueprintCallable)` go on the line **before** the declaration and register the function with Unreal's reflection system. `FORCEINLINE` sits in zone ② and asks the compiler much more firmly than `inline` to actually inline calls. Engine overrides follow the normal rules: `virtual void BeginPlay() override;` in the header, and `void AGuard::BeginPlay()` with neither keyword in the `.cpp`.

## Summary

1. Read declarations left to right: attributes → what kind of function → return type, name, parameters → what it may do to the object → base-class relationship → body choice.
2. Before the name (`static`, `virtual`, `inline`, `constexpr`, `explicit`, `friend`): the function's role.
3. After the `()` (`const`, `&`/`&&`, `noexcept`): a contract about the hidden `this` object and about throwing. Part of the function's type.
4. `const` and ref-qualifiers can overload a function. `noexcept` can't.
5. `override` and `final` come after the qualifiers, only on virtual functions, and are contextual keywords.
6. `= 0` makes a function pure virtual, `= default` asks for the generated body, and `= delete` forbids calls.
7. Out-of-class definitions repeat `const`, `&`, `noexcept` (and `constexpr`), and drop `static`, `virtual`, `explicit`, `friend`, `override`, `final`.
