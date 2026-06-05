# 🧮 Operators in C++

Operators are symbols or keywords that instruct the compiler to perform specific operations on variables or values.

→ They act on **operands** (the values or variables being operated on) and return a result.

**Example:**

```cpp
int x = 5 + 3;
// '+' is the operator
// 5 and 3 are operands
// The result is 8, which is assigned to x
```

Operators are essential building blocks of expressions in C++, enabling everything from basic arithmetic to complex logic and bitwise operations.

---

## 1. Arithmetic Operators

```cpp
int a = 10, b = 3;
int sum = a + b;      // Addition
int diff = a - b;     // Subtraction
int prod = a * b;     // Multiplication
int div = a / b;      // Division (integer division)
int mod = a % b;      // Modulus (remainder)
```

---

## 2. Relational (Comparison) Operators

```cpp
bool isEqual = (a == b);      // Equal to
bool notEqual = (a != b);     // Not equal to
bool greater = (a > b);       // Greater than
bool less = (a < b);          // Less than
bool greaterEq = (a >= b);    // Greater than or equal to
bool lessEq = (a <= b);       // Less than or equal to
```

---

## 3. Logical Operators

```cpp
bool result = (a > 5 && b < 5);    // Logical AND
bool result2 = (a > 5 || b < 2);   // Logical OR
bool result3 = !(a == b);          // Logical NOT
```

---

## 4. Assignment Operators

```cpp
int x = 5;
x += 3;   // x = x + 3
x -= 2;   // x = x - 2
x *= 2;   // x = x * 2
x /= 2;   // x = x / 2
x %= 2;   // x = x % 2
```

---

## 5. Increment / Decrement Operators

```cpp
int i = 0;
i++;   // Post-increment
++i;   // Pre-increment
i--;   // Post-decrement
--i;   // Pre-decrement
```

---

## 6. Bitwise Operators

```cpp
int bitA = 5;    // 0101
int bitB = 3;    // 0011
int andResult = bitA & bitB;   // bitwise AND -> 0001
int orResult = bitA | bitB;    // bitwise OR  -> 0111
int xorResult = bitA ^ bitB;   // bitwise XOR -> 0110
int notResult = ~bitA;         // bitwise NOT -> 1010 (in 2's complement)
int leftShift = bitA << 1;     // bitwise Left shift -> 1010
int rightShift = bitA >> 1;    // bitwise Right shift -> 0010
```

---

## 7. Ternary Operator

```cpp
int max = (a > b) ? a : b;  // If a > b, max = a; else max = b
```

---

## 8. Scope Resolution Operator

```cpp
#include <iostream>
using namespace std;

int globalX = 10;

int main() {
    int globalX = 20;
    cout << ::globalX << endl;  // Access global variable using scope resolution operator
    return 0;
}
```
