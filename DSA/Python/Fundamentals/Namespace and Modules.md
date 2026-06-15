# Namespaces and Modules in Python

## Namespace

A **namespace** is the actual name-to-object mapping.

### Common namespace types

- **Local namespace**: names created inside the function currently running.
- **Enclosing namespace**: names in an outer function when functions are nested.
- **Global namespace**: Each Python file is a **module**, and each module has its own global namespace.
- **Built-in namespace**: names Python provides automatically, like `print`, `len`, and `dict`.

### LEGB rule

Python resolves names using **LEGB**, searching outward and using the first matching name it finds:

```text
L -> Local
E -> Enclosing
G -> Global
B -> Built-in
```

Example:

```python
value = "global"

def outer():
    value = "enclosing"

    def show():
        value = "local"
        print(value) # Output: local
        print(len("hello")) # Output: 5

    show()

outer()
```

In this example:

- `value` exists in the local, enclosing, and global namespaces.
- Python prints `"local"` because the local `value` inside `show()` shadows the enclosing and global `value`.
- `len` and `print` are built-in names because Python provides them automatically.

### Not part of Python namespaces

Some things are related to names, but are not Python namespaces.

#### Keywords

Keyword like `if` ,`and`, `for`, `def` `class` are part of Python's grammar. They are different from Built-in names like `print` as they cannot be shadowed as variable names.

```python
if = "hello"  # SyntaxError
print = "hello"  # allowed, but bad idea
```

#### Environment variables

They are OS-level key-value settings available to a running program:

```python
import os  # os is a standard library module in Python

print(os.environ["HOME"])
```

Here, `os` is added to the current module's global namespace and maps to the `os` module.
Then `os.environ` gives access to OS-level environment variables. A `.env` file is a common way to define environment variables, but Python does not automatically load `.env` files unless a tool or library loads them.
`"HOME"` is a key inside that environment-variable mapping.

## Modules

At runtime, a module is an object with its own namespace.

When you import a module, Python adds that module's name to your current module's global namespace. That name maps to the imported module object.

```python
import math
from math import pow

print(math.sqrt(25))  # Output: 5.0
print(pow(2, 3))      # Output: 8.0
```

Here:

- `import math` adds `math` to the current module's global namespace.
- `math` maps to the `math` module object.
- `from math import pow` adds `pow` directly to the current module's global namespace.
- `math.sqrt` uses dot notation to look up `sqrt` inside `math`.
- `pow` works directly because it was imported as its own name.

### Standard library and third-party modules

**Standard library** modules come with Python but are not automatically added to your current namespace. Examples: `collections`, `math`, `os`, `random`, `datetime`, `json`.

**Third-party** packages are installed with package managers like `pip` or `uv`. A package can contain multiple modules/subpackages. Examples: `numpy`, `pandas`, `requests`.

### Importing modules into other modules

```python
# a.py
from collections import Counter

def count_letters(s):
    return Counter(s)

def most_common_letter(s):
    return Counter(s).most_common(1)
```

```python
# b.py
import a
from a import Counter
from a import most_common_letter

a.Counter("hello")              # use Counter through a.py
Counter("hello")                # use Counter directly in b.py
a.most_common_letter("hello")   # use a.py's own function through a
most_common_letter("hello")     # use a.py's own function directly
```

> Avoid `from module import *` because it copies many names into the current namespace, which can cause name collisions, accidental shadowing, and unclear code.

### `__name__`

At runtime, Python gives every module a special variable called `__name__`.

If a file is run directly, its `__name__` is `"__main__"`. If it is imported, its `__name__` is the module name.

This matters because importing a file runs its top-level code.

```python
# app.py
def main():
    print("Starting app")

main()
```

```python
# other.py
import app
print(app.__name__)
```

```bash
python app.py
# Starting app
```

```bash
python other.py
# Starting app
# app
```

Running `other.py` also runs `main()` from `app.py`, because the `main()` function call is top-level code in `app.py`.

To prevent that, guard the call:

```python
# app.py
def main():
    print("Starting app")

if __name__ == "__main__":
    main()
```

Now:

```bash
python app.py
# Starting app
```

```bash
python other.py
# app
```

When `app.py` runs directly, `__name__` is `"__main__"`, so `main()` runs. When `other.py` imports `app.py`, `app.__name__` is `"app"`, so `main()` does not run.

## Packages

A **package** is a folder that groups related Python modules together.

```text
shapes/
    __init__.py
    circle.py
    square.py
    three_d/
        __init__.py
        sphere.py
```

Here:

- `shapes` is a package.
- `circle.py` and `square.py` are modules inside `shapes`.
- `three_d` is a subpackage inside `shapes`.
- `sphere.py` is a module inside `three_d`.

### `__init__.py`

`__init__.py` tells Python to treat a folder as a package and runs when that package is imported.

It can also expose selected names from modules inside the package, so users can import from the package directly.

Example `__init__.py`:

```python
# shapes/__init__.py
from .circle import area
from .square import perimeter
```

```python
# shapes/three_d/__init__.py
from .sphere import volume
```

Now another file can import those names directly from the package/subpackage:

```python
# main.py
from shapes import area, perimeter
from shapes.three_d import volume

print(area(5))
print(perimeter(4))
print(volume(3))
```

This works because:

- `shapes/__init__.py` added `area` and `perimeter` to the `shapes` package namespace.
- `shapes/three_d/__init__.py` added `volume` to the `shapes.three_d` subpackage namespace.

Without exposing it in `__init__.py`, you would import from the module directly:

```python
# main.py
from shapes.circle import area
from shapes.square import perimeter
from shapes.three_d.sphere import volume

print(area(5))
print(perimeter(4))
print(volume(3))
```

> Names like `__name__` and `__init__.py` are often called **dunder** names because they use double underscores.
