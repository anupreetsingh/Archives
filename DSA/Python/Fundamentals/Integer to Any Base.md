# Integer to Any Base (between 2 and 36)

Code to convert an integer to its base of choice (between 2 and 36).

By dividing a `val` with `base`, you basically rightshift the entire `val` by one place. For example `divmod(437,10)=(43, 7)`.

If the base is different from the original base:

- The remainder (`val % base`) is the LSB (rightmost digit) of `val` in original base form, represented as LSB in new base.
- The Quotient (`val // base`) is whatever number remains from `val` after division (taking out LSB) represented in the original base form, we assign this as the updated `val` for next iteration.

---

## Function 1: Convert integer (Base 10) to any base between 2 and 10

```python
def int_to_integer_base(n, base):
    res="" # Empty string to return result
    val=n
    while val>0:
        val, digit=divmod(val,base)
        res=str(digit)+res # Prepending digit to string
    return res

print(int_to_integer_base(10,2))
print(int_to_integer_base(17,2))
print(int_to_integer_base(24,2))
```

## Function 2: Convert integer (Base 10) to any base between 2 and 36

```python
def int_to_any_base(n,base):
    res=""
    val=n
    digits="0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ" # Remainder will be used as index to see which digit it refers to
    while val>0:
        val, idx=divmod(val, base)
        digit=digits[idx]
        res=digit+res # Prepending digit to string
    return res

print(int_to_any_base(10,14))
print(int_to_any_base(255,16))
print(int_to_any_base(24,2))
```

## Function 3: Convert any base (2–36) back to integer (Base 10)

```python
def any_base_to_int(seq, base): # Sequence to base converted, The base it is in
    digits="0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ" # Index in this tells the value in base 10
    val=0 # Start from zero
    for c in seq:
        digit= digits.index(c) # Value of character in Integer(Base-10)
        val=val*base + digit # We left shift the current value by multiplying it with base and then add the value of the current character, and update val for next iteration
    return val

print(any_base_to_int("A",14)) # 10
print(any_base_to_int("FF",16)) # 255
print(any_base_to_int("11000",2)) # 24
```
