# Programming Language Characteristics

A reference covering 3 key language design dimensions:

1. **Static vs Dynamic** typing  
2. **Weak vs Strong** typing  
3. **Duck vs Nominal vs Structural** typing

---

## 1. Static vs Dynamic Typing

### Static typing

Variable types are checked at *compile* time. Once a variable type is declared explicitly or inferred by a compiler/type checker, it cannot be assigned to a different type in the same scope because that would lead to a type error before the program runs.
  
Type errors are caught by the compiler or a type checker before running.

Languages Examples: C++, TypeScript, Java, Rust, Go

Example:

```java
int count = 10;
count = "10"; // Type error: String cannot be assigned to int

var age = "24"; // inferred as String
age = 24;        // Type error: int cannot be assigned to String
```

Here, the errors are flagged before the program runs. The compiler/type checker rejects the assignments because "10" is not compatible with `int`, and 24 is not compatible with the inferred String type of age.

### Dynamic typing

Variable types are checked at *run* time. A name is not tied to any type and can be rebound to objects of different types in the same scope over time.

Type errors appear only when an operation is executed with an incompatible value.

Languages Examples: Python, JavaScript, Ruby

Example of Name rebinding:

```python
x = 3          # x -> int object
x = "three"    # now x -> str object (legal in Python)
```

Exampel of Runtime type error:

```python
value = "hello"
print(value.upper())  # OK: str has upper()

value = 42
# print(value.upper()) # AttributeError at runtime: int has no upper()
```

The name `value` is not permanently tied to one type. It can first refer to a `str` object and later be rebound to an `int` object, so an invalid method call like `value.upper()` is caught only when that line is executed at runtime.

---

## 2. Weak vs Strong Typing

### Weak typing

A weakly typed language performs more implicit type coercion. The language may automatically convert a value from one type to another during an operation.

Examples: JavaScript, PHP

Special Case: Because TypeScript adds static type checking on top of JavaScript, it catches many unsafe operations at compile time, such as `"5" - 2`, while still allowing JavaScript-valid coercions like `"5" + 2` as string concatenation; after compilation, the emitted JavaScript follows JavaScript's weak runtime coercion rules.

Example Usage

Implicit coercion:

```js
console.log("2" == 2);  // true
console.log("2" + 2);   // "22", converts to string because + applies to string concatenation and number arithmetic but the rules of implicit coercion in JS tell it to use concatentation in this scenario.
console.log("5" - 2);   // 3, converts to number because operator like -, *, / only really apply to numbers
```

### Strong typing

A strongly typed language resists implicit type coercion between incompatible types. If an operation needs a different type, the programmer usually has to convert the value explicitly.

Examples: Python, Java, Rust, Go

Example Usage

No implicit coercion:

```python
print(2 + "5") # unsupported operand types for +: int and str
```

Causes a TypeError at runtime because of strong typing

Explicit conversion:

```python
print(2 + int("5"))       # 7
print(str(2) + "5")       # "25"
```

---

## 3. Duck vs Nominal vs Structural Typing

This category is about **type compatibility**. It describes how a language decides whether a value is acceptable for an operation, function parameter, variable, or interface.

### Duck typing

Duck typing focuses on whether an object supports the operation being used, rather than whether it has a specific declared type.

It is named after the idea: "if it walks like a duck and quacks like a duck, then it is a duck."

Because Python and Ruby usually do not require declared parameter types, duck typing is common: an object is considered usable if it supports the operation being performed at runtime.

Example:

```python
def make_sound(obj):
    obj.speak()

class Dog:
    def speak(self):
        print("woof")

class Person:
    def speak(self):
        print("hello")

class Robot:
    def move(self):
        print("rolling")

make_sound(Dog())     # OK: Dog has speak()
make_sound(Person())  # OK: Person has speak()
# make_sound(Robot()) # AttributeError: Robot has no speak() method
```

The function does not require `obj` to be declared as a `Dog` or a `Person`. It only requires that `obj` supports the `.speak()` operation at runtime.

### Nominal typing

Nominal typing focuses on the declared or named type. A value is compatible because it belongs to a specific class, implements a specific interface, or is part of a declared inheritance relationship.

Nominal typing is common in languages such as Java, C#, and C++.

Example:

```java
interface Speaker {
    void speak();
}

class Dog implements Speaker {
    public void speak() {
        System.out.println("woof");
    }
}

void makeSound(Speaker obj) {
    obj.speak();
}

makeSound(new Dog()); 
```

Here, `makeSound` requires a value whose declared type is `Speaker` or a class that implements `Speaker`. The `Dog` object is accepted because `Dog` explicitly declares that it implements the named `Speaker` interface.

### Structural typing

Structural typing focuses on the shape or structure of a value. A value is compatible if it has the required fields or methods, even if it was not explicitly declared as a specific named type.

Structural typing is used by TypeScript and is also similar to how Go interfaces work.

Example:

```ts
type Speaker = {
    speak: () => void;
};

const dog = {
    speak() {
        console.log("woof");
    },
};

function makeSound(obj: Speaker) {
    obj.speak();
}

makeSound(dog); // OK: dog has the required speak method
```

The object `dog` does not have to explicitly declare that it is a `Speaker`. It is compatible because its structure matches the `Speaker` type.
