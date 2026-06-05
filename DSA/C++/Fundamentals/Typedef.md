# What is typedef?

```cpp
#include <iostream>
using namespace std;
```

In C++, `typedef` is used to create an alias for an existing type. This allows you to give a new name (alias) to an existing type OR simplifying custom complex types.

---

## 1. Simple Type Alias

`typedef` is commonly used to simplify complex types. Here's an example:

```cpp
typedef int Integer;   // Integer is now an alias for int
```

Now, instead of using `int`, we can use `Integer` to declare variables:

```cpp
typedef int Integer;

Integer x = 5;  // Equivalent to: int x = 5;
cout << "Value of x: " << x << endl; // Output: 5
```

---

## 2. Pointer Type Alias

You can also use typedef to create aliases for pointer types, making pointer declarations easier to read.

For example:

```cpp
typedef int* IntPointer;  // IntPointer is an alias for int* (pointer to int)
```

This means you can declare a pointer to an int using `IntPointer` instead of `int*`.

```cpp
typedef int* IntPointer;

IntPointer ptr;  // Equivalent to: int* ptr;
ptr = &x;
cout << "Value of ptr: " << *ptr << endl; // Output: 5
```

---

## 3. Function Pointer Alias

`typedef` can also be used to simplify function pointer declarations.

For example:

```cpp
typedef float (*FunctionPointer)(int);  // FunctionPointer is an alias for a pointer to a function that takes an int and returns a float
```

This means you can declare a function pointer that points to a function which accepts an int and returns a float.

Here, we define a function pointer and assign it to a function that matches the signature.

```cpp
typedef float (*FunctionPointer)(int);

// A function that matches the signature: takes int and returns float
float someFunction(int x) {
    return x * 2.5f;
}

int main() {
    // Declare and assign the function pointer
    FunctionPointer funcPtr = someFunction;

    // Use the function pointer to call the function
    float result = funcPtr(5);  // Equivalent to: someFunction(5)
    cout << "Result: " << result << endl;  // Output: 12.5

    return 0;
}
```

---

## 4. Struct Alias

You can also use typedef to alias struct types, making them more concise.

For example, suppose we define a struct:

```cpp
struct Person {
    string name;
    int age;
};
```

Now, we can use typedef to create an alias for the struct type, so we don't need to repeat the `struct` keyword:

```cpp
typedef struct Person PersonAlias;

PersonAlias person;  // Equivalent to: struct Person person;
person.name = "John";
person.age = 30;

cout << "Name: " << person.name << ", Age: " << person.age << endl; // Output: Name: John, Age: 30
```

---

## 5. Array Type Alias

You can use typedef to create an alias for arrays as well.

For example, to define an array of 10 integers:

```cpp
typedef int IntArray[10];  // IntArray is an alias for an array of 10 ints

IntArray arr;  // Equivalent to: int arr[10];
arr[0] = 100;
cout << "First element: " << arr[0] << endl; // Output: 100
```

---

## 6. Typedef with const

You can also use typedef with const to define constant types.

For example:

```cpp
typedef const int ConstantInt;  // ConstantInt is an alias for const int
```

Now you can use `ConstantInt` in place of `const int`:

```cpp
typedef const int ConstantInt;

ConstantInt x = 10;  // Equivalent to: const int x = 10;
cout << "Constant value: " << x << endl; // Output: 10
```

---

## Summary

1. `typedef` is used to create type aliases in C++.
2. It simplifies complex types (like function pointers, struct types, and arrays) and makes the code more readable.
3. You can use typedef for basic types, pointers, functions, structs, arrays, and more.

Remember, typedef doesn't create new types, it simply provides a new name (alias) for an existing type.
