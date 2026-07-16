# Lambda (In-line) Functions

## What is a lambda function?

A lambda is a small anonymous function defined using the `lambda` keyword. It can take any number of arguments but only one expression.

Syntax: `lambda arguments: expression`

---

## Basic example

```python
add = lambda x, y: x + y
# Here, `lambda x, y: x + y` creates a function object. That function object is then bound to the name `add`.

print(add(2, 3))  # Output: 5

# Equivalent to:
# def add(x, y):
#     return x + y
```

## Lambda as dictionary values

```python
ops = {
    '+': lambda a, b: a + b,
    '-': lambda a, b: a - b,
    '*': lambda a, b: a * b
}
print(ops['*'](4, 5))  # Output: 20
```

Very useful in cases like RPN calculators or interpreters.

## Use with `sorted()` for custom sort keys

```python
students = [('Alice', 25), ('Bob', 19), ('Charlie', 23)]

# Sort by age (2nd element)
sorted_by_age = sorted(students, key=lambda x: x[1])
print(sorted_by_age)
# Output: [('Bob', 19), ('Charlie', 23), ('Alice', 25)]
```

## Use with `map()`, `filter()`, and `reduce()`

```python
# map() – apply function to each element
squared = list(map(lambda x: x**2, [1, 2, 3, 4]))  # [1, 4, 9, 16]

# filter() – filter elements that meet a condition
evens = list(filter(lambda x: x % 2 == 0, [1, 2, 3, 4]))  # [2, 4]

# reduce() – apply function cumulatively
from functools import reduce
product = reduce(lambda a, b: a * b, [1, 2, 3, 4])  # 24
```

---

## Limitations of lambda

- Only one expression allowed (no statements like if/else, loops)
- No assignment (can't use `:=` inside it)
- Less readable when overused

## When to use lambda

- ✅ One-liner functions
- ✅ Inline usage (like key functions)
- ✅ When defining small functions you don't plan to reuse
- 🚫 Avoid for complex logic or when naming improves clarity
