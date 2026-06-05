# 🧠 Introduction to Throw and Catch in C++

```cpp
#include <iostream>
#include <stdexcept>  // For standard exceptions like std::out_of_range
using namespace std;
```

## 🔹 What is 'throw'?

- `throw` is used to signal that an error (exception) has occurred.
- The throw keyword creates an exception object.
- This object typically holds information about the error, such as an error message or an error code.

There are different types of exception. Standard exceptions are available in the `<stdexcept>` library. Some useful ones:

- `std::out_of_range`
- `std::invalid_argument`
- `std::runtime_error`

As soon as `throw` is executed:

- The loop ends immediately.
- The function exits immediately (unless the try is in the same function).
- Control jumps to the nearest matching catch block (catch block that can handle the same type of exception thrown).
- If you throw an exception without using a try or catch block, the exception will propagate up the call stack until it either finds a matching catch block or the program terminates.

## 🔹 What is 'try'?

- `try` defines a block of code in which exceptions may occur.
- It "tries" to run the code, and if an exception occurs, it passes control to the matching `catch`.
- The try block must be immediately followed by at least one catch block (or a `catch (...)` to catch all exceptions).

## 🔹 What is 'catch'?

- `catch` is used to handle exceptions that were thrown in a `try` block.
- It allows the program to recover from errors instead of crashing.
- A catch block must directly follow a try block.
- The catch block takes a parameter: the type of exception it handles.

## 🔹 Syntax

```cpp
try {
    // code that might throw
} catch (ExceptionType e) {
    // handle the exception
}
```

---

## Example

```cpp
// Function that might throw an exception
void checkValue(int value) {
    cout << "  -> checkValue called with: " << value << endl;
    if (value == 0) {
        throw runtime_error("Zero encountered!");//when an error occurs in a loop in a function in try, it terminates the loop, the function and jumps out of try
    }
    cout << "  -> checkValue completed successfully.\n";
}

int main() {
    int arr[] = {5, 3, 2, 0, 8, 1};  // Note the 0 in the middle
    int size = sizeof(arr) / sizeof(arr[0]);

    cout << "Starting loop over array...\n";

    try {
        for (int i = 0; i < size; ++i) {
            cout << "Loop iteration " << i << ": value = " << arr[i] << endl;
            checkValue(arr[i]);  // This contains throw
            cout << "Finished iteration " << i << "\n";
        }
        cout << "Loop completed without errors.\n";
    } catch (const runtime_error& e) {
        cout << "Exception caught in main: " << e.what() << endl;//what() is a member function of exception object e that returns the error message
    }

    cout << "Continuing program after try-catch block...\n";

    return 0;
}
```

## Output

```text
Starting loop over array...
Loop iteration 0: value = 5
  -> checkValue called with: 5
  -> checkValue completed successfully.
Finished iteration 0
Loop iteration 1: value = 3
  -> checkValue called with: 3
  -> checkValue completed successfully.
Finished iteration 1
Loop iteration 2: value = 2
  -> checkValue called with: 2
  -> checkValue completed successfully.
Finished iteration 2
Loop iteration 3: value = 0
  -> checkValue called with: 0
Exception caught in main: Zero encountered!
Continuing program after try-catch block...
```
