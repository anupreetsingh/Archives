# heapq Module Notes

- Python's built-in implementation of a **min-heap**.
- A heap is a binary tree stored as a list.
- In a min-heap: the smallest element is always at the root (index 0).
- Very useful for problems like:
    - finding kth largest/smallest element
    - priority queues
    - streaming data (e.g., running median)

```python
# Importing the standard python module for heaps
import heapq
```

---

## 1. Convert list into a heap

```python
nums = [3, 2, 5, 1, 4]
heapq.heapify(nums)   # rearranges nums in-place into a valid min-heap
# Complexity: O(n)  (efficient bottom-up heapify)
print("Heap after heapify:", nums)
# Smallest element (1) is now at index 0.
```

## 2. Push element into heap

```python
heapq.heappush(nums, 0)  # adds new element while maintaining heap property
# Complexity: O(log n) (because it may bubble up the new element)
print("Heap after pushing 0:", nums)
```

## 3. Pop smallest element

```python
smallest = heapq.heappop(nums)  # removes and returns the smallest element
# Complexity: O(log n) (because it may bubble down the new root)
print("Popped smallest:", smallest)
print("Heap after pop:", nums)
```

## 4. Peek at smallest element

```python
peek = nums[0]  # root element, no removal
print("Peek smallest:", peek)
```

## 5. Get n smallest / largest

```python
print("3 smallest elements:", heapq.nsmallest(3, nums))
print("2 largest elements:", heapq.nlargest(2, nums))
```
