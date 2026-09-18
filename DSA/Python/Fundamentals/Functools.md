# Functools Module in Python

`functools` is a module in standard-python library containing tools that work with functions and other callables. Its name means **function tools**. These tools adapt how functions are called, cache their results, combine values, and customize comparisons.

## Custom Comparator in Sorting

### Normal Key

Both `sorted()` and `list.sort()` use comparisons to determine the order of elements. By default, they compare the elements themselves. When you provide a `key` function, they call it once for each element and compare the returned values instead.

For example, this key function converts each string to an integer so that the strings are sorted by their numeric values:

```python
parts = ["30", "003", "34", "033"]

ordered = sorted(parts, key=lambda part: int(part))

print(ordered)  # ["003", "30", "033", "34"]
```

The key function receives **one element** and returns its comparison value. Here, `"30"` gets the key `30`, `"003"` gets the key `3`, `"34"` gets the key `34`, and `"033"` gets the key `33`. Sorting compares these integer keys, while the result contains the original strings, including their leading zeros.

### `cmp_to_key`

Suppose, however, that you want to define the order through a custom comparator, `compare(a, b)`. A comparator receives **two elements** and returns a number indicating their relative order:

- A negative number means `a` should come before `b`.
- Zero means `a` and `b` are equivalent for ordering.
- A positive number means `a` should come after `b`.

You cannot pass `compare` directly as the key function because sorting calls the key function with **one element**, whereas `compare` requires **two**.

`cmp_to_key()` is a function from the `functools` module that adapts the comparator function to the interface that `key` expects, so later on our comparator can be used by the sorting algorithm.

For example, to arrange numeric strings into the largest possible concatenated number, compare `a + b` with `b + a`:

```python
from functools import cmp_to_key

def compare(a, b):
    if a + b > b + a:
        return -1  # Putting a before b produces the larger concatenation.
    if a + b < b + a:
        return 1   # Putting b before a produces the larger concatenation.
    return 0

print(compare("3", "30"))  # -1: "330" > "303", so "3" comes first.
print(compare("3", "34"))  #  1: "334" < "343", so "34" comes first.
print(compare("3", "33"))  #  0: both concatenations are "333".

comparison_key = cmp_to_key(compare)
ordered = sorted(parts, key=comparison_key)

print(ordered)  # ["34", "3", "33", "30"]
```

### Internal Working

Here is how that adaptation works:

1. `cmp_to_key(compare)` returns a callable that creates wrapper objects.
2. Sorting calls that callable once for each element. Each call returns a wrapper holding that element; your comparator is not called yet.
3. When sorting compares two wrappers, their comparison methods call `compare()` with the elements stored inside them.

For sorting, you can picture the wrapper as a simplified class like this:

```python
class ComparisonKey:
    def __init__(self, element):
        self.element = element

    def __lt__(self, other):
        return compare(self.element, other.element) < 0

a = ComparisonKey("3")
b = ComparisonKey("30")

print(a < b)
# Calls a.__lt__(b)
# Evaluates compare("3", "30") < 0
# Evaluates -1 < 0 → True
```

The value returned by the key function is **the wrapper itself**. It holds the element and defines how `<` behaves when compared with another wrapper. It does not later turn into a numeric or string key: its comparison method returns the Boolean result that sorting needs.

Sorting therefore compares the wrappers using your comparator’s logic, while the final list contains the original elements.

## Adapting Function Calls

### `partial()`

`partial(function, *args, **kwargs)` creates a callable with some arguments supplied in advance. It does not call the original function immediately. Later calls supply the remaining arguments.

```python
from functools import partial

def power(base, exponent):
    return base ** exponent

# Pre-supply a positional argument: base = 2.
powers_of_two = partial(power, 2)
print(powers_of_two(5))  # power(2, 5) -> 32

# Pre-supply a keyword argument: exponent = 2.
square = partial(power, exponent=2)
print(square(5))              # power(5, exponent=2) -> 25
print(square(5, exponent=3))  # power(5, exponent=3) -> 125
```

In this usage, stored positional arguments come before the arguments supplied later. Stored keyword arguments can be overridden by keywords in a later call. Normal argument-binding rules still apply: `square(5, 3)` would give `exponent` both a positional value and its stored keyword value, raising `TypeError`.

Use `partial()` when an existing function already does what you need but a callback or repeated operation requires some arguments to be preconfigured.

## Reducing an Iterable to One Result

### `reduce()`

`reduce(function, iterable, initial)` repeatedly calls a two-argument function with the accumulated result and the next item. It processes items from left to right and returns the final accumulator. The accumulator can be any suitable object, not just a number.

```python
from functools import reduce

numbers = [1, 2, 3, 4]
multiply = lambda accumulated, item: accumulated * item

product = reduce(multiply, numbers, 1)
print(product)  # 24

# The same accumulation written explicitly:
product = 1
for number in numbers:
    product = multiply(product, number)
print(product)  # 24

print(reduce(multiply, [], 1))  # 1: an empty iterable returns the initial value.
```

Here, the accumulator starts at `1`, then becomes `1`, `2`, `6`, and `24`. The initial value participates in the calculation even when the iterable is nonempty.

`initial` is optional. Without it, the first item becomes the accumulator and processing starts with the second item. An empty iterable without an initial value raises `TypeError`.

Unlike the lazy iterators produced by [`map()` and `filter()`](<Iterables.md#map>), `reduce()` consumes its input to compute a final result. If that input is an iterator, its consumed items are no longer available. Use a clearer specialized operation when one exists, such as `sum()`, `min()`, `max()`, or `math.prod()`; the multiplication here demonstrates how accumulation works.

## Caching Results

These utilities use [decorators](<Decorators.md#the-core-idea>) to remember computed results. For the algorithmic use of memoization, see [Memoization and Tabulation](<../Traversal and Search Algorithms/DP(Memoization & Tabulation).md>).

### `cache()` and `lru_cache()`

Both decorators store function results using call arguments as cache keys. On a cache hit, the stored result is returned without executing the function body again.

| Decorator | Stored results |
| --- | --- |
| `@cache` | Keeps results without a size limit; equivalent to `@lru_cache(maxsize=None)`. |
| `@lru_cache(maxsize=128)` | Keeps at most 128 entries, the default limit. |
| `@lru_cache(maxsize=None)` | Keeps results without a size limit. |

**LRU** means **least recently used**. When a bounded cache is full, adding a new entry removes the entry that has gone unused for the longest time. Reading a cached entry makes it recently used again.

```python
from functools import lru_cache

@lru_cache(maxsize=2)
def square(number):
    print(f"Computing {number}")
    return number * number

print(square(2))  # Prints "Computing 2", then 4.
print(square(3))  # Prints "Computing 3", then 9.
print(square(2))  # Prints only 4; 2 becomes the most recently used entry.
print(square(4))  # Prints "Computing 4", then 16; evicts the entry for 3.
print(square(3))  # Prints "Computing 3", then 9; that entry must be recomputed.

print(square.cache_info())
# CacheInfo(hits=1, misses=4, maxsize=2, currsize=2)

square.cache_clear()  # Removes all entries and resets the hit/miss statistics.
```

To keep every result for this same function, import `cache` and replace `@lru_cache(maxsize=2)` with `@cache`. The print statements above make cache hits and misses visible; caching is most useful when the actual calculation is expensive or frequently repeated.

#### Requirements and Limits

- Arguments must be **hashable**. Lists and dictionaries cannot be used directly as cache arguments; a tuple works only if all its elements are hashable.
- The cache stores and returns the result object itself, without copying it. Mutating a cached list or dictionary changes what later callers receive.
- Results are not automatically invalidated when external data changes. Use caching when reuse is appropriate, and call `.cache_clear()` when needed.
- Unbounded caches retain arguments and results until cleared or the cache is discarded. A bounded cache limits entries, not the number of bytes they occupy.

### `cached_property()`

`@cached_property` computes an attribute the first time it is accessed on an instance, then stores that result on the instance for later access. For the underlying property concept, see [method decorators](<Decorators.md#common-method-decorators-used-in-oop>).

```python
from functools import cached_property

class Dataset:
    def __init__(self, values):
        self.values = tuple(values)

    @cached_property
    def total(self):
        print("Computing total")
        return sum(self.values)

data = Dataset([1, 2, 3])
print(data.total)  # Prints "Computing total", then 6.
print(data.total)  # Prints only 6; reads the stored attribute.

data.values = (1, 2, 3, 4)
print(data.total)  # Still 6: changing the input does not invalidate the cache.

del data.total    # Remove the stored result so the next access recomputes it.
print(data.total)  # Prints "Computing total", then 10.
```

The cached value belongs to each instance rather than a function-level cache keyed by arguments. Instances need a writable `__dict__`; a class using `__slots__` without `__dict__` cannot use `cached_property` directly.

## Writing Function Wrappers

### `wraps()`

When a [decorator](<Decorators.md#the-core-idea>) replaces a function with a wrapper, the resulting callable normally exposes the wrapper's name and docstring. `@wraps(original)` copies metadata such as the original function's name, docstring, and annotations onto the wrapper.

```python
from functools import wraps

def trace(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

@trace
def add(a, b):
    """Return the sum of two values."""
    return a + b

print(add(2, 3))       # Prints "Calling add", then 5.
print(add.__name__)    # add
print(add.__doc__)     # Return the sum of two values.
print(add.__wrapped__(2, 3))  # 5: calls the original without this wrapper.
```

`wraps()` also sets `__wrapped__` to the original callable, supporting introspection and access to the wrapped function. The wrapper still needs to forward arguments and return the result itself; `wraps()` handles metadata, not that calling logic.

`update_wrapper(wrapper, original)` performs the metadata update directly. `wraps(original)` is its convenient decorator form.

## Dispatching by Argument Type

### `singledispatch()`

`@singledispatch` gives one function different implementations based on the runtime type of its **first argument**. The decorated function is the default implementation; `@function.register(Type)` adds an implementation for a particular type.

```python
from functools import singledispatch

@singledispatch
def describe(value):
    return f"Value: {value}"

@describe.register(int)
def describe_integer(value):
    return f"Integer: {value}"

@describe.register(list)
def describe_list(value):
    return f"List with {len(value)} items"

print(describe(7))          # Integer: 7
print(describe([10, 20]))   # List with 2 items
print(describe("Python"))   # Value: Python
```

Dispatch considers the first argument's type, not its value or the types of later arguments. A registered base-class implementation can also handle subclasses when no more specific registration applies. Use this when behavior differs by input type and separate implementations make the function easier to extend.
