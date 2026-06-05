# 🔁 Iteration

- Executes a block of code repeatedly in a loop until a condition is met.
- Flow is linear — no call stack growth.
- Usually used when:
  - The number of steps is known in advance.
  - We can build the solution progressively without needing recursion.
- Time and space efficiency is often better than recursion for simple repetition.

Example: factorial using iteration

```python
def factorial_iter(n):
    result = 1
    for i in range(2, n + 1):   # Loop from 2 to n
        result *= i             # Multiply progressively
    return result
```
