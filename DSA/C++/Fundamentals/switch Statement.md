# 📘 Notes on 'switch' statement in C++

```cpp
#include <iostream>
using namespace std;
```

## 📌 What is 'switch'?

The `switch` statement in C++ allows multi-way branching. It is used to execute different blocks of code based on the value of a single variable (typically int, char, or enum).

It's often used as a cleaner alternative to long chains of if-else-if statements.

**How it works:**
Switch statement basically applies the equality comparison (`==`) between the expression (the value in the switch statement) and each of the case labels. Then it executes whichever case statements holds true.

⚠️ Only constant integral types are allowed in case labels.

- Supported: `int`, `char`, `enum`
- Not supported: `float`, `double`, `string`

---

## 🔹 Example 1: Basic switch usage

```cpp
int main() {

    int day = 2;
    cout << "Day " << day << ": ";

    switch (day) {
        // case 1: If 'day' == 1, print "Monday"
        case 1:
            cout << "Monday\n";
            break;  // 'break' exits the switch after executing the corresponding case

        // case 2: If 'day' == 2, print "Tuesday"
        case 2:
            cout << "Tuesday\n";
            break;  // exit the switch

        // default: If none of the above cases match, execute this block. This is optional to include
        default:
            cout << "Invalid day\n";  // This will run if 'day' is not 1, 2, or 3
    }

    // The switch statement ends here, and the program continues...

    return 0;  // End of the main function, returning 0 to indicate successful execution
}
```

## 🔹 Example 2: Fall-through behavior (no 'break')

```cpp
char grade = 'B';

cout << "\nGrade " << grade << ": ";
switch (grade) {
    case 'A':
        cout << "Excellent\n";
        break;
    case 'B':
        cout << "Good\n";
        // no break -> fall through to next case
    case 'C':
        cout << "Satisfactory\n";
        break;
    default:
        cout << "Grade not recognized\n";
}
// Output:
// Good
// Satisfactory
```

## 🔹 Example 3: Using switch with enum

```cpp
enum Direction { North, South, East, West };
Direction dir = East;

cout << "\nDirection: ";
switch (dir) {
    case North:
        cout << "Going North\n";
        break;
    case South:
        cout << "Going South\n";
        break;
    case East:
        cout << "Going East\n";
        break;
    case West:
        cout << "Going West\n";
        break;
}
```
