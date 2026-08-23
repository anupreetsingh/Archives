# Built-in Functions in Python

Python makes built-in functions available without an import. Their names are
found through the built-in namespace when they have not been shadowed by a name
in a closer scope. See [Namespace and Scope Resolution](<Namespace and Scope.md>).

## `pow()`

`pow(base, exponent)` raises `base` to `exponent` and is equivalent to
`base ** exponent`.

The three-argument form, `pow(base, exponent, modulus)`, calculates
`(base ** exponent) % modulus` without first constructing the potentially huge
value produced by `base ** exponent`. Use this form for efficient modular
exponentiation.

```python
base = 2
exponent = 10
modulus = 7

print(pow(base, exponent))           # 1024
print(base ** exponent)              # 1024
print(pow(base, exponent, modulus))  # 2
print((base ** exponent) % modulus)  # 2, but may create a huge intermediate value
```

The three-argument form requires integer arguments, and `modulus` cannot be
zero. A negative integer exponent is supported when the base has a modular
inverse for the given modulus; otherwise, Python raises `ValueError`.

Do not confuse the built-in `pow()` with `math.pow()`. `math.pow()` accepts only
two arguments, converts them to floating-point numbers, and returns a `float`.
