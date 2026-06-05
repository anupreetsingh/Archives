# Difference Between List and Array in Python

## Array

- A data structure that stores elements of the same data type.
- Uses contiguous memory allocation.
- Allows fast and efficient indexing (O(1) time).
- More memory-efficient and better for numerical operations.
- Example: `array.array` or `numpy.array` in Python

```python
import array
arr = array.array('i', [1, 2, 3, 4])  # 'i' stands for signed integer
```

## List

- A built-in Python data structure that can store mixed data types.
- Internally stores references/pointers, not raw values.
- More flexible but slower for numerical computation.
- Not memory efficient for large homogeneous data.

```python
lst = [1, "two", 3.0, [4]]  # stores mixed types
```

## Summary

- Use arrays for performance and memory efficiency with numeric data.
- Use lists for general-purpose storage and flexibility.
