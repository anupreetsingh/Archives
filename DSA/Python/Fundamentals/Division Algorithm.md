# Division Algorithm in Python

```text
A = (B x Q) + R
```

Where `A` -> divident, `B`-> divisor, `Q` -> quotient, and `R`-> remainder.

## Python operators

- `Q = A // B`
- `R = A % B`

Substitute these values in the original equation:

```text
A = (B x Q) + R
A = ((A // B) x B ) + (A % B)
```

```python
A = 17
B = 5

Q = A // B
R = A % B

print(Q)  # Output: 3
print(R)  # Output: 2

result = (B * Q) + R
print(result)       # Output: 17
print(result == A)  # Output: True
```

## Using `divmod()`

`divmod(A, B)` is a standard built in function that returns both `Q` and `R` together.

```python
A = 17
B = 5

Q, R = divmod(A, B)

print(Q)  # Output: 3
print(R)  # Output: 2
```

> This concept is useful in number problems, base conversion, hashing, cyclic indexing, and even/odd checks.
