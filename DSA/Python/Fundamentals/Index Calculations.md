# Index Calculation in Lists and Matrices

## Middle Index

This section discusses a heuristic for finding the middle point of an array.

For an array of size `n`, there are two useful "middles":

- lower middle = last element of the left half
- upper middle = first element of the right half

For even `n`, they are different and equally important since there is no true middle.
For odd `n`, they collapse into the same element: the true middle.

### What `// 2` Gives You

**0-indexed:**

```text
last index = n - 1

(n - 1) // 2 gives: 
- even n: lower middle 
- odd n: True middle 

n // 2 gives:
- even n: Upper middle
- odd n: True middle
```

**1-indexed:**

```text
last_index = size = n

n // 2 gives:
- even n: lower middle
- odd n: element just before the true middle
```

### Formulas for Getting the Middle Index

These formulas give the lower middle and upper middle, which are different for even `n` and automatically converge to the same element( the true middle) for odd `n`.

**0-indexed:**

```text
lower_middle = (n - 1) // 2
upper_middle = n // 2
```

**1-indexed:**

```text
lower_middle = (n + 1) // 2
upper_middle = n // 2 + 1
```

Use either formula for the corresponding indexed array, depending on whether you need the lower middle or upper middle when `n` is even.

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
