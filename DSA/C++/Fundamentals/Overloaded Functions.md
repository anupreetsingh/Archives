# 📘 Function Overloading in C++

## 🔹 Definition

Function overloading is a feature in C++ where two or more functions can have the same name but differ in:

- Number of parameters
- Type of parameters
- Order of parameters

## 🔹 Why use it?

- Improves code readability
- Allows intuitive use of the same function name for similar operations

## 🔹 Why the term "overloaded"?

- The same function name is "loaded" more than once in the compiler's symbol table, each time with a different signature (parameter list).
- This gives the same name **multiple meanings**, based on the arguments.

## 🔹 Rules

- ✅ Functions must differ in parameter list (signature)
- ❌ Return type alone cannot be used to distinguish overloaded functions

---

## 🔹 Code Example

```cpp
int add(int a, int b) {
    return a + b;
}

double add(double x, double y) {
    return x + y;
}

int add(int a, int b, int c) {
    return a + b + c;
}
```

## 🔹 Usage

```cpp
cout << add(2, 3);           // Calls int version
cout << add(2.5, 3.1);       // Calls double version
cout << add(1, 2, 3);        // Calls 3-parameter version
```

## 🔹 Notes

- Function overloading is resolved at **compile time**
- Also known as **compile-time polymorphism**
