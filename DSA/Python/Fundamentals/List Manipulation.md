# List Manipulation Notes

## Adding Elements

### `list.append()`

`list.append()` adds one element to the end of the list. It updates the original list.

```python
list.append(x)
```

```python
# append(x) -> Add one element to the end of the list
lst = [1, 2, 3]
lst.append(4)  # [1, 2, 3, 4]
```

### `list.extend()`

`list.extend()` adds all elements from an iterable to the end of the list. It updates the original list.

```python
list.extend(iterable)
```

```python
# extend(iterable) -> Add all elements from another iterable
lst = [1, 2, 3, 4]
lst.extend([5, 6])  # [1, 2, 3, 4, 5, 6]
```

### `list.insert()`

`list.insert()` inserts one element at a specific index. Existing elements from that index onward shift to the right.

```python
list.insert(index, x)
```

```python
# insert(index, x) -> Insert an element at a specific index
lst = [1, 2, 3, 4, 5, 6]
lst.insert(2, 99)  # [1, 2, 99, 3, 4, 5, 6]
```

## Removing Elements

### `list.remove()`

`list.remove()` removes the first occurrence of a value from the list.

```python
list.remove(x)
```

- Raises `ValueError` if `x` is not found.

```python
# remove(x) -> Remove the first occurrence of x
lst = [1, 2, 99, 3, 4, 5, 6]
lst.remove(99)  # [1, 2, 3, 4, 5, 6]
```

### `list.pop()`

`list.pop()` removes and returns an item from the list.

```python
list.pop([index])
```

- `index` is optional. If not given, `pop()` removes and returns the last element.
- You can call `pop()` without storing the returned value.
- `pop()` from the end is `O(1)`.
- `pop(index)` is `O(n)` because elements after `index` shift left.

```python
# pop([index]) -> Remove and return an item
lst = [1, 2, 3, 4, 5, 6]

last = lst.pop()     # removes 6, returns 6 -> [1, 2, 3, 4, 5]
lst.pop()            # removes 5 -> [1, 2, 3, 4]
second = lst.pop(1)  # removes 2, returns 2 -> [1, 3, 4] 
```

### `del`

`del` deletes item(s) by index or slice. It does not return a value.

```python
del list[index]
del list[start:end]
```

```python
# del -> Delete item(s) by index or slice
nums = [10, 20, 30, 40, 50]
del nums[1]    # removes 20 -> [10, 30, 40, 50]
del nums[1:3]  # removes 30, 40 -> [10, 50]
```

### `list.clear()`

`list.clear()` removes all elements from the list.

```python
list.clear()
```

```python
# clear() -> Remove all elements from the list
lst = [1, 2, 3]
lst.clear()  # []
```

## Sorting Lists

### `list.sort()`

`list.sort()` sorts the list in-place. It updates the original list and does not return a new list.

```python
list.sort(*, key=None, reverse=False)
```

- `key` is optional. It accepts a callable that takes one list element and returns the value used for sorting.
- `reverse` is optional. If `reverse=True`, the list is sorted in descending order.

```python
# .sort() -> Sorts the list in place
words = ['banana', 'apple', 'cherry']
words.sort()  # ['apple', 'banana', 'cherry']
words.sort(reverse=True)  # ['cherry', 'banana', 'apple']

words = ['banana', 'kiwi', 'strawberry']
words.sort(key=len)  # ['kiwi', 'banana', 'strawberry']
# Here, key=len means each word is sorted based on value of output of len() 
```

### `sorted()`

`sorted()` returns a new sorted list. It does not change the original iterable.

```python
sorted(iterable, *, key=None, reverse=False)
```

- `key` is optional. It accepts a callable that takes one element and returns the value used for sorting.
- `reverse` is optional. If `reverse=True`, the result is sorted in descending order.

```python
# sorted() -> Returns a new sorted list
nums = [3, 1, 2]
sorted_nums = sorted(nums)  # [1, 2, 3], nums is still [3, 1, 2]
sorted_desc = sorted(nums, reverse=True)  # [3, 2, 1]

words = ['banana', 'kiwi', 'strawberry']
sorted_words = sorted(words, key=len)  # ['kiwi', 'banana', 'strawberry']
```

## Counting Elements

### `list.count()`

`list.count()` returns the number of times one element appears in the list.

Time Complexity= O(n)

```python
list.count(x)
```

```python
# count(x) -> Count occurrences of x
count_twos = [1, 2, 2, 3].count(2)  # 2
```

### `Counter`

`Counter` class counts how many times each element appears in an iterable such as a list, tuple, or string. It returns a `Counter` object, which behaves mostly like a dictionary with elements as keys and counts as values.

Time Complexity: `O(n)` to build the `Counter` object.

```python
Counter(iterable)
```

```python
from collections import Counter

nums = [4, 2, 4, 6, 2, 4, 8, 6, 2]

counts = Counter(nums)

print(counts)     # Counter({4: 3, 2: 3, 6: 2, 8: 1})
print(counts[4])  # 3
print(counts[6])  # 2
```

## Other Useful List Ops

### `len()`

`len()` returns the number of elements in a list.

```python
len(list)
```

```python
# len() -> Get number of elements
nums = [3, 1, 2]
length = len(nums)  # 3
```

### `in`

`in` checks whether a value exists in a list.

```python
value in list
```

- For lists, `in` has `O(n)` time complexity because Python may need to scan the list.

```python
# in -> Check if element exists
nums = [3, 1, 2]
is_present = 2 in nums  # True
```

### `list.index()`

`list.index()` returns the index of the first occurrence of a value.

```python
list.index(x[, start[, end]])
```

- `start` is optional. Search begins at this index.
- `end` is optional. Search stops before this index.
- Raises `ValueError` if `x` is not found.
- Square brackets in syntax show optional arguments.

```python
# index(x[, start[, end]]) -> Get index of first occurrence of x
lst = [10, 20, 30, 20, 40]
idx = lst.index(20)         # 1 -> returns index of first 20
idx2 = lst.index(20, 2)     # 3 -> starts search from index 2 onward
idx3 = lst.index(20, 0, 3)  # 1 -> searches within range [0, 3)
# lst.index(99)             # raises ValueError: 99 is not in list
```

### `list.reverse()`

`list.reverse()` reverses the list in-place. It updates the original list and does not return a new list.

```python
list.reverse()
```

```python
# reverse() -> Reverse the list in-place
nums = [3, 1, 2]
nums.reverse()  # [2, 1, 3]
```

### Slicing

Slicing returns part of a list.

```python
list[start:end:step]
```

- `start` is optional. It decides where the slice begins.
- `end` is optional. It decides where the slice stops, but this index is not included.
- `step` is optional. It decides how much to move after each element.

```python
# slicing -> Access part of the list
nums = [3, 1, 2]
part = nums[1:3]  # [1, 3]
```

### `list.copy()`

A shallow copy creates a new outer container but keeps references to the same inner objects.

A deep copy creates a new outer container and recursively copies the inner objects too.

`list.copy()` creates a shallow copy of the list.

```python
list.copy()
```

- The outer list is new.
- The elements inside are still the same object references.

```python
# copy() -> Create a shallow copy of the list
nums = [3, 1, 2]
copy_nums = nums.copy()
```

### Repetition with `*`

For sequence types like lists and tuples, the `*` operator creates a new sequence by repeating the original sequence `n` times.

This is shallow-copy behavior: the outer container object, either a list or tuple, is new, but the elements inside are references to the original element objects, repeated `n` times.

```python
[1, 2] * 3   # [1, 2, 1, 2, 1, 2]
[0] * 5      # [0, 0, 0, 0, 0]
[] * 5       # [] since there are no elements in `[]` to repeat
```

**Updating** elements of the new list

```python
# For immutable objects like numbers and strings, updating the object at an index only means rebinding the index to a new object
nums = [0] * 5      # [0, 0, 0, 0, 0]
nums[0] = 99        # [99, 0, 0, 0, 0] 
words = ["hi"] * 3      # ["hi", "hi", "hi"]
words[0] = "bye"        # ["bye", "hi", "hi"]

# For mutable values like lists, updating the object actually updates the object and hence changes value at all other repeated references. 
matrix = [[]] * 5 # [[], [], [], [], []]
matrix[0].append(1) # [[1], [1], [1], [1], [1]]
# This rebinds that index to a new list object and does not update the repeated list object
matrix[0] = [99]         # [[99], [1], [1], [1], [1]] 
```

Use list comprehension when you need separate inner list elements:

```python
matrix = [[] for _ in range(5)]
matrix[0].append(1)
# [[1], [], [], [], []]
```

### List Comprehension

A list comprehension is a compact way to build a new list from an iterable.

#### Basic Syntax

```python
[expression for item in iterable]
```

The `expression` decides **what value gets added** to the new list. Every item that reaches the expression is still included, but the value added depends on the condition.

```python
nums = [1, 2, 3, 4]

[x for x in nums]
# `x` is the expression
# [1,2,3,4]

squares = [x**2 for x in nums]
# Expression: `x**2` 
# [1, 4, 9, 16]

labels = ["even" if x % 2 == 0 else "odd" for x in nums]
# Ternary Expression: "even" if x % 2 == 0 else "odd"
# ['odd', 'even', 'odd', 'even']
```

#### Filtering Syntax

A filter controls **whether an item reaches the expression and gets included at all**.

```python
[expression for item in iterable if condition]

nums = [1, 2, 3, 4, 5, 6]

evens = [x for x in nums if x % 2 == 0]
# Filter: `if x % 2 == 0`
# [2, 4, 6]
```

#### Nested List Comprehension

A nested, or stacked, list comprehension has more than one `for` loop.

```python
[expression for outer_item in outer_iterable for inner_item in inner_iterable]
```

Example:

```python

# Example 1: Flattening the nested List
matrix = [[1, 2], [3, 4], [5, 6]]

flat = [num for row in matrix for num in row]
# `num` is the expression
# `for row in matrix` is the outer loop
# `for num in row` is the inner loop
# [1, 2, 3, 4, 5, 6]

# The `for` clauses are written in the same order as normal nested loops.

# Equivalent to:
flat = []
for row in matrix:
    for num in row:
        flat.append(num)

# Example 2: Doubling the nested list
doubled = [[num * 2 for num in row] for row in matrix]
# `[num * 2 for num in row]` is the outer expression
# `num * 2` is the inner expression
# [[2, 4], [6, 8], [10, 12]]

# Equivalent to:
doubled = []
for row in matrix:
    new_row = []
    for num in row:
        new_row.append(num * 2)
    doubled.append(new_row)
```

#### Combined Example

We can combine nested loops, filters, and a ternary expression in one list comprehension.

```python
matrix = [[1, 2], [3, 4, 5], [6]]

result = [
    num * 10 if num % 2 == 0 else num
    for row in matrix
    if len(row) > 1
    for num in row
    if num > 2
]
# Ternary Expression: `num * 10 if num % 2 == 0 else num`
# Outer Filter: `if len(row) > 1`
# Inner Filter: `if num > 2`
# [3, 40, 5]

# Equivalent to:
result = []
for row in matrix:
    if len(row) > 1:
        for num in row:
            if num > 2:
                result.append(num * 10 if num % 2 == 0 else num)
```
