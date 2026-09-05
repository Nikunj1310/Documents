**Expected to know:** [[Functions]]
**Topics Covered:**
**Tags:** [[Functions]]

![[Syntax of C++#**6. Function Overloading**]]

# Functions in C++: Basics and Key Features (for C Programmers)

If you know functions in C, you already understand the fundamentals of function declaration, definition, and calling. C++ retains these core concepts but introduces some powerful enhancements, especially with parameter passing and overloading. Here’s a comprehensive guide to functions in C++:

---
## 1. Function Declaration, Definition, and Calling

**Declaration (Prototype):**

- Tells the compiler about the function name, return type, and parameters.
- Syntax:
```cpp
return_type function_name(parameter_list);
```
Example:
```cpp
int add(int, int);
```

**Definition:**
- Contains the actual code (body) of the function.
- Syntax:
```cpp
return_type function_name(parameter_list) 
{     
	// function body 
}
```
Example:
```cpp
int add(int a, int b) 
{     
	return a + b; 
}
```
**Calling:**
- Use the function’s name and pass required arguments.
```cpp
int result = add(3, 4);
```

**Note:**  
If you define a function after `main()`, you must declare its prototype before `main()`.

---
## 2. Function Parameters and Arguments
- **Parameters** are variables in the function declaration/definition.
- **Arguments** are actual values passed to the function during a call.

Example:
```cpp
void greet(std::string name) 
{     
	std::cout << "Hello, " << name << "!"; 
} 

greet("Alice"); // "Alice" is the argument[2].
```
---
## 3. Return Type
- Functions can return any data type (including `void` for no return).
- Use `return value;` to return a result.

---
## 4. Function Overloading (C++-only Feature)
C++ allows multiple functions with the same name but different parameter types or counts:
```cpp
int sum(int a, int b) 
{ 
	return a + b; 
} 

double sum(double a, double b) 
{ 
	return a + b; 
}
```
The compiler chooses the correct function based on the arguments used.

---
## 5. Default Arguments (C++ only Feature)
You can specify default values for parameters:
```cpp
void display(int a, int b = 10) 
{     
	std::cout << a << " " << b; 
} 

display(5);      // Outputs: 5 10 
display(5, 20);  // Outputs: 5 20
```

---
## 6. Parameter Passing: Value, Pointer, Reference
- **Pass by Value:** (default) Function gets a copy; changes don’t affect the original.
- **Pass by Pointer:** Use pointers (`*`) to allow the function to modify the original variable.
- **Pass by Reference (C++-only):** Use `&` to pass a reference, allowing direct modification without pointers:
    
```cpp
void increment(int &x) { x++; } 

int a = 5; 
increment(a); // a is now 6.
```

---
## 7. Inline Functions (C++-only)
Suggests to the compiler to insert the function code at the call site for efficiency (for small, frequently called functions):
```cpp
inline int square(int x) 
{ 
	return x * x; 
}
```
---
## 8. Recursion
Functions in C++ can call themselves, just as in C.

---
## 9. Function Templates (Generic Functions, C++-only)
Templates let you write functions that work with any data type:
```cpp
template <typename T> T max(T a, T b) 
{     
	return (a > b) ? a : b; 
}
```
Now `max` works for `int`, `double`, etc.

---
## 10. Example: All Features Together
```cpp
#include <iostream> 
using namespace std; 

// Function prototype with default argument 
int add(int a, int b = 10); 

// Function template 
template <typename T> T multiply(T a, T b) 
{ 
	return a * b; 
} 

int main() 
{     
	cout << add(5) << endl;         
	// Uses default argument (outputs 15)    
	cout << add(5, 7) << endl;      
	// Outputs 12    
	cout << multiply(2, 3) << endl; // Outputs 6    
	cout << multiply(2.5, 3.0) << endl; // Outputs 7.5    
	return 0; 
} 
// Function definition 
int add(int a, int b) 
{     
	return a + b; 
}
```
---
