# Binary Search

Binary search is used when you have a **monotonic** search space. Using this knowledge of monotonicity in the search space we can disregard half the search space at each step, thus taking log n steps instead of n steps across the space.

## Monotonic

**Monotonic** means going in one direction. Different search spaces can be monotonic in different sense:

### Sorted Monotonicity

A sorted array of `n` elements is monotonic because its elements move in only one direction when going left to right

    - Non decreasing sorted: each element stays the same or increases.
    - Non increasing sorted: each element stays the same or decreases.

Usually, in such a case we leverage knowledge of this monotonic nature to look for a **target** element.

Example:

```python
def binary_search(nums, target):
    left = 0
    right = len(nums) - 1

    while left <= right:
        mid = (left + right) // 2

        if nums[mid] == target:
            return mid
        elif nums[mid] < target:
            left = mid + 1
        else:
            right = mid - 1

    return -1
```

We use knowledge of the monotonic nature of the array being non-decreasing to say that if `nums[mid] < target`, the target can only be on the right side. If `nums[mid] > target`, the target can only be on the left side.

### Partial Monotonicity

We can even have piecewise / partial sorted monotonicity, where we are only sure about one half of the array being monotonic, to disregard the other half and conduct binary search.

Example: finding the greatest peak in a mountain array.

```python
nums = [0, 1, 2, 4, 7, 9, 8, 6, 3, 1]

def peak_index(nums):
    left = 0
    right = len(nums) - 1

    while left < right:
        mid = (left + right) // 2

        if nums[mid] < nums[mid + 1]: # Increasing slope 
            left = mid + 1 # Peak must be to right
        else: # Not increasing slope(same or descending)
            right = mid # mid could be peak

    return left
```

If `nums[mid] < nums[mid + 1]`, we are on the increasing slope, so the peak must be to the right. Otherwise, `mid` may already be the peak, so we keep it and search left.

### Binary Monotonicity

A boolean array of `n` elements is monotonic if it changes value at most once when going left to right.
    - False to True (False False False False True True)
    - True to False (True True True False False False False)

This array doesn't have to be a literal array but could also be a theoretical search space.

Usually in such a case, we are looking for the **boundary** where the two booleans change.

Example:
Searching across a boolean answer space, where values change only once from False to True.

False False False True True True

In this example if `x` is feasible, then we are sure every larger value must also be feasible. Using that knowledge to inform our decision of shrinking the search space at each iteration.

Written using `while <` :

```python
def first_feasible(l, r, feasible):
    while l < r:
        mid = (l + r) // 2
        if feasible(mid): # Every thing above and including mid is feasible
            r = mid # Search left half for boundary, including mid 
        else: # mid is not feasible
            l = mid + 1 # Search right half for boundary, excluding mid 
    return l
```

Written using `while <=` :

```python
def first_feasible(l, r, feasible):
    ans = None
    while l <= r:
        mid = (l + r) // 2
        if feasible(mid): # Every thing above and including mid is feasible
            ans = mid # Store this feasible value
            r = mid - 1 
        else: # mid is not feasible
            l = mid + 1 # Search right half, excluding mid for boundary
    return ans
```

## Heuristic for Loop Design

Depending on the kind of exit condition you want, There are two ways to design a binary search:

### while l <= r

Exit condition: `l > r` i.e. `l = r + 1`, Stops when range is empty

`mid` is discarded from the next interval: `l = mid + 1` or `r = mid - 1`. Though, well to remember that means we probably check something about the monotonic condition at mid before discarding it and moving on.

When the loop ends:
l = first value on the high side. (Insertion index, since we insert at position just greater)
r = last value on the low side. (Square root floor since we round down.)

Commonly used in scenarios like:

- A **target search**, check whether `mid` is the solution and exit if it is.
- A **boundary search**, store `mid` as the current best answer when it satisfies the condition, then continue searching for the actual boundary.

### while l < r

Exit Condition: `l == r`, Stop when one candidate remains.

`mid` may still be the answer in one branch, so it can be kept in the next interval with `r = mid`. In the other branch, if `mid` cannot be the answer, it can be discarded with `l = mid + 1`.

**Avoid** `l = mid` and `r = mid - 1` to converge, because eventually when `l` and `r` are adjacent, `mid = (l + r) // 2` floors the division and keeps giving us the left number. So `mid` ends up equal to `l`, and if the update is `l = mid`, the loop can repeat forever.

For example, if `l = 3` and `r = 4`, then `mid = 3`, `l` stays `3`, and the loop can repeat forever.

When the loop ends, `l == r` so we can return either l or r.

Commonly used when you are sure there is at least **one possible** answer inside `[l, r]`.
