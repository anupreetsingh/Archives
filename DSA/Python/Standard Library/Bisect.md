# Bisect Module in Python

`bisect` is a Python standard library module for finding insertion positions in sorted sequences and inserting items into sorted lists. Its name means **divide into two**, referring to the binary search it uses.

The sequence must already be sorted in **ascending (non-decreasing) order** according to the values being compared. `bisect` does not sort the data or check that it is sorted.

## Finding Insertion Positions

### `bisect_left()` and `bisect_right()`

Both functions return an **index where a value could be inserted** while preserving sorted order. They leave the sequence unchanged.

| Function | Position returned | Treatment of existing equal values |
| --- | --- | --- |
| `bisect_left(a, x)` | First index whose value is `>= x` | Before all equal values |
| `bisect_right(a, x)` | First index whose value is `> x` | After all equal values |

These positions are often called the **lower bound** and **upper bound**, respectively. `bisect(a, x)` is an alias for `bisect_right(a, x)`.

```python
import bisect

numbers = [10, 20, 20, 30, 40]

print(bisect.bisect_left(numbers, 20))   # 1: before the first 20
print(bisect.bisect_right(numbers, 20))  # 3: after the last 20

# When the value is absent, both functions return the same position.
print(bisect.bisect_left(numbers, 25))   # 3: between 20 and 30
print(bisect.bisect_right(numbers, 25))  # 3

# An insertion position can be at either end.
print(bisect.bisect_left(numbers, 5))    # 0
print(bisect.bisect_left(numbers, 50))   # 5: len(numbers)
print(bisect.bisect_left([], 20))        # 0

print(numbers)  # [10, 20, 20, 30, 40]: unchanged
```

An insertion index does **not** prove that the value exists. It can also equal `len(a)`, which is valid for insertion but cannot be used to read an element.

### Restricting the Search with `lo` and `hi`

Both functions accept `lo` and `hi` to search the interval **`[lo, hi)`**: `lo` is included, and `hi` is excluded. Their defaults search the entire sequence.

```python
# Continue with numbers = [10, 20, 20, 30, 40].
# Search indexes 2, 3, and 4 without creating a slice.
index = bisect.bisect_left(numbers, 20, lo=2, hi=5)
print(index)  # 2: index in the original list
```

The result is between `lo` and `hi`, inclusive. It describes a position within the specified search interval, not necessarily the first suitable position in the whole list.

### Using the Positions for Lookups and Counts

For a sorted numeric list, let `left = bisect_left(a, x)` and `right = bisect_right(a, x)`:

| Question | Result |
| --- | --- |
| Does `x` exist? | `left < len(a) and a[left] == x` |
| How many values equal `x`? | `right - left` |
| How many values are `< x`? | `left` |
| How many values are `<= x`? | `right` |
| What is the greatest value `< x`? | `a[left - 1]`, if `left > 0` |
| What is the smallest value `>= x`? | `a[left]`, if `left < len(a)` |

These formulas assume a search of the **whole list**. Checking the bounds prevents an out-of-range access or accidentally reading the last item through index `-1`.

```python
# Continue with the same sorted numbers.
target = 20
left = bisect.bisect_left(numbers, target)
right = bisect.bisect_right(numbers, target)

found = left < len(numbers) and numbers[left] == target
print(found)         # True
print(right - left)  # 2 occurrences

# Count values in the inclusive interval [20, 30].
start = bisect.bisect_left(numbers, 20)
stop = bisect.bisect_right(numbers, 30)
print(stop - start)  # 3
print(numbers[start:stop])  # [20, 20, 30]
```

The final slice creates a new list; calculating the count does not require copying the matching elements.

## Inserting While Preserving Sorted Order

### `insort_left()` and `insort_right()`

These functions find an insertion position and then perform a [list insertion](<../Fundamentals/List Manipulation.md#listinsert>). They modify the original list and return `None`.

- `insort_left(a, x)` inserts before existing equal values.
- `insort_right(a, x)` inserts after existing equal values.
- `insort(a, x)` is an alias for `insort_right(a, x)`.

```python
# Continue with numbers = [10, 20, 20, 30, 40].
result = bisect.insort(numbers, 25)

print(numbers)  # [10, 20, 20, 25, 30, 40]
print(result)   # None

bisect.insort_left(numbers, 20)
print(numbers)  # [10, 20, 20, 20, 25, 30, 40]
```

They also accept `lo` and `hi`. Restricting insertion to a search interval preserves the whole list's order only if the value also fits between the elements outside that interval.

## Comparing Records with `key`

For records such as tuples or dictionaries, `key` selects the field used for comparison. The list must already be sorted by that same key. This parameter is available in Python 3.10 and later.

The value passed as `x` differs between searching and inserting:

- **`bisect_left()` / `bisect_right()`:** pass the search key itself. The function applies `key` to list elements, but not to `x`.
- **`insort_left()` / `insort_right()`:** pass the complete record. The function uses `key(x)` to find the position, then inserts the original record.

```python
players = [
    {"name": "Asha", "score": 10},
    {"name": "Ben", "score": 20},
    {"name": "Chen", "score": 20},
    {"name": "Dia", "score": 30},
]
by_score = lambda player: player["score"]

# Search with a score, not a player dictionary.
print(bisect.bisect_left(players, 20, key=by_score))   # 1
print(bisect.bisect_right(players, 20, key=by_score))  # 3

# Insert a complete player after the existing players with score 20.
bisect.insort_right(players, {"name": "Eli", "score": 20}, key=by_score)
print([player["name"] for player in players])
# ['Asha', 'Ben', 'Chen', 'Eli', 'Dia']
```

Using `insort_left()` instead would put Eli before Ben. The distinction becomes visible because different records can have equal comparison keys.

## Time Complexity

For a Python list of length `n`, assuming constant-time comparisons and key evaluation:

| Operation | Time |
| --- | --- |
| Find a position with `bisect_left()` or `bisect_right()` | `O(log n)` |
| Find a count using two insertion positions | `O(log n)` |
| Insert with `insort_left()` or `insort_right()` | `O(n)` |
| Copy `k` matches with a slice | `O(k)` |

Insertion includes both the search and the list operation, so the cost of shifting elements dominates. Repeated insertions can therefore take `O(n²)` total time when building a list of `n` items; if all items are available together, consider [sorting once](<../Fundamentals/List Manipulation.md#sorting-lists>).

Searches do not retain computed key values between calls. For repeated searches with an expensive key function, a parallel list of precomputed keys can avoid recalculating them; keep it synchronized with the records.
