
To understand Data Structures and Algorithms, we must first agree on how to measure them. In the academic tradition of CLRS (Cormen et al.), we don't measure an algorithm's speed in seconds—because a fast algorithm on an old laptop might take longer than a slow algorithm on a supercomputer. Instead, we measure **Growth**.

### 1. Asymptotic Notation: The Language of Efficiency

We use mathematical symbols to describe the "order of growth" of an algorithm's running time as the input size ($n$) becomes very large.

#### **Big-O Notation ($O$): The Upper Bound**

Think of $O(g(n))$ as a "worst-case" guarantee. If an algorithm is $O(n^2)$, it means the running time will grow **no slower** (or wont take more time) than a quadratic function. It provides an asymptotic upper bound.

- **Formal Definition:** $f(n) = O(g(n))$ if there exist positive constants $c$ and $n_0$ such that $0 \le f(n) \le cg(n)$ for all $n \ge n_0$.

#### **Big-Omega Notation ($\Omega$): The Lower Bound**

This provides an asymptotic **lower bound**. If an algorithm is $\Omega(n)$, it means it will take **at least** linear time. It’s the "best-case" floor.

- **Formal Definition:** $f(n) = \Omega(g(n))$ if there exist positive constants $c$ and $n_0$ such that $0 \le cg(n) \le f(n)$ for all $n \ge n_0$.

#### **Theta Notation ($\Theta$): The Tight Bound**

This is the most precise. If an algorithm is $\Theta(n \log n)$, it means it grows exactly at that rate (bounded above and below). We strive for $\Theta$ bounds because they describe the algorithm's behavior most accurately.

---

### 2. A Newbie's Guide to Time Complexity Classes

Here is the hierarchy of growth rates you will encounter, from most efficient to least efficient:

1. **$\Theta(1)$ - Constant Time:** The time taken is independent of $n$ (e.g., accessing an array element by index).
2. **$\Theta(\log n)$ - Logarithmic Time:** Usually seen in algorithms that "divide and conquer" (e.g., Binary Search).
3. **$\Theta(n)$ - Linear Time:** Time grows directly with $n$ (e.g., finding the maximum in an unsorted list).
4. **$\Theta(n \log n)$ - Linearithmic Time:** The gold standard for efficient sorting (e.g., Merge Sort, Heapsort).
5. **$\Theta(n^2)$ - Quadratic Time:** Time grows with the square of $n$ (e.g., Bubble Sort, Insertion Sort). Avoid this for large $n$.

![[Pasted image 20250603225929.png]]

### Visualization
```
For n = 100:
O(1)        = 1
O(log n)    = ~7
O(n)        = 100
O(n log n)  = ~700
O(n²)       = 10,000
O(2ⁿ)       = 1,267,650,600,228,229,401,496,703,205,376
```

### Examples with C++ Code

#### O(1) - Constant Time
```cpp
int getFirstElement(vector<int>& arr) {
    return arr[0];  // Always takes same time
}

void swap(int& a, int& b) {
    int temp = a;
    a = b;
    b = temp;  // Fixed number of operations
}
```

#### O(log n) - Logarithmic Time
```cpp
// Binary Search
int binarySearch(vector<int>& arr, int target) {
    int left = 0, right = arr.size() - 1;
    
    while(left <= right) {
        int mid = left + (right - left) / 2;
        
        if(arr[mid] == target) return mid;
        else if(arr[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    
    return -1;
}
// Each iteration cuts search space in half: n → n/2 → n/4 → ...
// Number of iterations = log₂(n)
```

#### O(n) - Linear Time
```cpp
int findMax(vector<int>& arr) {
    int maxVal = arr[0];
    
    for(int i = 1; i < arr.size(); i++) {  // n iterations
        if(arr[i] > maxVal) {
            maxVal = arr[i];
        }
    }
    
    return maxVal;
}
```

#### O(n log n) - Linearithmic Time
```cpp
// Merge Sort
void merge(vector<int>& arr, int left, int mid, int right) {
    vector<int> temp(right - left + 1);
    int i = left, j = mid + 1, k = 0;
    
    while(i <= mid && j <= right) {
        if(arr[i] <= arr[j]) temp[k++] = arr[i++];
        else temp[k++] = arr[j++];
    }
    
    while(i <= mid) temp[k++] = arr[i++];
    while(j <= right) temp[k++] = arr[j++];
    
    for(int i = 0; i < temp.size(); i++) {
        arr[left + i] = temp[i];
    }
}

void mergeSort(vector<int>& arr, int left, int right) {
    if(left < right) {
        int mid = left + (right - left) / 2;
        mergeSort(arr, left, mid);      // T(n/2)
        mergeSort(arr, mid + 1, right); // T(n/2)
        merge(arr, left, mid, right);   // O(n)
    }
}
// T(n) = 2T(n/2) + O(n) = O(n log n)
```

#### O(n²) - Quadratic Time
```cpp
// Bubble Sort
void bubbleSort(vector<int>& arr) {
    int n = arr.size();
    
    for(int i = 0; i < n; i++) {        // n iterations
        for(int j = 0; j < n - i - 1; j++) {  // n iterations
            if(arr[j] > arr[j + 1]) {
                swap(arr[j], arr[j + 1]);
            }
        }
    }
}
// Total: n × n = O(n²)
```

#### O(2ⁿ) - Exponential Time
```cpp
// Naive Fibonacci (VERY SLOW!)
int fibonacci(int n) {
    if(n <= 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);  // Each call makes 2 more calls
}
// Creates a binary tree of calls: 2^n nodes
```

#### O(n!) - Factorial Time
```cpp
// Generate all permutations
void permute(vector<int>& arr, int start, vector<vector<int>>& result) {
    if(start == arr.size()) {
        result.push_back(arr);
        return;
    }
    
    for(int i = start; i < arr.size(); i++) {
        swap(arr[start], arr[i]);
        permute(arr, start + 1, result);
        swap(arr[start], arr[i]);  // backtrack
    }
}
// For n elements: n! permutations
```


---

### 3. Implementation: The C++ Perspective

In C++, efficiency isn't just about the algorithm; it's about how you handle memory. For example, let's look at a simple search.

**The "Knuth" Way (Clean and Direct):**

```C++
#include <iostream>
#include <vector>

// Linear Search: O(n) Time Complexity
int findElement(const std::vector<int>& arr, int target) {
    for (size_t i = 0; i < arr.size(); ++i) {
        if (arr[i] == target) return i; // Found!
    }
    return -1; // Not found
}
```

- **Memory Management:** Notice the use of `const std::vector<int>&`. By passing by **reference**, we avoid copying the entire list into the function, which would take $O(n)$ time and space unnecessarily. This is the hallmark of a disciplined C++ programmer.
    

---

### 4. Randomization: Why "Average" Matters

Sometimes, an algorithm's worst case is rare but disastrous. **Randomized Algorithms** (like Randomized Quicksort) use a random number generator to ensure that we don't get stuck in that worst-case scenario. By picking a "random pivot," we ensure that on average, the algorithm performs beautifully regardless of how the input is ordered.

---

## How to Measure Time Complexity (Quick Rules to get estimates)

### Rule 1: Drop Constants
```cpp
// O(2n + 5) → O(n)
void example1(int n) {
    for(int i = 0; i < n; i++) {}     // n
    for(int i = 0; i < n; i++) {}     // n
    for(int i = 0; i < 5; i++) {}     // 5
}
// 2n + 5 ≈ O(n) for large n
```

### Rule 2: Drop Lower Order Terms
```cpp
// O(n² + n + 1) → O(n²)
void example2(int n) {
    for(int i = 0; i < n; i++) {           // n
        for(int j = 0; j < n; j++) {}      // n²
    }
    for(int k = 0; k < n; k++) {}          // n
}
// n² dominates, so O(n²)
```

### Rule 3: Sequential = Add
```cpp
// O(n + m)
void example3(int n, int m) {
    for(int i = 0; i < n; i++) {}  // O(n)
    for(int i = 0; i < m; i++) {}  // O(m)
}
// Total: O(n + m)
```

### Rule 4: Nested = Multiply
```cpp
// O(n × m)
void example4(int n, int m) {
    for(int i = 0; i < n; i++) {      // n times
        for(int j = 0; j < m; j++) {  // m times each
        }
    }
}
// Total: O(n × m)
```

### Rule 5: Halving/Doubling Input = Logarithmic
```cpp
// O(log n)
void example5(int n) {
    for(int i = 1; i < n; i *= 2) {  // i: 1 → 2 → 4 → 8 → ... → n
        // ...
    }
}

void example6(int n) {
    for(int i = n; i > 0; i /= 2) {  // i: n → n/2 → n/4 → ... → 1
        // ...
    }
}
```

---

## Tricks and Tips

### Trick 1: Identify Loop Patterns
```cpp
// Pattern: Single loop → O(n)
for(int i = 0; i < n; i++) { /* ... */ }

// Pattern: Nested same-size loops → O(n²)
for(int i = 0; i < n; i++) {
    for(int j = 0; j < n; j++) { /* ... */ }
}

// Pattern: Loop with halving → O(log n)
for(int i = 1; i < n; i *= 2) { /* ... */ }

// Pattern: Loop with recursion → Check Master Theorem
```

### Trick 2: Amortized Analysis
```cpp
// Vector push_back is O(1) amortized
vector<int> v;
for(int i = 0; i < n; i++) {
    v.push_back(i);  // Occasionally O(n), but O(1) on average
}
// Total: O(n), not O(n²)
```

### Trick 3: Two Pointers Pattern
```cpp
// O(n) - NOT O(n²) because each pointer moves at most n times total
bool hasPairWithSum(vector<int>& arr, int target) {
    int left = 0, right = arr.size() - 1;
    
    while(left < right) {  // At most n iterations combined
        int sum = arr[left] + arr[right];
        if(sum == target) return true;
        else if(sum < target) left++;
        else right--;
    }
    
    return false;
}
```

### Trick 4: Sliding Window Pattern
```cpp
// O(n) - Each element added/removed once
int maxSumSubarray(vector<int>& arr, int k) {
    int maxSum = 0, windowSum = 0;
    
    // Initial window
    for(int i = 0; i < k; i++) {
        windowSum += arr[i];
    }
    maxSum = windowSum;
    
    // Slide window
    for(int i = k; i < arr.size(); i++) {
        windowSum += arr[i] - arr[i - k];  // Add new, remove old
        maxSum = max(maxSum, windowSum);
    }
    
    return maxSum;
}
```

### Trick 5: Recognize Recursive Complexity
```cpp
// Count recursive calls and work per call

// Example 1: O(2^n)
void allSubsets(int index, int n) {
    if(index == n) return;
    allSubsets(index + 1, n);  // Include
    allSubsets(index + 1, n);  // Exclude
}
// Tree depth = n, branches = 2 → 2^n nodes

// Example 2: O(n)
void printN(int n) {
    if(n == 0) return;
    cout << n << " ";
    printN(n - 1);  // Single recursive call
}
// Call stack depth = n, O(1) work per call → O(n)
```

### Trick 6: HashMap Operations
```cpp
// Unordered_map: O(1) average, O(n) worst case
#include <unordered_map>

unordered_map<int, int> countFrequency(vector<int>& arr) {
    unordered_map<int, int> freq;
    
    for(int num : arr) {  // O(n)
        freq[num]++;      // O(1) average per operation
    }
    
    return freq;  // Total: O(n) average
}

// Map (ordered): O(log n) per operation
#include <map>

map<int, int> countFrequencySorted(vector<int>& arr) {
    map<int, int> freq;
    
    for(int num : arr) {  // O(n)
        freq[num]++;      // O(log n) per operation
    }
    
    return freq;  // Total: O(n log n)
}
```

---
# Substitution Method
The **substitution method** is a formal way to solve recurrences by "guessing" the form of the solution and then using **mathematical induction** to prove that the guess is correct.

It is a two-step process:

1. **Guess** the form of the solution (often based on experience or a recursion tree).
2. **Prove** it using induction to find the specific constants $c$ and $n_0$.

---

## A Step-by-Step Example

Let’s solve the recurrence:

$$T(n) = 2T(n/2) + n$$

_(This is the recurrence for Merge Sort)_.

### Step 1: Make a Guess

Looking at the structure, we guess that the solution is $T(n) = O(n \log n)$. This means we need to prove that $T(n) \le cn \log n$ for some constant $c$.

### Step 2: The Inductive Proof

We assume our guess holds for all values smaller than $n$ (specifically for $n/2$).

**1. The Assumption:** Assume $T(n/2) \le c(n/2) \log(n/2)$.

**2. Substitute into the Recurrence:**

Take the original equation and replace $T(n/2)$ with our assumption:

$$T(n) \le 2 [c(n/2) \log(n/2)] + n$$

**3. Simplify the Algebra:**

- Multiply the $2$ and the $n/2$:
    
    $$T(n) \le cn \log(n/2) + n$$
    
- Use the log property $\log(a/b) = \log a - \log b$:
    
    $$T(n) \le cn (\log n - \log 2) + n$$
    
- Since $\log_2 2 = 1$:
    
    $$T(n) \le cn \log n - cn(1) + n$$
    
    $$T(n) \le cn \log n - cn + n$$
    

**4. Show it Matches the Form:**

We want to show that $T(n) \le cn \log n$. Looking at our simplified equation:

$$T(n) \le cn \log n - (cn - n)$$

For this to be true, the term $(cn - n)$ must be **greater than or equal to zero**.

$$cn - n \ge 0 \implies cn \ge n \implies c \ge 1$$

So, as long as we pick a constant **$c \ge 1$**, the proof works!

---

## Common Pitfalls to Avoid

### 1. The Base Case

Don't forget that induction requires a base case (e.g., $T(1)$). In asymptotic notation, we can often skip $n=1$ if the function is undefined there (like $\log 1 = 0$) and start our proof at $n=2$ or $n=3$.

### 2. Missing the Constant

You cannot say $T(n) \le O(n^2)$ inside your induction. You **must** use the specific constant $c$ (e.g., $T(n) \le cn^2$). Without the constant, the algebra won't actually prove anything.

### 3. Being "Almost" Right

If you guess $T(n) = cn^2$ but end up with $cn^2 + n$, the proof fails . In that case, you must **strengthen the hypothesis** by subtracting a lower-order term, like $cn^2 - dn$.

For example:

To show why the substitution proof fails and how to fix it, we will use the **Substitution Method**, which involves guessing the form of the solution and then using mathematical induction to find the constants.

---

**1. Why $T(n) \le cn^2$ Fails**

We want to prove that $T(n) = O(n^2)$ by assuming $T(n) \le cn^2$ for some constant $c > 0$.

- **The Assumption:** Assume $T(k) \le ck^2$ for all $k < n$.

- **The Substitution:**
    $$T(n) = 4T(n/2) + n$$
    $$T(n) \le 4(c(n/2)^2) + n$$
    $$T(n) \le 4(cn^2/4) + n$$
    $$T(n) \le cn^2 + n$$

- **The Problem:** For the induction to hold, we need to show that $cn^2 + n \le cn^2$. However, since $n > 0$, this is **impossible** for any positive constant $c$. The extra $+n$ term "pollutes" the proof, and we cannot get back to our original form of $cn^2$.


**2. Subtracting a Lower-Order Term**

To fix this, we make a stronger inductive hypothesis by subtracting a lower-order term. Let’s assume $T(n) \le cn^2 - dn$ for constants $c > 0$ and $d > 0$.

- **The New Assumption:** Assume $T(k) \le ck^2 - dk$ for all $k < n$.
    
- **The Substitution:**
    
    $$T(n) = 4T(n/2) + n$$
    
    $$T(n) \le 4(c(n/2)^2 - d(n/2)) + n$$
    
    $$T(n) \le 4(cn^2/4 - dn/2) + n$$
    
    $$T(n) \le cn^2 - 2dn + n$$
    
- **The Goal:** We need $cn^2 - 2dn + n \le cn^2 - dn$.
    
- **Solving for $d$:**
    
    $$-2dn + n \le -dn$$
    
    $$n \le dn$$
    
    $$d \ge 1$$


**Conclusion**

By choosing $d \ge 1$, the inequality holds. We then pick a $c$ large enough to satisfy the base cases of the recurrence. This proves that $T(n) = O(n^2)$. A similar proof can be used for the lower bound $\Omega(n^2)$ to establish $T(n) = \Theta(n^2)$.

# Time Complexity in Recursions:
![[Pasted image 20260308231957.png]]
![[Pasted image 20260308232032.png]]

# Master Theorem

The **Master Theorem** provides a formula to determine the time complexity of **divide-and-conquer** recursive algorithms.

### The Formula

For recurrence relations of the form:
```
T(n) = a·T(n/b) + f(n)
```

Where:
- **a** ≥ 1: Number of subproblems
- **b** > 1: Factor by which input size is divided
- **f(n)**: Cost of work done outside recursive calls (dividing + combining)

### The Three Cases

Let **$c = log_b(a)$** (critical exponent)

#### Case 1: $f(n) = O(n^d)$ where d < c
**Result**: $$T(n) = Θ(n^c) = Θ(n^{(log_ba)})$$

*Recursion dominates*

#### Case 2: $f(n) = O(n^d)$ where d = c
**Result**: $$T(n) = Θ(n^c · log n) = Θ(n^d · log n)$$

*Recursion and combining are balanced*

#### Case 3: $f(n) = O(n^d)$ where d > c
**Result**: $$T(n) = Θ(f(n)) = Θ(n^d)$$

*Combining work dominates*

### Examples with C++ Code

#### Example 1: Binary Search
```cpp
int binarySearch(vector<int>& arr, int left, int right, int target) {
    if(left > right) return -1;
    
    int mid = left + (right - left) / 2;
    
    if(arr[mid] == target) return mid;
    else if(arr[mid] < target) return binarySearch(arr, mid + 1, right, target);
    else return binarySearch(arr, left, mid - 1, target);
}
```

**Recurrence**: T(n) = T(n/2) + O(1)
- a = 1 (one subproblem)
- b = 2 (divide by 2)
- f(n) = O(1) = O(n^0)
- c = log₂(1) = 0

Compare: d = 0, c = 0 → **Case 2**

**Result**: T(n) = Θ(n^0 · log n) = **O(log n)**

#### Example 2: Merge Sort
```cpp
void mergeSort(vector<int>& arr, int left, int right) {
    if(left >= right) return;
    
    int mid = left + (right - left) / 2;
    mergeSort(arr, left, mid);      // T(n/2)
    mergeSort(arr, mid + 1, right); // T(n/2)
    merge(arr, left, mid, right);   // O(n)
}
```

**Recurrence**: T(n) = 2T(n/2) + O(n)
- a = 2 (two subproblems)
- b = 2 (divide by 2)
- f(n) = O(n) = O(n^1)
- c = log₂(2) = 1

Compare: d = 1, c = 1 → **Case 2**

**Result**: T(n) = Θ(n^1 · log n) = **O(n log n)**

#### Example 3: Binary Tree Traversal
```cpp
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
};

void inorder(TreeNode* root) {
    if(!root) return;
    inorder(root->left);       // T(n/2)
    cout << root->val << " ";  // O(1)
    inorder(root->right);      // T(n/2)
}
```

**Recurrence**: T(n) = 2T(n/2) + O(1)
- a = 2
- b = 2
- f(n) = O(1) = O(n^0)
- c = log₂(2) = 1

Compare: d = 0, c = 1 → d < c → **Case 1**

**Result**: T(n) = Θ(n^1) = **O(n)**

#### Example 4: Matrix Multiplication (Naive)
```cpp
vector<vector<int>> multiply(vector<vector<int>>& A, vector<vector<int>>& B, int n) {
    if(n == 1) {
        return {{A[0][0] * B[0][0]}};
    }
    
    // Divide into 4 submatrices, multiply recursively (8 calls)
    // Then combine (O(n²) work)
    // ... (simplified for illustration)
}
```

**Recurrence**: T(n) = 8T(n/2) + O(n²)
- a = 8
- b = 2
- f(n) = O(n²)
- c = log₂(8) = 3

Compare: d = 2, c = 3 → d < c → **Case 1**

**Result**: T(n) = Θ(n³) = **O(n³)**

#### Example 5: Strassen's Matrix Multiplication
**Recurrence**: T(n) = 7T(n/2) + O(n²)
- a = 7
- b = 2
- f(n) = O(n²)
- c = log₂(7) ≈ 2.807

Compare: d = 2, c ≈ 2.807 → d < c → **Case 1**

**Result**: T(n) = **O(n^2.807)** (Better than naive!)

#### Example 6: Linear Search with Split
```cpp
bool linearSearch(vector<int>& arr, int start, int end, int target) {
    if(start > end) return false;
    
    int mid = (start + end) / 2;
    
    // Check both halves (not efficient, but illustrative)
    return linearSearch(arr, start, mid, target) || 
           linearSearch(arr, mid + 1, end, target);
}
```

**Recurrence**: T(n) = 2T(n/2) + O(1)
- a = 2
- b = 2
- f(n) = O(1) = O(n^0)
- c = log₂(2) = 1

Compare: d = 0, c = 1 → d < c → **Case 1**

**Result**: T(n) = **O(n)** (Same as normal linear search!)

### Master Theorem Summary Table

| Recurrence | a | b | f(n) | c = log_b(a) | Case | Result |
|------------|---|---|------|--------------|------|--------|
| T(n) = T(n/2) + O(1) | 1 | 2 | O(1) | 0 | 2 | O(log n) |
| T(n) = 2T(n/2) + O(n) | 2 | 2 | O(n) | 1 | 2 | O(n log n) |
| T(n) = 2T(n/2) + O(1) | 2 | 2 | O(1) | 1 | 1 | O(n) |
| T(n) = 4T(n/2) + O(n) | 4 | 2 | O(n) | 2 | 1 | O(n²) |
| T(n) = 8T(n/2) + O(n²) | 8 | 2 | O(n²) | 3 | 1 | O(n³) |
| T(n) = 7T(n/2) + O(n²) | 7 | 2 | O(n²) | ~2.807 | 1 | O(n^2.807) |
| T(n) = T(n/2) + O(n) | 1 | 2 | O(n) | 0 | 3 | O(n) |





# Quick Reference Cheatsheet

### Common Data Structure Operations

| Structure          | Access    | Search    | Insert    | Delete    |
| ------------------ | --------- | --------- | --------- | --------- |
| Array              | O(1)      | O(n)      | O(n)      | O(n)      |
| Vector (end)       | O(1)      | O(n)      | O(1)*     | O(1)      |
| Linked List        | O(n)      | O(n)      | O(1)      | O(1)      |
| Stack              | O(n)      | O(n)      | O(1)      | O(1)      |
| Queue              | O(n)      | O(n)      | O(1)      | O(1)      |
| Hash Table         | -         | O(1)*     | O(1)*     | O(1)*     |
| Binary Search Tree | O(log n)* | O(log n)* | O(log n)* | O(log n)* |
| Heap               | -         | O(n)      | O(log n)  | O(log n)  |

*Amortized or average case

### Sorting Algorithms

| Algorithm      | Best       | Average    | Worst      | Space    |
| -------------- | ---------- | ---------- | ---------- | -------- |
| Bubble Sort    | O(n)       | O(n²)      | O(n²)      | O(1)     |
| Selection Sort | O(n²)      | O(n²)      | O(n²)      | O(1)     |
| Insertion Sort | O(n)       | O(n²)      | O(n²)      | O(1)     |
| Merge Sort     | O(n log n) | O(n log n) | O(n log n) | O(n)     |
| Quick Sort     | O(n log n) | O(n log n) | O(n²)      | O(log n) |
| Heap Sort      | O(n log n) | O(n log n) | O(n log n) | O(1)     |
