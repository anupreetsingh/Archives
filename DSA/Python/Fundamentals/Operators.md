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
print(a//b)   # Floor Division : 3 , returns the Quotient as an integer(Rounded down towards negative infinity aka leftwards on the number line), so -10//3 gives -4.
              # Particularly useful for reversing a list since n//2  gives middle index in case of odd elements and first element of second half in case of even elements. So you only have to traverse the first half of the list 
              # for i in range(n // 2):
              #     arr[i], arr[n - 1 - i] = arr[n - 1 - i], arr[i]
            
```

## 2. Relational / Comparison Operators

```python
a = 5
b = 7

print(a == b)  # Equal to : False
print(a != b)  # Not equal to : True
print(a < b)   # Less than : True
print(a > b)   # Greater than : False
print(a <= b)  # Less than or equal to Or equal? : True
print(a >= b)  # Greater than or equal to Or equal? : False
```

### Equality vs Identity

`==` checks whether two objects have the same **value**.

`is` checks whether two variables point to the same **object in memory**.

```python
a = tuple([1, 2])
b = tuple([1, 2])

print(a == b)  # True, same value
print(a is b)  # False, different objects in memory
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

A ternary operator is a compact way to choose between **two values** based on a condition.

It is called **ternary** because the expression has three main parts:

```python
value_if_true if condition else value_if_false
```

This is the expression form of:

```python
if a > b:
    max_value = a
else:
    max_value = b
```

Example:

```python
a = 10
b = 3

max_value = a if a > b else b
# 10
```

## 8. Scope Resolution Operator

Python does not have a C++-style scope resolution operator like `::`.

Python resolves plain names using the **LEGB rule** and resolves attributes using dot notation, such as `module.name`, `Class.attribute`, and `object.attribute`.

See [Scope Resolution.md](</Users/manpreetsingh/Downloads/Study/Notes/DSA/Python/Fundamentals/Scope Resolution.md>) for `global`, `nonlocal`, class attributes, and instance attributes.
