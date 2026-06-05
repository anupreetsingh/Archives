# Binary Search

When you see an ordered list, think Binary Search. At each iteration, binary search halves the search range by comparing the target with the middle element.

**Important distinction:**

- When searching for the **index of a particular element**, we use the condition `while low <= high`. Why? Because we want to keep checking even when `low == high` — the last single element might still be the target.

- When checking whether a certain **condition** holds in the array (e.g., "find the minimum element", "check feasibility", etc.), we often use `while low < high`. Here we're not looking for exact equality but for the boundary where the condition flips, so once `low == high` we've already converged on the answer.

This difference is subtle but crucial in practice.

## Example 1: Search for a target element (`<=` case)

```python
def binary_search(arr, target):
    low, high = 0, len(arr)-1  # first and last index in array
    while low <= high:         # <= because we are checking for an exact element
        mid = (low + high) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            low = mid + 1
        else:
            high = mid - 1
    return -1  # In case the target is not in the list
```

## Example 2: Find the minimum element in a rotated sorted array (`<` case)

Find the minimum element in a rotated (left rotation/right rotation) sorted array.

```python
def find_min_rotated(arr):
    low, high = 0, len(arr)-1
    while low < high:          # < because we are converging to a boundary
        mid = (low + high) // 2
        if arr[mid] > arr[high]:
            # Minimum must be in the right half
            low = mid + 1
        else:
            # Minimum is at mid or in the left half
            high = mid
    return arr[low]  # low == high → converged to the minimum element
```
