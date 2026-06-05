# Python If-Else Statements

## Basic syntax

```python
x = 10
# Python checks the first condition
if x > 5:
    print("x is greater than 5")
# If the first condition is False, it checks this one
elif x == 5:
    print("x is equal to 5")
# If none of the above conditions are True, this block runs
else:
    print("x is less than 5")
```

---

## Short-Circuit Evaluation

Python uses short-circuit evaluation (Left to Right) in logical expressions:

- For `or`: If the first condition is True, the second is not evaluated.
- For `and`: If the first condition is False, the second is not evaluated.

### Example 1: Using `or` to safely avoid IndexError

```python
s = "hi"
i = 2
if i == len(s) or s[i] != 'x':
    print("No error due to short-circuiting with 'or'")
```

**Explanation:**
`i == len(s)` is True, so `s[i]` is never evaluated. Prevents accessing an index that doesn't exist.
