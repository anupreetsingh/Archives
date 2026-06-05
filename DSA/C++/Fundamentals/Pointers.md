# Pointers in C++

```cpp
int main() {
```

## Static memory example

```cpp
int x = 42;         // x holds 42         (value)
                    // &x = 0x100 (say)    (address)

int* p = &x;        // p holds address of x i.e.&x = 0x100         (points to x)
int** pp = &p;      // pp holds &p = 0x200        (points to p)
int*** ppp = &pp;   // ppp holds &pp = 0x300      (points to pp)

// *p      = 42      (dereferences p to get value at address stored in p
// *pp     = p       = 0x100
// **pp    = *p      = 42
// *ppp    = pp      = 0x200
// **ppp   = *pp     = p = 0x100
// ***ppp  = **pp    = *p = 42
```

## Dynamic memory Allocation of Variable example

Dynamic memory allocation using `new` returns an address, in this case it will be stored in an int pointer named `dy_p`.

```cpp
int* dy_p = new int;      // Allocates memory on heap for 1 int, dy_p is a int pointer that stores that address
*dy_p = 99;               // *dy_p dereferences the pointer takes it to the int variable to store the value 99

int** dy_pp = new int*;   // Allocates memory on heap for pointer to int
*dy_pp = dy_p;            // Stores address of dy_p in dy_pp

int*** dy_ppp = new int**;// Allocates memory for pointer to pointer to int
*dy_ppp = dy_pp;          // Stores address of dy_pp in dy_ppp

// Now:
// *dy_p        = 99            //derefencing the pointer to int gives us the int
// *dy_pp       = dy_p      (address)
// **dy_pp      = *dy_p     = 99 //dereferencing the pointer to int pointer twice gives us the int
// *dy_ppp      = dy_pp     (address)
// **dy_ppp     = *dy_pp    = dy_p
// ***dy_ppp    = **dy_pp   = *dy_p = 99 //dereferencing the pointer to pointer to int pointer thrice gives us the int

// Cleanup (deallocate memory)
delete dy_p;
delete dy_pp;
delete dy_ppp;
```

## Dynamic memory Allocation of integer array Example

```cpp
int size = 5;
// Dynamically allocate array of 5 integers on heap
int* arr = new int[size];  //arr is an int pointer storing address for the first element of the array
                           //Allocating an array of size 5 is equivalent to allocating 5 integers in contiguous memory locations
// Assign values to the array using indexing
arr[0] = 10; //arr[0] is just a syntactic sugar for *(arr)
arr[1] = 20; //arr[1] is just a syntactic sugar for *(arr+1), incrementing an address moves it to the immediately next memory location
arr[2] = 30;
arr[3] = 40;
arr[4] = 50;

// Optional: using pointer to the array
int** arr_ptr = &arr;

// *arr_ptr  = arr      = 0x500
// **arr_ptr = *arr     = arr[0] = 10
// *(*arr_ptr + 1) = arr[1] = 20
// *(*arr_ptr + 4) = arr[4] = 50

// Cleanup
delete[] arr;   // use delete[] for arrays
return 0;
}
```
