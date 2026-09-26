**Expected to know:** [[Syntax]]
**Topics Covered:** 
**Tags:** [[Syntax]]

JavaScript uses C-like syntax for variables, loops, and conditionals, so constructs like `if`, `else`, `while`, `for`, and `switch` will look familiar.

[[How is the code written and executed in JS]]

## Variables:
Variables can be declared using `var`, `let`, or `const`. Example:
    
```javascript
let x = 10; 
const name = "Alice";
```
- There is no need to specify types; variables can hold any value:
```JS
let a = 10; 
a = "now a string";`
```

## Functions
- Functions are first-class objects: they can be assigned to variables, passed as arguments, and returned from other functions.
- You can declare functions in several ways:
```javascript
function add(x, y) {   
	return x + y; 
} 

const multiply = function(x, y) {   return x * y; };
```
- Functions return `undefined` by default if there is no explicit return value.

## Objects and Arrays
- JavaScript uses objects as flexible key-value stores, similar to structs or hash tables:
```javascript
let person = { name: "Bob", age: 25 }; 
person.job = "Engineer";`
```
- Arrays can hold elements of any type and can grow or shrink dynamically:
```javascript
let arr = [1, "two", true]; 
arr.push({ key: "value" });
```

## Dynamic Typing and Equality
- JavaScript is dynamically typed: variables can change type at runtime.
- It has both loose (`==`) and strict (`===`) equality operators. `==` performs type coercion, while `===` checks both type and value[5](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Language_overview).
```javascript
123 == "123";   // true 
123 === "123";  // false
```

## Interaction with Web Pages
- JavaScript can manipulate HTML and CSS, handle user events, and update the content of web pages in real time[4](https://www.w3schools.com/js/js_intro.asp).
```javascript
document.getElementById("demo").innerHTML = "Hello JavaScript";
```

## Running JavaScript
- You can run JavaScript directly in your browser's console or embed it in HTML using `<script>` tags.
- Example:
```xml
<script>   
	alert("Hello, World!"); 
</script>
```

## Notable Differences from C

| Feature           | JavaScript                    | C                      |
| ----------------- | ----------------------------- | ---------------------- |
| Typing            | Dynamic                       | Static                 |
| Memory management | Automatic (garbage collected) | Manual                 |
| Functions         | First-class objects, closures | Not first-class        |
| Execution         | Interpreted (browser/server)  | Compiled (native code) |
| Primary use case  | Web development, scripting    | Systems programming    |
