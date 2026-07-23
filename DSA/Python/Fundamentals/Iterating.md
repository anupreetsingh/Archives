# Iterating

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

## range()

`range()` represents an integer sequence, usually for loop counts or indexes.

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

### Internal Behavior

`range()` creates a `range` object immediately.

That object is itself an iterable sequence. It can be looped over, indexed, measured with `len()`, and checked with `in`.

```python
nums = range(5)

len(nums)
# 5

nums[0]
# 0

3 in nums
# True
```

`range()` does not store every integer in memory. It stores the `start`, `stop`, and `step`, then calculates values when they are accessed or iterated over.

Example:

```python
for i in range(1_000_000):
    pass
```

Here, `range(1_000_000)` represents one million integers from `0` up to `999_999`, but Python does not store those integers in a list somewhere. Each value is produced when the range is iterated over.

### Common Patterns

Use `list()` when you want to materialize the values represented by a range.

```python
nums = range(5)

print(nums)
# range(0, 5)

print(list(nums))
# [0, 1, 2, 3, 4]

print(list(range(5, 0)))
# []
```

Use a `for` loop when you want to consume the values one at a time.

```python
for i in range(3):
    print(i)

# Output:
# 0
# 1
# 2
```

---

## Wrap Around Logic

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

---

## Nested Loops and Flow Control in Python

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

---

## Traversing Multiple Iterables with zip()

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

---

## Enumerating an Iterable with enumerate()

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
