# Index and Counting Calculation in Lists and Matrices

## Middle Index

This section discusses a heuristic for finding the middle point of an array.

For an array of size `n`, there are two useful "middles":

- lower middle = last element of the left half
- upper middle = first element of the right half

For even `n`, they are different and equally important since there is no true middle.
For odd `n`, they collapse into the same element: the true middle.

**0-indexed:**

```text
lower_middle = (n - 1) // 2
upper_middle = n // 2
```

**1-indexed:**

1 - indexed is just +1 addition to 0 - indexed counterpart

```text
lower_middle = (n - 1) // 2 + 1 = (n + 1) // 2
upper_middle = n // 2 + 1
```

## Negative Indexing

In sequence types such as lists, strings, and tuples, negative indexing is a shortcut for referring to the nth element from the end of the sequence.

```text
-1 means 1st from the end
-2 means 2nd from the end
-3 means 3rd from the end
```

A negative index can be converted into a normal zero-based index by adding the sequence length.

```text
normal_index = len(sequence) + negative_index
```

This conversion is O(1) for built-in sequence types because they store their length on the object itself, so `len(sequence)` is also O(1).

Example:

For a sequence of length `5`, the normal indices and negative indices label the same positions:

```text
element:          10   20   30   40   50
normal index:      0    1    2    3    4
negative index:   -5   -4   -3   -2   -1
```

```python
nums = [10, 20, 30, 40, 50]
n = len(nums)

nums[-1] == nums[n - 1]  # nums[5 - 1] = nums[4] -> 50
nums[-2] == nums[n - 2]  # nums[5 - 2] = nums[3] -> 40
nums[-5] == nums[n - 5]  # nums[5 - 5] = nums[0] -> 10
```

Hence

```text
sequence[-k] = sequence[len(sequence) - k]
```

## Reversing Lists

The two case of:

- Reversing a list, which means reversing its elements so the list order actually changes.
- Reversing the indexing direction, where the elements stay in place but the index labels are read from the opposite side.

Both use the same mirror formula:

```text
new_i = (first_idx + last_idx) - i
```

Common forms:

```text
0-based indexing: new_i = (n - 1) - i
1-based indexing: new_i = (n + 1) - i
```

The reasoning behind the formula is explained in the subsections below.

### Reversing List Elements

After reversing the elements in a list, each element is mirrored around the center of the list. The element's distance from one end before reversing becomes its distance from the opposite end after reversing.

Before reversing, the element at index `i` is `i - first_idx` steps from the left side.

After reversing, that same element is at the new index `new_i`. Its distance from the right side is `last_idx - new_i`, which should equal its original distance from the left side.

```text
left_distance = right_distance

i - first_idx = last_idx - new_i

last_idx - new_i = i - first_idx

last_idx = i - first_idx + new_i

new_i = last_idx - i + first_idx

new_i = (first_idx + last_idx) - i
```

Example:

```text
original:       A  B  C  D  E
original index: 0  1  2  3  4

reversed:       E  D  C  B  A
reversed index: 0  1  2  3  4
```

### Reversing Indexing Directions

When the indexing direction is reversed, the index label assigned to each position changes, but the elements do not move, so each element's distance from the left end stays the same.

Before reversing the indexing direction, the element at normal index `i` is `i - first_idx` steps from the left side.

After reversing the indexing direction, the same element is still the same distance from the left side. However, the leftmost label is now `last_idx`, since labels go from right to left.

```text
normal_left_distance = reversed_left_distance

i - first_idx = last_idx - reversed_label

reversed_label = last_idx - i + first_idx

reversed_label = (first_idx + last_idx) - i
```

Example:

```text
element:        A  B  C  D  E
normal index:   0  1  2  3  4
reverse label:  4  3  2  1  0
```

### Reversing across Matrices

For an `m * n` matrix, where `m` is the number of rows and `n` is the number of columns, element reversal and index-direction reversal both use the same formula as lists.

Only apply the formula to the direction being reversed:

- If rows are reversed, use `new_r = (m - 1) - r`.
- If columns are reversed, use `new_c = (n - 1) - c`.
- If both are reversed, apply both formulas.

## Flat Index in Matrix

For an `m * n` matrix, the elements can be counted as one long flat list of elements scanned row by row.

### matrix position -> flat index

For an `m * n` matrix, each row has `n` columns. If you flatten the matrix row by row, every complete row before the current row contributes `n` elements.

So for a matrix position `(r, c)`:

```text
flat_index = r * n + c
```

Why this works:

- `r * n` skips all elements in the rows before row `r`.
- `c` moves to the correct column inside row `r`.

Example for a `3 * 4` matrix:

```text
matrix coordinates:

(0,0)  (0,1)  (0,2)  (0,3)
(1,0)  (1,1)  (1,2)  (1,3)
(2,0)  (2,1)  (2,2)  (2,3)

flat index in each position:

 0   1   2   3
 4   5   6   7
 8   9  10  11
```

So matrix position `(1, 2)` maps to:

```text
flat_index = 1 * 4 + 2
flat_index = 6
```

### flat index -> matrix position

The reverse calculation takes a flat index `i` and finds which row and column it belongs to:

```text
row = i // n
col = i % n
```

Why this works:

- Each row has `n` columns.
- `i // n` counts how many full rows come before index `i`.
- `i % n` gives the leftover position inside the current row.

Example for a `3 * 4` matrix:

```text
flat index in each matrix position:

 0   1   2   3
 4   5   6   7
 8   9  10  11

matrix coordinates:

(0,0)  (0,1)  (0,2)  (0,3)
(1,0)  (1,1)  (1,2)  (1,3)
(2,0)  (2,1)  (2,2)  (2,3)
```

So flat index `6` maps to:

```text
row = 6 // 4 = 1
col = 6 % 4 = 2

matrix[1][2]
```

## Subarrays

A subarray is a non-empty **contiguous** slice of the original array, fixed by choosing a start and end index.

For `[A, B, C]`, the subarrays are:

```text
length 1: [A], [B], [C]
length 2: [A, B], [B, C]
length 3: [A, B, C]
```

### Subarrays of Length `k`

In an array of length `n`, a subarray of length `k` can begin at any index from `0` through `n - k`.

The number of possible starting indices is therefore:

```text
(n - k) - 0 + 1 = n - k + 1
```

So:

```text
number of subarrays of length k = n - k + 1
```

For `[A, B, C]`:

```text
length 1: 3 - 1 + 1 = 3
length 2: 3 - 2 + 1 = 2
length 3: 3 - 3 + 1 = 1
```

### Total Number of Subarrays

#### Sum of different length of Subarrays

The total number of subarrays of an array of length `n` is the sum of the
number of subarrays of every possible length from `1` through `n`:

$$
\text{total} = \sum_{k=1}^{n}(n-k+1) = n+(n-1)+(n-2)+\cdots+1
$$

This is the sum of the first `n` positive integers:

```text
total non-empty subarrays = n(n + 1) / 2
```

For `[A, B, C]`:

```text
total = 3 + 2 + 1
      = 3(3 + 1) / 2
      = 6
```

#### Boundaries (Combinations)

Another way to see the same count is to choose two **distinct** boundaries from the `n + 1` boundaries surrounding the elements. The first boundary starts the subarray and the second ends it.

```text
| A | B | C |
0   1   2   3
```

Choosing boundaries `1` and `3`, for example, selects `[B, C]`. Thus:

```text
total non-empty subarrays = C(n + 1, 2) 
                          = (n + 1)! / ((n - 1)! * (2!))
                          = n * (n + 1) / 2
```

Requiring the two boundaries to be **distinct** is exactly what forbids the empty subarray: if the start and end boundary were allowed to coincide, that would add `n + 1` empty selections. So the empty case is excluded here by the choice rule itself, which is why the subarray count is stated for non-empty subarrays only.

## Subsequences

A subsequence is obtained by selecting `k` items from a sequence of `n` items without changing their relative order. The selected items do not need to be contiguous.

### Subsequences of Length `k`

To form a subsequence of length `k` from a sequence of length `n`, choose which `k` of the `n` distinct index positions `{0, 1, ..., n - 1}` to keep.

Once the positions are chosen, their order is assumed to be fixed as per the original sequence so we don't need to worry about permutations but just combinations/selections of the distinct positions.

Applying the size-`k` subset result to the index positions:

```text
number of subsequences of length k = subset of size k distinct positions = nCk
```

Important: `nCk` counts the ways to select `k` distinct **index positions** for forming `nCk` subsequences but those subsequences may not necessarily be unique depending on the values in the original sequence.

For example, consider:

```text
[1, 4, 1, 3, 1]

The sequence has five positions, so the number of positional subsequences of length `2` is: 5C2 = 10

However, choosing positions `(0, 2)`, `(0, 4)`, or `(2, 4)` produces the same value subsequence: [1, 1]
```

Thus, In this case there are `10` positional subsequences of length `2`, but only `6` distinct subsequences by value.

There is no standard formula for finding the distinct value subsequnce of size `k` from sequence of size `n`. The number of distinct value subsequences depends on the values and their order. For example, these sequences have the same length and element frequencies but different numbers of distinct length-2 subsequences:

```text
[1, 1, 2] -> [1, 1], [1, 2]                 -> 2
[1, 2, 1] -> [1, 2], [1, 1], [2, 1]         -> 3
```

### Total Number of Subsequences

There are `nCk` positional subsequences of length `k`. Summing over every possible length from `0` through `n` gives:

$$
\text{total subsequences}
= nC0 + nC1 + nC2 + \cdots + nCn
= \sum_{k=0}^{n} nCk
= 2^n
$$

Therefore:

```text
subsequences including the empty subsequence = 2^n
non-empty subsequences                       = 2^n - 1
```

The empty subsequence corresponds to the empty subset of positions. Contrast this with subarrays, where the empty case is excluded up front by requiring distinct boundaries.
