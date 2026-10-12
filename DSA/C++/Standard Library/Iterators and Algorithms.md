# Iterators and Algorithms in C++

The C++ standard library separates **containers** (which store data) from **algorithms** (which process data, such as sorting, searching, and counting). **Iterators** connect the two.

```text
 containers                  iterators                    algorithms
 vector, array, deque,  -->  begin() / end()  -->  std::sort, std::find, std::count_if,
 map, set, list, string      "a position"          std::transform, std::accumulate, ...
```

Any algorithm works with any container whose iterators support the operations it needs. So `std::find` is written once and works on a `vector`, a `list`, a `set`, a `string`, and a plain array.

Python splits things up differently: algorithms are either methods (`list.sort()`, `str.count()`) or built-ins that accept any iterable (`sorted()`, `sum()`, `min()`, `any()`). See [Iterables](<../../Python/Fundamentals/Iterables.md>).

---

## Iterators

An **iterator** is an object that represents a **position** in a container. It generalizes a pointer into an array (see [Pointers](<../Memory/Pointers.md#pointer-arithmetic>)), and supports the same core operations:

| Operation | Meaning | Pointer equivalent |
|---|---|---|
| `*it` | The element at this position | `*p` |
| `it->member` | A member of that element | `p->member` |
| `++it` | Move to the next position | `++p` |
| `it == other` / `it != other` | Same position? | `p == q` |

```cpp
std::vector<int> v = {10, 20, 30, 40, 50};

auto it = v.begin();      // position of the first element
*it;                      // 10
++it;                     // move forward
*it;                      // 20
*(v.begin() + 2);         // 30: vector iterators also support arithmetic
v.end() - v.begin();      // 5: distance = number of elements
```

### `begin()` and `end()`: Half-Open Ranges

Every container has `begin()`, the position of the first element, and `end()`, the position **one past the last** element. `end()` doesn't refer to an element and must never be dereferenced:

```text
 v:      [ 10 ][ 20 ][ 30 ][ 40 ][ 50 ]
           ^                             ^
        begin()                        end()   (one past the last)
```

The range is **half-open**, `[begin, end)`, which is the same convention as Python's `range(start, stop)` and slicing, where `stop` is excluded. It has the same benefits:

- `end - begin` is the number of elements.
- An empty container has `begin() == end()`.
- Loops have a single, uniform stopping condition: `it != end`.

```cpp
for (auto it = v.begin(); it != v.end(); ++it) {
    std::cout << *it << " ";    // 10 20 30 40 50
}

for (auto it = v.rbegin(); it != v.rend(); ++it) {   // reverse iterators: ++ moves backward
    std::cout << *it << " ";    // 50 40 30 20 10
}
```

`cbegin()` / `cend()` give **const** iterators, which can read elements but not modify them.

### "Not Found" Is `end()`

Algorithms that search return an iterator. When nothing is found, they return `end()`. This plays the role of Python's `-1` from `str.find()`, or the `ValueError` from `list.index()`:

```cpp
auto f = std::find(v.begin(), v.end(), 30);
if (f != v.end()) {
    std::cout << "found at index " << (f - v.begin());   // 2
}

std::find(v.begin(), v.end(), 99) == v.end()             // true: not found
```

### How Range-Based `for` Works

Python's `for x in obj` calls `iter(obj)`, then calls `next()` until `StopIteration` is raised. C++'s range-based `for` (see [Control Flow](<../Fundamentals/Control Flow.md#5-range-based-for-c11>)) is rewritten by the compiler into an iterator loop:

```cpp
for (int x : v) {
    body(x);
}

// is compiled as:
{
    auto it = v.begin();          // like iter(v)
    auto end = v.end();
    for (; it != end; ++it) {     // C++ compares against end instead of catching StopIteration
        int x = *it;              // like next(it)
        body(x);
    }
}
```

So any type with `begin()` and `end()` member functions works in a range-based `for`. That's the C++ version of implementing `__iter__` and `__next__`:

```cpp
class Countdown {
public:
    explicit Countdown(int start) : start_(start) {}

    struct Iterator {
        int value;
        int operator*() const { return value; }               // like __next__'s return value
        Iterator& operator++() { --value; return *this; }     // advance
        bool operator!=(const Iterator& o) const { return value != o.value; }
    };

    Iterator begin() const { return {start_}; }   // like __iter__
    Iterator end() const { return {0}; }          // the "stop" position

private:
    int start_;
};

for (int x : Countdown(3)) {
    std::cout << x;     // 321
}
```

Like a Python generator, `Countdown` produces values on the fly and stores none of them.

### Iterator Categories

Different containers support different iterator operations, and algorithms state which ones they need:

| Category | Supports | Containers |
|---|---|---|
| Forward | `++`, `*`, `==` | `std::forward_list`, `unordered_map`, `unordered_set` |
| Bidirectional | + `--` | `std::list`, `std::map`, `std::set` |
| Random access | + `it + n`, `it - other`, `it[n]`, `<` | `std::vector`, `std::deque`, `std::array`, `std::string`, raw arrays |

`std::sort` needs random access, so it can't sort a `std::list`:

```cpp
std::list<int> lst = {3, 1, 2};
std::sort(lst.begin(), lst.end());   // ERROR: invalid operands to binary expression ('__list_iterator' and '__list_iterator')
lst.sort();                          // OK: std::list provides its own member sort
```

### Iterator Invalidation

An iterator is only valid while the container's structure stays the same. Inserting into or erasing from a container can invalidate existing iterators (see [STL Containers](<STL Containers.md#iterator-pointer-and-reference-invalidation>)). To erase while looping, use the iterator that `erase` returns:

```cpp
std::vector<int> e = {1, 2, 3, 4};

for (auto it = e.begin(); it != e.end(); ) {
    if (*it % 2 == 0) {
        it = e.erase(it);    // erase returns an iterator to the next element
    } else {
        ++it;
    }
}
// e = {1, 3}

std::erase_if(e, [](int x) { return x % 2 == 0; });   // C++20: the same in one line
```

## Algorithms

`<algorithm>` and `<numeric>` contain more than 100 algorithms. Most of them take a range as two iterators, `(first, last)`, and many take a function (usually a lambda; see [Lambdas](<../Fundamentals/Lambdas.md>)) to customize them.

### Python to C++

| Python | C++ | Header |
|---|---|---|
| `lst.sort()` / `sorted(lst)` | `std::sort(v.begin(), v.end())` | `<algorithm>` |
| `lst.sort(reverse=True)` | `std::sort(v.begin(), v.end(), std::greater<>())` | `<algorithm>`, `<functional>` |
| `lst.sort(key=len)` | `std::sort(..., [](auto& a, auto& b) { return a.size() < b.size(); })` | |
| Stable sort (Python's sort is always stable) | `std::stable_sort(...)` (plain `std::sort` is **not** stable) | |
| `x in lst` | `std::find(b, e, x) != e` | |
| `next(x for x in lst if cond)` | `std::find_if(b, e, pred)` | |
| `lst.count(x)` | `std::count(b, e, x)` | |
| `sum(1 for x in lst if cond)` | `std::count_if(b, e, pred)` | |
| `min(lst)` / `max(lst)` | `*std::min_element(b, e)` / `*std::max_element(b, e)` | |
| `sum(lst)` | `std::accumulate(b, e, 0)` | `<numeric>` |
| `functools.reduce(f, lst, init)` | `std::accumulate(b, e, init, f)` | `<numeric>` |
| `any(...)` / `all(...)` | `std::any_of` / `std::all_of` / `std::none_of` | |
| `list(map(f, lst))` | `std::transform(b, e, out, f)` | |
| `[x for x in lst if cond]` | `std::copy_if(b, e, std::back_inserter(out), pred)` | `<iterator>` |
| `lst.reverse()` | `std::reverse(b, e)` | |
| `list(range(1, 6))` | `std::iota(b, e, 1)` | `<numeric>` |
| `sorted(set(lst))` | `std::sort` + `std::unique` + `erase` | |
| `bisect.bisect_left(lst, x)` | `std::lower_bound(b, e, x)` | |
| `bisect.bisect_right(lst, x)` | `std::upper_bound(b, e, x)` | |
| `x in sorted_lst` (binary search) | `std::binary_search(b, e, x)` | |

### Examples

```cpp
#include <algorithm>
#include <numeric>
#include <functional>
#include <iterator>

std::vector<int> v = {10, 20, 30, 40, 50};

*std::find_if(v.begin(), v.end(), [](int x) { return x > 25; })   // 30
*std::min_element(v.begin(), v.end())                             // 10
std::accumulate(v.begin(), v.end(), 0)                            // 150
std::any_of(v.begin(), v.end(), [](int x) { return x > 45; })     // true
std::count(v.begin(), v.end(), 20)                                // 1

// Binary search on a sorted range (see Bisect and Binary Search)
std::binary_search(v.begin(), v.end(), 40)                        // true
std::lower_bound(v.begin(), v.end(), 35) - v.begin()              // 3: first position where 35 could go
std::upper_bound(v.begin(), v.end(), 30) - v.begin()              // 3: first element > 30

// Sort descending
std::vector<int> s = {5, 2, 9, 1};
std::sort(s.begin(), s.end(), std::greater<>());                  // {9, 5, 2, 1}

// Sort just part of a range: algorithms take any [first, last)
std::vector<int> part = {1, 2, 3};
std::sort(part.begin() + 1, part.end(), std::greater<>());        // {1, 3, 2}

// Fill with 1, 2, 3, 4, 5
std::vector<int> io(5);
std::iota(io.begin(), io.end(), 1);                               // {1, 2, 3, 4, 5}

// Filter into a new vector
std::vector<int> big;
std::copy_if(v.begin(), v.end(), std::back_inserter(big),
             [](int x) { return x > 20; });                       // {30, 40, 50}
```

`std::back_inserter(big)` is an output iterator that calls `big.push_back()` for every value written to it. Without it, `copy_if` would write into `big`'s existing elements, and there aren't any.

The `lower_bound`/`upper_bound` pair matches the `bisect_left`/`bisect_right` semantics in [Bisect](<../../Python/Standard Library/Bisect.md>) and [Binary Search](<../../Python/Traversal and Search Algorithms/Binary Search.md>).

### Remove-Erase: Algorithms Can't Change a Container's Size

Algorithms see only iterators, never the container itself, so they can't add or remove elements. `std::unique` and `std::remove` therefore **move** the kept elements to the front and return the new logical end. You then `erase` the leftover tail:

```cpp
std::vector<int> d = {3, 1, 3, 2, 1, 3};
std::sort(d.begin(), d.end());                    // {1, 1, 2, 3, 3, 3}
auto new_end = std::unique(d.begin(), d.end());   // {1, 2, 3, ?, ?, ?}: adjacent duplicates moved out
d.erase(new_end, d.end());                        // {1, 2, 3}
```

For the common "remove all matching" case, C++20's `std::erase` / `std::erase_if` do both steps.

## Ranges and Views (C++20)

C++20's **ranges** library lets algorithms take a whole container instead of two iterators, and adds lazy **views**:

```cpp
#include <ranges>

std::vector<int> r = {4, 2, 3, 1};
std::ranges::sort(r);                    // no more r.begin(), r.end()
std::ranges::find(r, 3);
```

A **view** is a lazy, non-owning adaptor over a range. It doesn't compute or store anything until you iterate it, just like Python's lazy iterators (`map`, `filter`, `range`, and generator expressions; see [Lazy vs Eager Iterables](<../../Python/Fundamentals/Iterables.md#lazy-vs-eager-iterables>)). Views compose with `|`, like a Unix pipe:

```cpp
// Python: (x * x for x in v if x % 20 == 0)
auto evens_squared = v
    | std::views::filter([](int x) { return x % 20 == 0; })
    | std::views::transform([](int x) { return x * x; });

for (int x : evens_squared) std::cout << x << " ";   // 400 1600

// Python: range(5)
for (int i : std::views::iota(0, 5)) std::cout << i;   // 01234

// Python: list(reversed(range(10)))[:3]
for (int i : std::views::iota(0, 10) | std::views::reverse | std::views::take(3)) {
    std::cout << i;                                    // 987
}

// Python: zip(names, marks)   (C++23)
std::vector<std::string> names = {"Alice", "Bob"};
std::vector<int> marks = {85, 92};
for (auto [n, m] : std::views::zip(names, marks)) {
    std::cout << n << "=" << m << " ";                 // Alice=85 Bob=92
}
```

Since views don't own their data, a view must not outlive the container it looks at. That's the same rule as for `std::string_view` and references.

> Library support for the newest views varies. The libc++ that ships with Xcode on this Mac has `std::views::zip` but not yet `std::views::enumerate`. Unreal doesn't use STL ranges. `TArray` supports range-based `for`, and has its own member algorithms such as `Sort()`, `FindByPredicate()`, `RemoveAll()`, and `ContainsByPredicate()`.

## Summary

```text
Iterator         -> a position: *it to read, ++it to advance, compare with != end
[begin, end)     -> half-open, like Python's range(start, stop); end is never dereferenced
Not found        -> algorithms return end()
Range-for        -> sugar for a begin/end iterator loop; works on any type with begin()/end()
Algorithms       -> generic functions over [first, last), customized with lambdas
Size changes     -> algorithms can't: use erase / erase_if afterwards
Views (C++20)    -> lazy, composable with |, like Python generators; don't outlive the container
```
