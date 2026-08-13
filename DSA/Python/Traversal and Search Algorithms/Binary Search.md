# Binary Search

Binary search is used when you have a **monotonic** search space. Using this knowledge of monotonicity in the search space we can disregard half search search space at each step thus taking log n steps instead of n steps across the space.

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

We use knowledge of the monotonic nature of the array(being non-decreasin) to say that if `nums[mid] < target`, the target can only be on the right side. If `nums[mid] > target`, the target can only be on the left side.

### Partially Sorted Monotonicity

We can even have piecewise / partial sorted monotonicity, where we are only sure about one half of the array being monotonic to conduct binary search.

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

Usually in such a case, In such a case we are looking the **boundary** where the two booleans change.

Example:
Searching across a boolean answer space, where values change only once from False to True.

False False False True True True

if `x` is feasible, then we are sure every larger value must also be feasible.

Written using `while <` :

```python
def first_feasible(l, r, feasible):
    while l < r:
        mid = (l + r) // 2
        if feasible(mid): # Every thing above and including mid is feasible
            r = mid # Search left half including mid for boundary
        else: # mid is not feasible
            l = mid + 1 # Search right half, excluding mid for boundary
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

Not all algorithms can be written in both loop designs but depending on the scenario there is a heuristic for using the type of invariant in the while loop depending on the inner logic.

### while l <= r

Stop when range is empty: `l > r`.

Used when `mid` is discarded from the next interval: `l = mid + 1` or `r = mid - 1`.

Since `mid` is discarded from the next interval, it is common to handle the possible solution at `mid` before moving the bounds:

- In a **target search**, check whether `mid` is the solution and exit if it is.
- In a **boundary search**, store `mid` as the current best answer when it satisfies the condition, then continue searching for the actual boundary.

### while l < r

Stop when one candidate remains: `l == r`.

Used when `mid` may still be the answer, so it can be kept in the next interval: `r = mid` or `l = mid + 1`.

Commonly used when you are sure there is at least one possible answer inside `[l, r]`, especially in case of **N**

**Avoid** `l = mid` and `r = mid - 1` to converge, because eventually  when `l` and `r` are adjacent, `mid = (l + r) // 2` floors the division and l stays at mid, stuck in an infinite loop.
For example, if `l = 3` and `r = 4`, then `mid = 3`, `l` stays `3`, and the loop can repeat forever.
