# 🧮 Operators in Python

Operators in Python are special symbols or keywords that perform operations on variables and values.

→ They act on operands (the data being operated on) and return a result.

**Example:**

```python
x = 5 + 3
# '+' is the operator
# 5 and 3 are operands
# The result is 8, which is assigned to x
```

Operators are the foundation of expressions in Python and are used for:

- Arithmetic calculations
- Comparisons
- Logical operations
- Bitwise manipulation
- Assignments
- Identity and membership tests

They make Python code expressive and concise by handling common operations in a readable way.

---

## 1. Arithmetic Operators

```python
a = 10
b = 3

print(a + b)  # Addition ➕ : 13
print(a - b)  # Subtraction ➖ : 7
print(a * b)  # Multiplication ✖️ : 30
print(a / b)  # Division ➗ : 3.333..., returns the Quotient as a floating point number
print(a % b)  # Modulus 🧮 : 1, Returns the Remainder of the division
print(a//b)   # Floor Division : 3 , returns the Quotient as an integer(Rounded down), so for -10//3 it gives -4(since it rounds down a number)
              # Floor Division is particularly handy when trying to reverse a list cause n//2 will takes you upto index of last swap
```

## 2. Relational / Comparison Operators

```python
a = 5
b = 7

print(a == b)  # Equal to ❓ : False
print(a != b)  # Not equal to 🚫 : True
print(a < b)   # Less than 👈 : True
print(a > b)   # Greater than 👉 : False
print(a <= b)  # Less than or equal to 👈 Or equal? : True
print(a >= b)  # Greater than or equal to 👉 Or equal? : False
```

## 3. Logical Operators

```python
x = True
y = False

print(x and y)  # AND 🔌 : False
print(x or y)   # OR 🔌 : True
print(not x)    # NOT 🔌 : False
```

## 4. Assignment Operators

```python
x = 5  # Assignment 📦 : x = 5
x += 3  # Add and assign ➕ : x = 8
x -= 2  # Subtract and assign ➖ : x = 6
x *= 2  # Multiply and assign ✖️ : x = 12
x /= 4  # Divide and assign ➗ : x = 3.0
x %= 2  # Modulus and assign 🧮 : x = 1.0
```

## 5. Increment / Decrement Operators

```python
i = 0
i += 1  # Increment: i = 1
i -= 1  # Decrement: i = 0
```

## 6. Bitwise Operators

You don't need to convert int to binary first to apply these operators, they by default work on the binary form of the number.

```python
a = 5   # 0101
b = 3   # 0011
```

**Important Fact:** When numbers are of unequal binary length, Python aligns them by padding the shorter one with leading zeroes before applying the operator.

```python
# AND (&)
# Syntax: x & y
# Action: 1 if both bits are 1
# 0101 & 0011 = 0001 (1)
print(a & b)   # 1

# OR (|)
# Syntax: x | y
# Action: 1 if at least one bit is 1
# 0101 | 0011 = 0111 (7)
print(a | b)   # 7

# XOR (^)
# Syntax: x ^ y
# Action: 1 if bits are different
# 0101 ^ 0011 = 0110 (6)
print(a ^ b)   # 6

# NOT (~)
# Syntax: ~x
# Action: flips all bits (2's complement form)
# ~0101 = ...1010 = -6
print(~a)      # -6

# LEFT SHIFT (<<)
# Syntax: x << n
# Action: shifts binary number x to the left by n bits by adding n zeroes to the right
# Equivalent to multiplying x by 2^n (similar to multiplying decimal by 10^n)
# 0101 << 1 = 1010 (10)
print(a << 1)  # 10

# RIGHT SHIFT (>>)
# Syntax: x >> n
# Action: shifts binary number x to the right by n bits by discarding rightmost bits
# Equivalent to floor dividing x by 2^n
# 0101 >> 1 = 0010 (2)
print(a >> 1)  # 2
```

## 7. Ternary Operator

```python
a = 10
b = 3
max_value = a if a > b else b  # Conditional expression: max_value = 10
print(max_value)
```

## 8. Scope Resolution Operator

```python
global_x = 100

def print_global():
    global global_x  # Using global variable
    print(global_x)   # Output: 100

print_global()
```
