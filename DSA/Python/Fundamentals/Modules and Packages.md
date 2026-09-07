# Modules and Packages in Python

Better to be familiar with the broader discussion of namespaces in [Namespace and Scope Resolution](<Namespace and Scope.md>).

## Modules

Every `.py` file is a module.

At runtime, a module is an object with its own global namespace containing the names defined or imported in that module.

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
- `pow` works directly because it was imported as its own name. In this module,
  that imported name shadows Python's built-in `pow()`; see
  [`pow()`](<Built-in Functions.md#pow>).

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

A **package** is a directory that groups related Python modules and subpackages together.

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

When a package is imported, Python loads it as a special kind of module object with its own namespace, just like a regular `.py` module:

```python
import shapes

print(type(shapes))  # <class 'module'>
```

Unlike a normal module object, A package module object also has a `__path__` attribute that identifies where Python should search for the package's submodules and subpackages when importing names such as `shapes.circle` or `shapes.three_d`:

```python
import math
import shapes

print(hasattr(math, "__path__"))    # False: regular module
print(hasattr(shapes, "__path__"))  # True: package module

import shapes.circle
import shapes.three_d.sphere
```

### `__init__.py`

`__init__.py` is a Python file used to initialize a package. It tells Python to treat a directory as a package.

When the package is first imported, Python runs `__init__.py` as part of creating and initializing the package's module object and namespace.

Because `__init__.py` defines the package's initial namespace, it can also **expose** selected names—such as modules, classes, functions, or variables—from elsewhere inside the package at the package's **top level**. This is done by importing those names into `__init__.py`, allowing users to access or import them directly from the package without referring to the specific internal module where they are defined.

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

## Standard Library and Third-Party Packages

### Standard Library

Python's **standard library** is a collection of modules and packages that comes with a normal Python installation. You can use them by importing them directly into a python module's namespace without any separate installs.

The following example shows several commonly used modules and packages from Python’s standard library. It doesn't show any :

```python
Python Standard Library
├── Modules
│   ├── heapq
│   ├── math
│   ├── random
│   ├── os
│   ├── subprocess
│   └── pathlib
│
└── Packages
    ├── collections
    |   └── defaultdict
    |   └── deque
    |   └── Counter
    │   └── abc
    │
    ├── json
    │   ├── decoder
    │   ├── encoder
    │   └── scanner
    │
    ├── urllib
    │   ├── request
    │   ├── response
    │   ├── error
    │   ├── parse
    │   └── robotparser
    │
    └── unittest
        ├── mock
        ├── result
        └── runner
```

You can import them in your module directly like :

```python
import math
import os
import heapq
import collections
import json
```

#### `os` Module

`os` is a module in Python's standard library that gives you access to operating-system functionality from your python script. It's uses could includes access to:

##### 1. Environment Variables

**Environment variables** are key-value pairs that the operating system attaches to a process (running program) when it starts. A parent process, such as a shell, can set up the initial environment, and a child process, such as a Python program started from that shell, inherits it.

> Environment variables are not names in a Python namespace.

The `os` module helps us access the current process's environment variables from a Python script. It exposes them through `os.environ`, which behaves like a mutable dictionary:

```python
import os

home = os.environ["HOME"]                    # Raises KeyError if HOME is missing, just like a dictionary does
home = os.environ.get("HOME")                # Returns None if HOME is missing, .get() returns None just like in a dictionary
home = os.getenv("HOME")                     # A wrapper function equivalent to os.environ.get().
mode = os.environ.get("APP_MODE", "development") # Returns default value "development" in case `APP_MODE` key is missing in os.environ

os.environ["APP_DEBUG"] = "true"            # Add or update a variable, just like in a dictionary

print(home)
print(mode)
```

A `.env` file is a common way to record environment-variable settings, but Python does not load `.env` files automatically. A tool or library must read the file and add its values to the process environment.

For example, the third-party package like `python-dotenv` can load a `.env` file:

```text
# .env
DATABASE_URL=postgresql://localhost/app
APP_MODE=development
```

```python
from dotenv import load_dotenv
import os

load_dotenv() # Calls the function that reads the .env file in the current working directory and loads those key value pairs as the OS environment variables for the current process. 

print(os.environ["DATABASE_URL"])
print(os.getenv("APP_MODE"))
```

`load_dotenv()` is a function call that searches the current working directory or travels outward to parent directories for finding the `.env` file load the variables there as environment variables of the current process. By deafult, it preserves the current value when the same variable already exists. Use `load_dotenv(override=True)` when values from `.env` should replace existing variables with matching names.

##### 2. Directories

The `os` module can inspect the current directory, create directories, list their contents, change directories, and remove empty directories.

```python
import os

original_directory = os.getcwd()
# "/Users/manpreet/Projects"
project_directory = os.path.join(original_directory, "example_project")
# "/Users/manpreet/Projects/example_project"
data_directory = os.path.join(project_directory, "data")
# "/Users/manpreet/Projects/example_project/data"

# Creates the data_directory if it doesn't exist. `exist_ok=False`, Raise a FileExistsError if it already exists.
os.makedirs(data_directory, exist_ok=True)

try:
    # Change the process's current working directory from original_directory to project_directory
    os.chdir(project_directory)

    print("Current directory:", os.getcwd())
    # Current directory: /Users/manpreet/Projects/example_project

    print("Directory contents:", os.listdir("."))
    # Directory contents: ['data']

    # Check whether a directory exists.
    if os.path.isdir("data"):
        print("The data directory exists.")
finally:
    # Restore the original directory.
    os.chdir(original_directory)
```

Changing the working directory affects the entire process. For most file operations, constructing an explicit path is safer than calling `os.chdir()`.

##### 3. OS Information

The `os` module provides information about the operating system, current process, CPU, and environment.

```python
import os

print("Operating-system type:", os.name)
# Operating-system type: posix
# posix means the operating system follows the POSIX(Portable Operating System Interface) standards, a common set of rules for Unix-like systems(macOS, Linux)

print("Logical CPU count:", os.cpu_count())
# Logical CPU count: 8 , Logical CPU cores.

print("Current process ID:", os.getpid())
# Current process ID: 4582, The Unique number assigned by OS to uniquely identify.

print("Parent process ID:", os.getppid())
# Parent process ID: 4510, Parents process's Process-ID

print("Home directory:", os.path.expanduser("~"))
# Gives the currents users home directory path when "~" is passed in.
# Home directory: /Users/manpreet

print(os.path.expanduser("~/Documents"))
# /Users/manpreet/Documents
```

##### 4. File Permissions

```python
import os

# Another module in Standard Library, for named permission-bit constants such as S_IRUSR, S_IWUSR, and S_IXUSR.
import stat

# Store the path of the file whose permissions will be changed.
script_path = "backup.sh"

# Open backup.sh for writing. This creates the file if it does not exist and
# replaces its existing contents if it does. The with statement closes it afterward.
with open(script_path, "w", encoding="utf-8") as script:
    # Write the interpreter directive as the first line of the shell script.
    script.write("#!/bin/sh\n")

    # Write a command that will print a message when the script is executed.
    script.write('echo "Creating backup..."\n')

# Read the file's current metadata and save its complete mode, which includes
# its permission bits. Reading it first lets us preserve the existing permissions.
current_mode = os.stat(script_path).st_mode

# S_IXUSR represents the owner's execute bit. Bitwise OR turns that bit on
# without changing any of the other bits, and chmod applies the resulting mode.
os.chmod(script_path, current_mode | stat.S_IXUSR)

# Read the mode again so the following checks use the updated permissions.
updated_mode = os.stat(script_path).st_mode

# Bitwise AND isolates each owner permission bit. A nonzero result means that
# permission is enabled, and bool converts the result to True or False.
print("Owner can read:", bool(updated_mode & stat.S_IRUSR))
print("Owner can write:", bool(updated_mode & stat.S_IWUSR))
print("Owner can execute:", bool(updated_mode & stat.S_IXUSR))
```

To remove the owner's execute permission:

```python
# Read the latest mode so every permission other than owner-execute is preserved.
current_mode = os.stat(script_path).st_mode

# ~ inverts the S_IXUSR mask, and bitwise AND therefore turns off only the
# owner's execute bit. chmod then applies the updated mode to backup.sh.
os.chmod(script_path, current_mode & ~stat.S_IXUSR)
```

### Third-Party Libraries and Packages

Third-party libraries and packages are not part of Python or its standard library. They must be installed separately, commonly from a package index by using a package manager such as `pip` or `uv`, and then imported by the program.

Examples include `requests`, `httpx`, `django`, `FastAPI`, `numpy`, and `pandas`.

#### `requests` Package

`requests` is a third-party package for making client-side synchronous HTTP requests in Python. It is less verbose than the standard library `urllib` and handles things like sessions, headers, JSON parsing and timeouts more cleanly.

The Python program acts as the client: it sends a request to a web server or API and receives a response. It is commonly used to call external APIs, download web content, or communicate with another service. It does not define routes or receive incoming requests like a web framework such as Django or FastAPI.

The package exposes functions named after HTTP methods, including `requests.get()`, `requests.post()`, `requests.put()`, `requests.patch()`, and `requests.delete()`. Each function constructs and sends an HTTP request, waits for the server, and returns a `Response` object.

```python
import requests

try:
    response = requests.get(
        "https://api.github.com/repos/psf/requests/issues",
        params={"state": "open", "per_page": 5},
        headers={"Accept": "application/vnd.github+json"},
        timeout=10,
    )
    
    # Raise an HTTPError for an unsuccessful 4xx or 5xx response.
    # Helps catch the response code for error, rather than checking manually everytime
    response.raise_for_status()

    # Convert a JSON response body into Python objects(such as dictionaries or lists)
    issues = response.json()
    # Here issues is a list of dictionaries
    # So we look at the title of each issue(bound by its dictionary)
    for issue in issues:
        print(issue["title"])

# if the requests.get() times out, it raises Timeout exception(A subclass of RequestException) and we print this message:
except requests.exceptions.Timeout:
    print("The server took too long to respond")
# RequestException is the base class for exceptions raised by the Requests library.
# This catches any Requests exception not handled by an earlier, more specific block.
except requests.exceptions.RequestException as error:
    print(type(error).__name__)
    print(f"The request failed: {error}")

# Example Output: 
# ConnectionError
# The request failed: HTTPSConnectionPool(host='api.example.com', port=443): Max retries exceeded with url: /users
```

The keyword arguments describe parts of the outgoing request:

| Argument | Purpose |
|---|---|
| `params={...}` | Adds URL query parameters, such as `?state=open&per_page=5`. |
| `headers={...}` | Adds HTTP headers containing metadata about the request. |
| `json={...}` | Serializes a Python object as a JSON request body, commonly for `POST`, `PUT`, or `PATCH`. |
| `data={...}` | Sends form data or another non-JSON request body. |
| `auth=(username, password)` | Supplies HTTP authentication credentials. |
| `timeout=10` | Stops waiting and raises `Timeout` when the remote service is unresponsive. Production code should normally set a timeout explicitly. |

#### `HTTPX`

`httpx` is a third-party package for making client side synchronous and asynchronous HTTP requests in Python.

For synchronous code, its top-level functions work much like those in `requests`:

```python
import httpx

response = httpx.get("https://api.github.com/users/psf", timeout=10)
response.raise_for_status()
account = response.json()
```

The familiar request arguments such as `params`, `headers`, `json`, `data`, `auth`, and `timeout` are also available. The returned `httpx.Response` object provides attributes and methods such as `status_code`, `headers`, `text`, `content`, `json()`, and `raise_for_status()`.

Asynchronous HTTP calls are made through an `httpx.AsyncClient`. Its request methods are coroutine methods, so they must be called with `await` from inside an `async def` function:

```python
import asyncio
import httpx


async def fetch_account(client, username):
    try:
        response = await client.get(f"/users/{username}")
        response.raise_for_status()
        return response.json()
    except httpx.TimeoutException:
        print(f"The request for {username} timed out")
    except httpx.HTTPStatusError as error:
        print(
            f"The server returned {error.response.status_code} "
            f"for {error.request.url}"
        )
    except httpx.RequestError as error:
        print(f"The request to {error.request.url} failed: {error}")

    return None


async def main():
    usernames = ["psf", "pallets", "django"]

    async with httpx.AsyncClient(
        base_url="https://api.github.com",
        headers={"Accept": "application/vnd.github+json"},
        timeout=10,
    ) as client:
        accounts = await asyncio.gather(
            *(fetch_account(client, username) for username in usernames)
        )

    for account in accounts:
        if account is not None:
            print(account["login"])


asyncio.run(main())
```

`asyncio` is part of Python's standard library and manages the event loop. When `fetch_account()` reaches `await client.get(...)`, that coroutine pauses while it waits for network I/O, allowing the event loop to advance another coroutine. `asyncio.gather()` therefore lets the three requests make progress concurrently instead of waiting for each one to finish before starting the next.

One `AsyncClient` is shared by all three calls. The client stores common configuration such as the base URL, headers, and timeout, and it maintains a connection pool so connections can be reused. The `async with` statement closes the client and its connections after the block finishes.
