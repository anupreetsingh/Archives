# String Manipulation in Python

## Common String Methods

In Python Strings are immutable, so any function working on a string would always returns a new string object, and we often reassign that to a variable and not update the string in-place.

Time complexities below are worst-case bounds, with `n` as the input string's length and fixed-length string arguments unless stated otherwise.

```python
text = "Hello World"
```

### 1. `.split()` → Splits a string into a list using a delimiter (default is whitespace)

**Time complexity:** `O(n)` — scans the string and creates the resulting substrings.

```python
sentence = "apple banana cherry"
words = sentence.split()
print(words)        # ['apple', 'banana', 'cherry']

csv_line = "a,b,c"
parts = csv_line.split(",")
print(parts)        # ['a', 'b', 'c']
```

### 2. `.join()` → Joins string elements of an iterable into one string

**Time complexity:** `O(k + L)`, where `k` is the number of strings and `L` is the output length, including separators; excludes work needed to generate or convert the input elements.

Syntax: `"<separator>".join(iterable)`

`.join()` combines the string elements of an iterable into one string, placing the separator between each element.

```python
joined = "-".join(["2025", "07", "16"])
print(joined)       # '2025-07-16'
```

If any element in the iterable is not a string, Python raises a `TypeError`.

```python
values = ["2025", 7, 16]

joined = "-".join(values)
print(joined)       # TypeError
```

To avoid this, convert each element to a string first:

```python
values = ["2025", 7, 16]

joined = "-".join(str(value) for value in values)
print(joined)       # '2025-7-16'
```

### 3. `.replace()` → Replaces occurrences of a substring

**Time complexity:** `O(n)` for fixed-length search and replacement strings — scans for matches and builds the result.

`.replace()` replaces **all non-overlapping occurrences** by default, scanning from left to right. Pass a third argument, `count`, to limit the number of replacements; `1` replaces only the first occurrence.

```python
repeated_text = "World World World"

print(repeated_text.replace("World", "Python"))     # 'Python Python Python'
print(repeated_text.replace("World", "Python", 1))  # 'Python World World'

# Overlapping matches are not replaced:
print("aaa".replace("aa", "X"))                     # 'Xa'
```

In `"aaa"`, `"aa"` matches at positions `0–1` and `1–2`, but these overlap. Once the first match is replaced, its characters cannot participate in another match.

### 4. `.lower()`, `.upper()`, `.capitalize()`, `.title()`, `.swapcase()`

**Time complexity:** `O(n)` for all five — processes the characters and creates the result.

These methods change letter case; characters without uppercase/lowercase forms, such as digits, spaces, and punctuation, remain unchanged.

Returns a new string to be passed into the print function, doesn't do in-place update. The strings made this way but not assigned to a variable are eligible for garbage collection, so python deletes them based on memory requirements.

```python
print(text.lower())       # 'hello world'
print(text.upper())       # 'HELLO WORLD'
print(text.capitalize())  # 'Hello world'
print(text.title())       # 'Hello World'
print(text.swapcase())    # 'hELLO wORLD'
```

### 5. `.strip()`, `.lstrip()`, `.rstrip()` → Remove whitespace (or characters) from edges

**Time complexity:** `O(n)` for all three — scans the relevant edges and may copy the remaining characters into a new string.

With no argument, these methods remove whitespace, including spaces, tabs (`\t`), and newlines (`\n`). `.strip()` removes it from both ends, `.lstrip()` from the left, and `.rstrip()` from the right.

Pass a string argument to remove any of its characters from the corresponding edges instead. The argument is treated as a **set of individual characters**, not an exact substring; removal stops at the first character outside that set at each edge. Characters inside the string are left unchanged.

```python
messy = "  hello  "
print(messy.strip())      # 'hello'
print(messy.lstrip())     # 'hello  '
print(messy.rstrip())     # '  hello'

messy = "--__hello__--"
print(messy.strip("-_"))   # 'hello'
print(messy.lstrip("-_"))  # 'hello__--'
print(messy.rstrip("-_"))  # '--__hello'
```

### 6. `.startswith()`, `.endswith()`

**Time complexity:** `O(m)` for both, where `m` is the length of a single prefix or suffix being checked — compares only the relevant edge.

```python
print(text.startswith("Hel"))  # True
print(text.endswith("ld"))     # True
```

### 7. `.find()`, `.rfind()`, `.index()`, `.count()`

**Time complexity:** `O(n)` for all four with a fixed-length search string — may scan the entire input.

```python
print(text.find("o"))      # 4 (first occurrence)
print(text.rfind("o"))     # 7 (last occurrence)
# .find() returns -1 when the element is not found.
# .index() is like find() but raises ValueError if not found
print(text.count("l"))     # 3 (number of occurrences)
# `.count()` counts only non-overlapping occurrences: `"aaaa".count("aa")` returns `2`, not `3`.
```

### 8. `.zfill()` → Left-pads a string with zeros

**Time complexity:** `O(max(n, width))` — when padding is needed, creates an output of length `width` by adding zeros and copying the original characters.

`.zfill(width)` returns a new string padded with leading zeros until the string reaches `width` characters.

```python
print("42".zfill(5))    # "00042"
print("123".zfill(2))   # "123", already at least 2 characters
print("-42".zfill(5))   # "-0042", sign stays at the front
```

Use `.zfill()` when the fill character is specifically zero. Use `format()` or an f-string when you need different fill characters, alignment, numeric formatting, or base conversion.

---

## f-strings (formatted strings)

```python
name = "Anu"
age = 24
print(f"My name is {name} and I am {age} years old.")  # 'My name is Anu and I am 24 years old.'
```

## String Concatenation

```python
greeting = "Hello"
greeting=greeting + " Anu," + " How are you?" # Concatenation also Creates a new string object, and then  reassigns reference to the variable "greeting"
print(greeting)     # 'Hello World'
```

## Repetition with `*`

For Strings the `*` operator creates a new string by repeating the original string n times. This is not shallow copy since strings are just one immutable object and not containers of mutable element references.

```python
print("ha" * 3)      # 'hahaha'
```

## Check membership

```python
print("e" in text)   # True
print("z" not in text)  # True
```
