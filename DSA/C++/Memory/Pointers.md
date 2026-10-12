# Pointers in C++

A **pointer** is a variable whose value is a **memory address**: the location of some other object.

→ Every Python variable already works a little like a pointer: a name holding a reference to an object stored elsewhere. Python just never lets you see the address, do arithmetic on it, or have it point at garbage. C++ pointers make the address an explicit value you can store, copy, compare, and follow. That's what makes them powerful, and dangerous.

```cpp
int score = 42;
int* p = &score;     // p holds the address of score

std::cout << p;      // 0x16f249fc8   the address
std::cout << *p;     // 42            the value stored at that address
```

Conceptually:

```text
        address 0x16f249fc8
             |
             v
 score: [   42   ]
             ^
             |
 p:     [ 0x16f249fc8 ]   (p is itself a variable, 8 bytes, with its own address)
```

---

## 1. The Two Operators: `&` and `*`

| Syntax | Name | Meaning |
|---|---|---|
| `&x` | Address-of | "Give me the address where `x` lives" |
| `*p` | Dereference | "Go to the address in `p` and use the object there" |

They're opposites: `*&x` is just `x`.

```cpp
int score = 42;
int* p = &score;   // & : take the address

*p = 50;           // * : write through the pointer
std::cout << score;  // 50: score itself changed, because p points at it
```

### Declaring Pointers

`T*` means "pointer to `T`". The type of the pointer says what kind of object lives at the address, so the compiler knows how many bytes to read and how to interpret them:

```cpp
int*    pi;          // pointer to int
double* pd;          // pointer to double
Player* pp;          // pointer to Player
char*   pc;          // pointer to char
```

Every pointer is the same size whatever it points to: 8 bytes on a 64-bit machine.

```cpp
sizeof(int*)      // 8
sizeof(Player*)   // 8, even if a Player is 500 bytes
```

> **Declaration gotcha:** the `*` binds to the **name**, not the type.
>
> ```cpp
> int* a, b;     // a is int*, but b is a plain int!
> int *a, *b;    // two pointers
> ```
>
> Declaring one variable per line avoids the problem entirely.

### `&` and `*` Mean Different Things in Declarations

This is the most confusing part of the syntax. The same symbols mean different things depending on where they appear:

```cpp
int* p = &x;    // in a DECLARATION, * means "p is a pointer"; in the expression, & means "address of x"
int& r = x;     // in a DECLARATION, & means "r is a reference" (see References)
*p = 5;         // in an EXPRESSION, * means "dereference p"
a * b;          // between two values, * is multiplication
a & b;          // between two values, & is bitwise AND
```

## 2. `nullptr`: Pointing at Nothing

A pointer that doesn't point at any object should hold `nullptr`. It plays the same role as Python's `None` for "no object here":

```cpp
Player* target = nullptr;      // no target yet

if (target != nullptr) {       // like Python's: if target is not None:
    target->attack();
}

if (target) { }                // same check: a null pointer converts to false
```

Dereferencing a null pointer is **undefined behavior**. In practice it's an immediate crash:

```cpp
int* p = nullptr;
std::cout << *p;   // Segmentation fault, exit code 139. No AttributeError, no traceback
```

> Older code uses `NULL` or `0` for null pointers. Always use `nullptr` (C++11). It has its own type, so it can't be confused with the integer `0` in overloaded functions.

> In Unreal, checking pointers before use is essential: `if (IsValid(Target))` for `UObject`s, since a pointer can be non-null but point to an object that is pending destruction.

## 3. Pointers to Objects: `->`

To reach a member through a pointer, you dereference and then use `.`. `->` is shorthand for that:

```cpp
struct Player {
    int health = 100;
    void heal() { health = 100; }
};

Player pl;
Player* ptr = &pl;

(*ptr).health -= 10;   // dereference, then member access (parentheses needed: . binds tighter than *)
ptr->health -= 10;     // the same thing; this is how everyone writes it
ptr->heal();           // calling a member function through a pointer
```

`->` shows up constantly in Unreal code, since almost every engine object is handled through a pointer: `GetOwner()->GetActorLocation()`.

## 4. Uninitialized, Dangling, and Wild Pointers

A pointer is only as valid as the object it points to, and the compiler can't check that:

```cpp
int* wild;                  // UNINITIALIZED: holds a random address. Dereferencing it is UB

int* dangling;
{
    int temp = 5;
    dangling = &temp;
}                           // temp is destroyed here
*dangling = 10;             // DANGLING: points at a dead object. UB

int* heap = new int(7);
delete heap;
*heap = 3;                  // USE-AFTER-FREE: the memory was returned to the allocator. UB
```

Python makes all three impossible: names must be bound before use, and an object stays alive as long as any reference to it exists. In C++, it's your job to make sure a pointer never outlives what it points at. AddressSanitizer catches these at runtime (see [Undefined Behavior and Sanitizers](<Undefined Behavior and Sanitizers.md>)).

Rules that prevent most of these bugs:

1. Initialize every pointer: to a real address, or to `nullptr`.
2. Never keep a pointer to a local variable after its scope ends.
3. Don't use raw `new`/`delete` for ownership. Use smart pointers (see [Smart Pointers](<Smart Pointers.md>)).

## 5. Pointers and Arrays

### Array-to-Pointer Decay

The name of a C-style array **converts** to a pointer to its first element in almost every expression:

```cpp
int nums[4] = {10, 20, 30, 40};
int* q = nums;        // same as &nums[0]

std::cout << *q;      // 10
```

That conversion loses the array's size:

```cpp
sizeof(nums)   // 16: the whole array (4 ints * 4 bytes)
sizeof(q)      // 8:  just a pointer

void print_size(int arr[]) {     // int arr[] in a parameter really means int* arr
    sizeof(arr);                 // 8, and clang warns:
}                                // sizeof on array function parameter will return size of 'int *'
```

That's why C-style functions take the length as a separate parameter (`void f(int* arr, int n)`), and why modern C++ prefers `std::array`, `std::vector`, and `std::span`, which keep their size. See [Arrays and Strings](<../Fundamentals/Arrays and Strings.md>).

### Pointer Arithmetic

Adding `n` to a pointer moves it forward by `n` **elements**, not `n` bytes. The compiler multiplies by `sizeof(T)` for you:

```cpp
int nums[4] = {10, 20, 30, 40};
int* q = nums;

q          // 0x16f249fd0
q + 1      // 0x16f249fd4: 4 bytes later, one int forward

*(q + 1)   // 20
q[2]       // 30: subscripting IS pointer arithmetic. q[i] means *(q + i)
*(nums + 3)  // 40

int* end = nums + 4;    // one past the last element: a valid address to compare against, but never dereference it
end - nums              // 4: subtracting pointers gives the number of elements between them
```

Iterating with a pointer is the low-level version of a `for` loop, and it's exactly what iterators generalize (see [Iterators and Algorithms](<../Standard Library/Iterators and Algorithms.md>)):

```cpp
for (int* it = nums; it != nums + 4; ++it) {
    std::cout << *it << " ";   // 10 20 30 40
}
```

Pointer arithmetic that leaves the array (anywhere other than its elements or one past the end) is UB. There is no bounds check: `q[100]` just computes an address and reads whatever is there.

## 6. Pointers to Pointers

A pointer is itself a variable with an address, so you can point at it:

```cpp
int score = 42;
int* p = &score;
int** pp = &p;       // pointer to (pointer to int)

**pp = 7;            // follow two hops: pp -> p -> score
std::cout << score;  // 7
```

```text
pp: [addr of p] ---> p: [addr of score] ---> score: [ 7 ]
```

You'll see `T**` in C-style APIs that need to change which object a caller's pointer points to, and in `char** argv` (an array of C strings; see [Program Structure](<../Fundamentals/Program Structure.md#command-line-arguments>)).

## 7. Pointers and `const`

`const` can apply to the **pointed-to object**, to the **pointer itself**, or to both. Read declarations **right to left**:

```cpp
int x = 1, y = 2;

const int* a = &x;        // "a is a pointer to a const int"
*a = 5;                   // ERROR: can't modify x through a
a = &y;                   // OK: a itself can be re-pointed

int* const b = &x;        // "b is a const pointer to an int"
*b = 5;                   // OK: can modify x through b
b = &y;                   // ERROR: b can't be re-pointed

const int* const c = &x;  // "c is a const pointer to a const int": neither can change
```

| Declaration | Change the value through it? | Re-point it? |
|---|---|---|
| `int* p` | Yes | Yes |
| `const int* p` (same as `int const* p`) | No | Yes |
| `int* const p` | Yes | No |
| `const int* const p` | No | No |

`const T*` is what you'll use most: a function that only needs to **look at** an object takes `const Player*`. More in [Const Correctness](<Const Correctness.md>).

## 8. `void*`: A Pointer to Anything

`void*` holds an address without saying what type lives there. You can't dereference it until you cast it back to the right type:

```cpp
int score = 7;
void* vp = &score;                    // any object pointer converts to void*
// *vp;                               // ERROR: can't dereference void*
int* back = static_cast<int*>(vp);    // you must know the real type
std::cout << *back;                   // 7
```

It's mostly found in C APIs (`malloc` returns `void*`). In C++, templates (see [Templates](<../Fundamentals/Templates.md>)) give you "works with any type" while keeping type safety.

## 9. Function Pointers

Functions live in memory too (the code region in [Stack and Heap](<Stack and Heap.md#the-process-memory-layout>)), so you can point at them and call through the pointer. It's the low-level version of passing a function as a value in Python:

```cpp
int add(int a, int b) { return a + b; }
int mul(int a, int b) { return a * b; }

int apply(int (*op)(int, int), int x, int y) {   // op: pointer to a function taking (int, int) returning int
    return op(x, y);
}

apply(add, 3, 4);   // 7
apply(mul, 3, 4);   // 12
```

The syntax is ugly, so a type alias helps (`using BinaryOp = int (*)(int, int);`, see [Enums and Type Aliases](<../Fundamentals/Enums and Type Aliases.md#typedef-older-syntax>)). In modern code, lambdas and `std::function` usually replace function pointers (see [Lambdas](<../Fundamentals/Lambdas.md>)).

## 10. `char*` Is Special for Output

`std::cout` treats a `char*` as a C-style string and prints the **text**, not the address. Every other pointer type prints as an address:

```cpp
const char* s = "hello";
std::cout << s;            // hello
std::cout << (void*)s;     // 0x100bb5652: cast to void* to see the address
std::cout << *s;           // h: dereferencing gives the first char
std::cout << s[1];         // e
```

## 11. When to Use Pointers

In modern C++, raw pointers are for **observing**, not **owning**:

| Situation | Use |
|---|---|
| Function parameter that must refer to an object, never null | Reference: `Player&` / `const Player&` |
| Function parameter where "no object" is valid | Pointer: `Player*` (check for `nullptr`) |
| Owning a heap object (responsible for deleting it) | `std::unique_ptr<Player>` / `std::shared_ptr<Player>` |
| Non-owning link to an object owned elsewhere (e.g. an AI's current target) | Raw pointer `Player*`, or `std::weak_ptr` if the owner is a `shared_ptr` |
| Dynamic array | `std::vector<T>`, not `new T[n]` |
| Iterating through an array at a low level | Pointer arithmetic, or better, iterators |

> In Unreal, a raw `UObject*` member is acceptable only when marked `UPROPERTY()`, so the garbage collector can see it. Newer code often uses `TObjectPtr<T>` for members.

The other way to refer to an object without copying it is a reference, which is safer and more common for parameters. The comparison is at the end of [References](<References.md#pointers-vs-references>).
