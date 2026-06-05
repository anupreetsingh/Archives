# 🚀 Dynamic Programming (DP) Overview

Dynamic Programming (DP) is a method to solve complex problems by breaking them into smaller overlapping subproblems.

**Conditions for a DP solution:**

1️⃣ **Overlapping Subproblems** → Same subproblems are solved multiple times. Example: Fibonacci → `fib(4)` requires `fib(3) + fib(2)`, and `fib(3)` also requires `fib(2)`.

2️⃣ **Optimal Substructure** → The optimal solution to a big problem can be built from optimal solutions of its smaller subproblems. Example: Shortest path in a graph = min(shortest paths of sub-routes).

**Key Idea:**

- Store results of subproblems to avoid recomputation.
- Trade space (extra memory) for time (faster execution).
- Converts exponential recursion → polynomial-time solution.

Two main strategies:

- Memoization (Top-Down): recursion + cache
- Tabulation (Bottom-Up): iterative + table

---

## 📦 Dynamic Programming (Memoization — Top-Down)

- Use recursion and a dictionary (cache) to store results.
- Start from the main problem and break into smaller subproblems.
- Cache ensures repeated calls are avoided.

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

## 📊 Dynamic Programming (Tabulation — Bottom-Up)

- Iteratively build the solution from the smallest subproblems.
- Use a table (list) to store values up to n.
- More memory-predictable than memoization.

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
