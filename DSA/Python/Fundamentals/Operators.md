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
print(a ** b) # Exponentiation: 10 raised to the power of 3 = 1000

# Matrix multiplication (@)
# Used by matrix-like objects, such as NumPy arrays or custom classes.
# Normal Python lists do not support @ by default.
```

### Unary Arithmetic Operators

Unary operators work on one operand.

```python
x = 5

print(+x)  # Unary plus keeps the value/sign the same: 5
print(-x)  # Unary minus switches the sign: -5
```

## 2. Comparison Operators

```python
a = 5
b = 7

print(a == b)  # Equal to : False. `==` is the "Equality operator". It checks whether two objects have the same **value**.
print(a != b)  # Not equal to : True
print(a < b)   # Less than : True
print(a > b)   # Greater than : False
print(a <= b)  # Less than or equal to Or equal? : True
print(a >= b)  # Greater than or equal to Or equal? : False
```

Comparison operators can be chained. Python reads `a == b == c` as `a == b and b == c`.

```python
a = 5
b = 5
c = 5

print(a == b == c)  # True, because a == b and b == c
```

Under the hood, comparison operators call special dunder methods on objects:

| Operator | Dunder method |
|---|---|
| `==` | `__eq__` |
| `!=` | `__ne__` |
| `<` | `__lt__` |
| `>` | `__gt__` |
| `<=` | `__le__` |
| `>=` | `__ge__` |

For built-in types like integers, strings, lists, and tuples, Python already defines these methods.

For custom classes, you can define these dunder methods yourself when you want objects to compare based on your own rules.

```python
class Student:
    def __init__(self, name, grade):
        self.name = name
        self.grade = grade

    def __eq__(self, other):
        return self.grade == other.grade


a = Student("Ava", 90)
b = Student("Leo", 90)

print(a == b)  # True, because Student.__eq__ compares just grades
```

## 3. Identity Operators

Identity operators check whether two names are bound to the same object in memory.

`is` and `is not` cannot be customized. They mean object identity everywhere in Python.

```python
a = tuple([1, 2])
b = tuple([1, 2])
c = a

print(a == b)  # True, same value
print(a is b)  # False, different objects in memory
print(a is c)  # True, both names point to the same object
```

Use `is` or `is not` when comparing with `None`, because `None` is a singleton object in Python, which means there is only one such object instance.

```python
value = 0

if not value:
    print("No value")  # Runs because 0 is falsy. This would also run if value were None.

if value is None:
    print("Missing value")  # Does not run, because value is 0, not None.

if value is not None:
    print("Value exists")  # Runs because value is not None.
```

## 4. Membership Operators

Membership operators check whether a value exists inside a container object.

```python
nums = [1, 2, 3]
name = "Manpreet"

print(2 in nums)          # True
print(4 not in nums)      # True
print("Man" in name)      # True
print("preet" not in name)  # False
```

Under the hood, membership operators usually call `__contains__` on the container object:

| Operator | Dunder method |
|---|---|
| `in` | `__contains__` |
| `not in` | Negates the result of `__contains__` |

For built-in containers, the time complexity depends on the container type:

| Container | Average time complexity |
|---|---|
| `list` / `tuple` | `O(n)` |
| `str` | `O(n)` |
| `set` | `O(1)` average |
| `dict` | `O(1)` average, checks keys by default |
| `range` | `O(1)`, checks using arithmetic instead of scanning |

`range` is efficient for membership checks because Python can calculate whether a number is within the range bounds and fits the step pattern, instead of scanning each value.

Example:

```python
nums = range(0, 10, 2)  # 0, 2, 4, 6, 8

print(6 in nums)  # True
print(7 in nums)  # False

# Internally the __contains__ is doing something like 
start <= x < stop
and
(x - start) % step == 0
```

For custom classes, you can define `__contains__` yourself when you want `in` and `not in` to use your own membership rules.

```python
class Team:
    def __init__(self, members):
        self.members = members

    def __contains__(self, name):
        return name in self.members


team = Team(["Ava", "Leo"])

print("Ava" in team)  # True, because Team.__contains__ checks members
```

## 5. Logical Operators and Truth-Value Testing

Truth-value testing is the rule Python uses to decide whether an object should count as true or false in a Boolean context.

### Truthy and Falsy Values

Python can treat any object as either truthy or falsy.

Common falsy values:

```python
False
None
0
""
[]
{}
set()
```

Most other values are truthy:

```python
5
"hello"
[1, 2, 3]
{"name": "Ava"}
```

### Logical Operators

Logical operators use truth-value testing to combine or reverse truth values.

```python
x = True
y = False

print(x and y)  # AND : False
print(x or y)   # OR : True
print(not x)    # NOT : False
```

`not` tests the truthiness of a value and always returns a Boolean: either `True` or `False`.

```python
print(not 0)     # True
print(not None)  # True
print(not 5)     # False
```

`and` and `or` also use truthiness, but they do not always return `True` or `False`. They return one of the original values which could further be assessed to be truthy or falsy.

`and` and `or` are short-circuit operators. Python evaluates from left to right and stops as soon as the final result is already determined.

- For `and`, Python stops at the first falsy value. If no value is falsy, it returns the last value.
- For `or`, Python stops at the first truthy value. If no value is truthy, it returns the last value.

```python
print(0 and "hello")   # 0, stops at the first falsy value
print(5 and "hello")   # "hello", because both values are truthy, so the last value is returned

print("" or "backup")  # "backup", stops at the first truthy value
print("Ava" or "backup")  # "Ava", stops at the first truthy value
```

### Control Flow Statements

Control flow statements are not operators, but they use the same truthy/falsy rules to decide whether a code block should run.

`if` and `while` statement use truth value testing to execute code blocks:

```python
name = "Ava"

if name:
    print("Name exists")  # Runs because non-empty strings are truthy

items = [1, 2, 3]

while items:
    print(items.pop())  # Runs until the list becomes empty and falsy
```

## 6. Assignment Operators

```python
x = 5    # Assignment 📦 : x = 5
x += 3   # Add and assign ➕ : x = 8
x -= 2   # Subtract and assign ➖ : x = 6
x *= 2   # Multiply and assign ✖️ : x = 12
x /= 4   # Divide and assign ➗ : x = 3.0
x %= 2   # Modulus and assign 🧮 : x = 1.0
x //= 2  # Floor-divide and assign
x **= 3  # Exponentiate and assign
```

Assignment can also be stacked when multiple variables should reference the same value.

```python
a = b = c = 10

print(a)  # 10
print(b)  # 10
print(c)  # 10
```

Bitwise operators also have assignment versions:

```python
x &= 3   # Bitwise AND and assign
x |= 3   # Bitwise OR and assign
x ^= 3   # Bitwise XOR and assign
x <<= 1  # Left-shift and assign
x >>= 1  # Right-shift and assign
```

### Assignment Expression Operator

```python
items = ["a", "b", "c"]

if (count := len(items)) > 0:
    print(count)  # 3
```

`:=` is called the walrus operator. It assigns a value inside an expression.

## 7. Increment / Decrement Operators

Python does not have `++` or `--` operators.

```python
i = 0
i += 1  # Increment by 1
i -= 1  # Decrement by 1
```

## 8. Bitwise Operators

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

## 9. Ternary Expression

It is called **ternary** expression because the expression has three main parts:

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

## 10. Scope Resolution Operator

Python does not have a C++-style scope resolution operator like `::`.

Python resolves plain names using the **LEGB rule** and resolves attributes using dot notation, such as `module.name`, `Class.attribute`, and `object.attribute`.

See [Namespace and Scope.md](</Users/manpreetsingh/Downloads/Study/Notes/DSA/Python/Fundamentals/Namespace and Scope.md>) for `global`, `nonlocal`, class attributes, and instance attributes.
