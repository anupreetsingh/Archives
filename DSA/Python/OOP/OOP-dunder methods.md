# Common Dunder Functions in Python OOP

- Dunder functions are methods with double underscores before and after their name.
- `__init__` and `__str__` are two common examples, but there are many others.
- These methods let custom objects work with normal Python syntax.

**KEY IDEA: Dunder functions customize object behavior** — Example: object creation, printing, comparison, addition, length, indexing, and function calls.

---

## Most Common Dunder Functions

| Method | What It Controls | Example Trigger | Trigger Meaning |
|---|---|---|---|
| `__init__` | Initializes object attributes | `Student("Aman", 21)` | Creating an instance of the class |
| `__str__` | User-friendly string output | `print(obj)` | Printing the instance for users |
| `__repr__` | Developer-friendly string output | `repr(obj)` | Inspecting the instance for debugging |
| `__len__` | Length of an object | `len(obj)` | Asking how many items the object represents |
| `__eq__` | Equality comparison | `obj1 == obj2` | Checking whether two objects should count as equal |
| `__lt__` | Less-than comparison | `obj1 < obj2` | Checking whether one object should come before another |
| `__gt__` | Greater-than comparison | `obj1 > obj2` | Checking whether one object should come after another |
| `__add__` | Addition behavior | `obj1 + obj2` | Adding or combining two objects |
| `__sub__` | Subtraction behavior | `obj1 - obj2` | Subtracting one object from another |
| `__getitem__` | Indexing or key access | `obj[index]` | Reading an item from the object |
| `__setitem__` | Assigning by index or key | `obj[index] = value` | Updating an item inside the object |
| `__contains__` | Membership checking | `value in obj` | Checking whether the object contains a value |
| `__call__` | Makes an instance of the class callable like a function | `obj()` | Calling the instance like a function |
| `__iter__` | Makes an object iterable | `for item in obj` | Looping over the object |
| `__next__` | Gets the next item from an iterator | `next(obj)` | Getting the next value during iteration |
| `__enter__` | Starts a context manager | `with obj:` | Entering a `with` block |
| `__exit__` | Ends a context manager | `with obj:` | Leaving a `with` block |

## Basic Example

```python
class Book:
    def __init__(self, title, pages):
        self.title = title
        self.pages = pages

    def __str__(self):
        return f"{self.title} has {self.pages} pages"

    def __len__(self):
        return self.pages

    def __eq__(self, other):
        return self.title == other.title and self.pages == other.pages


book1 = Book("Python Basics", 250)
book2 = Book("Python Basics", 250)

print(book1)
print(len(book1))
print(book1 == book2)
# Output:
# Python Basics has 250 pages
# 250
# True
```

## Important Categories

### Object Setup

- `__init__`: sets up object attributes when the object is created.

### String Representation

- `__str__`: readable output for users.
- `__repr__`: more detailed output for developers/debugging.

### Comparison

- `__eq__`: equal to
- `__lt__`: less than
- `__gt__`: greater than
- `__le__`: less than or equal to
- `__ge__`: greater than or equal to

### Math Operators

- `__add__`: addition with `+`
- `__sub__`: subtraction with `-`
- `__mul__`: multiplication with `*`
- `__truediv__`: division with `/`

### Container Behavior

- `__len__`: length using `len()`
- `__getitem__`: access items using square brackets
- `__setitem__`: update items using square brackets
- `__contains__`: membership using `in`

### Callable and Iterable Behavior

- `__call__`: lets an object behave like a function.
- `__iter__`: lets an object be used in a loop.
- `__next__`: returns the next value during iteration.

## Why They Matter

- They make your custom classes feel like built-in Python objects.
- They allow Python syntax to work with your own classes.
- They are commonly used when building cleaner, more Pythonic OOP code.
