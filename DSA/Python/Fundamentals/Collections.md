# Collections Module in Python

`collections` is a standard library module in python that provides specialized container data types. Named `collections` because its gives useful ways to **collect**, organize, and access groups of values.

A **container** is an object that stores other objects. Python already has built-in containers like `list`, `tuple`, `dict`, and `set`, but the `collections` module provides specialized containers that are better suited for specific tasks and reduce the boilerplate needed to do the same thing with built-in containers.

Many of these specialized containers can still be recreated using built-in data types, but the implementation usually becomes more complex and requires more custom logic. That is why it is usually better to use the container types from the `collections` module when they match the problem you are solving.

Example import:

```python
from collections import Counter, defaultdict, deque, namedtuple, OrderedDict, ChainMap
```

## `defaultdict`

`defaultdict` is a dictionary-like type that automatically creates a default value when a missing key is accessed.

`defaultdict` takes a function. When a key is missing, Python runs that function and uses the returned value as the default value:

```python
defaultdict(list)   # calls list(), so a missing key starts as []
defaultdict(int)    # calls int(), so a missing key starts as 0
defaultdict(float)  # calls float(), so a missing key starts as 0.0
defaultdict(set)    # calls set(), so a missing key starts as set()
defaultdict(str)    # calls str(), so a missing key starts as ""

# Adding custom value
defaultdict(lambda: 23)  # calls lambda: 23, so a missing key starts as 23
```

Example:

```python
from collections import defaultdict

word_counts = defaultdict(int)

words = ["apple", "banana", "apple"]

for word in words:
    word_counts[word] += 1

print(word_counts)
# defaultdict(<class 'int'>, {'apple': 2, 'banana': 1})
```

Since we use `defaultdict`,  the incrememnt is just cleaner and more natural but we could achieve the same thing with a normal `dict` using `.get()`:

```python
word_counts = {}

words = ["apple", "banana", "apple"]

for word in words:
    word_counts[word] = word_counts.get(word, 0) + 1

print(word_counts)
# {'apple': 2, 'banana': 1}
```

Here, `word_counts.get(word, 0)` gets the current count for `word`. If `word` is not in the dictionary yet, it uses `0` instead.

## `Counter`

`Counter` is a dictionary-like type used for counting how many times each value appears.

It stores each unique item from the original iterable as a key and the number of times that item appears as the value. This is useful when you need a frequency table, such as counting characters, words, numbers, or categories.

Example:

```python
from collections import Counter

letters = ["a", "b", "a", "c", "b", "a"]

counts = Counter(letters)

print(counts)       # Counter({'a': 3, 'b': 2, 'c': 1})
print(counts["a"])  # 3
print(counts["z"])  # 0
```

Here, `Counter` counts how many times each letter appears.

Unlike a normal dictionary, asking for a missing key returns `0` instead of raising a `KeyError`.

We could do the same counting with a normal dictionary, but then we would have to manually loop through the iterable and update each count ourselves:

```python
letters = ["a", "b", "a", "c", "b", "a"]

counts = {}

for letter in letters:
    counts[letter] = counts.get(letter, 0) + 1

print(counts)       # {'a': 3, 'b': 2, 'c': 1}
print(counts["a"])  # 3
```

Here, `.get(letter, 0)` gets the current count for `letter`. If `letter` is not in the dictionary yet, it starts from `0`.

`Counter` is useful because it handles this counting loop for us.

## `deque`

`deque` stands for **double-ended queue**.

It is a list-like container optimized for adding and removing items from both the left end and the right end.

This makes it useful for queues, stacks, sliding windows, and BFS.

Example:

```python
from collections import deque

items = deque(["b", "c"])

items.append("d")
items.appendleft("a")

print(items)  # deque(['a', 'b', 'c', 'd'])

right_item = items.pop()
left_item = items.popleft()

print(right_item)  # d
print(left_item)   # a
print(items)       # deque(['b', 'c'])
```

The most common `deque` operations are `append()`, `appendleft()`, `pop()`, and `popleft()`. These let us add and remove items from either end of the container.

## `namedtuple`

`namedtuple` creates a new tuple type whose positions also have names.

Syntax:

```python
TypeName = namedtuple("TypeName", ["field1", "field2"])
```

The first argument, `"TypeName"`, is the name of the new tuple type.

The second argument, `["field1", "field2"]`, gives names to each position in the tuple.

The variable on the left side is bound to the new tuple type. After that, use that variable to create actual tuples that conform to the namedtuple.

It is useful when you want immutable records with predefined named elements, so their values are easier to access by name instead of just by index.

Example:

```python
from collections import namedtuple

Point = namedtuple("Point", ["x", "y"]) # Defines a namedtuple type

p = Point(3, 4) # Makes a tuple that conforms to that namedtuple type

print(p[0])  # 3
print(p.x)   # 3
print(p.y)   # 4
```

Here, `Point` is the new tuple type, and `p` is a named tuple value created from that type.

The value `3` goes into the `x` field, and the value `4` goes into the `y` field.

`p` behaves like a tuple, so indexing still works. But it also supports named fields like `p.x` and `p.y`, which makes the code clearer.

## `OrderedDict`

Dictionaries using the built in `dict` type preserve insertion order.

`OrderedDict` is a dictionary-like type that preserves insertion order and gives us extra order-based operations that a normal dictionary does not provide.

Common order-based operations:

```python
recent.move_to_end("home")              # moves "home" to the end
recent.move_to_end("home", last=False)  # moves "home" to the front
recent.popitem()                        # removes and returns the last item
recent.popitem(last=False)              # removes and returns the first item
```

Example:

```python
from collections import OrderedDict

recent = OrderedDict()

recent["home"] = "Home Page"
recent["search"] = "Search Page"
recent["profile"] = "Profile Page"

recent.move_to_end("home")

print(recent)
# OrderedDict([('search', 'Search Page'), ('profile', 'Profile Page'), ('home', 'Home Page')])

oldest = recent.popitem(last=False)

print(oldest)
# ('search', 'Search Page')

print(recent)
# OrderedDict([('profile', 'Profile Page'), ('home', 'Home Page')])
```

## `ChainMap`

`ChainMap` groups multiple dictionaries together and searches them as one combined mapping.

It does not merge the dictionaries into a new dictionary. Instead, it keeps references to the original dictionaries and checks them from left to right.

Example:

```python
from collections import ChainMap

user_settings = {"theme": "dark"}
default_settings = {"theme": "light", "font_size": 14}

settings = ChainMap(user_settings, default_settings)

print(settings["theme"])      # dark
print(settings["font_size"])  # 14
```

Here, Python first checks `user_settings`. If a key is not found there, it checks `default_settings`.

If the key is not found in any dictionary inside the `ChainMap`, Python raises a `KeyError`.

This is useful for configuration systems where user-provided values should override defaults.
