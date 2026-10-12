# Arrays and Strings in C++

C++ has two generations of both arrays and strings:

| | Inherited from C | Modern C++ |
|---|---|---|
| Fixed-size array | `int a[5]` | `std::array<int, 5>` |
| Growable array | `new int[n]` (manual) | `std::vector<int>` (see [STL Containers](<../Standard Library/STL Containers.md>)) |
| Text | `char s[]` / `const char*` (null-terminated) | `std::string`, `std::string_view` |

You need to understand the C versions because they show up everywhere (string literals, `argv`, C libraries, and underneath the modern types), but write new code with the modern ones.

Better to be familiar with [Pointers](<../Memory/Pointers.md>) first, because C arrays and C strings are mostly pointers in disguise.

---

## 1. C-Style Arrays

A C array is a **fixed** number of elements of one type, stored **contiguously** in memory. The size must be a compile-time constant:

```cpp
int a[5] = {1, 2, 3};     // the remaining elements are zero-initialized: {1, 2, 3, 0, 0}
int b[] = {4, 5, 6};      // size deduced from the initializer: 3
int c[5];                 // local array: elements are UNINITIALIZED garbage
int d[5] = {};            // all zeros

a[0] = 10;                // index from 0, like Python
```

This matches the "array" side of [Array vs List](<../../Python/Fundamentals/Array vs List.md>): homogeneous, contiguous, O(1) indexing. Unlike a Python list, the elements are stored **inline** with no references:

```text
int a[5]:   [ 10 ][ 2 ][ 3 ][ 0 ][ 0 ]     20 contiguous bytes, the values themselves
            ^ a (the address of a[0])
```

### What C Arrays Can't Do

```cpp
int a[5] = {1, 2, 3, 4, 5};

a[10] = 7;                 // NO bounds check: writes past the end. Undefined behavior
a[-1];                     // no negative indexing either: also out of bounds, UB
int e[5] = a;              // ERROR: arrays can't be copied by assignment or initialization
a == b;                    // compares ADDRESSES, not contents
a.size();                  // ERROR: arrays have no member functions
```

For the length, use `std::size` (C++17), or the old `sizeof` trick, which only works where the array hasn't decayed to a pointer:

```cpp
std::size(a)                  // 5
sizeof(a) / sizeof(a[0])      // 20 / 4 = 5
```

### Decay

Passing a C array to a function **decays** it into a pointer to its first element, and the size is lost (see [Pointers](<../Memory/Pointers.md#array-to-pointer-decay>)):

```cpp
void print_all(int arr[], int n);   // int arr[] really means int* arr, so the size must be passed separately
```

### Multidimensional Arrays

```cpp
int grid[2][3] = {
    {1, 2, 3},
    {4, 5, 6},
};

grid[1][2]       // 6
sizeof(grid)     // 24: 6 ints, one contiguous block, row after row
```

The rows are laid out back to back ("row-major"), so `grid[r][c]` lives at flat index `r * cols + c`, the same formula as in [Array Calculations](<../../Python/Fundamentals/Array Calculations.md#matrix-position---flat-index>). In C++ that formula describes how the memory is actually arranged.

## 2. `std::array`: the Modern Fixed-Size Array

`std::array<T, N>` (from `<array>`) wraps a C array in a class. It has exactly the same memory layout and speed, but behaves like a normal value:

```cpp
#include <array>

std::array<int, 4> arr = {10, 20, 30, 40};

arr.size()         // 4: it knows its size
arr.front()        // 10
arr.back()         // 40
arr[1]             // 20: unchecked, like a C array
arr.at(10)         // throws std::out_of_range: "array::at". The checked version, like Python's IndexError

std::array<int, 4> copy = arr;   // copies all elements (value semantics)
copy[0] = 99;                    // arr[0] is still 10
arr == copy                      // false: compares contents

for (int x : arr) { }            // range-based for works
```

Use `std::array` when the size is fixed at compile time, and [`std::vector`](<../Standard Library/STL Containers.md>) when it isn't.

### `std::span`: a View of Any Contiguous Array (C++20)

`std::span<T>` is a pointer plus a length. It lets one function accept C arrays, `std::array`s, and `std::vector`s alike, without copying and without losing the size:

```cpp
#include <span>

int sum(std::span<const int> xs) {   // span of const int: read-only view
    int total = 0;
    for (int x : xs) total += x;
    return total;
}

int a[3] = {1, 2, 3};
std::array<int, 4> arr = {10, 20, 30, 40};
std::vector<int> v = {5, 5};

sum(a);     // 6
sum(arr);   // 100
sum(v);     // 10
```

## 3. C-Style Strings

A C string is an array of `char` that ends with a **null terminator**, the character `'\0'` (byte value 0). There's no stored length: code finds the end by scanning for that zero byte.

```cpp
char cs[] = "hello";
```

```text
cs:  [ 'h' ][ 'e' ][ 'l' ][ 'l' ][ 'o' ][ '\0' ]
       0      1      2      3      4      5
```

```cpp
#include <cstring>

sizeof(cs)          // 6: five letters + the terminator
std::strlen(cs)     // 5: scans until '\0', O(n) every call
```

A **string literal** like `"hello"` has type `const char[6]`. It lives in read-only memory for the whole program, and usually decays to `const char*`:

```cpp
const char* lit = "hello";   // points at the literal's first char
lit[0] = 'J';                // ERROR: lit points to const char
```

C strings are where the "`'A'` vs `"A"`" distinction from [Variables and Types](<Variables and Types.md#characters>) comes from: `'A'` is one `char`, while `"A"` is the two-char array `{'A', '\0'}`.

### Why Not to Use C Strings

```cpp
"a" + "b"           // ERROR: invalid operands to binary expression ('const char[2]' and 'const char[2]')
                    // two arrays can't be added: there's no concatenation
const char* p = "apple";
p == "apple"        // compares addresses, not text
char buf[5];
std::strcpy(buf, "too long for buf");   // writes past the end of buf: buffer overflow, UB
```

Manual buffer sizes, no concatenation, O(n) length, and silent overflows are why C strings are a famous source of security bugs. Use `std::string`.

## 4. `std::string`

`std::string` (from `<string>`) owns a heap buffer of characters that grows as needed, tracks its own length, and cleans up after itself. It's the closest thing to Python's `str`, with **one big difference**:

> A C++ `std::string` is **mutable**. Python's `str` is immutable, so every "modification" in Python builds a new string. In C++ you can change characters in place.

```cpp
#include <string>

std::string s = "Hello, World";

s.size()          // 12 (s.length() is the same thing), O(1): the length is stored
s[0]              // 'H': a char, not a 1-character string as in Python
s.back()          // 'd' (like Python's s[-1]; there's no negative indexing)
s[0] = 'J';       // "Jello, World": modified in place. A TypeError in Python
```

### Common Operations, Python to C++

Matching [String Manipulation](<../../Python/Fundamentals/String Manipulation.md>):

| Python | C++ | Notes |
|---|---|---|
| `len(s)` | `s.size()` | |
| `s[i]` | `s[i]` / `s.at(i)` | `at` is bounds-checked and throws |
| `s[-1]` | `s.back()` | |
| `s[7:]` | `s.substr(7)` | `substr(pos, count)` takes a **length**, not an end index |
| `s[0:5]` | `s.substr(0, 5)` | |
| `a + b` | `a + b` | At least one side must be a `std::string` |
| `s += "!"` | `s += "!"` / `s.append("!")` / `s.push_back('!')` | In place, amortized O(1) |
| `s.find("World")` | `s.find("World")` | Returns `std::string::npos` (not `-1`) when not found |
| `s.rfind("o")` | `s.rfind('o')` | |
| `"sub" in s` | `s.find("sub") != std::string::npos` | `s.contains("sub")` in C++23 |
| `s.startswith("J")` | `s.starts_with("J")` | C++20 |
| `s.endswith("d")` | `s.ends_with("d")` | C++20 |
| `s.count("l")` (single char) | `std::count(s.begin(), s.end(), 'l')` | `<algorithm>` |
| `s.upper()` | `std::transform` + `std::toupper` | See below |
| `s[::-1]` | `std::reverse(s.begin(), s.end())` | Reverses **in place** |
| `s.replace("-", "+")` (single char) | `std::replace(s.begin(), s.end(), '-', '+')` | For substrings, use `find` + `replace` in a loop |
| `s.split()` | `std::istringstream` loop | No built-in split, see below |
| `", ".join(words)` | Loop with `+=` | No built-in join |
| `s.strip()` | `find_first_not_of` / `find_last_not_of` | No built-in strip |
| `"0" * 3` | `std::string(3, '0')` | Repeat one char. For a substring, use a loop |
| `f"{name}: {score}"` | `std::format("{}: {}", name, score)` | C++20 |
| `a == b`, `a < b` | `a == b`, `a < b` | Compare contents, lexicographically |

### Examples of the Missing Built-ins

```cpp
#include <algorithm>
#include <cctype>
#include <sstream>
#include <vector>

// upper(): apply toupper to every char, in place
std::string up = "Jello, World";
std::transform(up.begin(), up.end(), up.begin(),
               [](unsigned char c) { return std::toupper(c); });
// "JELLO, WORLD"

// split() on whitespace: read words out of a string stream
std::string text = "the quick  brown fox";
std::istringstream iss(text);
std::vector<std::string> words;
std::string w;
while (iss >> w) {             // >> skips any amount of whitespace, like Python's split()
    words.push_back(w);
}
// {"the", "quick", "brown", "fox"}

// split(",") on a delimiter: getline with a separator char
std::string csv = "a,b,,c";
std::stringstream ss(csv);
std::vector<std::string> parts;
std::string item;
while (std::getline(ss, item, ',')) {
    parts.push_back(item);
}
// {"a", "b", "", "c"}: keeps empty fields, like Python's "a,b,,c".split(",")

// ", ".join(words)
std::string joined;
for (std::size_t i = 0; i < words.size(); ++i) {
    if (i > 0) joined += ", ";
    joined += words[i];
}
// "the, quick, brown, fox"

// strip()
std::string padded = "  hi  ";
auto first = padded.find_first_not_of(' ');
auto last = padded.find_last_not_of(' ');
std::string trimmed = padded.substr(first, last - first + 1);   // "hi"
```

> Unreal's `FString` includes most of what's missing here: `ToUpper()`, `ParseIntoArray()` (split), `FString::Join()`, `TrimStartAndEnd()`, `Contains()`, and `FString::Printf()`.

### Building Strings Efficiently

Since `std::string` is mutable, appending in a loop is efficient (amortized O(1) per append, the same capacity-doubling as a vector). In Python, repeated `+=` on immutable strings can be slow, which is why `"".join()` is idiomatic there. In C++, `+=` in a loop is the idiom. Call `reserve(n)` first if you know the final size.

### Converting To and From C Strings

```cpp
std::string s = "Jello, World";
const char* c = s.c_str();   // a null-terminated view of s's buffer, for C APIs
                             // valid only until s is modified or destroyed

std::string from_c = c;      // copy a C string into a std::string
```

### Numbers and Strings

There's no implicit conversion. See [Type Conversion](<Type Conversion.md#3-text-conversions>) for `std::to_string`, `std::stoi`, `std::format`, and the `s += 65` trap.

### Characters Are Bytes, Not Unicode Code Points

`std::string` stores **bytes**. With UTF-8 text, one accented letter or emoji takes several bytes, so `s.size()` counts bytes and `s[i]` may land in the middle of a character:

```cpp
std::string e = "é";   // 2 bytes in UTF-8
e.size();              // 2 (Python's len("é") is 1)
```

Python's `str` is a sequence of Unicode code points. C++ has no standard type that handles this for you.

> Unreal's `FString` stores wide `TCHAR` characters instead of bytes, and string literals are wrapped in `TEXT("...")` to match.

## 5. `std::string_view`: a Non-Owning String View (C++17)

A `std::string_view` is a pointer plus a length that **looks at** characters owned by someone else. It never allocates or copies, and `substr` on a view is O(1):

```cpp
#include <string_view>

void print_label(std::string_view label) {   // accepts std::string, literals, and char*, all without copying
    std::cout << label << "\n";
}

std::string s = "Jello, World";
print_label(s);                 // views s's buffer
print_label("literal");         // views the literal, no std::string is created
std::string_view first = std::string_view(s).substr(0, 5);   // "Jello": no copy
sizeof(std::string_view)        // 16: a pointer + a size
```

It's the string equivalent of `std::span`, and it's a good parameter type for functions that only read text.

The danger is the same as with references: the view doesn't keep the characters alive.

```cpp
std::string_view bad() {
    std::string s = "temporary";
    return s;          // warning: address of stack memory associated with local variable 's' returned
}                      // s is destroyed; the returned view dangles
```

Rule: use `std::string_view` for **parameters**. Use `std::string` when you need to **store** text or return it.

## Summary

```text
int a[5]               fixed size, no bounds check, decays to a pointer, can't be copied   -> avoid in new code
std::array<int, 5>     fixed size, knows its size, copyable, .at() checks bounds           -> fixed-size arrays
std::vector<int>       runtime size, growable                                              -> default "list"
std::span<const int>   non-owning view of any contiguous array                             -> parameters

const char* / char[]   null-terminated, no length, no safety                               -> only for C APIs and literals
std::string            owning, mutable, growable text                                      -> storing text
std::string_view       non-owning view of text                                             -> read-only parameters
```
