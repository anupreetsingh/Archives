# Comparison Sort

## Bubble Sort

Each iteration forms a bubble from start to one lesser element at the end, and successive swaps push the largest element in the bubble to the end, placing it in its correct position.

```python
def bubble_sort(arr):
    n = len(arr)
    for i in range(n):
        for j in range(0, n-i-1):
            #if the current element in the bubble is greater than next element swap them
            if arr[j] > arr[j+1]:
                arr[j], arr[j+1] = arr[j+1], arr[j]#Simultaneous assignment using tuple unpacking(RHS forms a tuple and then the values are assigned to LHS)
    return arr
```

## Selection Sort

Repeatedly iterates through the list exploring sections smaller by one element from the beginning to the end and selecting the smallest value of that section to put it in front.

Think of this as making a bubble but shortening the size of the bubble from the left instead of the right like we do in bubble sort, that is why we wanna select the smallest element instead of the largest one like we do in bubble sort.

```python
def selection_sort(arr):
    n = len(arr)
    for i in range(n):
        min_idx = i
        for j in range(i+1, n):
            if arr[j] < arr[min_idx]:
                min_idx = j
        arr[i], arr[min_idx] = arr[min_idx], arr[i] #In place swap, so no additional memory used
    return arr
```

## Insertion Sort

In each iteration we select a "key" element, then for that iteration we insert that element in the correct position inside the sublist behind that element.

```python
def insertion_sort(arr):
    n = len(arr)
    for i in range(1, n):
        key = arr[i]
        j = i - 1
        #Loop breaks if it reaches end of the list while decrementing indexes(moving right to left in the sublist) OR the current element is no longer greater than the key which means we found the spot to insert the "key" element
        while j >= 0 and arr[j] > key:
            arr[j + 1] = arr[j]  # Shift element to the right
            j -= 1 #decrementing counter for traversing leftward sublist
        arr[j + 1] = key  # Inserting key at its correct position
    return arr
```

## Merge Sort

An algorithm that splits the list at the middle into smaller sublists, then makes recursive calls on the left and right halves, then uses a helper function to merge the sorted left and right halves.

```python
def merge_sort(arr):
    # Recursion Base Case:
    if len(arr) <= 1:
        return arr

    #In this approach, In case of odd split, we make it right heavy. For instance in case of 5 elements, left sublist will have 2 and right will have 3
    mid = len(arr) // 2 # floor division gives you the integer part of the quotient
    left = merge_sort(arr[:mid])# right heavy, so middle excluded here
    right = merge_sort(arr[mid:])# rigth heavy, so middle included here

    return merge(left, right)

# Helper function for carrying out the merge of 2 sorted lists
def merge(left, right): #Remember both left and right lists are sorted in themselves
    result = []
    i = j = 0

    # Compare and merge
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1

    # Append remaining elements
    result.extend(left[i:])
    result.extend(right[j:])
    return result

```

## Quick Sort

An algorithm that selects a pivot and makes recursive calls at each step forming 2 new sub-lists to store lesser and greater elements than the pivot using list comprehension.

```python
def quick_sort(arr):
    if len(arr) <= 1:
        return arr

    # Choose pivot (typically the last element)
    pivot = arr[-1]
    left = [x for x in arr[:-1] if x < pivot]
    middle=[pivot]+[x for x in arr[:-1] if x==pivot] #using Middle instead of just pivot ensures that if the pivot element appears multiple times in the array we don't have call quick_sort on that element multiple times, so it is a little more efficient
    right = [x for x in arr[:-1] if x > pivot]

    # Recursively sort left and right partitions
    return quick_sort(left) + middle + quick_sort(right)

# Example
arr = [8, 4, 7, 3, 1, 5, 2, 6]
print("Sorted array:", quick_sort(arr))
```

## Heap Sort

Heap sort works in 2 steps:

- **Step 1:** by first rearranging the array and making a Max Heap by calling heapify on all non leaf nodes starting from the last non leaf node and moving towards the root,
- **Step 2:** Then it repetitively takes the root of the max heap swaps it with the last element, making the last element the largest in the array, and calls heapify on the root of the sub-array excluding the most recently swapped element.

### Heap as an Array

A binary Heap stored as complete binary trees using 0 indexed array, the indexing rule for accessing each node is:

For a node at Index `i`:

- Left Child is at index `2i+1`, Since nodes double at each level in a binary tree, and children of a node appear consecutively in the array
- Right Child is at index `2i+2`
- Parent is at index `(i-1)//2`

**Leaf And Non-Leaf Nodes:**
if there are n nodes, the indexes go upto `n-1`, so for `2i+1>=n`, node `i` has no children because the child will be out of bounds. Solving for `i`:

- `i>(n-1)//2` will all be nodes that don't have children because their children's index will be out of bounds
- Leaf Nodes range from `n//2` to `n-1`

### The main function that sorts the array

```python
def heap_sort(arr):
    n = len(arr)

    # Step 1: Build a max heap (rearrange array)
    for i in range(n//2-1, -1, -1): #We start from the last non leaf node, and decrement downwards to the root
        heapify(arr, n, i) #Calling heapify() on each node to ensure the subtree rooted at that node satisfies the max-heap property.

    # Step 2: Extract elements one by one
    for i in range(n-1, 0, -1):
        arr[i], arr[0] = arr[0], arr[i]  # Swap
        heapify(arr, i, 0)
```

### heapify helper function

`heapify` is a helper function that makes sure the tree with root `i` (and all its subtrees through the recursive calls) follow the Max Heap property.

```python
def heapify(arr, n, i):
    largest = i #In the beginning we suppose the root of the subtree will be the largest
    left = 2 * i + 1 #left child of the current root in the current subtree
    right = 2 * i + 2 #Right child of the current root in the current subtree

    # Check if left child is larger than root
    if left < n and arr[left] > arr[largest]:
        largest = left

    # Check if right child is larger than current largest
    if right < n and arr[right] > arr[largest]:
        largest = right

    # Swap and continue heapifying if needed
    if largest != i: #if i(the root of the subtree) was not the largest, we make the largest the new root
        arr[i], arr[largest] = arr[largest], arr[i]
        heapify(arr, n, largest)

# Example
arr = [8, 4, 7, 3, 1, 5, 2, 6, 10]
heap_sort(arr)
print("Sorted array:", arr)
```
