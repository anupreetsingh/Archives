# Comparison Sort

## Bubble Sort

It is called bubble sort because each pass makes the extreme value in the unsorted prefix subarray "bubble up" toward the end through adjacent swaps.

In this version, we bubble up the maximum value to the end of the unsorted section.

- Time Complexity: O(n^2), because the algorithm repeatedly scans the unsorted section with adjacent comparisons.
- Space Complexity: O(1), because sorting happens in place.

```python
def bubble_sort(nums):
    n = len(nums)

    for i in range(n):
        for j in range(n - 1 - i):
            if nums[j] > nums[j + 1]:  # Successive swaps bubble up the max element
                nums[j], nums[j + 1] = nums[j + 1], nums[j]

    return nums
```

## Selection Sort

Selection sort repeatedly scans the unsorted section, selects one extreme value from it, and moves that value into its final sorted position. That extreme value can be either the minimum or maximum, depending on the implementation.

In this version, we select the maximum value from the unsorted prefix and move it to the end of that prefix. After the swap, that position becomes part of the sorted section.

- Time Complexity: O(n^2), because each pass scans the remaining unsorted section.
- Space Complexity: O(1), because sorting happens in place.

```python
def selection_sort(nums):
    n = len(nums)

    for i in range(n):
        max_idx = 0
        end = n - 1 - i

        for j in range(end + 1):
            if nums[j] > nums[max_idx]:
                max_idx = j  # Update index of the max element in the unsorted section

        nums[max_idx], nums[end] = nums[end], nums[max_idx]  # Move max to end of unsorted section

    return nums
```

## Insertion Sort

Insertion sort repeatedly takes the next unsorted value and inserts it into its correct position inside the sorted prefix behind it.

In this version, each pass selects a key value, shifts larger values in the sorted prefix one position to the right, and places the key in the gap that remains.

- Time Complexity: O(n^2), because each key may need to scan and shift across the sorted prefix.
- Space Complexity: O(1), because sorting happens in place.

```python
def insertion_sort(nums):
    n = len(nums)

    for i in range(1, n):
        key = nums[i]
        j = i - 1
        while j >= 0 and key < nums[j]:
            nums[j + 1] = nums[j] # Shift Larger element forward
        nums[j + 1] = key

    return nums
```

## Merge Sort

Merge sort repeatedly splits the array into smaller halves until each subarray has at most one element, then merges sorted subarrays back together into larger sorted arrays.

It is called merge sort because the important work happens during merging: two already-sorted arrays are combined by interleaving their elements in sorted order, instead of simply placing one array after the other.

- Time Complexity: O(n log n), because there are log n levels for splitting(going down the tree) and then merging(coming up) and n elements at each level
- Space Complexity: O(n), because merging creates new arrays to store the sorted result.

```python
def merge_sort(nums):
    n = len(nums)
    # Recursion Base case
    if n <= 1:
        return nums

    # Recursive calls to sort left and right halves
    left_arr = merge_sort(nums[: n // 2]) # Up to, but excluding upper middle
    right_arr = merge_sort(nums[n // 2 :]) # From upper middle to end
    # Merging left and right halves
    return merge(left_arr, right_arr)

# Helper function that performs merging of two sorted subarrays
def merge(left_arr, right_arr):
    l, r = 0, 0
    res = []
    i = 0
    while l < len(left_arr) and r < len(right_arr):
        if left_arr[l] <= right_arr[r]:
            res.append(left_arr[l])
            l += 1
        else:
            res.append(right_arr[r])
            r += 1
        i += 1

    res.extend(left_arr[l:])
    res.extend(right_arr[r:])
    return res
```

## Quick Sort

Recursively splitting arrays around a pivot element selected from the array until we get single element subarrays and then join(solder) them together to get the final sorted array.

It is called Quick sort because it is usually quick in practice and quick to write using list comprehension.

Quick sort can use either 2-way partitioning or 3-way partitioning around the pivot. 3-way partitioning is better when the array has many duplicates because all values equal to the pivot are grouped in the middle and do not need to be recursively sorted again.

### List Comprehension Implementation

For the list-comprehension implementation:

- Average Time Complexity: O(n log n), because balanced pivots create log n levels and each level partitions n values.
- Worst Time Complexity: O(n^2), because repeatedly choosing the smallest or largest value as the pivot creates n levels.
- Average Space Complexity: O(n), because the extra lists across a balanced recursion path hold a shrinking fraction of the array at each level.
- Worst Space Complexity: O(n^2), because highly unbalanced pivots can keep many large copied subarrays alive across the recursion path.

```python
def quick_sort(nums):
    if len(nums) <= 1:
        return nums

    pivot = nums[-1]  # Selecting the last element as pivot

    # Three-way partition around the pivot
    lesser_sub = [x for x in nums if x < pivot]
    equal_sub = [x for x in nums if x == pivot]
    greater_sub = [x for x in nums if x > pivot]

    # Solder sorted partitions together; no recursive call is needed for equal_sub
    return quick_sort(lesser_sub) + equal_sub + quick_sort(greater_sub)
```

### In-place 3-way partitioning Implementation

- Average Time Complexity: O(n log n), because balanced pivots create log n recursive levels and each level partitions n values in place.
- Worst Time Complexity: O(n^2), because repeatedly choosing the smallest or largest value as the pivot creates n recursive levels.
- Average Space Complexity: O(log n), because balanced recursion uses log n stack frames.
- Worst Space Complexity: O(n), because highly unbalanced pivots can create n stack frames.

```python
def quick_sort(nums):
    if len(nums) <= 1:
        return nums

    # Helper function to recursively 3-way partition around a pivot in place
    def split(l, r):
        # Recursion base case: invalid and single-element subarrays return immediately
        if l >= r:
            return

        LT, GT = l, r
        E = l
        pivot = nums[r]

        while E <= GT:
            if nums[E] < pivot:
                nums[E], nums[LT] = nums[LT], nums[E]
                E += 1
                LT += 1
            elif nums[E] == pivot:
                E += 1
            else:
                nums[E], nums[GT] = nums[GT], nums[E]
                GT -= 1

        split(l, LT - 1)
        split(GT + 1, r)

    split(0, len(nums) - 1)
    return nums
```

## Heap Sort

Heap sort repeatedly treats the array as a heap: first it builds a heap from the values, then it repeatedly removes the heap root into the next final sorted position.

It is called heap sort because it uses the heap data structure to sort the list. The important work happens through the heap property: each parent is the extreme value of its local subtree compared to its children.

For a max heap, the main heap property is that each parent is greater than or equal to its children, so the largest value is always at the root. Heap sort repeatedly moves that root value to the end of the list, shrinks the active heap, and restores the heap property.

### `heapq` module Implementation

- Time Complexity: O(n log n), because heapifying takes O(n), then each of the n pops takes O(log n).
- Space Complexity: O(n), because this version builds a result array while popping values out of the heap.

```python
import heapq

def heap_sort_heapq(nums):
    heapq.heapify(nums)  # Turns nums into a min heap in place

    res = []
    while nums:
        res.append(heapq.heappop(nums))  # heappop returns the smallest value and shrinks nums

    return res
```

### In-place max_heap Implementation

- Time Complexity: O(n log n), because building the heap takes O(n), then each of the n extractions may sift a value down log n levels.
- Space Complexity: O(1), because sorting happens in place.

A binary heap is stored as a complete binary tree inside a 0-indexed array. For a node at index `i`:

- Left child is at index `2 * i + 1`
- Right child is at index `2 * i + 2`
- Parent is at index `(i - 1) // 2`

Leaf nodes do not need to be heapified because the heap property only relates a node to its children. Since leaves have no children, each leaf already satisfies the heap property for its own subtree. If a small or large value is sitting in a leaf, it will be handled when one of its ancestors is heapified.

For `n` nodes in a 0-indexed array heap:

- Leaf nodes range from `n // 2` to `n - 1` (from upper middle to end)
- Non-leaf nodes range from `0` to `n // 2 - 1` (from start to element before upper middle)
- The last non-leaf node is at index `n // 2 - 1`

The two phases of constructing the max heap and sorting the array both rely on the same `sift_down` assumption: the child subtrees below the current index are already heaps.

That is why, during heap construction, `sift_down(n, i)` is called on non-leaf nodes from the bottom up. This ensures that the child subtrees have already been heapified and therefore follow the heap property. In this case, we call `sift_down(n, i)` with `i` going from `n // 2 - 1` down to `0`. The total work is still `O(n)` because most nodes near the lower levels swap down only `0` or `1` level. Only a small number of upper-level nodes can swap down up to `log n` levels, which is the depth of the tree.

During the sorting phase, `sift_down(end, 0)` is called on the root of the active heap `n` times. After the root is swapped with the end of the active heap, the reduced heap size is passed into `sift_down`, so `sift_down` acts only on the smaller heap currently under consideration. Since the heap gets smaller after each extraction, the cost of the calls is more precisely like `log n + log(n - 1) + log(n - 2) + ... + log 1`, which is `log(n!)`. Because `n! <= n^n`, we get `log(n!) <= log(n^n) = n log n`. Therefore, even though each successive heap is smaller, the total sorting phase is still bounded by `O(n log n)`.

```python
def heap_sort(nums):
    n = len(nums)
    def sift_down(end_idx, i):
        while True:
            largest = i
            left_child = 2 * i + 1
            right_child = 2 * i + 2

            # Check both children within bounds to determine the which is greater
            if left_child <= end_idx and nums[left_child] > nums[largest]:
                largest = left_child

            if right_child <= end_idx and nums[right_child] > nums[largest]:
                largest = right_child

            # Parent is the largest, subtree satifies heap property
            if largest == i:
                return 
            
            # Swap the parent with largest child
            nums[largest], nums[i] = nums[i], nums[largest]
            # Updating parent's current index to check if we sift in next iteration
            i = largest 


    # Heapify all non-leaf nodes within the main array
    for i in range(n // 2 - 1, -1, -1):
        sift_down(n - 1, i)

    # Repeatedly move the max value to the end and then call sift_down on first idx and a smaller active heap excluding the max_value
    for last in range(n - 1, 0, -1):
        nums[0], nums[last] = nums[last], nums[0]
        sift_down(last - 1, 0)
    
    return nums
```
