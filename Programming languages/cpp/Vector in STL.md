
A vector is a **dynamic array** from C++ Standard Template Library (STL) that can grow or shrink in size automatically. Unlike regular arrays with fixed size, vectors manage their own memory and resize when needed.

## Why Use Vector Over Array

- **Dynamic sizing**: No need to know size beforehand; it grows/shrinks automatically[^1]
- **Memory safety**: Built-in bounds checking with `at()` method[^2]
- **No size passing**: When passed to functions, vectors track their own size (no separate parameter needed)[^2]
- **STL integration**: Works seamlessly with algorithms like `sort()`, `find()`, etc[^2]

Use arrays only when size is fixed and known at compile time.[^3]

## How to Use Vectors

### Include and Declare

```cpp
#include <vector>
using namespace std;

vector<int> v1;              // empty vector, will need to initialize first
vector<int> v2(5);           // 5 elements (default value 0)
vector<int> v3(5, 12);       // 5 elements, all value 12
vector<int> v4 = {1,2,3,4,5}; // initialize with values
```


### Add Elements

```cpp
v1.push_back(10);    // adds 10 at end
v1.emplace_back(20); // faster than push_back, constructs in place
```


### Access Elements

```cpp
v4[^0];        // first element (no bounds check)
/*
If v4[0] is out of bounds, you get undefined behavior - the program doesn't crash or throw an error, it just accesses random memory
*/
v4.at(0);     // first element (throws exception if out of bounds)
v4.front();   // first element
v4.back();    // last element
```


### Modify Elements

```cpp
v4[^1] = 99;      // change second element
v4.at(4) = 77;   // change fifth element with bounds check
```


### Remove Elements

```cpp
v4.pop_back();                    // removes last element
v4.erase(v4.begin() + 1);         // removes element at index 1
v4.erase(v4.begin(), v4.begin()+3); // removes range [0,3)
v4.clear();                       // removes all elements
```

### Get size of the vector:

```C++
v4.size();      // number of elements
```

### Important Methods

```cpp

v4.capacity();  // allocated storage size
v4.empty();     // returns true if vector is empty
v4.resize(10);  // changes size to 10 elements
```


## Iterating Through Vector

### Index-based Loop

```cpp
for(int i = 0; i < v4.size(); i++) {
    cout << v4[i] << " ";
}
```


### Range-based Loop (Preferred)

```cpp
for(int element : v4) {
    cout << element << " ";
}
```


### Iterator Loop

```cpp
for(vector<int>::iterator it = v4.begin(); it != v4.end(); it++) {
    cout << *it << " ";
}
```


## Passing Vectors to Functions

### Pass by Value (Creates Copy)

```cpp
void func(vector<int> v) {
    // changes won't affect original
}
```


### Pass by Reference (Modifies Original)

```cpp
void func(vector<int>& v) {
    // changes affect original vector
}
```


### Pass by Const Reference (Read-only, No Copy)

```cpp
void func(const vector<int>& v) {
    // can't modify, no copy overhead
}
```


## 2D Vectors

```cpp
vector<vector<int>> matrix = {
    {1, 2, 3},
    {4, 5, 6},
    {7, 8, 9}
};

// Access element
int val = matrix[^1][^2];  // gets 6

// Iterate
for(int i = 0; i < matrix.size(); i++) {
    for(int j = 0; j < matrix[i].size(); j++) {
        cout << matrix[i][j] << " ";
    }
}
```

