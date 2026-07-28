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
parent index = (i - 1) // 2
left child   = 2 * i + 1
right child  = 2 * i + 2
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

```python
# Importing the standard python module for heaps
import heapq
```

### 1. Convert list into a heap

```python
nums = [3, 2, 5, 1, 4]
heapq.heapify(nums)   # rearranges nums in-place into a valid min-heap
# Complexity: O(n)  (efficient bottom-up heapify)
print("Heap after heapify:", nums)
# Smallest element (1) is now at index 0.
```

### 2. Push element into heap

```python
heapq.heappush(nums, 0)  # adds new element while maintaining heap property
# Complexity: O(log n) (because it may bubble up the new element)
print("Heap after pushing 0:", nums)
```

### 3. Pop smallest element

```python
smallest = heapq.heappop(nums)  # removes and returns the smallest element
# Complexity: O(log n) (because it may bubble down the new root)
print("Popped smallest:", smallest)
print("Heap after pop:", nums)
```

### 4. Peek at smallest element

```python
peek = nums[0]  # root element, no removal
print("Peek smallest:", peek)
```

### 5. Get n smallest / largest

```python
print("3 smallest elements:", heapq.nsmallest(3, nums))
print("2 largest elements:", heapq.nlargest(2, nums))
```
