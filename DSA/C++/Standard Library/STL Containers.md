# STL Containers in C++

The C++ standard library's containers (historically called the **STL**, Standard Template Library) are class templates (see [Templates](<../Fundamentals/Templates.md>)) that own and manage collections of objects. They're RAII types: each one frees its memory when it's destroyed (see [Object Lifetime and RAII](<../OOP/Object Lifetime and RAII.md>)).

Two differences from Python's built-in collections apply to all of them:

- **One element type per container**, fixed at compile time: `std::vector<int>` holds only `int`s. A Python list can mix types because it stores references.
- **Elements are stored by value**. `std::vector<Player>` holds the `Player` objects themselves, not references to them, and copying a container copies every element.

## Python to C++ at a Glance

| Python | C++ | Header | Underlying structure |
|---|---|---|---|
| `list` | `std::vector<T>` | `<vector>` | Dynamic array |
| `tuple` (fixed size) | `std::array<T, N>`, `std::pair`, `std::tuple` | `<array>`, `<utility>`, `<tuple>` | Inline |
| `dict` | `std::unordered_map<K, V>` | `<unordered_map>` | Hash table |
| `dict` with sorted keys | `std::map<K, V>` | `<map>` | Red-black tree (balanced BST) |
| `set` | `std::unordered_set<T>` | `<unordered_set>` | Hash table |
| sorted set | `std::set<T>` | `<set>` | Red-black tree |
| `collections.deque` | `std::deque<T>` | `<deque>` | Array of fixed-size blocks |
| `list` used as a stack | `std::stack<T>` | `<stack>` | Adaptor over `deque` |
| `collections.deque` used as a queue | `std::queue<T>` | `<queue>` | Adaptor over `deque` |
| `heapq` | `std::priority_queue<T>` | `<queue>` | Binary heap over `vector` |
| `collections.Counter` | `std::unordered_map<T, int>` | | |
| `collections.defaultdict` | `std::unordered_map` (its `[]` default-inserts) | | |

---

## 1. `std::vector`: the Default Container

`std::vector` is a growable, contiguous array. It's the right choice most of the time, even where Python programmers might expect a linked list or a deque, because contiguous memory is very cache-friendly (see [Object Memory Layout](<../Memory/Object Memory Layout.md#7-why-layout-matters-the-cache>)).

```cpp
#include <vector>

std::vector<int> a;               // empty
std::vector<int> b = {10, 20, 30};
std::vector<int> z(5);            // five zeros: {0, 0, 0, 0, 0}
std::vector<int> f(3, 7);         // three sevens: {7, 7, 7}, like Python's [7] * 3

std::vector<std::vector<int>> grid(3, std::vector<int>(4, 0));   // 3x4 grid of zeros
grid[1][2] = 5;                   // each row is an independent copy, unlike Python's [[0] * 4] * 3 trap
```

### Common Operations

Matching [List Manipulation](<../../Python/Fundamentals/List Manipulation.md>):

| Python | C++ | Cost |
|---|---|---|
| `lst.append(x)` | `v.push_back(x)` / `v.emplace_back(args...)` | Amortized O(1) |
| `lst.pop()` | `v.pop_back()` (returns **nothing**; read `v.back()` first) | O(1) |
| `lst[-1]` | `v.back()` | O(1) |
| `lst[0]` | `v.front()` / `v[0]` | O(1) |
| `lst[i]` | `v[i]` (unchecked) / `v.at(i)` (throws `std::out_of_range`) | O(1) |
| `len(lst)` | `v.size()` | O(1) |
| `not lst` | `v.empty()` | O(1) |
| `lst.insert(i, x)` | `v.insert(v.begin() + i, x)` | O(n) |
| `lst.pop(i)` / `del lst[i]` | `v.erase(v.begin() + i)` | O(n) |
| `lst.clear()` | `v.clear()` | O(n) |
| `x in lst` | `std::find(v.begin(), v.end(), x) != v.end()` | O(n) |
| `lst.sort()` | `std::sort(v.begin(), v.end())` | O(n log n) |
| `lst.reverse()` | `std::reverse(v.begin(), v.end())` | O(n) |
| `[x for x in lst if cond]` (in place) | `std::erase_if(v, pred)` (C++20) | O(n) |
| `lst2 = lst.copy()` | `auto v2 = v;` (plain assignment copies) | O(n) |
| `lst2 = lst` (alias) | `auto& v2 = v;` (reference) | O(1) |

```cpp
std::vector<int> nums = {10, 20, 30, 40};
nums.insert(nums.begin() + 1, 15);   // {10, 15, 20, 30, 40}
nums.erase(nums.begin());            // {15, 20, 30, 40}
nums.pop_back();                     // {15, 20, 30}

nums.at(10);                         // throws std::out_of_range: "vector"
nums[10];                            // UB: no check at all
```

`emplace_back(args...)` constructs the element **directly inside** the vector from constructor arguments, instead of building a temporary and moving it in:

```cpp
std::vector<Player> players;
players.push_back(Player("Ava", 100));   // constructs a temporary, then moves it in
players.emplace_back("Ava", 100);        // constructs in place
```

### Size vs Capacity

Exactly the same idea as [List Structure](<../../Python/Fundamentals/List Structure.md#size-vs-capacity>): `size()` is how many elements exist, and `capacity()` is how many fit before the vector must reallocate. When it runs out, it allocates a bigger block, **moves** the elements across, and frees the old block.

Watching the capacity while pushing 20 elements with libc++ (macOS):

```text
caps: 1 2 4 8 16 32      <- doubles each time (growth factor 2; MSVC uses 1.5)
```

That's why `push_back` is amortized O(1). The difference from Python is **what** gets copied on a resize: Python copies references, while a C++ vector moves the **elements themselves** (using their move constructors, if those are `noexcept`; see [Copy and Move Semantics](<../OOP/Copy and Move Semantics.md#7-why-noexcept-matters-on-move-operations>)).

If you know the final size, `reserve` allocates once up front:

```cpp
std::vector<int> r;
r.reserve(100);      // size 0, capacity 100: no reallocations for the first 100 push_backs
```

`reserve` changes capacity only. `resize(n)` changes the **size**, creating or destroying elements.

### Iterator, Pointer, and Reference Invalidation

Because a reallocation moves every element to new memory, **any** pointer, reference, or iterator into a vector is invalidated by an operation that may reallocate:

```cpp
std::vector<int> v = {1, 2, 3};
int& first = v[0];
int* p = &v[1];
auto it = v.begin();

v.push_back(4);      // may reallocate
// first, p, and it may all now point into freed memory: using them is UB
```

See the AddressSanitizer report for exactly this bug in [Undefined Behavior and Sanitizers](<../Memory/Undefined Behavior and Sanitizers.md#addresssanitizer-reports>).

| Operation on `std::vector` | Invalidates |
|---|---|
| `push_back`, `emplace_back`, `insert`, `reserve`, `resize` | **Everything**, if a reallocation happens; otherwise only positions at or after the insertion point |
| `erase`, `pop_back` | Positions at or after the erased element |
| Reading, or `v[i] = x` | Nothing |

The same goes for a range-based `for` loop: don't `push_back` into a vector while looping over it.

## 2. `std::unordered_map`: Hash Table (Python `dict`)

```cpp
#include <unordered_map>

std::unordered_map<std::string, int> ages = {{"Ava", 30}};

ages["Mia"] = 25;                     // insert or overwrite
ages["Ava"] += 1;                     // 31
ages.insert({"Zoe", 22});             // insert only if the key is absent
ages.emplace("Kai", 40);              // construct in place, only if absent
ages.insert_or_assign("Mia", 26);     // insert or overwrite (C++17)

ages.contains("Zoe")                  // true (C++20)   Python: "Zoe" in ages
ages.count("Bob")                     // 0              pre-C++20 membership test
ages.erase("Zoe");                    //                Python: del ages["Zoe"]
ages.at("Bob");                       // throws std::out_of_range: like Python's KeyError
```

### `operator[]` Inserts Missing Keys

This is the most important difference from a Python `dict`:

```python
ages["Leo"]        # KeyError
```

```cpp
ages["Leo"]        // returns 0, and INSERTS {"Leo", 0} into the map!
ages.size()        // grew by one
```

`[]` has to return a reference to a value it can hand back, so for a missing key it inserts a default-constructed value. That makes it behave like `collections.defaultdict`, which is handy for counting:

```cpp
std::unordered_map<char, int> counts;
for (char c : std::string("hello")) {
    counts[c]++;                      // missing keys start at 0: like Counter("hello")
}
```

For a **lookup that must not insert**, use `find`, `contains`, or `at`:

```cpp
if (auto it = ages.find("Ava"); it != ages.end()) {
    std::cout << it->first << "=" << it->second;   // Ava=31. The element is a std::pair<const Key, Value>
}
```

`[]` also can't be used on a `const` map, since it might insert.

### Iteration

```cpp
for (const auto& [name, age] : ages) {     // like: for name, age in ages.items():
    std::cout << name << ": " << age << "\n";
}
```

The iteration order is **unspecified** and can change after any insertion. This differs from Python, where dicts remember insertion order (see [Dictionary Manipulation](<../../Python/Fundamentals/Dictionary Manipulation.md#insertion-order>)). C++ has no insertion-ordered map in the standard library.

### Under the Hood

Everything in [Hash Maps Structure](<../../Python/Fundamentals/Hash Maps Structure.md>) applies, with one implementation difference. `std::unordered_map` resolves collisions with **separate chaining** (each bucket holds a linked list of nodes), whereas CPython's `dict` uses open addressing. Its default `max_load_factor()` is `1.0` (an average of one element per bucket), and it rehashes into more buckets when that's exceeded.

```cpp
ages.load_factor()       // 0.6
ages.bucket_count()      // 5
ages.max_load_factor()   // 1
```

Because every element is a separately allocated node, `std::unordered_map` is cache-unfriendly. That's why game engines often use their own hash maps (Unreal's `TMap` keeps its elements in one sparse array instead of separately allocated nodes).

### Custom Key Types Need a Hash

Like Python, where a key must be hashable (`__hash__` + `__eq__`), a C++ key type needs `operator==` and a hash function:

```cpp
struct GridPos {
    int x, y;
    bool operator==(const GridPos&) const = default;
};

struct GridPosHash {
    std::size_t operator()(const GridPos& p) const {
        return std::hash<int>{}(p.x) ^ (std::hash<int>{}(p.y) << 1);   // combine the member hashes
    }
};

std::unordered_map<GridPos, int, GridPosHash> cost;   // the hasher is the 3rd template argument
cost[{1, 2}] = 7;
```

Without a hash, the error is `call to implicitly-deleted default constructor of 'std::hash<GridPos>'`.

## 3. `std::map`: Sorted Keys

`std::map` keeps its keys **sorted**, using a balanced binary search tree (a red-black tree). Operations are O(log n) instead of O(1) on average, but you get ordered iteration and range queries:

```cpp
#include <map>

std::map<std::string, int> sorted = {{"zed", 1}, {"amy", 2}, {"kim", 3}};

for (const auto& [k, v] : sorted) std::cout << k << " ";   // amy kim zed: always in key order
sorted.begin()->first                                       // "amy": smallest key
sorted.rbegin()->first                                      // "zed": largest key
sorted.lower_bound("b")->first                              // "kim": first key >= "b"
```

`lower_bound` and `upper_bound` are the tree version of the [Bisect](<../../Python/Standard Library/Bisect.md>) module's `bisect_left` and `bisect_right`.

The key type needs `operator<` (or `<=>`) instead of a hash (see [Operator Overloading](<../OOP/Operator Overloading.md#2-comparison-operator-and-operator>)).

| | `std::unordered_map` | `std::map` |
|---|---|---|
| Lookup / insert / erase | O(1) average, O(n) worst | O(log n) guaranteed |
| Iteration order | Unspecified | Sorted by key |
| Key requirement | `std::hash<K>` + `==` | `<` |
| Use when | You just need fast lookup | You need ordering, min/max, or range queries |

## 4. Sets

```cpp
#include <set>
#include <unordered_set>

std::set<int> s = {5, 1, 3, 3};        // {1, 3, 5}: sorted, duplicates dropped
std::unordered_set<int> us = {1, 2, 3};
us.insert(2);                           // already present: no change
us.contains(2);                         // true
us.erase(2);
```

There are no `|`, `&`, `-` set operators like Python's (see [Set Manipulation](<../../Python/Fundamentals/Set Manipulation.md>)). Use `std::set_union`, `std::set_intersection`, and `std::set_difference` from `<algorithm>`, which work on **sorted** ranges, or loop over one set and test membership in the other.

## 5. `std::deque`, `std::stack`, `std::queue`

The same structures as [Stack and queue](<../../Python/Fundamentals/Stack and queue.md>):

```cpp
#include <deque>
#include <stack>
#include <queue>

std::deque<int> dq = {2, 3};
dq.push_front(1);       // like appendleft: O(1)
dq.push_back(4);        // like append: O(1)
dq.pop_front();         // like popleft: O(1)

std::stack<int> st;     // LIFO
st.push(1);
st.push(2);
st.top();               // 2: look at the top
st.pop();               // remove it (returns nothing)

std::queue<int> q;      // FIFO
q.push(1);
q.push(2);
q.front();              // 1
q.pop();
```

`std::stack` and `std::queue` are **adaptors**: they wrap another container (a `deque` by default) and expose only the stack or queue operations.

> As with `vector::pop_back`, the `pop()` functions return nothing. Read `top()` or `front()` first, then `pop()`. Python's `pop()` does both at once.

## 6. `std::priority_queue`: Heap

The binary heap from [Heap Data Structure](<../../Python/Fundamentals/Heap Data Structure.md>), with one inversion to remember: **`std::priority_queue` is a max-heap by default**, while Python's `heapq` is a min-heap.

```cpp
#include <queue>

std::priority_queue<int> max_heap;
for (int x : {3, 1, 4, 1, 5}) max_heap.push(x);   // O(log n) each
max_heap.top();                                   // 5: the LARGEST
max_heap.pop();                                   // O(log n)

// Min-heap (like heapq): pass std::greater as the comparator
std::priority_queue<int, std::vector<int>, std::greater<int>> min_heap;
for (int x : {3, 1, 4, 1, 5}) min_heap.push(x);
min_heap.top();                                   // 1: the smallest
```

For Dijkstra-style searches, push `(distance, node)` pairs into a min-heap, exactly like pushing tuples into `heapq`. `std::pair` compares lexicographically, like a Python tuple:

```cpp
using Entry = std::pair<int, std::string>;        // (distance, node)
std::priority_queue<Entry, std::vector<Entry>, std::greater<>> pq;
pq.push({5, "far"});
pq.push({1, "near"});
pq.top().second;                                  // "near"
```

## 7. `std::pair` and `std::tuple`

Fixed-size groups of values, possibly of different types, like Python's tuples (see [Tuples](<../../Python/Fundamentals/Tuples.md>)):

```cpp
#include <utility>
#include <tuple>

std::pair<std::string, int> p = {"Ava", 90};
p.first;                          // "Ava"
p.second;                         // 90
auto [name, score] = p;           // unpacking: like name, score = p

std::tuple<int, double, std::string> t{1, 2.5, "x"};
std::get<1>(t);                   // 2.5: the index must be a compile-time constant
```

For anything with meaning beyond "two values", a small `struct` with named members reads better than `.first` and `.second`.

## Complexity Summary

| Container | Access by index | Search | Insert / erase at end | Insert / erase in middle | Ordered |
|---|---|---|---|---|---|
| `vector` | O(1) | O(n) | Amortized O(1) | O(n) | Insertion order |
| `deque` | O(1) | O(n) | O(1) both ends | O(n) | Insertion order |
| `unordered_map` / `unordered_set` | n/a | O(1) average | O(1) average | O(1) average | No |
| `map` / `set` | n/a | O(log n) | O(log n) | O(log n) | Sorted |
| `priority_queue` | Top only, O(1) | n/a | Push / pop O(log n) | n/a | Heap |

> Unreal Engine code uses its own containers instead of the STL: `TArray` (≈ `std::vector`), `TMap` (≈ `std::unordered_map`), `TSet`, and `TQueue`. They integrate with Unreal's reflection, garbage collector, serialization, and memory allocators. The concepts in this note (contiguous storage, capacity growth, invalidation, hashing requirements) carry over directly.
