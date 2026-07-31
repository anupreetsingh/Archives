# Type Conversion in Python

Type conversion means creating a value of one type from a value of another type.

Since Python is strongly typed, there is no implicit type coercion for incompatible types. You need to manually convert one type to another using explicit **constructor function** calls. They are called constructors because they actually construct new objects of the desired type.

```python
# 2 + "5"  # TypeError

print(2 + int("5"))  # 7
print(str(2) + "5")  # "25"
```

## Numeric Conversions

### `int()`

`int()` creates an integer.

Form:

```python
int() # Default integer value = 0
int(value) 
int(value, base)
```

`value` is the thing being converted.

`base` is used with strings and tells Python what base the string-like `value` is written in. `base=10` is the default.

Example:

```python
print(int())              # 0
print(int(True))          # 1
print(int(False))         # 0
print(int(3.9))           # 3
print(int(-3.9))          # -3
print(int("42"))          # 42
print(int("101", 2))      # 5
print(int("10", 16))      # 16
print(int("FF", 16))      # 255
```

Important detail: `int(float_value)` truncates toward zero. It does not round.

```python
print(int(3.99))   # 3
print(int(-3.99))  # -3
```

Base `0` means Python should infer the base from the prefix:

```python
print(int("0b101", 0))  # 5, binary prefix
print(int("0o10", 0))   # 8, octal prefix
print(int("0x10", 0))   # 16, hexadecimal prefix
```

Invalid conversions raise errors:

```python
# int("3.5")      # ValueError, because this is float text, not int text
# int("hello")    # ValueError
# int([1, 2, 3])  # TypeError
```

### `float()`

`float()` creates a floating point number.

Form:

```python
float() # Default float value = 0.0
float(value)
```

`value` can be a number-like value or numeric string-like text.

Example:

```python
print(float())        # 0.0
print(float(5))       # 5.0
print(float(True))    # 1.0
print(float("3.14"))  # 3.14
print(float("1e3"))   # 1000.0
```

Important detail: `float()` can parse decimal text like `"3.14"`, while `int()` cannot parse `"3.14"` directly.

Invalid conversions raise errors:

```python
# float("hello")    # ValueError
# float("3,14")     # ValueError
# float([1, 2, 3])  # TypeError
```

### `complex()`

`complex()` creates a complex number with a real part and an imaginary part.

Form:

```python
complex() # Default complex value = 0j
complex(real)
complex(real, imag)
complex(value)
```

`value` can be number-like or string-like complex number text.

Example:

```python
print(complex())          # 0j
print(complex(3))         # (3+0j)
print(complex(3, 4))      # (3+4j)
print(complex("3+4j"))    # (3+4j)
```

## Text Conversion

### `str()`

`str()` creates a string representation of a value.

Form:

```python
str() # Default string value = ""
str(value)
```

`value` can be almost any object.

Example:

```python
print(str())            # ""
print(str(42))          # "42"
print(str(3.14))        # "3.14"
print(str(True))        # "True"
print(str([1, 2, 3]))   # "[1, 2, 3]"
```

Important detail: `str()` gives a display representation. It does not parse, clean, or decode data.

```python
data = b"hello"

print(str(data))        # "b'hello'"
print(data.decode())    # "hello"
```

Use `.decode()` when converting bytes that represent encoded text.

### Converting decimal to Other Number Systems

#### Using Number system constructors

The decimal int values can be converted into binary, octal and hexadecimal number systems as a string type

Use `bin()`, `oct()`, and `hex()` when you want Python's standard prefixes:

```python
number = 42

print(bin(number))  # "0b101010", binary
print(oct(number))  # "0o52", octal
print(hex(number))  # "0x2a", hexadecimal
```

The result is a string with a 2 letter prefix representing the kind of number system this string represents.

#### Using format

Use `format()` or an f-string when you want digits without the prefix:

```python
number = 42

print(format(number, "b"))  # "101010"
print(format(number, "o"))  # "52"
print(format(number, "x"))  # "2a"

print(f"{number:b}")        # "101010"
print(f"{number:o}")        # "52"
print(f"{number:x}")        # "2a"
```

Use a width when you want the result to take up a minimum number of characters:

Format pattern:

```python
format(value, "<fill><align><width><type>")
```

```python
number = 5

print(format(number, "08b"))  # "00000101", width is 8
print(format(number, "04x"))  # "0005", width is 4
```

Use alignment when you want to control which side gets padded:

```python
print(format(number, ">8b"))   # "     101", right aligned
print(format(number, "<8b"))   # "101     ", left aligned
```

Use a fill character before the alignment symbol when you want something other than spaces:

```python
print(format(number, "*>8b"))  # "*****101"
```

## Boolean Conversion

### `bool()`

`bool()` converts a value to its truth value.

Form:

```python
bool() # Default boolean value = False
bool(value)
```

Falsy values:

```python
False
None
0
0.0
""
[]
()
{}
set()
```

Most other values are truthy.

Example:

```python
print(bool())        # False
print(bool(0))       # False
print(bool(""))      # False
print(bool([]))      # False

print(bool(1))       # True
print(bool("False")) # True, because this is a non-empty string
print(bool([0]))     # True, because this is a non-empty list
```

Important detail: `bool("False")` is `True` because Python checks whether the string is empty. It does not parse the word `"False"` as the boolean value `False`.

## Collection Conversions

Collection constructors usually take an iterable.

An iterable is something Python can loop over, such as a string, list, tuple, set, dictionary, range object, or generator.

### `list()`

`list()` creates a list.

Form:

```python
list() # Default list value = []
list(iterable)
```

`iterable` is something Python can loop over. Each item becomes an element in the new list.

Example:

```python
print(list())              # []
print(list("abc"))         # ['a', 'b', 'c']
print(list((1, 2, 3)))     # [1, 2, 3]
print(list({1, 2, 3}))     # [1, 2, 3], order may vary
print(list(range(3)))      # [0, 1, 2]
print(list({"a": 1, "b": 2}))  # ['a', 'b'], dictionaries iterate over keys
```

Invalid conversions raise errors:

```python
# list(5)  # TypeError, because 5 is not iterable
```

### `tuple()`

`tuple()` creates a tuple.

Form:

```python
tuple() # Default tuple value = ()
tuple(iterable)
```

`iterable` is something Python can loop over. Each item becomes an element in the new tuple.

Example:

```python
print(tuple())          # ()
print(tuple("abc"))     # ('a', 'b', 'c')
print(tuple([1, 2, 3])) # (1, 2, 3)
print(tuple(range(3)))  # (0, 1, 2)
```

### `set()`

`set()` creates a set.

Form:

```python
set() # Default set value = set()
set(iterable)
```

`iterable` is something Python can loop over. Each unique item becomes an element in the new set.

Example:

```python
print(set())              # set()
print(set("banana"))      # {'b', 'a', 'n'}, order may vary
print(set([1, 2, 2, 3]))  # {1, 2, 3}
```

Important detail: sets remove duplicates and do not preserve a dependable order.

### `dict()`

`dict()` creates a dictionary.

Form:

```python
dict() # Default dictionary value = {}
dict(mapping)
dict(iterable_of_pairs)
dict(key=value)
```

`mapping` means another dictionary-like object. `iterable_of_pairs` means each item must contain exactly one key and one value.

Example:

```python
print(dict())                         # {}
print(dict({"a": 1, "b": 2}))          # {'a': 1, 'b': 2}
print(dict([("a", 1), ("b", 2)]))      # {'a': 1, 'b': 2}
print(dict(a=1, b=2))                  # {'a': 1, 'b': 2}
```

Invalid conversions raise errors:

```python
# dict([1, 2, 3])      # TypeError, each item must be a key-value pair
# dict([("a", 1, 2)])  # ValueError, each pair must have exactly 2 values
```

## Common Parsing Pattern

User input from `input()` is always a string, so numeric input usually needs explicit conversion.

```python
age_text = input("Age: ")
age = int(age_text)

price_text = input("Price: ")
price = float(price_text)
```

For multiple numbers on one line:

```python
nums = list(map(int, input().split()))
```

How this works:

1. `input()` reads a full line as a string.
2. `.split()` breaks the string into a list of smaller strings.
3. `map(int, ...)` applies `int()` to each string.
4. `list(...)` stores the converted integers in a list.

Example:

```python
# Input line: 10 20 30

nums = list(map(int, "10 20 30".split()))

print(nums)  # [10, 20, 30]
```
