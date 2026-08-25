# Hash Maps Structure

It is a **non-sequence type**, specifically a **mapping type**: values are accessed by hashable keys, not by integer positions.

A hash map stores **key-value pairs** and uses a key's hash value to decide where the pair should live internally.

In Python, a hash map is implemented by built-in `dict` type and called a dictionary.

```python
student_scores = {
    "Ava": 95,
    "Leo": 88,
    "Mia": 91
}
```

Conceptually, a hash map has an internal array of slots:

```text
index:  0      1      2      3      4      5
slots: [ ]    [ ]    [ ]    [ ]    [ ]    [ ]
```

Each stored entry contains:

```text
key, value, hash
```

The hash is stored so the dictionary does not always need to recompute it during lookup or collision checks.

## Hashing a Key

A key is passed into a hash function to get a unique value that is specific to the key.

To make that hash output fit into the Hash Table:

```python
index = hash("Ava") % table_size
```

Keys must be **hashable** because the dictionary needs a stable hash value.

### Hashable key examples

```python
class Person:
    pass

p = Person()

valid_keys = {
    "name": "string key",
    10: "integer key",
    3.14: "float key",
    True: "boolean key",
    None: "None key",
    (1, 2): "tuple key",
    frozenset({1, 2, 3}): "frozenset key",
    p: "custom object key"
}
```

Default custom objects are hashable because they compare by identity and their default hash is based on identity.

### Non-hashable key examples

```python
invalid_keys = {
    [1, 2]: "list key",        # TypeError
    {"a": 1}: "dict key",      # TypeError
    {1, 2, 3}: "set key"       # TypeError
}
```

Lists, dictionaries, and sets are not valid dictionary keys because they are mutable. If a key could change after insertion, its hash location could become incorrect.

A tuple is hashable only if all values inside it are hashable.

```python
hash((1, 2))        # works
hash(("a", 10))     # works
hash(([1, 2], 3))   # TypeError, because the tuple contains a list
```

## Collision Handling

A collision happens when two different keys with different hash output map to the same index because the `%` tries to make the index land within the table size

```text
10 % 100 -> 10
110 % 100 -> 100
```

There are two common ways to handle collisions in a hash map:

### Separate chaining

Separate chaining means each array slot in the hash table points to a separate data structure, most commonly a linked list or dynamic array.

If two keys map to the same index, they are stored in the same **bucket** attached to that index. The bucket holds all entries for that index, while the main table slot only points to the bucket.

```text
index:  0      1      2
slots: [ ]    [ ]   [("Ava", 95), ("Leo", 88)]
```

### Open Addressing

Open addressing means all entries are stored directly inside the main table.

Python dictionaries use open addressing.

```text
index:  0      1      2      3       4      5      6      7
slots: [ ]    [ ]    [ ]    Ava     Leo    [ ]    [ ]    [ ]
```

If the desired slot is already occupied, the dictionary searches for another open slot in the same table. The process of searching for another slot is called **probing**.

#### Probing

**linear probing**: Keep on checking the next slot in case of collision until you find an unoccupied one.

```text
3 -> 4 -> 5 -> 6
```

**perturbation-based probing**: Advanced probing technique used by python where it starts by using original hash value to generate a spread-out sequence of possible slots.

Conceptually:

```python
perturb = hash_value

while slot_is_occupied_by_a_different_key:
    index = formula_using(index, perturb)
    perturb >>= 5 # Right shifting pert to use a different value for next index calculation
```

The goal is to avoid too many collided keys clustering in the same nearby area.

## Lookup

Lookup follows the same path as insertion.

```python
student_scores["Ava"]
```

```text
1. Hash the key
2. Convert hash to starting index
3. Keep probing until the key matches
```

This works because insertion and lookup use the same hash and probing sequence.

## Load Factor & Resizing

Load factor measures how full the hash table is.

```python
load_factor = number_of_entries / number_of_slots
```

Example:

```text
entries = 6
slots = 8

load factor = 6 / 8 = 0.75
```

As the table gets fuller:

- collisions become more likely
- probing chains get longer
- lookup, insertion, and deletion become slower

To keep operations close to `O(1)` average time, hash maps resize when the table gets too full.

**Resizing** means creating a larger internal table and placing the existing keys into the new table.

```text
old table size: 8
new table size: 16
```

The keys need to be placed again because the index depends on the table size.

```python
old_index = hash(key) % 8
new_index = hash(key) % 16
```

The hash value may be the same, but the final index can change because the table size changed.

Resizing is expensive when it happens, but it does not happen on every insertion. This is why dictionary operations are still `O(1)` on average.

## Deletion

Deleting from an open-addressed hash map needs special handling.

Suppose `"Ava"` and `"Leo"` collided, and `"Leo"` was placed after probing.

```text
index:  0      1      2      3       4      5
slots: [ ]    [ ]    [ ]   Ava     Leo    [ ]
```

If `"Ava"` is deleted and slot `3` is made completely empty:

```text
index:  0      1      2      3       4      5
slots: [ ]    [ ]    [ ]    [ ]    Leo    [ ]
```

Now looking up `"Leo"` can break.

The lookup starts at index `3`. If it sees an empty slot, it may conclude that `"Leo"` was never inserted, even though `"Leo"` is at index `4`.

So Python leaves a special marker called a **dummy marker** or **tombstone**.

```text
index:  0      1      2      3        4      5
slots: [ ]    [ ]    [ ]  dummy     Leo    [ ]
```

The dummy marker means:

```text
This slot used to contain something, so keep probing during lookup.
```

During insertion, a dummy slot can be reused.

So deletion is:

```text
1. Find the key using the normal lookup/probing path
2. Remove the key-value pair
3. Leave a dummy marker so future lookups do not stop too early
```

## Time Complexity

Average case:

```text
lookup:    O(1)
insert:    O(1)
delete:    O(1)
```

Worst case:

```text
lookup:    O(n)
insert:    O(n)
delete:    O(n)
```

The worst case can happen if many keys collide or if probing chains become very long.

In practice, Python dictionaries are designed to keep operations very close to `O(1)` average time by using:

- hashing
- open addressing (Using probing policies like linear, quadratic, double hash or perturbation-based probing) or separate chaining.
- resizing based on load factor
- Lazy deletion

General hash maps can also use separate chaining, but Python's `dict` specifically uses open addressing.
