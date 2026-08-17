# Function Syntax

Function syntax controls how parameters are written in a function definition and how arguments are written in a function call.

In a function definition, **parameters** name the values the function expects.

In a function call, **arguments** provide the actual values for those parameters.

## Function definitions and parameters

Parameters are the names listed inside the parentheses of a function definition:

```python
def convert(value, base=10):
    print(value, base)
```

Here, `value` and `base` are parameters.

Normal parameters can be:

- **Required parameters:** do not have a default value.
- **Default parameters:** have a default value.

**Rule:** In the normal function definition, required parameters must come before default parameters:

```python
def f(a, b, c=0):  # valid
    print(a, b, c)

def f(a=0, b):     # SyntaxError
    print(a, b)
```

A normal parameter can be passed either positionally or by keyword when the function is called.

But if you want to be particular about certain parameters being positional and keyword.

Use `/` in the definition to make parameters before it positional-only:

```python
def convert(value, /, base=10):
    print(value, base)
```

Use `*` in the definition to make parameters after it keyword-only:

```python
def connect(host, *, timeout=5):
    print(host, timeout)
```

## Function calls and arguments

Arguments are the actual values sent into a function call.

Arguments can be written in two common ways:

- **Positional arguments:** matched to parameters by order.
- **Keyword arguments:** matched to parameters by name.

Example:

```python
def convert(value, base=10):
    print(value, base)

convert("101")
convert("101", 2)
convert(value="101", base=2)
convert("101", base=2)
```

**Rule:** In a function call, positional arguments must come before keyword arguments:

```python
def f(a, b, c):
    print(a, b, c)

f(1, 2, c=3)     # valid
f(a=1, 2, 3)     # SyntaxError
```

A parameter cannot receive two values in the same call:

```python
def f(a, b, c):
    print(a, b, c)

f(1, a=2, b=3)  # TypeError: multiple values for argument 'a'
```

If a parameter is positional-only because it appears before `/`, it cannot be passed by keyword:

```python
def convert(value, /, base=10):
    print(value, base)

convert("101", base=2)
convert(value="101", base=2)  # TypeError
```

If a parameter is keyword-only because it appears after `*`, it must be passed by keyword:

```python
def connect(host, *, timeout=5):
    print(host, timeout)

connect("localhost", timeout=10)
connect("localhost", 10)  # TypeError
```

## Type hints

Type hints can be added to function parameters with `: type` and to the return value with `-> type`:

These serve as indicators of the expected type and are not usually enforced by Python itself.

```python
def greet(name: str) -> str:
    return f"Hello, {name}"
```

`Optional[T]` means the value can be either type `T` or `None`.

```python
from typing import Optional

def find_user(user_id: int) -> Optional[str]:
    if user_id == 1:
        return "Ava"
    return None
```

`Optional` does not make a parameter optional by itself. A default value makes the argument optional in the call:

```python
from typing import Optional

def greet(name: Optional[str] = None) -> str:
    if name is None:
        return "Hello"
    return f"Hello, {name}"

greet()
greet("Ava")
```

In modern Python, `Optional[str]` can also be written as `str | None`:

```python
def greet(name: str | None = None) -> str:
    if name is None:
        return "Hello"
    return f"Hello, {name}"
```
