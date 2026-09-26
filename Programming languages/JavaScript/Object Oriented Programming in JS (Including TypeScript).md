**Expected to know:** [[Objects]], [[Class or Type]], [[Object Oriented Programming]], [[Control Flow Statements and functions in JS]]
**Topics Covered:** [[#Constructor Functions with prototypes]], [[#Classes]]
**Tags:** [[Object Oriented Programming]]

The concept for [[Objects]] and the need for them is same everywhere.
In JavaScript, almost everything is an object (or behaves like one), but they fall into different categories based on how they are built and what they are used for.

Think of "Object" as the base DNA, and different types are just specialized versions of that DNA.

---

## 1. The Core Categories

There are three main buckets that "objects" fall into:

|**Category**|**Description**|**Examples**|
|---|---|---|
|**Standard Built-in Objects**|Objects provided by the JavaScript language itself.|`Object`, `Array`, `Date`, `RegExp`, `Map`, `Set`|
|**Browser/Environment Objects**|Provided by the "host" (like the browser or Node.js).|`window`, `document` (DOM), `fetch`, `process`|
|**User-Defined Objects**|Objects you create yourself in your code.|`const user = { name: "Alex" };`|

---

## 2. Specialized "First-Class" Objects

While all of these are objects, JavaScript gives some of them special superpowers:

### **Functions**

As we discussed, these are "Callable Objects." They have all the properties of a regular object but include a hidden property that allows them to be executed using `()`.

### **Arrays**

An Array is just a special type of object designed to store ordered lists.

- **Regular Object:** Uses named keys (e.g., `user.name`).
    
- **Array:** Uses numeric indexes (e.g., `list[0]`) and has a special `.length` property that updates automatically.
    

### **Dates & RegEx**

These are objects with specialized methods for handling time or searching through text strings.

---

## 3. Objects vs. Primitives

To understand objects, you have to know what _isn't_ an object. JavaScript has **Primitives** (simple data):

- `String`, `Number`, `Boolean`, `null`, `undefined`, `Symbol`, `BigInt`.
    

**The Trick:** JavaScript uses something called "Autoboxing." When you try to access a method on a primitive (like `"hello".toUpperCase()`), JavaScript temporarily wraps that string in a **String Object** so it can use the method, then throws the object away.

---

## The "Everything is an Object" Hierarchy

In JavaScript, almost everything inherits from the **Global Object Prototype**. This is why different "types" of objects still feel similar—they all share the same family tree.

> **The Big Takeaway:** JavaScript doesn't really have "strict types" like other languages. Instead, it has **one Object type** that wears many different hats (Function hat, Array hat, Date hat).


# Getting into the implementation of it:

In JavaScript, we also saw [[Data Structures in JS#Object Literals]], so you might be wondering what is the difference?
<blockquote>An object in JavaScript is a fundamental data type that stores collections of key-value pairs, where keys (also called properties or names) are strings (or symbols), and values can be any type—numbers, strings, functions, other objects, etc. Objects are used to represent real-world entities or structured data, and are central to how JavaScript programs are structured and interact</blockquote>

An **object literal** (also called an _object initializer_) is a specific syntax for creating objects directly in your code, using curly braces `{}` with a comma-separated list of key-value pairs:
```JS
const person = {
  firstName: "John",
  lastName: "Doe",
  age: 50,
  eyeColor: "blue"
};
```

## There are multiple ways to create an object in JS:
We have already seen how to create objects via Object Literals, but we can create objects via:
### Constructor Functions with prototypes
A constructor function in JavaScript is a special type of function used to create and initialize objects. By convention, constructor functions are named with an uppercase first letter i.e., the `PascalCase`(e.g., `Person`, `Car`) to distinguish them from regular functions. 
**Example:**
```javascript
function Person(firstName, lastName) 
{   
	this.firstName = firstName;  
	this.lastName = lastName; 
}
```
(Where the identifier followed by `this` is the name of the attribute of the object created via this constructor function. `this` actually means the thing itself, the block , and the block belongs to the function, and when used, it would belong to the object we are creating using `new`)
When you use the `new` keyword with a constructor function, it does the following:
- Creates a new empty object.
- Sets the object's `[[Prototype]]` to the constructor's `prototype` property.
- Executes the constructor function with `this` bound to the new object.
- Returns the new object (unless the constructor explicitly returns something else).

**Example:**
```JS
const person1 = new Person("Alice", "Smith");
```
Another example would be:
```JS
function Person(firstName, lastName, Dob){
	this.firstNameVariable = firstName;
	this.lastNameVariable = lastName;
	this.dateOfBirthVariable = Dob;
}
```

And now, we can create the object:
```JS
const person1 = new Person("Krish", "Kachru", 655525);
```

However, we can also pass in objects as the value:
```JS
function Person(firstName, lastName, Dob){
	this.firstNameVariable = firstName;
	this.lastNameVariable = lastName;
	this.dateOfBirthVariable = new Date(dob);
}
```
(**Here, Date is a built-in class provided by JavaScript**, and hence, we can use methods provided by that class in our constructor!!!)

We can also define methods inside the constructors:
```JS
function Person(firstName, lastName, Dob){
	this.firstNameVariable = firstName;
	this.lastNameVariable = lastName;
	this.dateOfBirthVariable = new Date(dob);
	this.getYearOfBirth = function (){
		return this.dateOfBirthVariable.getFullYear();
	}
}
```
```JS
function Person(firstName, lastName, Dob){
	this.firstNameVariable = firstName;
	this.lastNameVariable = lastName;
	this.dateOfBirthVariable = new Date(dob);
	this.getYearOfBirth = function (person){
		return person.dateOfBirthVariable.getFullYear();
	}
}
```
Where `getFullYear()` is a method in `Date` class.
When we create a constructor for an object, then it's instances can be accessed just like how we access any other data type's variable.
For example:
```JS
function printFullName(variableOfPerson){
	console.log(`he name is ${variableOfPerson.firstNameVariable} ${variableOfPerson.lastNameVariable}`);
}
```

#### Prototypes and Constructor Functions
When you define methods inside a constructor function, every time you create a new instance, a new copy of each method is created and stored in memory for that instance. For example:
```javascript
function Person(firstName, lastName) 
{     
	this.firstName = firstName;    
	this.lastName = lastName;    
	this.getFullName = function() 
		{        
			return this.firstName + " " + this.lastName;    
		} 
} 

let person1 = new Person("Alice", "Smith"); 
let person2 = new Person("Bob", "Jones"); 
// person1.getFullName !== person2.getFullName (different function objects in memory)
```
Here, both `person1` and `person2` have their own separate copies of `getFullName`, which wastes memory if you create many instances.

And every function in JavaScript (except arrow functions and some special cases) has a `prototype` property by default, i.e., as soon as the function is created. 
Hence, every function in JS has this prototype object that stores attributes and methods attached to it, and can be accessed via :
```JS
(functionName).prototype.(MethodOrVariableName) = ...(WhateverYouWantToSet)...;
```

And when we use a function as a constructor (with `new`), the newly created objects also have a `prototype`, which is linked to the constructor's `prototype` object, i.e., a new prototype is not created, but the function and all it's instances use a single prototype object that belongs to the function.

Hence, the `prototype` object is where you can define methods and properties that should be shared across all instances created by the constructor. 
It now should be obvious that this is more memory-efficient than defining methods inside the constructor, as all instances share the same methods through the prototype chain rather than each instance having their own copy of that method.
**Adding Methods with Prototypes:**
```javascript
Person.prototype.getFullName = function() {   
	return this.firstName + " " + this.lastName; 
}; 

console.log(person1.getFullName()); // "Alice Smith"`
```
Here, `getFullName` is not duplicated for each instance; all `Person` objects share this method via the prototype.

##### The `constructor` Property
- The `prototype` object of a constructor function has a `constructor` property that points back to the constructor function itself.
- This property can be useful for type checking or recreating objects, but it's not commonly relied upon in modern code due to its mutability and potential for being overwritten.
- If you manually replace a constructor's prototype, you should restore the `constructor` property to maintain this link.

**Example of Restoring the Constructor:**
```javascript
function Animal() {}

Animal.prototype = {   
		speak: function() { console.log("Animal speaks"); 
	},  
	constructor: Animal // Restoring the constructor property 
};
```

---
### Classes
Till now we have seen how to use a constructor to make objects, and use the prototype of that constructor to make methods effectively, but that is tedious, so we have something called a **class**:
- When you define a class in JavaScript using the class syntax, it provides a default constructor if you do not specify one. If you define your own constructor, as in your Animal example, it will use that instead.    
- Methods defined inside the class body (like speak) are placed on the class's prototype, not on each instance. This means all instances share the same method, which is efficient and avoids duplicating the method for every object.
- This class syntax is essentially syntactic sugar over the older constructor function and prototype pattern, making the code cleaner and easier to write and understand.
**Example**:
```js
class Animal 
{ 	
	constructor(name) 
	{ 		
		this.name = name; 	
	} 	
	
	speak() 
	{ 		
		console.log(`${this.name} makes a noise.`); 	
	} 
}
```
The speak method is shared across all Animal instances, and you can still define your own constructor to initialize instance properties.

#### Using Class Variables Without a Constructor
You can define and use class instance variables even if you don’t explicitly declare them in the constructor. For example:
```javascript
class Person 
{   
	greet() 
	{    
		return `Hello, my name is ${this.name}`;  
	} 
} 

const p = new Person(); 
p.name = "Alice"; // You can add properties dynamically 

console.log(p.greet()); // Hello, my name is Alice
```
- JavaScript is dynamic: you can add properties to objects at any time, not just in the constructor.
	For example, here, the variable `name` did not exist(i.e., undefined) until we defined it by writing `p.name = "Alice";`.
- However, if you want to initialize properties when the object is created, you usually do so in the constructor, i.e., before the introduction of class fields
We can declare properties inside the Class itself:
```JS
class Person{
    let name;
    greet(){
        return `Hello my name is ${this.name}`;
    }
}

const person = new Person();

console.log(person.greet());
```
But this would be invalid, don't forget about the key:value thing.
##### Why is this invalid?
- **You cannot use `let`, `const`, or `var` to declare properties inside a class body.**
- In JavaScript class syntax, you define properties directly, without any keyword, or inside the constructor using `this`.    

##### Correct ways to define properties:
**1. Using the constructor:**
```javascript
class Person {
    constructor() {
        this.name = ""; // property defined for each instance
    }
    greet() {
        return `Hello my name is ${this.name}`;
    }
}
const person = new Person();
person.name = "Alice";
console.log(person.greet()); // Hello my name is Alice
```
**2. Using class fields (modern JavaScript):**
```JS
class Person {
    name; // property declared directly in the class body
    greet() {
        return `Hello my name is ${this.name}`;
    }
}
const person = new Person();
person.name = "Alice";
console.log(person.greet()); // Hello my name is Alice

```
- Here, `name;` is a _class field_ declaration, which is valid and supported in modern JavaScript (ES2022+)

**3. With an initializer:**
```JS
class Person {
    name = "Default Name"; // property with a default value
    greet() {
        return `Hello my name is ${this.name}`;
    }
}
const person = new Person();
console.log(person.greet()); // Hello my name is Default Name

```


# Interfaces in JS
In JavaScript, the concept of **Interfaces** is a bit of a "good news/bad news" situation.

The "bad news" is that **JavaScript (Vanilla JS) does not have a formal `interface` keyword.** Unlike Java or C#, JavaScript is a dynamically typed language, so it doesn't natively enforce structure at compile time.

The "good news" is that we achieve the same goal using **TypeScript** or **Object Composition**. [[Introduction to TypeScript]]

---

## 1. The Modern Standard: TypeScript Interfaces

If you are working on a professional project, you are likely using TypeScript. Here, interfaces are a first-class citizen. They define the "shape" an object must have.

### Syntax

TypeScript

```
interface User {
  readonly id: number;      // Cannot be changed after creation
  name: string;
  email: string;
  age?: number;             // Optional property
  greet(): string;          // Method signature
}

const person: User = {
  id: 1,
  name: "Alex",
  email: "alex@example.com",
  greet() {
    return `Hello, I'm ${this.name}`;
  }
};
```

### Key Features

- **Duck Typing:** If it looks like a duck and quacks like a duck, it’s a duck. If an object matches the interface's structure, TypeScript is happy.
    
- **Extending:** You can build interfaces on top of each other.
    
    TypeScript
    
    ```
    interface Admin extends User {
      role: 'superadmin' | 'editor';
    }
    ```
    

---

## 2. Vanilla JavaScript: The "Pattern" Approach

Since JS doesn't have an `interface` keyword, we use **Class Inheritance** or **Documentation** to mimic the behavior.

### The Abstract Class Pattern

You can create a "Base" class that throws errors if a method isn't implemented. This forces a developer to "implement" the interface in the subclass.

JavaScript

```
class ShapeInterface {
  constructor() {
    if (this.constructor === ShapeInterface) {
      throw new Error("Object of Abstract Class cannot be created");
    }
  }

  getArea() {
    throw new Error("Method 'getArea()' must be implemented.");
  }
}

class Circle extends ShapeInterface {
  constructor(radius) {
    super();
    this.radius = radius;
  }

  getArea() {
    return Math.PI * this.radius ** 2;
  }
}
```

---

## 3. Important Concepts to Remember

|**Concept**|**Description**|
|---|---|
|**Contract**|An interface is a contract. It doesn't tell you _how_ to do something; it just says _what_ must be there.|
|**Zero Runtime Cost**|In TypeScript, interfaces are completely removed during transpilation. They don't bloat your final JS file.|
|**Structural Typing**|Unlike Java, you don't always have to say `implements InterfaceName`. If the object has the required properties, it passes.|

---

## Summary

- **Vanilla JS:** Use classes and throw errors for missing methods.
    
- **TypeScript:** Use the `interface` keyword for robust, type-safe code.
    
- **Purpose:** To ensure different parts of your code can communicate reliably by agreeing on a specific data structure.
    

so in interfaces, the vbariables are like `key:value` or what? how do we decalre methods and all? Hopw does it actually work and all?