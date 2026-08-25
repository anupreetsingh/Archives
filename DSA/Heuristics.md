## Using input constraints to determine Algorithm Type

![Input-Complexity-Algorithm Relation](<Media/Input-Complexity Relation.png>)

Most coding platforms start hitting **TLE (Time Limit Exceeded)** around `10^7` to `2 * 10^7` operations. So the input size `N` usually gives a strong hint about the acceptable time complexity.

These are **strong guidelines, not hard rules**.

1. If `0 < N < 20`

   Acceptable worst-case complexity:

   ```text
   O(2^N)
   ```

   Common approaches:

   - Brute force
   - Backtracking
   - Bitmask DP
   - Recursion

   If `0 < N < 12`, even this may be acceptable:

   ```text
   O(N!)
   ```

2. If `0 < N < 10^3`

   Acceptable worst-case complexity:

   ```text
   O(N^2)
   ```

   Common approaches:

   - 2D Dynamic Programming
   - All-pairs graph algorithms
   - Nested-loop comparisons

3. If `0 < N < 10^6`

   Acceptable worst-case complexity:

   ```text
   O(N)
   O(N log N)
   O(K log N)
   ```

   Common approaches:

   - Sorting
   - Single-pass HashMap
   - Two pointers
   - Sliding window
   - Prefix/suffix arrays
   - Greedy

4. If `N > 10^6`

   Acceptable worst-case complexity:

   ```text
   O(log N)
   O(1)
   ```

   Common approaches:

   - Binary search
   - Math-based operations
   - Bit manipulation

## Dictionary vs Array for Lookup

A dictionary is usually the natural choice for mapping keys to values. It works with arbitrary hashable keys, dynamically stores only the keys that are present, and provides `O(1)` average lookup.

There is one useful optimization: when the keys belong to a small, dense range and can be mapped directly to integer indices, an array can act as the lookup table instead. Array indexing is also `O(1)`, but it avoids the hashing and collision-handling overhead of a dictionary. Arrays also tend to have lower memory overhead and better cache locality.

Use an array instead of a dictionary when:

- Each key is an integer or maps naturally to an integer index. For example, a fixed, sequential character set such as the digits `0` to `9` or the letters `a` to `z` can be mapped to array indices.
- The complete key range is small enough to allocate. For example, values from `1` to `1_000` require only `1_000` array positions when each key is mapped to `key - 1`.
- Most of the possible keys are likely to be used. Do not allocate an array covering `1` to `1_000_000` when only `1_000` scattered keys are expected to occur.

### Example 1: Counting Lowercase Letters

A dictionary works for counting characters:

```python
counts = {}

for char in text:
    counts[char] = counts.get(char, 0) + 1
```

However, if the input contains only lowercase English letters, the key space is known and dense: the 26 letters from `a` to `z`. An array can represent the same mapping with less lookup overhead:

```python
counts = [0] * 26

for char in text:
    index = ord(char) - ord("a")
    counts[index] += 1
```

### Example 2: Tracking Visited Graph Nodes

A dictionary can track whether each graph node has been visited:

```python
visited = {}

visited[start] = True

if not visited.get(neighbor, False):
    visited[neighbor] = True
```

If the nodes are numbered from `0` to `n - 1`, every node already corresponds to an array index. A boolean array provides the same lookup with less overhead:

```python
visited = [False] * n

visited[start] = True

if not visited[neighbor]:
    visited[neighbor] = True
```

Keep the dictionary when keys are arbitrary or sparse. For example, allocating an array for only the IDs `{12, 50_000, 900_000_000}` would waste an enormous amount of space.
