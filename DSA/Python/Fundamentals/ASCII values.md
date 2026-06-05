# ASCII Notes and Functions in Python

ASCII (American Standard Code for Information Interchange) is a character encoding standard. It assigns a unique integer (0 to 127) to represent characters such as letters, digits, and symbols.

Example ASCII values:

```text
'A' -> 65
'a' -> 97
'0' -> 48
'&' -> 38
```

## `ord()` function

`ord` comes from the ordinal which means position of something in a sorted sequence. Takes a single character as input and returns its ASCII (Unicode) integer value.

```python
print(ord('A'))  # Output: 65
print(ord('&'))  # Output: 38
print(ord('z'))  # Output: 122
```

## `chr()` function

Takes an integer ASCII value and returns the corresponding character.

```python
print(chr(65))   # Output: 'A'
print(chr(38))   # Output: '&'
print(chr(122))  # Output: 'z'
```

## Important notes

- `ord()` only accepts a single character string.
- ASCII covers characters from 0 to 127; Python's `ord()` works with Unicode values.
- `chr()` converts integers back to characters (within valid range).

## Examples

```python
# Example: Printing all lowercase letters using ASCII values
for i in range(97, 123):  # ASCII values for 'a' to 'z'
    print(chr(i), end=' ')  # Output: a b c d ... z

# Example: Calculate difference between two characters using ASCII values
diff = ord('z') - ord('a')
print("\nDifference between 'z' and 'a':", diff)  # Output: 25
```
