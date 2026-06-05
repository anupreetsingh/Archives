# Hash Maps (Python's dict) — Structure & Collision Handling

## 1. Structure of a Hash Map

- A hash map stores key-value pairs.
- Keys are passed through a hash function -> produces an integer (hash code).
- That integer is mapped to an index in an internal array (the "bucket array").
- Buckets hold entries (key, value, and original hash).

In Python, `dict` is a highly optimized hash table implementation. It's built with:

- A dynamic array of slots (buckets).
- Each slot can either be:
    - Empty
    - Occupied with (key, value)
    - A "dummy" marker (from a deleted key)

Keys must be immutable & hashable because the hash value must not change.

```python
# Example
my_map = {}
my_map["apple"] = 5
my_map["banana"] = 10
print(hash("apple"))   # Under the hood, this decides the bucket location
```

## 2. Hash Collisions

Collision = Two different keys produce the same hash bucket index. Collisions are inevitable because:

- Infinite possible keys
- Finite number of buckets

Python uses **Open Addressing with Probing**:

- If a bucket is full, the algorithm searches the next slot(s) according to a probing sequence until it finds an empty one.

In CPython:

- Perturbation-based probing (kind of quadratic probing + randomization).
- Prevents clustering and spreads out colliding keys.

**Example:** Suppose "dog" and "cat" hash to the same index.

1. Insert "dog" -> goes to index i.
2. Insert "cat" -> index i is occupied -> probing -> finds next available slot.

## 3. Load Factor & Resizing

- Load factor = (# of elements) / (# of buckets)
- To keep operations efficient (~O(1) average), Python resizes (doubles) when load factor gets too high.
- Resizing involves rehashing all keys into a new larger bucket array.

## 4. Deletion

- When a key is deleted, Python doesn't leave a bucket completely empty.
- Instead, a "dummy marker" is left so probing still works correctly.
- This ensures that later lookups for colliding keys don't get cut off.

## 5. Complexity

Average case:

- Insert: O(1)
- Lookup: O(1)
- Delete: O(1)

Worst case (rare, with too many collisions or bad hash function):

- O(n)
