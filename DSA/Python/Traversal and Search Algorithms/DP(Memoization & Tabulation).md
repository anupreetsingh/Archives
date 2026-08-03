# Dynamic Programming (DP) Overview

Dynamic Programming (DP) is a method to solve complex problems by breaking them into smaller overlapping subproblems.

**Conditions for a DP solution:**

1. **Overlapping Subproblems** → Same subproblems are solved multiple times. Example: Fibonacci → `fib(4)` requires `fib(3) + fib(2)`, and `fib(3)` also requires `fib(2)`.

2. **Optimal Substructure** → The optimal solution to a big problem can be built from optimal solutions of its smaller subproblems. Example: Shortest path in a graph = min(shortest paths of sub-routes).

**Key Idea:**

- Store results of subproblems to avoid recomputation.
- Trade space (extra memory) for time (faster execution).
- Converts exponential recursion → polynomial-time solution.

Two main strategies:

- Memoization (Top-Down): recursion + cache
- Tabulation (Bottom-Up): iterative + table

---

## Dynamic Programming (Memoization — Top-Down)

- Start with the final problem and recursively break it into smaller subproblems, caching each answer as it is computed.
- Use a dictionary, array, or `@cache` to store results.
- The cache prevents repeated computation of the same subproblems.

```python
def fib_memo(n, memo={}):
    if n in memo:        # Reuse cached result if available
        return memo[n]
    if n <= 1:           # Base case
        return n
    # Store result in cache
    memo[n] = fib_memo(n - 1, memo) + fib_memo(n - 2, memo)
    return memo[n]
```

## Dynamic Programming (Tabulation — Bottom-Up)

- Start with the smallest subproblems and iteratively build up to the final answer.
- Use a table, such as a list or array, to store values for subproblems.
- Often has more predictable memory usage than memoization because the table size and fill order are explicit.

```python
def fib_tab(n):
    if n <= 1:
        return n
    dp = [0] * (n + 1)  # Create table
    dp[1] = 1
    for i in range(2, n + 1):
        dp[i] = dp[i - 1] + dp[i - 2]
    return dp[n]
```

## 🔍 Quick Test

```python
if __name__ == "__main__":
    n = 10
    print("Fibonacci using Memoization:", fib_memo(n))
    print("Fibonacci using Tabulation:", fib_tab(n))
```
