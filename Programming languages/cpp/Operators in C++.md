**Expected to know:** [[Introduction to Programming]]
**Topics Covered:**
**Tags:** [[Operators]]

[[Operators]] mentions all the operators that are used generally in majority of the Programming language.
In the bitwise Operators, << and >> has a special work when it comes to input and output.

---
## 1. `new` and `delete` Operators

- **Purpose:** Dynamic memory allocation and deallocation.
- **Syntax:**    
```cpp
int* p = new int;      // Allocates memory for an int 
delete p;              // Frees the memory 
int* arr = new int[10]; // Allocates array of 10 ints 
delete[] arr;           // Frees the array
```
- **Details:**
    - `new` calls constructors and allocates memory on the heap, and returns the pointer to that memory.
    - `delete` calls destructors and frees memory.
    - These replace `malloc` and `free` from C with type safety and constructor/destructor support.

---
## **2. `and`, `or`, `not` (Alternative Logical Operators)**

- **Purpose:** Keyword alternatives to `&&`, `||`, and `!`.
- **Syntax:**
```cpp
if (a and b) 
{ ... }    // Same as if (a && b) 

if (a or b) 
{ ... }     // Same as if (a || b) 

if (not a) 
{ ... }      // Same as if (!a)`
```
- **Details:**
    - These are standard keywords in C++ (and C99), improving code readability for some programmers.

---
## **3. Operator Overloading**
- **Purpose:** Allow user-defined types (classes) to define custom behavior for operators (`+`, `-`, `*`, `[]`, etc.).
- **Syntax:**
```cpp
class Complex 
{ 
public:     
	Complex operator+(const Complex& rhs) 
	{ ... } 
};
```

- **Details:**
    - Enables intuitive usage of operators with user-defined types, e.g., `a + b` for complex numbers.
    - Not available for all operators (e.g., `.` cannot be overloaded).

---

## Bit-Wise operators 