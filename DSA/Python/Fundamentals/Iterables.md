# Iterable

```python
numbers = [10, 20, 30, 40, 50]
print("For Loop Traversal:")
for num in numbers:
    # Iterates over each element in the list
    print(num)

print("While Loop Traversal:")
i = 0
while i < len(numbers):
    print(numbers[i])
    i += 1  # Don't forget to increment!
```

---

## Iterable vs Iterator

An **iterable** is an object that can be looped over, such as a list, string, or tuple. Calling `iter()` on it creates an **iterator**.

An **iterator** produces one value at a time with `next()` and remembers its current position. Every iterator is iterable, but not every iterable is an
iterator.

```python
numbers = [10, 20, 30]  # Iterable, but not an iterator
# next(numbers)         # TypeError

number_iterator = iter(numbers)
print(next(number_iterator))  # 10
print(next(number_iterator))  # 20

for number in number_iterator:
    print(number)              # 30; iteration continues from its current position
```

Objects returned by `map()`, `filter()`, `zip()`, and generator expressions are iterators. Once consumed, they do not restart automatically.

## Lazy vs Eager Iterables

**Lazy iterable**  → produces values only when needed

**Eager iterable** → already stores or computes its values immediately. Example: Strings, lists, tuples, dictionary, set.

### range()

`range()` is a lazy iterable and represents an integer sequence. Usually used for loop counts or indexes.

Syntax:

```python
range(stop)              # default: start = 0, step = 1
range(start, stop)       # default step = 1
range(start, stop, step)
```

Rules:

- `start` is included and `stop` is excluded.
- `step` cannot be `0`.
- If `step` is positive, `start < stop` to produce values.
- If `step` is negative, `start > stop` to produce values.

```python
range(5)
# represents: 0, 1, 2, 3, 4

range(2, 6)
# represents: 2, 3, 4, 5

range(2, 10, 2)
# represents: 2, 4, 6, 8

range(5, 0, -1)
# represents: 5, 4, 3, 2, 1

range(5, 0)
# represents: no values because default step = 1, but start > stop

range(0, 5, -1)
# represents: no values because step is negative, but start < stop
```

**Internal Behavior**

`range()` creates a `range` object. Instead of storing every integer in memory, it stores only the `start`, `stop`, and `step` values. Python then calculates each number when it is accessed or iterated over.

Because a `range` object stores metadata about the sequence, Python can use arithmetic to check membership in **O(1)** time for integers.

```python
for i in range(1_000_000):
    pass

print(3478 in range(1, 10000, 6))   # False, O(1)
```

Here, `range(1_000_000)` represents one million integers from `0` up to `999_999`, but Python does not store those integers in a list. Each value is produced only when the range is iterated over.

To create and store an actual list of the elements represented by a range, use `list()`:

```python
nums = range(5)

print(nums)
# range(0, 5)

print(list(nums))
# [0, 1, 2, 3, 4]

print(list(range(5, 0)))
# []
```

### Generator Expression

A generator is a lazy iterator that produces values one at a time. Unlike `range`, a generator is consumed after one full iteration.

Generator expressions use parentheses:

```python
nums = (x * 2 for x in range(5))

print(nums)
# <generator object <genexpr> at ...>

print(list(nums))
# [0, 2, 4, 6, 8]
# The generator expression is converted to a list

print(list(nums))
# []
# This second `list(nums)` is empty because the generator has already been consumed.

nums = tuple(x * 2 for x in range(5))
print(nums)
# (0, 2, 4, 6, 8)
# Builds a tuple using a generator expression. Since there is no inherent tuple comprehension in python like there is for lists and dictionary.

print([x * 2 for x in range(5)])
# [0, 2, 4, 6, 8] 
# Uses a list comprehension to build a list. 
# The written structure for the command and the final result are the same for lists created using both generator expression and the list comprehension but form by different mechanisms.
```

### `map()`

`map(function, iterable, *additional_iterables)` lazily applies a function and returns a `map` iterator. With multiple iterables, the function receives one item from each, and `map()` stops at the shortest iterable.

```python
strings = ["10", "20", "30"]

numbers = map(int, strings)  # Conversion happens as the iterator is consumed.
print(list(numbers))  # [10, 20, 30]

# A generator expression can perform the same lazy transformation.
numbers = (int(value) for value in strings)
print(list(numbers))  # [10, 20, 30]
```

### `filter()`

`filter(function, iterable)` lazily keeps items for which the function returns a truthy value and returns a `filter` iterator. `map()` transforms items; `filter()` selects them.

```python
evens = list(filter(lambda number: number % 2 == 0, [1, 2, 3, 4]))
print(evens)  # [2, 4]

# A generator expression can perform the same lazy filtering.
evens = (number for number in [1, 2, 3, 4] if number % 2 == 0)
print(list(evens))  # [2, 4]

# With None, filter() removes falsy items.
values = [0, 1, "", "Python", None, False]
print(list(filter(None, values)))  # [1, "Python"]
```

## Sequence and Non-Sequence Types

Iterables can also be classified as **sequence** or **non-sequence** types.

A **sequence** is an ordered iterable whose elements can be accessed by index. Example: list, tuple, str and range.

Examples:

```python
items = ["a", "b", "c"]

print(items[0])   # a
print(items[1])   # b
```

A **non-sequence** iterable does not support indexed access. Some non-sequence types may preserve insertion order, but they are still not sequences because you cannot access elements by position using an index.

Examples:

```python
data = {"name": "Ana", "age": 24}
items = {"a", "b", "c"}

print(data[0])    # KeyError
print(items[0])   # TypeError
```

Common non-sequence iterable types include:

```python
dict
set
generator
map
filter
```

## Iterating Methods

### Wrap Around Logic

To traverse a list starting from any index and wrap around in circular way:

```python
n = len(numbers)
start_index = 3  # Let's start from the 4th element (index 3)

print("Circular traversal starting at index 3:")
for i in range(n):
    # (start_index + i) % n wraps around to the beginning when index exceeds list length
    idx = (start_index + i) % n
    print(numbers[idx])

# Output:
# 40
# 50
# 10
# 20
# 30
```

### Nested Loops and Flow Control in Python

The inner loop runs completely for every iteration of the outer loop.

**Control Flow:**

1. Outer loop condition is checked.
2. If true, inner loop starts.
3. Inner loop runs all its iterations.
4. Then control goes back to the top of the outer loop.
5. Outer loop condition is checked again.

**Note:** Even if the inner loop changes a variable used in the outer loop's condition, the outer loop won't stop immediately. The outer loop condition is only checked when its next iteration begins.

```python
# Example: Demonstrating this behavior
i = 0
while i < 2:  # Outer loop
    print(f"Outer loop start: i = {i}")
    j = 0
    while j < 5:  # Inner loop
        print(f"  Inner loop: j = {j}")
        i = 5 # Even though this violates outer loop condition, outer loop won't terminate until control goes to it
        j += 1
    i += 1  # Even though i is already 5, this line still runs before next check
    print(f"Outer loop end: i = {i}")

# Output:
# Outer loop start: i = 0
#   Inner loop: j = 0
#   Inner loop: j = 1
#   Inner loop: j = 2
#   Inner loop: j = 3
#   Inner loop: j = 4
# Outer loop end: i = 6
```

### Traversing Multiple Iterables with zip()

`zip()` allows parallel iteration over multiple iterables. It pairs elements from each iterable into tuples until the shortest iterable is exhausted.

```python
names = ["Alice", "Bob", "Charlie"]
scores = [85, 92, 78]

print("Traversing multiple iterables with zip:")
for name, score in zip(names, scores):
    print(f"{name}: {score}")

# Output:
# Alice: 85
# Bob: 92
# Charlie: 78
```

### Enumerating an Iterable with enumerate()

`enumerate()` returns both the index and the element while iterating.

Syntax: `enumerate(iterable, start=0)` — `start` is optional; defaults to 0 if not provided.

```python
print("Enumerating a list (default start=0):")
for index, value in enumerate(numbers):  # start defaults to 0
    print(f"Index {index}: {value}")

# Output:
# Index 0: 10
# Index 1: 20
# Index 2: 30
# Index 3: 40
# Index 4: 50

print("Enumerating a list (custom start=100):")
for index, value in enumerate(numbers, start=100):  # start is set to 100
    print(f"Index {index}: {value}")

# Output:
# Index 100: 10
# Index 101: 20
# Index 102: 30
# Index 103: 40
# Index 104: 50
```
