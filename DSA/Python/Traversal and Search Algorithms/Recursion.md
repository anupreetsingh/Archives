# 📌 Recursion in Python — Notes & Examples

## Introduction

Recursion is a technique where a function calls itself to solve a smaller version of the same problem.

Key idea:

- Break a big problem into smaller subproblems of the same type.
- Stop at a "base case" that can be solved directly.
- Build the solution back up from those base cases.

## Designing a Recursive Function — Ground Rules

1. Define the TRUE BASE CASE (smallest solvable input).
2. Handle the base case correctly (all results depend on it).
3. Consider GUARD CASES (invalid/out-of-bounds situations).
4. Ensure PROGRESS toward the base case in each recursive call.
5. Combine the results of recursive calls to form the solution.
6. Optimize with memoization if subproblems repeat.

## True Base Case vs Guard Case

- **True Base Case:** smallest *valid* problem instance with direct answer.
- **Guard Case:** Defensive stop for invalid or unsafe situations. Prevents runtime errors or meaningless work.

---

## Example: Path Counting in a Grid (with guard case)

```python
def count_paths(grid, r, c):
    """
    Count paths from (r,c) to bottom-right in a grid.
    True base case: reached destination cell → 1 path
    Guard case: out-of-bounds or blocked cell → 0 paths
    """
    rows, cols = len(grid), len(grid[0])

    # Guard case first → prevents index errors
    if r >= rows or c >= cols or grid[r][c] == 1:
        return 0

    # True base case
    if r == rows - 1 and c == cols - 1:
        return 1

    # Recursive step → explore right and down
    return count_paths(grid, r+1, c) + count_paths(grid, r, c+1)
```

---

## Summary

- Recursion works by breaking a problem into smaller instances.
- Always define:
    - TRUE BASE CASE → smallest valid problem.
    - PROGRESS → recursive step reduces the problem.
    - GUARD CASES → defensive stops for invalid/unsafe cases.
- If recursion recomputes the same subproblems, consider memoization or bottom-up dynamic programming.

Pattern:

```python
def recursive_function(params):
    if <guard condition>:
        return <sentinel/0/None>
    if <true base case>:
        return <direct answer>
    return combine( recursive_function(smaller_params) )
```
