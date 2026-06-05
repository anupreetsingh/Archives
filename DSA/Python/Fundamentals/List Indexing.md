# Index Math for Mirroring and Index Reversing

Setup:

- `n` = length of array
- `first` = first index (0 for 0-based, 1 for 1-based... so on)
- `last` = last index (n-1 for 0-based, n for 1-based... so on)
- Indexing goes from left → Right

---

## Case 1: Reverse the *elements* (Actually Mirror)

**Question:** After reversing the elements in an array/list, where does the element originally at index `i` end up?

Reasoning:

- Element at index `i` was (`i - first`) steps away from the LEFT end.
- After reversing, the same element must be (`i - first`) steps away from the RIGHT end.

```text
Distance from left (original):     i - first
Distance from right (after flip):  last - new_index
Equate:                            last - new_index = i - first
Solve:                             new_index = last - (i - first)
Simplify:                          new_index = (first + last) - i
```

- 0-based indexing : `new_index = (n-1) - i`
- 1-based indexing : `new_index = (n+1) - i`

## Case 2: Reverse the *indexing direction* (relabeling)

**Q:** Without moving elements, if we now change indexing to run right→left, what is the new index of the element that was at `i`?

Reasoning:

- The element at index `i` stays at the same physical spot.
- Its distance from the LEFT END (in space) is the same before and after.

```text
Old indexing (left->right):
  left-distance = i - first

New indexing (right->left):
  left-distance (same physical distance) = last - new_label

Equate:     last - new_label = i - first
Solve:      new_label = (first + last) - i
```

- 0-based indexing : `new_label = (n-1) - i`
- 1-based indexing : `new_label = n - i + 1`

## Common Formula (both cases)

No matter if we flip elements or flip labels, the mapping rule is:

```text
mapped_index = (first + last) - i
```

---

# Using Arithmetic Progression (AP) to reason about lists and its Indexes

General nth term of an AP:

```text
T_n = a + (n - 1) * d
```

where:

- `a` = first term
- `d` = common difference

## 1) Finding the nth index/position (indexing concept)

A list's index sequence is an AP with `d = 1`. The starting index is the first term `a` (e.g., 0-based vs 1-based). Therefore the nth index is:

```text
T_n = a + (n - 1) * 1 = a + (n - 1)
```

Plugging common starts:

```text
start at 0 → T_n = 0 + (n - 1)*1 = n - 1
start at 1 → T_n = 1 + (n - 1)*1 = n
start at 4 → T_n = 4 + (n - 1)*1 = n + 3
```

## 2) Steps vs Elements between two terms

Let `T_n` and `T_r` be the n-th and r-th terms of the SAME AP, assuming n ≥ r. Using `T_k = a + (k - 1)*d`:

```text
T_n - T_r = [a + (n - 1)d] - [a + (r - 1)d] = (n - r) * d
```

Divide by `d`:

```text
(T_n - T_r) / d = n - r
```

Interpretation:

- `n - r` = number of AP "steps" (jumps of size d) from the r-th term to the n-th term.
- Elements strictly BETWEEN the r-th and n-th terms = `(n - r - 1)`.
- Elements INCLUDING both endpoints (i.e., count from r through n) = `(n - r + 1)`.

**Example with AP** — `AP = 3, 7, 11, 15, 19, 23, 27, 31` (a = 3, d = 4)

```text
Pick T_7 and T_3:
  T_7 = 27
  T_3 = 11

Steps apart:
  (T_7 - T_3) / d = (27 - 11) / 4 = 16 / 4 = 4 steps

Elements strictly between:
  steps - 1 = 4 - 1 = 3  → {15, 19, 23}

Elements including both endpoints:
  steps + 1 = 4 + 1 = 5  → {11, 15, 19, 23, 27}
```
