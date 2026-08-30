# Built-in Functions in Python

Python makes built-in functions available without an import. Their names are
found through the built-in namespace when they have not been shadowed by a name
in a closer scope. See [Namespace and Scope Resolution](<Namespace and Scope.md>).

## `pow()`

`pow(base, exponent, modulus=None)` is the how it is internally  definition. This gives us option for **runtime polymorphism** and we can use pow() in two ways:

### `pow(base, exponent)`

The two-argument form `pow(base, exponent)` calculates `base ** exponent`, raising `base` to `exponent` using **binary exponentiation**.

Binary exponentiation recursively squares the base and halves the exponent. When the exponent is odd, one copy of the base is first taken outside so that the remaining exponent is even.

```text
if exponent is even:
    base^exponent = (base^2)^(exponent // 2)

if exponent is odd:
    base^exponent = base * (base^2)^(exponent // 2)
```

For `pow(2, 100)`:

```text
2^100
= 2^(2 * (100 // 2))
= (2^2)^(100 // 2)
= 4^50

= 4^(2 * (50 // 2))
= (4^2)^(50 // 2)
= 16^25

= 16^(2 * (25 // 2) + 1)
= 16 * (16^2)^(25 // 2)
= 16 * 256^12

= 16 * (256^2)^(12 // 2)
= 16 * 65,536^6

= 16 * (65,536^2)^(6 // 2)
= 16 * 4,294,967,296^3

= 16 * 4,294,967,296 * (4,294,967,296^2)^(3 // 2)
= 16 * 4,294,967,296 * 18,446,744,073,709,551,616

= 1,267,650,600,228,229,401,496,703,205,376
```

### `pow(base, exponent, modulus)`

The three-argument form, `pow(base, exponent, modulus)`, calculates `(base ** exponent) % modulus` using **binary exponentiation** along with **modular arithmetic** at every step so that it calculates final answer without first constructing the potentially huge value produced by `base ** exponent`.

The three-argument form requires integer arguments, and `modulus` cannot be zero. A negative integer exponent is supported when the base has a modular inverse for the given modulus; otherwise, Python raises `ValueError`.

For `pow(2, 100, 7)`, the same transformations are used, but each newly squared base is reduced modulo `7`:

```text
2^100 mod 7
= [((2)^2) mod 7]^(100 // 2) mod 7
= (4 mod 7)^50 mod 7
= 4^50 mod 7

= [((4)^2) mod 7]^(50 // 2) mod 7
= (16 mod 7)^25 mod 7
= 2^25 mod 7

= (2 * [((2)^2) mod 7]^(25 // 2)) mod 7
= (2 * (4 mod 7)^12) mod 7
= (2 * 4^12) mod 7

= (2 * [((4)^2) mod 7]^(12 // 2)) mod 7
= (2 * (16 mod 7)^6) mod 7
= (2 * 2^6) mod 7

= (2 * [((2)^2) mod 7]^(6 // 2)) mod 7
= (2 * (4 mod 7)^3) mod 7
= (2 * 4^3) mod 7

= (2 * 4 * [((4)^2) mod 7]^(3 // 2)) mod 7
= (2 * 4 * (16 mod 7)) mod 7
= (2 * 4 * 2) mod 7
= 16 mod 7
= 2
```

### Example Use

```python
import math

base = 2
exponent = 100
modulus = 7

print(pow(base, exponent))           # 1267650600228229401496703205376 (exact int)
print(base ** exponent)              # 1267650600228229401496703205376 (exact int)

print(math.pow(base, exponent))      # 1.2676506002282294e+30 (returns float; and may lose precision) # Also there is no three argument form for math.pow()

print(pow(base, exponent, modulus))  # 2 (avoids constructing the huge power)
print((base ** exponent) % modulus)  # 2 (constructs the huge power before applying modulo)
```

## `divmod()`

The syntax is:  `Q, R = divmod(A, B)`

`divmod(A, B)` returns the quotient and remainder together as the tuple `(Q, R)`. The first result is `A // B`, and the second result is `A % B`:

```text
Q = A // B
R = A % B
A = Q * B + R
```

### Example Use

```python
A = 17
B = 5

Q, R = divmod(A, B) # Q, R = 3, 2

# Negative dividend
Q, R = divmod(-17, 5) # Q, R = -4, 3
# Check: -17 = (-4) * 5 + 3

# Negative divisor
Q, R = divmod(17, -5)  # Q, R = -4, -3
# Check: 17 = (-4) * (-5) + (-3)

# Negative dividend and divisor
Q, R = divmod(-17, -5) #  Q, R = 3, -2
# Check: -17 = 3 * (-5) + (-2)
```

For positive and negative inputs, the floor division `//` equivalence in divmod() still rounds the quotient down toward negative infinity.

> A nonzero remainder has the same sign as the divisor `B`.
