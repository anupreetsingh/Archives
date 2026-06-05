# Tuples in Python

A tuple is an immutable sequence type, meaning once created, it cannot be changed (no adding, removing, or modifying elements).

- Tuples are defined by comma-separated values, usually enclosed in parentheses `()`, but not necessarily.
- They can hold elements of different data types (integers, strings, lists, etc.).

## Creating tuples

```python
empty_tuple = ()                      # An empty tuple
single_element_tuple = (5,)           # Tuple with one element — note the trailing comma!
multi_element_tuple = (1, 2, 3, 4)   # Tuple with multiple elements

# Tuples can be created without parentheses as well (tuple packing):
packed_tuple = 10, 20, 30
```

## Accessing tuple elements (indexing and slicing)

```python
print(multi_element_tuple[0])   # Output: 1 (index starts at 0)
print(multi_element_tuple[-1])  # Output: 4 (negative index counts from the end)
print(multi_element_tuple[1:3]) # Output: (2, 3) (slice from index 1 up to but not including 3)
```

## Iteration

Tuples support iteration:

```python
for item in multi_element_tuple:
    print(item)
```

## Immutability

Since tuples are immutable, operations that modify contents will raise an error:

```python
# multi_element_tuple[0] = 10  # This will cause a TypeError!
```

Tuples can be used as keys in dictionaries because they are hashable (unlike lists).

## Unpacking tuples

```python
a, b, c = (1, 2, 3)   # Assign elements of tuple to variables
a, b, c = 1, 2, 3     # Same thing without parantheses
print(a, b, c)       # Output: 1 2 3
```

Tuples are often used for fixed collections of heterogeneous data, e.g., coordinates, database records, etc.

## Summary

- Immutable ordered collection
- Can contain mixed data types
- Defined with parentheses and commas
- Supports indexing, slicing, iteration
- Useful for fixed-size, read-only data structures
