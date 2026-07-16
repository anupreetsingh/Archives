# List Structure in Python

A Python list is a **sequence type**, meaning it is an ordered collection where elements can be accessed by position.

```python
nums = [10, 20, 30]

nums[0]  # 10
nums[1]  # 20
```

## Internal Storage

Although it may feel like the elements in a Python list are stored contiguously, the actual element objects are not stored directly inside the list.

Internally, a Python list stores a contiguous **array of references**. Each reference points to an object somewhere in memory. When we access `my_list[i]`, Python uses the index to access the reference at position `i`, then follows that reference to the actual object.

The internal array is homogeneous because every slot stores the same kind of thing: a reference to a Python object.

The list itself can still appear heterogeneous because those references can point to objects of different types.

```python
items = [10, "hi", [1, 2], True]
```

Conceptually:

```text
list object
  |
  v
contiguous array of object references

index 0 -> int object 10
index 1 -> str object "hi"
index 2 -> list object [1, 2]
index 3 -> bool object True
```

## Size vs Capacity

A list has two important ideas:

- `size`: how many elements are currently in the list.
- `capacity`: how many reference slots are currently allocated internally

That is why `len()` is an `O(1)` operation for lists: it returns the list's stored `size` value instead of counting the elements one by one.

Example:

```python
nums = [10, 20, 30]
```

Conceptually:

```text
size = 3
capacity = 6

[ptr][ptr][ptr][empty][empty][empty]
```

The exact capacity is an implementation detail, but the core idea is that Python usually allocates more space than the list currently needs.

This is called **over-allocation**.

## Appending to a List

When there is extra capacity, `append()` places the new element reference into the next empty slot.

```python
nums.append(40)
```

Conceptually:

```text
before:
[ptr][ptr][ptr][empty][empty][empty]

after:
[ptr][ptr][ptr][ptr][empty][empty]
```

This is `O(1)`.

## Resizing a List

When the list runs out of capacity, Python cannot keep extending the same block forever.

Instead, it:

1. Allocates a new larger contiguous array of references.
2. Copies the old references into the new array.
3. Adds the new element reference.
4. Frees the old reference array.

Conceptually:

```text
old array:
[ptr][ptr][ptr][ptr]

new larger array:
[ptr][ptr][ptr][ptr][ptr][empty][empty]
```

The actual element objects are not copied. Only the references are copied.

This is why `append()` is `O(1)` **amortized**:

- Most appends are cheap because there is already empty capacity.
- Some appends are expensive because Python must resize and copy references.
- Averaged over many appends, the cost is still treated as `O(1)`.

## List Resizing vs Dictionary Resizing

List resizing is conceptually similar to hash map resizing because both structures grow their internal storage when they become too full.

But the work is different.

For a list:

```text
old pointer array too small
-> allocate larger contiguous pointer array
-> copy old element references in the same order
-> append new reference
-> free old pointer array
```

The index positions stay the same.

For a dictionary:

```text
hash table too full
-> allocate a larger hash table
-> redistribute key-value entries based on hash values and new table size
-> free old table
```

Dictionary entries may land in different internal slots after resizing because slot placement depends on the table size.

```python
index = hash(key) % table_size
```

When `table_size` changes, the calculated index can change.
