## JavaScript Execution Order and Basic Syntax
## **Basic JavaScript Writing Syntax**
- **Statements**: JavaScript code is written as a series of statements, each ending with a semicolon (optional but recommended).
- **Blocks**: Code blocks are enclosed in curly braces `{}` (used in functions, loops, conditionals).
- **Functions**: Defined using the `function` keyword or arrow syntax.
- **Variables**: Declared with `let`, `const`, or `var`.
- **Comments**: Single-line (`//`) and multi-line (`/* ... */`) comments are supported.

**Example:**
```js
// This is a single-line comment 
let x = 5; 
function greet(name) {   console.log(`Hello, ${name}`); } 
greet("Alice");
```

---
## Execution Order in JavaScript


### 1. Top-to-Bottom, Line-by-Line
- JavaScript executes code **sequentially**, from the top of the file to the bottom, one line at a time.
- For example:
```js
console.log("One"); 
console.log("Two"); 
console.log("Three");`
```
**Output:**
```text
One Two Three
```    
Each statement runs in the order it appears.


### 2. Execution Context and Call Stack
- When code runs, JavaScript creates a **global execution context** and pushes it onto the **call stack**.
- Each time a function is called, a new execution context is created and pushed onto the stack. When the function finishes, its context is popped off.
- The call stack ensures that functions are executed in the correct order (last-in, first-out).


### 3. Synchronous vs. Asynchronous Execution

- **Synchronous code**: Runs one statement at a time, blocking the next statement until the current one finishes.

- **Asynchronous code**: Uses mechanisms like `setTimeout`, Promises, or event handlers to schedule code to run later, allowing the rest of the script to continue executing.


### 4. Event Loop and Queues

- JavaScript uses an **event loop** to handle asynchronous tasks:
    
    - **Call Stack**: Executes synchronous code.
    - **Job Queue / Microtask Queue**: Handles microtasks (like resolved Promises).
    - **Callback Queue / Macrotask Queue**: Handles callbacks from events, timers, etc.
    
- Synchronous code runs first. When the call stack is empty, the event loop checks the microtask queue, then the callback queue, and processes jobs in order.

### 5. Function Calls and Execution Order

- When a function is called, its code runs before the next statement after the call.
    
- Example:
    
    ```js
    function sayHello() 
    {   
	    console.log("Hello"); 
	} 
	
	console.log("Start"); 
	
	sayHello(); 
	
	console.log("End");
    ```
    
    **Output:**
    ```text
    Start 
    Hello 
    End
    ```
    The function's code runs in the order it is called.

### 6. Asynchronous Example

- Asynchronous code (like `setTimeout`) is placed in a queue and runs after synchronous code finishes:
    
    js
    
    `console.log("A"); setTimeout(() => console.log("B"), 0); console.log("C");`
    
    **Output:**
    
    text
    
    `A C B`
    
    `setTimeout`'s callback runs after the synchronous code, even if the delay is zero[3](https://demirels-organization.gitbook.io/javascript-tutorial/execution-order)[4](https://stackoverflow.com/questions/36348556/understanding-of-javascript-code-execution-order)[6](https://dev.to/jahid6597/javascript-execution-context-a-deep-dive-4kno).
    

---

## **Summary Table**

|Code Type|Execution Order|Example|
|---|---|---|
|Synchronous|Top-to-bottom, in order|`console.log("A")`|
|Function Call|In order of invocation|`myFunc()`|
|Asynchronous|After sync code, via event loop|`setTimeout`, Promises|
