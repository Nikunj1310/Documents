**Topics Covered:**
**Tags:** [[Operators]] [[Introduction to JavaScript]]

## Operators in JavaScript
Operators in JavaScript are special symbols or keywords used to perform operations on values (operands). They are fundamental to writing expressions and controlling logic in your code.

---
## **Types of JavaScript Operators**

| Operator Type         | Purpose/Example                                                |
| --------------------- | -------------------------------------------------------------- |
| Arithmetic            | Math operations: `+`, `-`, `*`, `/`, `%`, `**`, `++`, `--`     |
| Assignment            | Assign values: `=`, `+=`, `-=`, `*=`, `/=`, `%=`, `**=`        |
| Comparison            | Compare values: `==`, `===`, `!=`, `!==`, `>`, `<`, `>=`, `<=` |
| Logical               | Combine/negate conditions: `&&`, `                             |
| String                | Concatenate strings: `+`, `+=`                                 |
| Bitwise               | Bit-level operations: `&`, `                                   |
| Ternary (Conditional) | Shorthand if-else: `condition ? expr1 : expr2`                 |
| Type                  | Type checking: `typeof`, `instanceof`                          |
| Spread/Rest           | Spread/rest syntax: `...array`                                 |
| Delete                | Remove object property: `delete obj.prop`                      |
| Others                | `void`, `in`, `new`, `this`, `super`, `yield`                  |

---
## Key Operator Categories and Examples

### 1. Arithmetic Operators

Perform mathematical calculations.
```js
let a = 10, b = 3; 
a + b;    // 13 (Addition) 
a - b;    // 7  (Subtraction) 
a * b;    // 30 (Multiplication) 
a / b;    // 3.333... (Division) 
a % b;    // 1  (Modulus, remainder) 
a ** b;   // 1000 (Exponentiation) 
a++;      // 11 (Increment) 
b--;      // 2  (Decrement)
```
(It does follow pre and post increment and decrement)

### 2. Assignment Operators
Assign or update variable values.
```js
let x = 5; 
x += 2; // x = x + 2 => 7 
x *= 3; // x = x * 3 => 21 
x -= 1; // x = x - 1 => 20 
x /= 2; // x = x / 2 => 10 
x %= 3; // x = x % 3 => 1 
x **= 2; // x = x ** 2 => 1
```

### 3. Comparison Operators
Compare two values and return a boolean.
```js
x == 5;    // true ( checks only for equal value) 
x === "5"; // false (equal value and type) 
x != 8;    // true (not equal) 
x !== "5"; // true (not equal value or type) 
x > 4;     // true (greater than) 
x < 10;    // true (less than) 
x >= 5;    // true (greater than or equal) 
x <= 7;    // true (less than or equal)
```

So here, we find some new things like == and === , === means is equal to in both value and type, while == doesn't actually care about the type.
So [[How does == compare in JS]] 

### 4. Logical Operators
Combine or invert boolean values.
```js
let a = true, b = false; 
a && b; // false (AND)
a || b; // true  (OR) 
!a;     // false (NOT)
```

### 5. String Operators
Concatenate strings.
```js
let str = "Hello" + " " + "World"; // "Hello World" 
str += "!"; // "Hello World!"
```

### 6. Bitwise Operators
Operate on binary representations of numbers.
```js
5 & 1;  // 1 (AND) 
5 | 1;  // 5 (OR) 
5 ^ 1;  // 4 (XOR) 
~5;     // -6 (NOT) 
5 << 1; // 10 (left shift) 
5 >> 1; // 2  (right shift)
```

### 7. Ternary (Conditional) Operator
Shorthand for if-else.
```js
let age = 18; 
let status = (age >= 18) ? "adult" : "minor"; // "adult"
```
### 8. Type Operators

Check or assert types.
```js
typeof "hello"; // "string" 
x instanceof Array; // true if x is an array
```

### 9. Other Useful Operators

- **Spread/Rest:** `let arr = [1, ...[2][3]]; // [1][2][3]` 
- **Delete:** `delete obj.prop;`
- **in:** `'key' in obj; // true if obj has property 'key'`
- **void:** `void(0); // returns undefined`  

---
## Operator Precedence

Operator precedence determines the order in which operations are evaluated. For example, multiplication has higher precedence than addition:

```js
let result = 2 + 3 * 4; // result is 14, not 20
```

Use parentheses to override precedence:
```js
let result = (2 + 3) * 4; // result is 20
```