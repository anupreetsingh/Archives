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

---

## Python Example

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
- `else` runs only if no exception happens.
- `finally` always runs, whether there was an error or not.

To manually create an exception, Python uses `raise`.

```python
def divide(a, b):
    if b == 0:
        raise ZeroDivisionError("Cannot divide by zero.")
    return a / b
```

---

## JavaScript Example

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

How this works:

- `throw` creates an exception.
- `try` runs code that might fail.
- `catch` handles the thrown error.
- `finally` runs cleanup code.

---

## Key Difference

Python says `raise` and `except`.

JavaScript, Java, and C++ usually say `throw` and `catch`.

The concept is the same: when normal execution cannot continue, an exception jumps to the nearest matching handler.
