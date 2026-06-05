# 📌 Sets in Python

✅ Internally, a set in Python is backed by a hash table just like a Dictionary. In fact, a set can be thought of as a dictionary with only keys and no values. Every item added to a set is hashed, and stored in a way that allows:

- very fast lookup (O(1) average case)
- automatic elimination of duplicates

Under the hood:

```text
set: {'a', 'b', 'c'}    ≈    dict: {'a': True, 'b': True, 'c': True}
```

---

## 🔧 Creating a Set

```python
# Using curly braces
my_set = {1, 2, 3, 4}

# Using the set() constructor (useful when creating an empty set)
empty_set = set()  # {} is an empty dict, NOT a set

# Duplicates are automatically removed
set_with_duplicates = {1, 2, 2, 3, 4, 4}  # {1, 2, 3, 4}
```

## 🛠 Common Set Functions

```python
# Add an item
my_set.add(5)  # {1, 2, 3, 4, 5}

# Remove an item (raises KeyError if item not present)
my_set.remove(3)

# Discard an item (safe to use if not sure item exists)
my_set.discard(10)  # No error if 10 isn't in the set

# Clear all elements
my_set.clear()  # my_set is now an empty set

# Copying a set
new_set = {10, 20, 30}
copied_set = new_set.copy()  # A shallow copy of the set
```

## 🔍 Set Membership Testing

```python
numbers = {1, 2, 3, 4}
print(3 in numbers)     # True
print(5 not in numbers) # True
```

## 🔄 Set Operations (like in math)

```python
a = {1, 2, 3}
b = {3, 4, 5}

# Union (all unique elements from both)
print(a.union(b))      # {1, 2, 3, 4, 5}

# Intersection (common elements)
print(a.intersection(b))  # {3}

# Difference (items in a but not in b)
print(a.difference(b))    # {1, 2}

# Symmetric Difference (items in a or b, but not both)
print(a.symmetric_difference(b))  # {1, 2, 4, 5}
```

## 🧪 Looping through a Set

```python
for item in a:
    print(item)  # Order is not guaranteed
```

---

## ✅ Summary

- Sets are built on hash tables → fast lookup, no duplicates
- Very similar to dictionaries without values
- Great for membership testing, deduplication, and set algebra
