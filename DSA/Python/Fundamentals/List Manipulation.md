# 📌 Python List Operations

## Adding Elements

```python
# 1. append(x) → Add an element to the end of the list
lst = [1, 2, 3]
lst.append(4)  # [1, 2, 3, 4]

# 2. extend(iterable) → Add all elements from another iterable (e.g. list)
lst.extend([5, 6])  # [1, 2, 3, 4, 5, 6]

# 3. insert(index, x) → Insert an element at a specific index (shifts others)
lst.insert(2, 99)  # [1, 2, 99, 3, 4, 5, 6]
```

## Removing Elements

```python
# 4. remove(x) → Remove the **first** occurrence of x (error if not found)
lst.remove(99)  # [1, 2, 3, 4, 5, 6]

# 5. pop([index]) → Remove and return item at index (last if index not given)
last = lst.pop()    # removes 6, returns 6 → [1, 2, 3, 4, 5]
lst.pop() # Also valid, doesn't necessarily require you to store the returned value
second = lst.pop(1) # removes 2, returns 2 → [1, 3, 4, 5]

# 6. del → Delete item(s) by index or slice (no return)
nums = [10, 20, 30, 40, 50]
del nums[1]      # removes 20 → [10, 30, 40, 50]
del nums[1:3]    # Allows slicing unlike pop(), removes 30, 40 → [10, 50]

# 7. clear() → Remove all elements from the list
lst.clear()  # []
```

## Sorting Lists

```python
# 8. sort() → Sorts the list In-Place through a series of operations
words = ['banana', 'apple', 'cherry']
words.sort()  # ['apple', 'banana', 'cherry']
words.sort(reverse=True)  # ['cherry', 'banana', 'apple']
words = ['banana', 'kiwi', 'strawberry']
words.sort(key=len) # ['kiwi', 'banana', 'strawberry'] # key becomes the key characteristic on the basis of which each element is sorted. In this case it is the function len

# 9. sorted() → Gives a sorted version of the argument list
nums = [3, 1, 2]
sorted_nums = sorted(nums)  # [1, 2, 3], nums is still [3, 1, 2]
sorted_desc = sorted(nums, reverse=True)  # [3, 2, 1]
```

## Other Useful List Ops

```python
# 10. len() → Get number of elements
length = len(nums)  # 3

# 11. in → Check if element exists, "in" operator for list has a O(n) time complexity
is_present = 2 in nums  # True

# 12. count(x) → Count occurrences of x
count_twos = [1, 2, 2, 3].count(2)  # 2

# 13. index(x[, start[, end]]) → Get index of **first** occurrence of x (raises error if not found), [] brackets in syntax show that those arguments are optional
lst = [10, 20, 30, 20, 40]
idx = lst.index(20)            # 1 → returns index of first 20
idx2 = lst.index(20, 2)        # 3 → starts search from index 2 onward
idx3 = lst.index(20, 0, 3)     # 1 → searches within range [0, 3)
# lst.index(99)                # ❌ raises ValueError: 99 is not in list

# 14. reverse() → Reverse the list in-place
nums.reverse()  # [2, 1, 3]

# 15. slicing → Access part of the list
part = nums[1:3]  # [1, 3]

# 16. copy() → Create a shallow copy of the list
copy_nums = nums.copy()
```

## 17. List Comprehension (Quick way to build a list)

Full Syntax: `[ expression_if_true if condition else expression_if_false for item in iterable if filter_condition ]`

```python
result = [x**2 if x > 5 else x for x in range(10) if x % 2 == 0]
# [0, 2, 4, 36, 64]
```

We use either none, conditional part, filtering part, or both depending on our needs.

**Basic Syntax (no conditions)** — `[expression for item in iterable]`

```python
squares = [x**2 for x in range(5)]
# [0, 1, 4, 9, 16]
```

**Conditional Expression Syntax** (if BEFORE → controls the value). All items in the iterable are included, just their form vary depending on the expression applied to them based on the condition — `[expression_if_true if condition else expression_if_false for item in iterable]`

```python
pos_or_neg = [x if x % 2 == 0 else -x for x in range(6)]
# [0, -1, 2, -3, 4, -5]
```

**Filtering Syntax** (if AFTER expression → controls inclusion). Items are filtered out from the iterable depending on the condition — `[expression for item in iterable if condition]`

```python
evens = [x for x in range(10) if x % 2 == 0]
# [0, 2, 4, 6, 8]
```
