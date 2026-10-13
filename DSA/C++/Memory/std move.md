# `std::move`

`std::move(x)` doesn't move anything. It is a **cast** that makes `x` count as an rvalue. That changes which overload the compiler picks, and the overload it picks (the move constructor or move assignment) does the actual work.

```text
std::move(x)  ──cast──►  x counts as an rvalue  ──overload resolution──►  operator=(T&&)       if it exists → move
                                                                     ╰─►  operator=(const T&)  otherwise     → copy
```

Read [Copy and Move Semantics](<../OOP/Copy and Move Semantics.md>) first. This note is a condensed reference for `std::move`, with the memory picture behind it. Lvalues and rvalues are introduced in [References](<References.md#7-lvalues-rvalues-and--preview>).

---

## 1. Lvalues and Rvalues

Every expression is one of two kinds:

- **lvalue**: refers to an object with a lasting identity. It has an address, and code after this line can still use it.
  `name`, `v[0]`, `*ptr`
- **rvalue**: a temporary with no name. It is destroyed at the end of the full expression (the `;`), so no later code can refer to it.
  `5`, `a + b`, `std::string("hi")`, a function's returned value

Lvalue/rvalue is a property of the **expression**, not the type. A named `T&&` variable is still an lvalue when you use it, because it has a name.

## 2. The Three Kinds of Reference

| Reference | Binds to lvalues | Binds to rvalues |
|---|---|---|
| `T&` | ✅ | ❌ |
| `const T&` | ✅ | ✅ (extends the temporary's lifetime) |
| `T&&` | ❌ | ✅ (extends the temporary's lifetime) |

`std::string& r = name + " Singh";` is an error. The only object holding `"Anu Singh"` is a temporary that dies at the `;`, so `r` would dangle, and anything written through `r` would be lost. `const T&` makes it read-only, and `T&&` states that binding to a temporary is deliberate.

## 3. What Is Actually in a `std::string` or `std::vector`

Both are small, fixed-size objects (typically 24 bytes) that live on the stack. They hold **pointers** to the real data, which lives on the heap:

```text
        stack                              heap
┌───────────────────┐
│ std::string name  │
│   ptr ────────────┼──────────►  [ A n u   S i n g h ]
│   size     = 9    │
│   capacity = 15   │
└───────────────────┘

┌───────────────────┐
│ std::vector<int> v│
│   begin ──────────┼──────────►  [ 1 | 2 | 3 | 4 | _ | _ ]
│   end ────────────┼──────────────────────────────┘   │
│   cap_end ────────┼──────────────────────────────────┘
└───────────────────┘
```

`sizeof(name)` and `sizeof(v)` are the same whether the data is 3 elements or 3 million. The heap buffer holds the bulk of the data, and the stack object owns it (RAII: the destructor frees it).

> **Small String Optimization (SSO):** most `std::string` implementations store short strings (about 15–22 chars) inside the stack object, with no heap buffer. Moving a short string copies those few chars, which is still cheap. `std::vector` has no equivalent; its elements are always on the heap.

## 4. Copy vs Move

**Copy** (`x = name;`) allocates a **new** heap buffer and copies every element into it. That gives two independent owners and costs O(n):

```text
name: ptr ───► [ A n u   S i n g h ]
x:    ptr ───► [ A n u   S i n g h ]     ← new allocation
```

**Move** (`x = std::move(name);`) copies only the **pointers** into `x`, then sets the source's pointer to null. The cost is O(1) whatever the size:

```text
name: ptr = nullptr, size = 0            ← empty shell, still a valid object
x:    ptr ───► [ A n u   S i n g h ]     ← the same buffer name used to own
```

Nulling the source is necessary. Each object's destructor frees its buffer, so if both kept the pointer, the buffer would be freed twice (double free). After the move:

- `x`'s destructor frees the buffer.
- `name`'s destructor still runs at the end of its scope, but `delete[] nullptr` does nothing.

The buffer is freed exactly once. A move changes **which object owns** the buffer, and RAII still guarantees it gets freed.

## 5. Why Value Category Decides Copy vs Move

A move empties the source, so it is only safe when nothing will look at the source again. The compiler can't know your intentions, so it uses value category as the test:

```cpp
x = name;                // lvalue: later code might read name      → const T&  → copy
x = name + " Singh";     // rvalue: unnamed, dies at ';'             → T&&       → move (automatic)
x = std::move(name);     // lvalue cast to rvalue: "I'm done with it" → T&&       → move
```

`T&&` binds **only** to rvalues, so the move overload is selected **only** when stealing is safe. `std::move` is how you tell the compiler that a named object qualifies.

## 6. What `std::move` Does Not Guarantee

- **Not always faster.** `int`, `double` and plain structs have no buffer, so moving them is copying. A class with no move operations falls back to the copy overload.
- **Can fail to compile.** If the move overload is declared `= delete`, it still wins overload resolution for an rvalue, and the call is an error. It does not fall back to copy. (A *defaulted* move that ends up deleted is ignored, so the copy is used.)
- **Moved-from is not UB.** Standard library types are left **valid but unspecified**. Assigning a new value, calling `clear()`, or destroying the object is fine. Don't rely on its value.
- **No guarantee that `x != y`.** For `int`, `x == y` after the move, and `x = std::move(x)` is a self-move.

> Unreal uses `MoveTemp(x)` in place of `std::move(x)`. `FString` and `TArray` have the same stack-object → heap-buffer layout and move the same way.
