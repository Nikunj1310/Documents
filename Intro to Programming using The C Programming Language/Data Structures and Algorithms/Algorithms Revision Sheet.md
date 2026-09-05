# Sorting Algorithms
## ![[BubbleSort Rev]]
## ![[InsertionSort Rev]]
## ![[QuickSort Rev]]
## ![[MergeSort Rev]]
## ![[Binary Search Rev]]
# Array Related Algorithm

# Kadane's Algorithm

**Problem:** Given an array (with possibly negative numbers), find the contiguous subarray with the largest sum.

## Core Idea

At each index, decide: _extend the previous subarray_ or _start fresh from here_. If the running sum becomes negative, it can only drag down future sums — so drop it and restart.

## Recurrence

```
maxEndingHere = max(arr[i], maxEndingHere + arr[i])
maxSoFar      = max(maxSoFar, maxEndingHere)
```

- `maxEndingHere` → best sum of subarray **ending at index i**
- `maxSoFar` → best sum seen **overall**

## Code (Java-style, works same in C++/Python)

```cpp
int kadane(vector<int>& arr) {
    int maxSoFar = arr[0];
    int maxEndingHere = arr[0];

    for (int i = 1; i < arr.size(); i++) {
        maxEndingHere = max(arr[i], maxEndingHere + arr[i]);
        maxSoFar = max(maxSoFar, maxEndingHere);
    }
    return maxSoFar;
}
```

## Dry Run

`arr = [-2, 1, -3, 4, -1, 2, 1, -5, 4]`

|i|arr[i]|maxEndingHere|maxSoFar|
|---|---|---|---|
|0|-2|-2|-2|
|1|1|max(1, -2+1)=1|1|
|2|-3|max(-3, 1-3)=-2|1|
|3|4|max(4, -2+4)=4|4|
|4|-1|max(-1,4-1)=3|4|
|5|2|max(2,3+2)=5|5|
|6|1|max(1,5+1)=6|6|
|7|-5|max(-5,6-5)=1|6|
|8|4|max(4,1+4)=5|6|

**Answer: 6** → subarray `[4, -1, 2, 1]`

## Complexity

- Time: **O(n)** — single pass
- Space: **O(1)**

## Edge Cases to Remember

- **All negative numbers** → answer is the _least negative_ single element (works correctly since we init `maxSoFar = arr[0]`, not 0). Don't initialize to 0 or you'll wrongly allow an empty subarray.
- **Empty subarray not allowed** → must pick at least one element.
- To also **track the subarray itself** (not just sum), keep a `start` index that resets whenever you restart (`maxEndingHere = arr[i]`), and update `bestStart`/`bestEnd` whenever `maxSoFar` updates.

## Variants Worth Knowing

- **Maximum Product Subarray** → same idea, but track both max _and_ min ending here (since a negative × negative can flip small to large).
- **Circular array version** → `max(Kadane(arr), totalSum - MinSubarraySum(arr))`, handling all-negative edge case separately.
- **Print the subarray** → track indices as shown above.

That's the whole algorithm — one pass, one simple choice at each step: extend or restart.

# Dutch National Flag Algorithm:
 The **Dutch National Flag Algorithm**, proposed by Edsger W. Dijkstra, is the optimal way to sort an array consisting of only three distinct elements (typically represented as `0`, `1`, and `2`).

Instead of using a standard sorting algorithm that takes $O(N \log N)$ time, or counting the elements which requires two passes, this algorithm sorts the array in a **single pass** with $O(N)$ time complexity and $O(1)$ auxiliary space.

Before diving into the code, stepping through the visual process helps make the pointer logic click:

## The Logic (Three Pointers)

The algorithm divides the array into four sections using three pointers: `low`, `mid`, and `high`.

1. **`[0` to `low - 1]`:** Contains all `0`s.
    
2. **`[low` to `mid - 1]`:** Contains all `1`s.
    
3. **`[mid` to `high]`:** The unknown/unsorted elements we are currently evaluating.
    
4. **`[high + 1` to `N - 1]`:** Contains all `2`s.
    

You iterate through the array using the `mid` pointer. Depending on the value at `arr[mid]`, you take one of three actions:

- **If `arr[mid] == 0`:** Swap `arr[low]` and `arr[mid]`. Increment both `low` and `mid`. (You've successfully moved a `0` to the start and a `1` to the middle).
    
- **If `arr[mid] == 1`:** Increment `mid`. (It's already in the correct middle section, just move forward).
    
- **If `arr[mid] == 2`:** Swap `arr[mid]` and `arr[high]`. Decrement `high`. **Do not increment `mid`.** (You've moved a `2` to the end, but the element swapped _into_ the `mid` position still needs to be evaluated).
    

## C++ Implementation

Here is how that logic translates directly into code:

```C++
#include <iostream>
#include <vector>

using namespace std;

void sort012(vector<int>& arr) {
    int low = 0;
    int mid = 0;
    int high = arr.size() - 1;

    // Loop continues until the unknown region is completely evaluated
    while (mid <= high) {
        if (arr[mid] == 0) {
            swap(arr[low], arr[mid]);
            low++;
            mid++;
        } 
        else if (arr[mid] == 1) {
            mid++;
        } 
        else { // arr[mid] == 2
            swap(arr[mid], arr[high]);
            high--;
        }
    }
}

int main() {
    vector<int> arr = {2, 0, 2, 1, 1, 0};
    
    sort012(arr);
    
    cout << "Sorted array: ";
    for(int num : arr) {
        cout << num << " ";
    }
    cout << endl;
    
    return 0;
}
```

### Complexity Breakdown

- **Time Complexity:** $O(N)$. The algorithm traverses the array exactly once. Each step either increments `mid` or decrements `high`, shrinking the search space by one.
    
- **Space Complexity:** $O(1)$. It modifies the array completely in place without requiring any additional data structures.


# Moore's Voting Algorithm:

**Problem:** Given an array, find the element that appears **more than n/2 times** (the majority element). Assumes a majority element exists (unless stated otherwise — then verify).

## Core Idea

Think of it as a **cancellation game**: if two _different_ elements fight, they cancel each other out (count--). Since the majority element occurs more than half the time, it can never be fully cancelled out — whatever survives at the end is the candidate.

## Algorithm (2 variables: `candidate`, `count`)

1. If `count == 0` → pick current element as new `candidate`.
2. If current element == `candidate` → `count++`.
3. Else → `count--`.

## Code

```cpp
int majorityElement(vector<int>& arr) {
    int candidate = arr[0];
    int count = 0;

    for (int num : arr) {
        if (count == 0) candidate = num;
        count += (num == candidate) ? 1 : -1;
    }
    return candidate;
}
```

## Dry Run

`arr = [2, 2, 1, 1, 1, 2, 2]`

|num|count==0?|candidate|count|
|---|---|---|---|
|2|yes → cand=2|2|1|
|2|no|2|2|
|1|no|2|1|
|1|no|2|0|
|1|yes → cand=1|1|1|
|2|no|1|0|
|2|yes → cand=2|2|1|

**Answer: candidate = 2** ✔ (appears 4/7 times, majority)

## Complexity

- Time: **O(n)** — single pass
- Space: **O(1)**

## Important Gotcha

If a majority element (**> n/2**) isn't guaranteed to exist, you **must verify** with a second pass:

```cpp
int freq = count(arr.begin(), arr.end(), candidate);
if (freq > arr.size() / 2) return candidate;
else return -1; // no majority element
```

Without this check, the algorithm can output garbage on inputs with no true majority.

## Variant: Moore's Voting for n/3 (up to 2 majority elements)

Find all elements appearing **more than n/3 times** (there can be at most 2 such elements).

- Maintain **2 candidates + 2 counts** (`cand1, cand2, count1, count2`)
- Same cancellation logic, but skip cancelling against yourself
- **Verify both candidates** with a second pass (mandatory here — false positives are common)

```cpp
vector<int> majorityElementN3(vector<int>& arr) {
    int cand1 = 0, cand2 = 1, count1 = 0, count2 = 0;

    for (int num : arr) {
        if (num == cand1) count1++;
        else if (num == cand2) count2++;
        else if (count1 == 0) { cand1 = num; count1 = 1; }
        else if (count2 == 0) { cand2 = num; count2 = 1; }
        else { count1--; count2--; }
    }

    count1 = count2 = 0;
    for (int num : arr) {
        if (num == cand1) count1++;
        else if (num == cand2) count2++;
    }

    vector<int> result;
    if (count1 > arr.size() / 3) result.push_back(cand1);
    if (count2 > arr.size() / 3) result.push_back(cand2);
    return result;
}
```

## Quick Comparison with Kadane's

|                     | Kadane's      | Moore's Voting                   |
| ------------------- | ------------- | -------------------------------- |
| Tracks              | running sum   | running count                    |
| Reset trigger       | sum < 0       | count == 0                       |
| Answer              | best sum seen | surviving candidate              |
| Needs verification? | no            | yes (if majority not guaranteed) |
|                     |               |                                  |

Same "single pass, greedy local decision" flavor as Kadane's — that's why they pair well for revision.


# Power in O(log n) : Binary Exponentiation (Fast Power)

## Problem

Compute `a^n` faster than the naive O(n) loop.

## Core Idea

Write `n` in binary. Then:

```
a^13 = a^(8+4+1) = a^8 * a^4 * a^1      (13 = 1101 in binary)
```

- At each step: **square the base**, and if the current bit of `n` is `1`, multiply it into the result.
- Shift `n` right by 1 (i.e., `n /= 2`) each iteration.
- Number of bits in `n` = `log2(n)` → hence **O(log n)** time.

## Iterative Version (preferred — O(1) space)

```cpp
long long power(long long a, long long n) {
    long long result = 1;
    while (n > 0) {
        if (n & 1) result *= a;   // bit set -> include current a
        a *= a;                   // square the base
        n >>= 1;                  // move to next bit
    }
    return result;
}
```

## Recursive Version

```cpp
long long power(long long a, long long n) {
    if (n == 0) return 1;
    long long half = power(a, n / 2);
    long long result = half * half;
    if (n % 2 != 0) result *= a;
    return result;
}
```

Space: O(log n) due to recursion stack.

## Modular Exponentiation (common in CP)

```cpp
const long long MOD = 1e9 + 7;

long long power(long long a, long long n, long long mod = MOD) {
    a %= mod;
    long long result = 1;
    while (n > 0) {
        if (n & 1) result = (result * a) % mod;
        a = (a * a) % mod;
        n >>= 1;
    }
    return result;
}
```

## Dry Run: a = 3, n = 13 (binary 1101)

| n (binary) | bit | result  | a (after squaring) |
| ---------- | --- | ------- | ------------------ |
| 13 (1101)  | 1   | 3       | 9                  |
| 6 (110)    | 0   | 3       | 81                 |
| 3 (11)     | 1   | 243     | 6561               |
| 1 (1)      | 1   | 1594323 | —                  |

`3^13 = 1594323`

## Complexity

- Time: **O(log n)**
- Space: **O(1)** iterative / **O(log n)** recursive

## Key Points to Remember

- Also called **exponentiation by squaring**.
- Same idea generalizes to **matrix exponentiation** (e.g., Fibonacci in O(log n)).
- Use `long long` — values overflow fast.
- Negative `n`: compute `1.0 / power(a, -n)` (use double if fractional result needed).
- `n & 1` checks last bit; `n >>= 1` is faster than `n /= 2` for the same effect.

# Two Pointer Approch:

## the core idea:
We can use two indices (pointers) to traverse a data structure to avoid using a nested loop, which usually help us cut down the time complexity to O(n). 
For example: **Container with Most Water problem**
![[Pasted image 20260820023217.png]]

## When to Use

- Array/string is **sorted** (or can be sorted).
- Looking for a **pair/triplet** with a target sum, difference, etc.
- Need to **shrink/expand a window** based on a condition.
- Detecting **cycles** or finding **middle** of a linked list (fast-slow variant).

## Types

### 1. Opposite Direction (Converging Pointers)

Start: `left = 0`, `right = n-1`. Move inward based on a condition.

**Example: Pair with given sum in sorted array**

```cpp
bool pairSum(vector<int>& arr, int target) {
    int left = 0, right = arr.size() - 1;
    while (left < right) {
        int sum = arr[left] + arr[right];
        if (sum == target) return true;
        else if (sum < target) left++;   // need bigger sum
        else right--;                    // need smaller sum
    }
    return false;
}
```

**Example: Container With Most Water**

```cpp
int maxArea(vector<int>& height) {
    int left = 0, right = height.size() - 1, best = 0;
    while (left < right) {
        int area = min(height[left], height[right]) * (right - left);
        best = max(best, area);
        // move the shorter wall (moving taller one can't help)
        if (height[left] < height[right]) left++;
        else right--;
    }
    return best;
}
```

### 2. Same Direction (Slow-Fast / Window Pointers)

Both pointers start at `0`, move forward; one leads, one trails.

**Example: Remove duplicates from sorted array (in-place)**

```cpp
int removeDuplicates(vector<int>& arr) {
    int slow = 0;
    for (int fast = 1; fast < arr.size(); fast++) {
        if (arr[fast] != arr[slow]) {
            slow++;
            arr[slow] = arr[fast];
        }
    }
    return slow + 1; // new length
}
```

**Example: Sliding window — max sum subarray of size k**

```cpp
int maxSumK(vector<int>& arr, int k) {
    int windowSum = 0, best = 0;
    for (int i = 0; i < arr.size(); i++) {
        windowSum += arr[i];
        if (i >= k - 1) {
            best = max(best, windowSum);
            windowSum -= arr[i - k + 1];  // shrink from left
        }
    }
    return best;
}
```

### 3. Fast-Slow Pointers (Floyd's Cycle Detection)

Used on linked lists — `slow` moves 1 step, `fast` moves 2 steps.

```cpp
bool hasCycle(ListNode* head) {
    ListNode *slow = head, *fast = head;
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) return true;
    }
    return false;
}
```

## Dry Run: pairSum(`[2,7,11,15]`, target=9)

| left  | right  | arr[left]+arr[right] | action                 |
| ----- | ------ | -------------------- | ---------------------- |
| 0 (2) | 3 (15) | 17                   | sum > target → right-- |
| 0 (2) | 2 (11) | 13                   | sum > target → right-- |
| 0 (2) | 1 (7)  | 9                    | found! return true     |

## Complexity

- Time: **O(n)** (single pass, each pointer moves at most n steps)
- Space: **O(1)** : no extra data structures

## Key Points to Remember

- Array **must be sorted** for the converging (opposite-direction) type to work correctly.
- Deciding which pointer to move is the crux — usually driven by comparing current sum/condition to target.
- Sliding window problems (fixed or variable size) are a same-direction two-pointer variant.
- Fast-slow pointers solve cycle detection / middle-of-list problems in O(n) time, O(1) space.
- Don't confuse with **Binary Search** — two pointer is linear scan, not divide-and-conquer.