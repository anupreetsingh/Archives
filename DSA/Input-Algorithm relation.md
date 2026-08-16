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
