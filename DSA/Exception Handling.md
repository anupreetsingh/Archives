# Exception Handling

Exception handling is a way to deal with runtime errors without immediately crashing the program.

The basic idea is:

1. Put risky code inside a protected block.
2. If an error happens, control jumps to a handler.
3. The handler decides what to do next.

Common examples of exceptions:

- Dividing by zero
- Opening a missing file
- Converting invalid input
- Accessing an invalid index

## Try Blocks

### Python Example

Python uses `try`, `except`, `else`, `finally`, and `raise`.

```python
try:
    x = int(input("Enter a number: "))
    result = 10 / x
except ValueError:
    print("Input must be a valid number.")
except ZeroDivisionError:
    print("Cannot divide by zero.")
else:
    print("Result:", result)
finally:
    print("Finished running.")
```

How this works:

- `try` contains code that might fail.
- `except` catches specific errors.
- `else` runs only if no exception happens in the `try` block. It separates the code that should run after a successful `try` from the code that might raise the exceptions being handled.
- `finally` always runs, whether there was an error or not.

### JavaScript/Java Example

JavaScript uses `try`, `catch`, `finally`, and `throw`.

```javascript
function divide(a, b) {
    if (b === 0) {
        throw new Error("Cannot divide by zero.");
    }
    return a / b;
}

try {
    const result = divide(10, 0);
    console.log(result);
} catch (error) {
    console.log("Error:", error.message);
} finally {
    console.log("Finished running.");
}
```

```java
public class Main {
    public static double divide(double a, double b) {
        if (b == 0) {
            throw new ArithmeticException("Cannot divide by zero.");
        }

        return a / b;
    }

    public static void main(String[] args) {
        try {
            double result = divide(10, 0);
            System.out.println(result);
        } catch (ArithmeticException error) {
            System.out.println("Error: " + error.getMessage());
        } finally {
            System.out.println("Finished running.");
        }
    }
}
```

How this works:

- `throw` creates an exception.
- `try` runs code that might fail.
- `catch` handles the thrown error.
- `finally` runs cleanup code.

### Key Difference

Python says `raise` and `except`.

JavaScript, Java, and C++ usually say `throw` and `catch`.

The concept is the same: when normal execution cannot continue, an exception jumps to the nearest matching handler.

## `with` Block

The `with` statement is used for resource management. It sets up a resource before a block runs and automatically cleans it up when the block exits.

This is most common with files. Calling `open()` does not load the whole file into memory. It asks the operating system to establish an active handle, or connection, to that file and returns a Python file object that represents that connection.

The file object keeps track of state such as the current file position, also called the **offset**. As the program reads or writes, that offset moves through the file.

```python
with open("notes.txt", "r") as file:
    contents = file.read()

print(file.closed)  # True
```

Here, `open("notes.txt", "r")` opens a connection to the file in read mode as a file object. The `as file` part binds that object to the name `file` inside the surrounding scope.

The actual file contents are only read into memory when code calls something like `file.read()`, `file.readline()`, or loops over the file. When the block finishes, Python closes the file automatically.

This cleanup still happens if an exception is raised inside the block:

```python
try:
    with open("notes.txt", "r") as file:
        number = int(file.read())
except ValueError:
    print("The file did not contain a valid integer.")
```

Conceptually, `with` is similar to using `try` and `finally` for cleanup, but it keeps the setup and cleanup behavior attached to the object being used.

```python
file = open("notes.txt", "r")

try:
    contents = file.read()
finally:
    file.close()
```

Objects that work with `with` are called **context managers**. They define how to enter the block and how to exit it, usually through `__enter__` and `__exit__`.

Example of Other common resources opened using `with` block:

```python
import sqlite3
import tempfile
from contextlib import suppress
from pathlib import Path
from threading import Lock


# 1. Temporary directories
# The directory exists inside the block and is deleted afterward.
with tempfile.TemporaryDirectory() as folder:
    path = Path(folder) / "scratch.txt"
    path.write_text("temporary work")


# 2. Locks
# The lock is acquired before the block and released afterward.
lock = Lock()

with lock:
    print("Only one thread should run this section at a time.")


# 3. Database transactions
# sqlite3 commits on success and rolls back if an exception happens.
# Other database libraries may also close sessions or connections this way.
with sqlite3.connect("app.db") as connection:
    connection.execute("CREATE TABLE IF NOT EXISTS users (name TEXT)")
    connection.execute("INSERT INTO users VALUES (?)", ("Aman",))


# 4. Suppressing specific exceptions
# FileNotFoundError is ignored, but other exceptions still raise normally.
with suppress(FileNotFoundError):
    Path("missing.txt").unlink()
```

To manually create an exception, Python uses `raise`.

```python
def divide(a, b):
    if b == 0:
        raise ZeroDivisionError("Cannot divide by zero.")
    return a / b
```

---
