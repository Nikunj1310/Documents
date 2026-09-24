**Expected to know:** [[Introduction to Programming]] (till functions)
**Topics Covered:** Conditional statements, loops, jump statements
**Tags:**

When a code is executed, there is a flow that is followed during the execution. The flow in which the statements in a programme is executed is called Control Flow/flow of control.
Example:
```c
#include <stdio.h>

int main()
{
	printf("Lets play a game. I'll count and you will run\n");
	printf("If when I open my eyes, I see you, then I'll kill you\n");
	printf("if I open my eyes and I don't see you, then I'll go into next room.");
	printf("And if I find you in that room, then the same process will repeat\n");
	printf("Or else I die :)");
	return 0;
}
```

So here, the execution is sequential, statement is executed one after another. However, we can use certain statements in order to control the flow of execution of code. These statements are called Control Flow Statements.

---
# Control Flow Statements

**Control Flow Statements** determine the **order in which instructions are executed**. They help make decisions, loop through code, and control execution flow.

There are various types of Control Flow Statements:
- **[[Conditional Statements]]** (Decision-Making)
    
    - `if`
    - `if-else`
    - `else if`
    - `switch`

- **[[Looping Statements]]** (Iteration)
    
    - `for`
    - `while`
    - `do-while`

- **[[Jump Statements]]** (Branching)
    
    - `break`
    - `continue`
    - `goto`
    - `return`

---
# **Can we say Functions are also control flow statements?**

No, **function declaration** is **not** a control flow statement. Here's why:

### **1. Function Declaration vs. Control Flow Statements**

- A **function declaration** (or function prototype) **does not control execution flow**; it only tells the compiler about the function's return type, name, and parameters.
- **Control flow statements**, on the other hand, **directly affect how and when code executes** (e.g., loops, conditionals, jumps).

---

### **2. Example of a Function Declaration**

```c
#include <stdio.h>

// Function declaration (not a control flow statement)
int add(int, int);

int main() {
    int result = add(3, 4);
    printf("Sum: %d\n", result);
    return 0;
}

// Function definition
int add(int a, int b) {
    return a + b;
}
```

🔹 **The function declaration `int add(int, int);` does not execute anything.** It only informs the compiler that `add` exists.

---

### **3. Example of Control Flow Statements**

```c
if (x > 0) {         // Controls execution based on condition
    printf("Positive");
}

for (int i = 0; i < 5; i++) {  // Controls repetition of code
    printf("%d ", i);
}
```

🔹 These statements **actively decide what part of the program runs**.

---

### **4. Why Function Calls Can Affect Control Flow**

Although **function declarations are not control flow statements**, calling a function **does affect the flow** because execution jumps to the function definition and returns.

```c
#include <stdio.h>

void greet() {
    printf("Hello\n");  // Function affects execution flow
}

int main() {
    greet();  // Function call redirects execution flow
    printf("Back to main\n");
    return 0;
}
```

 **Output:**

```
Hello
Back to main
```

🔹 The function **call** (`greet();`) affects the control flow, but **declaring** the function does not.

---

### **5. Conclusion**

❌ **Function declaration ≠ Control Flow Statement**  
✅ **Function calls affect execution flow but are not considered control flow statements.**

---
---
# **Using assignment operators in Control Flow Statements**

In C, an **assignment operator (`=`)** can be used inside conditionals. However, it is important to understand **how it behaves** and **what it returns**.

---
### **What Does an Assignment Expression Return?**
An assignment expression like `x = 5` **returns the assigned value**,[but why?](Operators.md)(i.e., `5` in this case).  
This means you can use assignment in conditionals, but it can be confusing!

### **Example 1: Assignment in an `if` Statement**

```c
#include <stdio.h>

int main() {
    int x;

    if (x = 10) {  // Assignment, not a comparison!
        printf("Condition is true! x = %d\n", x);
    }

    return 0;
}
```

### **What Happens Here?**
- `x = 10` assigns `10` to `x`.
- Since **`10` is a non-zero value**, it is considered **true** in C.
- The condition **executes the `if` block**.

### **Output:**
```
Condition is true! x = 10
```

---
### **Example 2: Using Assignment in a While Loop**

```c
#include <stdio.h>

int main() {
    int x = 5;

    while (x = x - 1) {  // Assigns `x - 1` to `x`, then checks if nonzero
        printf("x = %d\n", x);
    }

    return 0;
}
```

### **Explanation:**

- `x = x - 1` reduces `x` by `1` and **returns the new value of `x`**.
- The loop **continues** until `x` becomes `0` (which is treated as false).
- When `x` is `0`, the loop stops.

### **Output:**

```
x = 4
x = 3
x = 2
x = 1
```

---

### **Example 3: Common Mistake – Using `=` Instead of `==`**

A common mistake is **accidentally using `=` (assignment) instead of `==` (comparison)**.

#### **Wrong Code (Bug!)**

```c
if (x = 0) {  // Oops! Assignment instead of comparison
    printf("x is zero\n");
}
```

#### **What Happens?**

- `x = 0` assigns `0` to `x`, and the assignment **returns `0`**.
- `if (0)` evaluates to **false**, so the block does **not execute**.

✅ **Correct Way (Using `==` for Comparison):**

```c
if (x == 0) {  // Now it correctly checks if x is equal to 0
    printf("x is zero\n");
}
```

---

### **Summary**

|**Expression**|**What Happens?**|
|---|---|
|`if (x = 5)`|Assigns `5` to `x`, returns `5` (true). ✅|
|`while (x = x - 1)`|Assigns `x - 1` to `x`, loop stops when `x = 0`. ✅|
|`if (x = 0)`|Assigns `0` to `x`, returns `0` (false), condition **does not run**. ❌|
|`if (x == 0)`|Checks if `x` is `0`. **Correct comparison**. ✅|

---

### **Key Takeaways**

✅ **Assignment expressions return the assigned value**  
✅ **Nonzero values are treated as true**, and `0` is false  
✅ **Be careful not to use `=` instead of `==` in conditionals**
