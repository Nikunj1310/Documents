
C++ STL (Standard Template Library) is a powerful library of pre-built, reusable template classes and functions that help you efficiently manage data and implement common algorithms without writing everything from scratch.

## Core Components

STL consists of three main components:

**Containers** - Pre-built data structures to store and organize data. Examples include vectors (dynamic arrays), lists (linked lists), stacks, queues, sets, and maps.

**Algorithms** - Ready-to-use functions for common operations like sorting, searching, counting, and reversing data. These are defined in the `<algorithm>` header file.

**Iterators** - Pointer-like objects that let you access and navigate through elements in containers. They connect algorithms with containers.

## Why Use STL?

Instead of manually coding data structures like linked lists or binary search trees, STL provides:

- **Fast** - Optimized implementations tested rigorously
- **Reliable** - Industry-standard, bug-free code
- **Type-safe** - Works with any data type using templates
- **Efficient** - Compile-time optimization with zero runtime overhead


## Simple Example

```cpp
#include <vector>
#include <algorithm>
using namespace std;

vector<int> numbers = {5, 2, 8, 1, 3};  // Container
sort(numbers.begin(), numbers.end());    // Algorithm with iterators
```

This sorts the vector without writing sorting logic yourself.

![[Data Structures in C++#2. STL (Standard Template Library) Containers]]


# [[Linked Lists in STL]]

# [[Stack in STL]]

# [[Queue in STL]]

# Pair in STL
`pair` in C++ STL is a simple container that stores two values together as a single unit. The values can be of different data types.

## Basic Syntax

Include the `<utility>` header to use pair:[^3]

```cpp
#include <utility>
using namespace std;

pair<int, string> p;  // Declares a pair
```


## Creating Pairs

**Method 1: Direct initialization**

```cpp
pair<int, string> p1 = {1, "Apple"};
```

**Method 2: Using make_pair()**

```cpp
pair<int, string> p2 = make_pair(2, "Banana");
```

**Method 3: Assign later**

```cpp
pair<int, char> p3;
p3.first = 100;
p3.second = 'G';
```


## Accessing Values

Use `.first` and `.second` to access elements:[^1][^3]

```cpp
pair<int, string> p = {5, "Hello"};
cout << p.first;   // Output: 5
cout << p.second;  // Output: Hello
```


## Modifying Values

You can directly change the values:[^1]

```cpp
pair<int, string> p = {1, "Geeks"};
p.first = 2;
p.second = "ForGeeks";
```


## Important Functions

**swap()** - Swaps contents of two pairs:[^4][^5]

```cpp
pair<char, int> pair1 = make_pair('A', 1);
pair<char, int> pair2 = make_pair('B', 2);
pair1.swap(pair2);  // Now pair1 = ('B', 2), pair2 = ('A', 1)
```


## Comparison Operators

Pairs support comparison operators like `==`, `!=`, `<`, `>`:[^6][^3]

```cpp
pair<int, int> p1 = {1, 2};
pair<int, int> p2 = {1, 3};
if (p1 < p2) { /* true - compares first, then second */ }
```

Comparison is lexicographical: it first compares the first element, then the second if first elements are equal.[^1]

## Nested Pairs

You can create pairs inside pairs:[^4]

```cpp
pair<int, pair<int, int>> nested = {1, {2, 3}};
cout << nested.first;           // Output: 1
cout << nested.second.first;    // Output: 2
cout << nested.second.second;   // Output: 3
```


# [[Vector in STL]]

# map in STL

`map` in C++ STL is a container that stores data as **key-value pairs**, where each key is unique. It automatically keeps keys **sorted** and is implemented using a **Red-Black Tree** (balanced binary search tree).

## Basic Syntax

```cpp
#include <map>
using namespace std;

map<int, string> m;  // key type: int, value type: string
```


## Creating a Map

```cpp
// Empty map
map<int, string> m1;

// Initialize with values
map<int, string> m2 = {{1, "Apple"}, {2, "Banana"}, {3, "Cherry"}};
```


## Inserting Elements

**Method 1: Using `[]` operator**

```cpp
m[^1] = "One";
m[^2] = "Two";
```

**Method 2: Using `insert()`**

```cpp
m.insert({3, "Three"});
m.insert(make_pair(4, "Four"));
```

Note: `insert()` only adds if the key doesn't exist; `[]` will overwrite.[^1]

## Accessing Elements

```cpp
map<int, string> m = {{1, "Hello"}, {2, "World"}};

// Using []
cout << m[^1];  // Output: Hello

// Using at()
cout << m.at(2);  // Output: World
```

**Difference**: `at()` throws an error if key doesn't exist; `[]` creates a new entry with default value.[^4][^1]

## Modifying Values

```cpp
m[^1] = "Updated";
m.at(2) = "Changed";
```


## Deleting Elements

```cpp
// Erase by key
m.erase(2);

// Erase by iterator
m.erase(m.begin());

// Erase range
m.erase(m.begin(), m.end());
```


## Searching for a Key

**Using `find()`**

```cpp
if (m.find(3) != m.end()) {
    cout << "Key 3 exists";
}
```

**Using `count()`**

```cpp
if (m.count(3) > 0) {
    cout << "Key 3 exists";
}
```


## Iterating Through a Map

```cpp
map<int, string> m = {{1, "A"}, {2, "B"}, {3, "C"}};

// Range-based loop
for (auto& p : m) {
    cout << p.first << " -> " << p.second << endl;
}

// Iterator
for (auto it = m.begin(); it != m.end(); it++) {
    cout << it->first << " -> " << it->second << endl;
}
```


## Important Functions

| Function | Description |
| :-- | :-- |
| `size()` | Returns number of elements [^5] |
| `empty()` | Checks if map is empty [^5] |
| `clear()` | Removes all elements [^4] |
| `begin()` | Returns iterator to first element [^5] |
| `end()` | Returns iterator past last element [^5] |

## Time Complexity (Integer/Char Keys)

| Operation               | Time Complexity |
| :---------------------- | :-------------- |
| Insert                  | O(log n)        |
| Access (`[]` or `at()`) | O(log n)        |
| Find/Search             | O(log n)        |
| Erase                   | O(log n)        |

Where `n` = number of elements in the map.[^3]

## Time Complexity with String Keys

When the key is a **string**, time complexity changes because **string comparison is not O(1)**.[^6][^8]

**String comparison takes O(k)**, where `k` = average length of the string.[^8][^6]

**Adjusted complexities:**


| Operation | Time Complexity |
| :-- | :-- |
| Insert | O(k × log n) [^6][^8] |
| Access | O(k × log n) [^8] |
| Find/Search | O(k × log n) [^8] |
| Erase | O(k × log n) [^8] |

Where `k` = average string length, `n` = number of elements.[^6][^8]

**Example:**

```cpp
map<string, int> m;
m["apple"] = 1;  // Time: O(k × log n), k = 5
```

Each comparison during tree traversal compares strings character-by-character.[^8]

## Map vs Unordered_Map

- **map**: O(log n), keeps keys **sorted**, uses Red-Black Tree[^7][^3]
- **unordered_map**: O(1) average, **unsorted**, uses hashing[^7]

Use `map` when you need sorted order; use `unordered_map` for faster lookups.


# unordered_map in STL

`unordered_map` in C++ STL is a container that stores **key-value pairs** where keys are unique. Unlike `map`, it is **unordered** (no sorting) and uses a **hash table** for storage instead of a balanced tree.

## Basic Syntax

```cpp
#include <unordered_map>
using namespace std;

unordered_map<int, string> um;  // key: int, value: string
```


## Creating an Unordered Map

```cpp
// Empty map
unordered_map<int, string> um1;

// Initialize with values
unordered_map<int, string> um2 = {{1, "Apple"}, {2, "Banana"}, {3, "Cherry"}};

// Uniform initialization
unordered_map<string, int> um3{{"One", 1}, {"Two", 2}};
```


## Inserting Elements

**Using `[]` operator**

```cpp
um[^1] = "One";
um[^2] = "Two";
```

**Using `insert()`**

```cpp
um.insert({3, "Three"});
um.insert(make_pair(4, "Four"));

// Insert multiple
um.insert({{5, "Five"}, {6, "Six"}});
```

Note: `insert()` skips if key exists; `[]` overwrites existing value.[^10]

## Accessing Elements

```cpp
unordered_map<int, string> um = {{1, "Hello"}, {2, "World"}};

// Using []
cout << um[^1];  // Output: Hello

// Using at()
cout << um.at(2);  // Output: World
```

**Difference**: `at()` throws error if key doesn't exist; `[]` creates new entry with default value.[^10]

## Modifying Values

```cpp
um[^1] = "Updated";
um.at(2) = "Changed";
```


## Deleting Elements

```cpp
// Erase by key
um.erase(2);

// Erase by iterator
um.erase(um.begin());

// Clear all
um.clear();
```


## Searching for a Key

**Using `find()`**

```cpp
auto it = um.find(3);
if (it != um.end()) {
    cout << "Key 3 exists with value: " << it->second;
}
```

**Using `count()`**

```cpp
if (um.count(3) > 0) {
    cout << "Key 3 exists";
}
```


## Iterating Through Unordered Map

```cpp
unordered_map<int, string> um = {{1, "A"}, {2, "B"}, {3, "C"}};

// Range-based loop
for (auto& p : um) {
    cout << p.first << ": " << p.second << endl;
}

// Using iterators
for (auto it = um.begin(); it != um.end(); it++) {
    cout << it->first << ": " << it->second << endl;
}
```

Note: Elements are **not printed in any specific order**.[^9][^10]

## Important Functions

| Function | Description |
| :-- | :-- |
| `size()` | Returns number of elements [^10] |
| `empty()` | Checks if map is empty [^10] |
| `clear()` | Removes all elements [^10] |
| `find(key)` | Returns iterator to element with key [^10] |
| `count(key)` | Returns 1 if key exists, 0 otherwise [^10] |
| `erase(key)` | Removes element with key [^10] |

## Time Complexity (Integer/Char Keys)

`unordered_map` uses **hashing**, giving **average O(1)** complexity for most operations.[^13][^10]


| Operation | Average Case | Worst Case |
| :-- | :-- | :-- |
| Insert | O(1) [^10][^13] | O(n) [^10] |
| Access (`[]` or `at()`) | O(1) [^10][^13] | O(n) [^10] |
| Find/Search | O(1) [^10][^13] | O(n) [^10] |
| Erase | O(1) [^10][^13] | O(n) [^10] |

**Worst case O(n)** happens when many keys **hash to same bucket** (hash collisions).[^13][^10]

## Time Complexity with String Keys

When the key is a **string**, hashing and comparison require **O(k)** time, where `k` = average string length.[^10]

**Adjusted complexities:**


| Operation | Average Case | Worst Case |
| :-- | :-- | :-- |
| Insert | O(k) | O(n × k) |
| Access | O(k) | O(n × k) |
| Find/Search | O(k) | O(n × k) |
| Erase | O(k) | O(n × k) |

Where `k` = average string length, `n` = number of elements.

**Why?**

- Computing the hash of a string requires scanning all `k` characters[^10]
- Comparing strings during collision resolution also takes O(k) time

**Example:**

```cpp
unordered_map<string, int> um;
um["hello"] = 1;  // Hash computation: O(5), average insertion: O(5)
```


## Unordered_Map vs Map

| Feature | unordered_map | map |
| :-- | :-- | :-- |
| Implementation | Hash Table [^9][^10] | Red-Black Tree [^10] |
| Order | Unordered [^9] | Sorted by key [^10] |
| Average Access | O(1) [^10] | O(log n) [^10] |
| Worst Access | O(n) [^10] | O(log n) [^10] |

# Sets(Both ordered and unordered) in STL
## Set (Ordered)

`set` in C++ STL stores **unique elements in sorted order**. It's implemented using a **Red-Black Tree** (balanced binary search tree).[^1][^2][^3][^4]

### Basic Syntax

```cpp
#include <set>
using namespace std;

set<int> s;  // Creates an empty set of integers
```


### Creating a Set

```cpp
// Empty set
set<int> s1;

// Initialize with values
set<int> s2 = {5, 2, 8, 1, 3};  // Auto-sorted: {1, 2, 3, 5, 8}

// Using initializer list
set<string> s3{"apple", "banana", "cherry"};
```


### Inserting Elements

```cpp
set<int> s;

// Using insert()
s.insert(10);
s.insert(5);
s.insert(10);  // Duplicate ignored, not inserted

// Using emplace() (constructs in-place)
s.emplace(20);
```

**Note**: Duplicates are **automatically ignored**.[^5][^1]

### Accessing Elements

Sets don't support direct indexing (no `[]` operator). Use **iterators** or **find()**:[^1]

```cpp
set<int> s = {1, 2, 3, 5, 8};

// Using find()
auto it = s.find(3);
if (it != s.end()) {
    cout << "Found: " << *it;
}

// Check if element exists
if (s.count(5) > 0) {
    cout << "5 exists";
}
```


### Deleting Elements

```cpp
// Erase by value
s.erase(5);

// Erase by iterator
s.erase(s.find(3));

// Clear all
s.clear();
```


### Iterating Through a Set

```cpp
set<int> s = {1, 2, 3, 5, 8};

// Range-based loop (sorted order)
for (int x : s) {
    cout << x << " ";  // Output: 1 2 3 5 8
}

// Using iterators
for (auto it = s.begin(); it != s.end(); it++) {
    cout << *it << " ";
}
```


### Important Functions

| Function | Description |
| :-- | :-- |
| `size()` | Returns number of elements [^1] |
| `empty()` | Checks if set is empty [^1] |
| `clear()` | Removes all elements [^1] |
| `find(val)` | Returns iterator to element [^1] |
| `count(val)` | Returns 1 if exists, 0 otherwise [^1] |
| `erase(val)` | Removes element [^1] |

### Time Complexity (Integer/Char)

| Operation | Time Complexity |
| :-- | :-- |
| Insert | O(log n) [^1][^4] |
| Find/Search | O(log n) [^1][^4] |
| Erase | O(log n) [^1][^4] |
| Access (iterator) | O(log n) [^1] |

### Time Complexity with String Keys

When elements are **strings**, comparison takes **O(k)** time, where `k` = average string length.

**Adjusted complexities:**


| Operation | Time Complexity |
| :-- | :-- |
| Insert | O(k × log n) |
| Find/Search | O(k × log n) |
| Erase | O(k × log n) |

Where `k` = average string length, `n` = number of elements.

***

## Unordered_Set

`unordered_set` stores **unique elements in no particular order**. It's implemented using a **hash table**.[^4][^6][^7]

### Basic Syntax

```cpp
#include <unordered_set>
using namespace std;

unordered_set<int> us;  // Creates empty unordered_set
```


### Creating an Unordered Set

```cpp
// Empty unordered_set
unordered_set<int> us1;

// Initialize with values
unordered_set<int> us2 = {5, 2, 8, 1, 3};  // No sorting

// Using initializer list
unordered_set<string> us3{"apple", "banana", "cherry"};
```


### Inserting Elements

```cpp
unordered_set<int> us;

// Using insert()
us.insert(10);
us.insert(5);
us.insert(10);  // Duplicate ignored

// Using emplace()
us.emplace(20);
```


### Accessing Elements

```cpp
unordered_set<int> us = {1, 2, 3, 5, 8};

// Using find()
auto it = us.find(3);
if (it != us.end()) {
    cout << "Found: " << *it;
}

// Check existence
if (us.count(5) > 0) {
    cout << "5 exists";
}
```


### Deleting Elements

```cpp
// Erase by value
us.erase(5);

// Erase by iterator
us.erase(us.find(3));

// Clear all
us.clear();
```


### Iterating Through Unordered Set

```cpp
unordered_set<int> us = {1, 2, 3, 5, 8};

// Range-based loop (random order)
for (int x : us) {
    cout << x << " ";  // Output: 8 2 5 1 3 (order varies)
}

// Using iterators
for (auto it = us.begin(); it != us.end(); it++) {
    cout << *it << " ";
}
```


### Important Functions

Same as `set`: `size()`, `empty()`, `clear()`, `find()`, `count()`, `erase()`.[^7]

### Time Complexity (Integer/Char)

| Operation | Average Case | Worst Case |
| :-- | :-- | :-- |
| Insert | O(1) [^6][^8] | O(n) [^8] |
| Find/Search | O(1) [^6][^8] | O(n) [^8] |
| Erase | O(1) [^6][^8] | O(n) [^8] |

**Worst case O(n)** occurs when many elements **hash to the same bucket**.[^8][^6]

### Time Complexity with String Elements

When elements are **strings**, hashing and comparison take **O(k)** time, where `k` = average string length.

**Adjusted complexities:**


| Operation | Average Case | Worst Case |
| :-- | :-- | :-- |
| Insert | O(k) | O(n × k) |
| Find/Search | O(k) | O(n × k) |
| Erase | O(k) | O(n × k) |

**Why?**

- Computing hash of a string requires scanning all `k` characters
- Comparing strings during collision resolution also takes O(k)

***

## Set vs Unordered_Set

| Feature | set | unordered_set |
| :-- | :-- | :-- |
| Implementation | Red-Black Tree [^2][^4] | Hash Table [^7] |
| Order | Sorted [^1][^4] | Unordered [^4][^7] |
| Average Search | O(log n) [^4] | O(1) [^4][^8] |
| Worst Search | O(log n) [^4] | O(n) [^8] |

**When to use:**

- Use `set` when you need **sorted elements** or guaranteed O(log n)[^4]
- Use `unordered_set` when you need **fast lookups** and don't care about order


# Multiset in STL

`multiset` in C++ STL stores **elements in sorted order and allows duplicates**. It's implemented using a **Red-Black Tree** (balanced binary search tree), just like `set`.[^1][^2][^3]

## Basic Syntax

```cpp
#include <set>
using namespace std;

multiset<int> ms;  // Creates an empty multiset
```


## Creating a Multiset

```cpp
// Empty multiset
multiset<int> ms1;

// Initialize with values (duplicates allowed)
multiset<int> ms2 = {5, 3, 3, 1, 5};  // Stores as {1, 3, 3, 5, 5}

// Using initializer list
multiset<string> ms3{"apple", "banana", "apple"};
```


## Inserting Elements

```cpp
multiset<int> ms;

// Using insert() - duplicates are stored
ms.insert(5);
ms.insert(3);
ms.insert(3);  // Duplicate stored (unlike set)
ms.insert(1);

// Result: {1, 3, 3, 5}
```

**Key difference from `set`**: `multiset` **allows and stores duplicates**.[^3][^4][^1]

## Accessing Elements

No direct indexing. Use **iterators** or **find()**:[^1][^2]

```cpp
multiset<int> ms = {1, 3, 3, 5, 5};

// Using find() - returns iterator to first occurrence
auto it = ms.find(3);
if (it != ms.end()) {
    cout << "Found: " << *it;  // Output: 3
}

// Count occurrences
cout << ms.count(3);  // Output: 2
```


## Deleting Elements

```cpp
multiset<int> ms = {1, 3, 3, 5, 5};

// Erase all occurrences of value
ms.erase(3);  // Removes both 3's

// Erase single occurrence (using iterator)
auto it = ms.find(5);
ms.erase(it);  // Removes only one 5

// Clear all
ms.clear();
```

**Important**: `erase(value)` removes **all occurrences**; `erase(iterator)` removes **only one**.[^1]

## Iterating Through a Multiset

```cpp
multiset<int> ms = {5, 3, 3, 1};

// Range-based loop (sorted order with duplicates)
for (int x : ms) {
    cout << x << " ";  // Output: 1 3 3 5
}

// Using iterators
for (auto it = ms.begin(); it != ms.end(); it++) {
    cout << *it << " ";
}
```


## Important Functions

| Function | Description |
| :-- | :-- |
| `size()` | Returns total number of elements (including duplicates) [^2][^5] |
| `empty()` | Checks if multiset is empty [^2] |
| `clear()` | Removes all elements [^2] |
| `find(val)` | Returns iterator to first occurrence [^1] |
| `count(val)` | Returns number of occurrences [^1][^2] |
| `erase(val)` | Removes all occurrences of value [^1] |

## Time Complexity (Integer/Char)

| Operation | Time Complexity |
| :-- | :-- |
| Insert | O(log n) [^1] |
| Find/Search | O(log n) [^1] |
| Erase (by value) | O(log n + k) [^1] |
| Count | O(log n + k) [^1] |

Where `n` = total elements, `k` = number of occurrences of the element.

## Time Complexity with String Elements

When elements are **strings**, comparison takes **O(m)** time, where `m` = average string length.

**Adjusted complexities:**

[^6][^7]

| Operation | Time Complexity |
| :-- | :-- |
| Insert | O(m × log n) |
| Find/Search | O(m × log n) |
| Erase | O(m × (log n + k)) |
| Count | O(m × (log n + k)) |

Where `m` = average string length, `n` = total elements, `k` = occurrences.

***

## Unordered_Multiset

`unordered_multiset` stores **elements in no particular order and allows duplicates**. It's implemented using a **hash table**.[^6][^7]

### Basic Syntax

```cpp
#include <unordered_set>[^6][^7]
using namespace std;

unordered_multiset<int> ums;
```


### Operations

All operations are similar to `multiset`, but elements are **not sorted**.[^7]

### Time Complexity

| Operation | Average Case | Worst Case |
| :-- | :-- | :-- |
| Insert | O(1) [^6][^7] | O(n) [^6] |
| Find/Search | O(1) [^6][^7] | O(n) [^6] |
| Erase | O(1) [^6][^7] | O(n) [^6] |

**With string elements**: Multiply by **O(m)** where `m` = average string length.

***

## Multiset: Ordered or Unordered?

**`multiset` is ORDERED** (sorted).[^4][^3][^2][^1]

There are **two types**:

1. **`multiset`** - Ordered (sorted), allows duplicates
2. **`unordered_multiset`** - Unordered (unsorted), allows duplicates

***

## Complete Comparison Table

| Feature | set | multiset | unordered_set | unordered_multiset |
| :-- | :-- | :-- | :-- | :-- |
| Order | Sorted [^3] | Sorted [^2][^3][^4] | Unsorted | Unsorted [^6][^7] |
| Duplicates | No [^3][^4] | Yes [^2][^3][^4] | No | Yes [^6][^7] |
| Implementation | Red-Black Tree [^3] | Red-Black Tree [^1] | Hash Table | Hash Table [^7] |
| Average Insert | O(log n) | O(log n) [^1] | O(1) | O(1) [^7] |
| Average Search | O(log n) | O(log n) [^1] | O(1) | O(1) [^7] |

**When to use `multiset`:**

- Need **sorted elements** with **duplicate values**[^3]
- Frequency counting with sorted order[^3]
- Examples: storing scores with duplicates, maintaining sorted history



# ITERATOR in STL

Iterators in C++ STL are pointer-like objects that let you access and navigate through elements in containers like vectors, sets, and maps. Think of them as smart pointers that know how to move through container elements.[^1][^2]

## Basic Syntax

```cpp
#include <vector>
using namespace std;

vector<int> v = {10, 20, 30, 40};
vector<int>::iterator it;  // Declare iterator
```

Or use `auto` to simplify:[^1]

```cpp
auto it = v.begin();  // Compiler deduces type
```


## Essential Functions

**begin()** - Returns iterator pointing to the first element[^3][^1]

**end()** - Returns iterator pointing to the position *after* the last element (not the last element itself)[^3][^1]

**cbegin()** - Returns constant iterator to beginning (read-only)[^1]

**cend()** - Returns constant iterator to end (read-only)[^1]

## Basic Operations

**Dereferencing** - Use `*` to access the value:[^1]

```cpp
auto it = v.begin();
cout << *it;  // Output: 10 (first element)
*it = 100;    // Modify value
```

**Incrementing/Decrementing**:[^1]

```cpp
++it;  // Move to next element
--it;  // Move to previous element
```

**Arithmetic** (for random access iterators like vector):[^1]

```cpp
it = it + 2;  // Jump forward 2 positions
it = it - 1;  // Jump backward 1 position
```

**Comparison**:[^1]

```cpp
if (it1 == it2) { /* same position */ }
if (it != v.end()) { /* not at end */ }
```


## Traversing Containers

**Method 1: Traditional loop**:[^1]

```cpp
vector<int> v = {10, 20, 30, 40};
for (auto it = v.begin(); it != v.end(); ++it) {
    cout << *it << " ";
}
```

**Method 2: Range-based for** (simpler):[^4]

```cpp
for (const auto &element : v) {
    cout << element << " ";
}
```


## Iterator Types

Based on capabilities, iterators are classified into five types:[^2][^1]

**1. Input Iterator** - Read only, single pass (e.g., reading from input stream)[^1]

**2. Output Iterator** - Write only, single pass (e.g., writing to output stream)[^1]

**3. Forward Iterator** - Read/write, multiple passes (e.g., `forward_list`, `unordered_set`)[^1]

**4. Bidirectional Iterator** - Can move forward and backward (e.g., `list`, `set`, `map`)[^2][^1]

**5. Random Access Iterator** - Can jump to any position directly (e.g., `vector`, `deque`)[^2][^1]

## Common Container Examples

**Vector**:[^1]

```cpp
vector<int> v = {10, 20, 30};
auto it = v.begin();
cout << *it;  // 10
```

**Set**:[^1]

```cpp
set<int> s = {10, 20, 30, 40};
auto it = s.begin();
while (it != s.end()) {
    cout << *it << " ";
    ++it;
}
```

**Map**:

```cpp
map<string, int> m = {{"apple", 5}, {"banana", 3}};
for (auto it = m.begin(); it != m.end(); ++it) {
    cout << it->first << ": " << it->second << endl;
}
```


## Accessing First and Last Elements

**First element**:[^5]

```cpp
int first = *v.begin();
```

**Last element** (not using `end()` directly):

```cpp
int last = *(v.end() - 1);  // Or v.back()
```

Remember: `end()` points *past* the last element, so you must subtract 1 or use decrement.


