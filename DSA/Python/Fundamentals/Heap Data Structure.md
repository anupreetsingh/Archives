# Heap Data Structure

A **heap** is a tree-based data structure used when we need repeated access to the minimum or maximum element.

- A heap is usually represented as a **complete binary tree**, where every level is filled from left to right before a new level starts.

- Although heaps are conceptually trees, they are commonly stored in arrays/lists instead of explicit tree nodes because parent-child positions can be calculated from indexes.

- Heaps can be of 2 types:
  - In a **min-heap**, every parent is less than or equal to its children, so the smallest value is at the root.
  - In a **max-heap**, every parent is greater than or equal to its children, so the largest value is at the root.

- The root can be read in O(1), while insertion and removal take O(log n) because values may need to move up or down the tree.

## Why Not Use a Sorted Stack/List?

A sorted stack or sorted list can also keep the minimum or maximum value easy to access, but it usually pays more when new values are inserted.

| Operation | Sorted list/array | Heap |
| --- | --- | --- |
| Read min/max | O(1) | O(1) |
| Insert new value | O(n) | O(log n) |
| Remove min/max | O(1) or O(n), depending on which end is used | O(log n) |
| Keep every element fully sorted | Yes | No |

The key tradeoff is that a heap is **partially ordered**, not fully sorted. It only guarantees that each parent has the correct relationship to its children, so the min or max stays at the root. Because the whole structure does not need to stay sorted, insertion and removal only move values along one root-to-leaf path instead of shifting many elements.

Use a heap when the goal is repeatedly getting the next smallest or largest item, such as in a priority queue. Use a sorted list when the full sorted order of all elements matters.

## Calculating Index

For a 0-indexed array representation:

```text
left child   = 2 * i + 1
right child  = 2 * i + 2
parent index = (i - 1) // 2
```

These formulas work because the array stores the complete binary tree in **level order**: root first, then its children from left to right, then the next level from left to right.

```text
Tree position:        Array index:

        A                  0
      /   \              /   \
     B     C            1     2
    / \   / \          / \   / \
   D   E F   G        3   4 5   6
```

Every node before index `i` has already taken its level-order position. Since each parent owns two child slots, the children of index `i` start after all earlier parent-child pairs:

- left child = `2 * i + 1`
- right child = `2 * i + 2`

The parent formula reverses that relationship. Both children of a parent land next to each other, so subtracting `1` and using integer division by `2` maps either child back to the same parent:

- parent of left child `2 * i + 1`: `((2 * i + 1) - 1) // 2 = i`
- parent of right child `2 * i + 2`: `((2 * i + 2) - 1) // 2 = i`

Example min-heap:

```text
        1
      /   \
     3     5
    / \
   7   4
```

This can be stored as:

```python
heap = [1, 3, 5, 7, 4]
```

## heapq Module

- Python's built-in implementation of a **min-heap**.
- In a min-heap: the smallest element is always at the root (index 0).
- Very useful for problems like:
  - finding kth largest/smallest element
  - priority queues
  - streaming data (e.g., running median)
- A list managed by `heapq` can store comparable elements. When those elements are sequence types, such as tuples, lists, or strings, Python compares them lexicographically: it compares index `0` first, and if there is a tie, it compares index `1`, then index `2`, and so on.

```python
import heapq

tasks = [(2, "write"), (1, "debug"), (1, "build"), (2, "test")]
heapq.heapify(tasks)

print(heapq.heappop(tasks))  # (1, "build")
print(heapq.heappop(tasks))  # (1, "debug")
print(heapq.heappop(tasks))  # (2, "test")
print(heapq.heappop(tasks))  # (2, "write")
```

This is useful for priority queues where the first value is the main priority and later values break ties. If two elements tie up to a later field, that later field must still be comparable, or Python will raise a `TypeError`.

```python
# Importing the standard python module for heaps
import heapq

# You could directly import functions you want as well, like this;
from heapq import heapify, heappush, heappop, heappushpop, heapreplace

# But usually people just import the module and use whatever function they need from the module
```

### 1. Convert list into a heap

Syntax: `heapq.heapify(nums)`

`heapify` rearranges the list in place and returns `None`, so do not assign its result to another variable.

Python 3.14 and newer also provide `heapq.heapify_max(nums)` for building a max-heap in place. In older Python versions, use negated values(opposite sign) when you need max-heap behavior with `heapq`.

```python
import heapq

nums = [9, 5, 6, 2, 3, 8, 1, 7, 4, 10]

heapq.heapify(nums)

print("Heap after heapify:", nums)
# Heap after heapify: [1, 2, 6, 4, 3, 8, 9, 7, 5, 10]
# For a given array, heapify deterministically constructs one valid min-heap layout out of multiple possible valid min-heap layouts.
# The main guarantee is that the smallest element, 1 in this case, will be at index 0.
# Complexity: O(n), because heapify uses efficient bottom-up heap construction.


heapq.heapify_max(nums)  # Available in Python 3.14 and newer
print("Max-heap after heapify_max:", nums)
# Max-heap after heapify_max: [10, 9, 8, 7, 5, 6, 1, 2, 4, 3]
# For a given array, heapify_max deterministically constructs one valid max-heap layout out of multiple possible valid max-heap layouts.
# The main guarantee is that the largest element, 10 in this case, will be at index 0.
# Complexity: O(n), because heapify_max uses efficient bottom-up heap construction.
```

### 2. Push element into heap

Syntax: `heapq.heappush(heap, item)`

```python
import heapq

nums = [9, 5, 6, 2, 3, 8, 1, 7, 4, 10]
heapq.heapify(nums) # Develops the min_heap in place

heapq.heappush(nums, 0)  

print("Heap after pushing 0:", nums)
# Heap after pushing 0: [0, 1, 6, 4, 2, 8, 9, 7, 5, 10, 3]
# heappush adds the new value in the valid position of the heap, then repairs the list so it remains a valid min-heap.
# Internally, the new value is first added at the next open position at the end, which keeps the complete-tree shape intact.
# That new value is then compared with its parent and swapped upward until every parent is <= its children again.
# Since 0 is now the smallest value, it becomes the root at index 0.
# Complexity: O(log n), because the new value may need to bubble up(sift) through the heap height.
```

### 3. Pop smallest element

Syntax: `heapq.heappop(heap)`

```python
import heapq

nums = [9, 5, 6, 2, 3, 8, 1, 7, 4, 10]
heapq.heapify(nums)

smallest = heapq.heappop(nums)

print("Smallest value:", smallest)
print("Heap after pop:", nums)
# Smallest value: 1
# Heap after pop: [2, 3, 6, 4, 10, 8, 9, 7, 5]
# heappop removes and returns the root, then repairs the remaining list into another valid min-heap.
# Internally, the last element is moved to the root because removing the last node keeps the complete-tree shape intact.
# That last-element replacement for the root is then compared with its smaller child and swapped downward until every parent is <= its children again.
# Complexity: O(log n), because the last-element replacement may need to sift down through the heap height.
```

### 4. Push and pop in one operation

Syntax: `heapq.heappushpop(heap, item)`

```python
import heapq

nums = [9, 5, 6, 2, 3, 8, 1, 7, 4, 10]
heapq.heapify(nums)

removed = heapq.heappushpop(nums, 11)

print("Removed value:", removed)
print("Heap after push-pop with 11:", nums)
# Removed value: 1
# Heap after push-pop with 11: [2, 3, 6, 4, 10, 8, 9, 7, 5, 11]
# heappushpop pushes the new value first, then removes and returns the smallest value.
# It is more efficient than heappush followed by heappop as two separate operations.
# Complexity: O(log n), because the heap may need to be repaired after the root changes.
```

### 5. Replace element

Syntax: `heapq.heapreplace(heap, item)`

```python
import heapq

nums = [9, 5, 6, 2, 3, 8, 1, 7, 4, 10]
heapq.heapify(nums)

removed = heapq.heapreplace(nums, 11)

print("Removed value:", removed)
print("Heap after replacing with 11:", nums)
# Removed value: 1
# Heap after replacing with 11: [2, 3, 6, 4, 10, 8, 9, 7, 5, 11]
# heapreplace removes the root first, then pushes the new value while maintaining the min-heap property.
# It is equivalent to heappop followed by heappush, but it is more efficient as one combined operation.
# The returned value can be larger than the inserted item; use heappushpop when you need to keep the larger value in the heap.
# Complexity: O(log n), because the new root may need to sift down through the heap height.
```

### 6. Get n smallest / largest

Syntax: `heapq.nsmallest(n, iterable, key=None)` and `heapq.nlargest(n, iterable, key=None)`

```python
import heapq

nums = [9, 5, 6, 2, 3, 8, 1, 7, 4, 10]

print("3 smallest elements:", heapq.nsmallest(3, nums))
print("2 largest elements:", heapq.nlargest(2, nums))
# 3 smallest elements: [1, 2, 3]
# 2 largest elements: [10, 9]
# nsmallest returns the k smallest values in sorted order, and nlargest returns the k largest values in descending order.
# key is optional and lets you compare values by a derived value, such as key=len or key=str.lower.
# Complexity: O(n log k), because Python keeps a heap of size k while scanning the n input values.
```

```python
import heapq

names = ["manpreet", "amy", "christopher", "zoe"]

print("2 shortest names:", heapq.nsmallest(2, names, key=len))
print("2 longest names:", heapq.nlargest(2, names, key=len))
# 2 shortest names: ['amy', 'zoe']
# 2 longest names: ['christopher', 'manpreet']
# key=len means names are compared by length instead of alphabetical value.
```

### 7. Merge sorted iterables

Syntax: `heapq.merge(*iterables, key=None, reverse=False)`

`heapq.merge` is especially useful when merging more than two already-sorted iterables, because it keeps only the current next value from each iterable in the heap.

```python
import heapq

a = [1, 4, 7]
b = [2, 5, 8]
c = [3, 6, 9]

merged = heapq.merge(a, b, c)

print("Merged values:", list(merged))
# Merged values: [1, 2, 3, 4, 5, 6, 7, 8, 9]
# merge returns an iterator, so list(...) is used here only to display all values at once.
# Each input iterable must already be sorted from smallest to largest.
# Complexity: O(n log k), where n is the total number of values and k is the number of input iterables.
```

```python
import heapq

a = [7, 4, 1]
b = [8, 5, 2]
c = [9, 6, 3]

merged = heapq.merge(a, b, c, reverse=True)

print("Merged descending values:", list(merged))
# Merged descending values: [9, 8, 7, 6, 5, 4, 3, 2, 1]
# reverse=True assumes each input iterable is already sorted from largest to smallest.
```
