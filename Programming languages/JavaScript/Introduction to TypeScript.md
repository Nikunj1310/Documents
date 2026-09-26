get to know:
Record<string, unknown>; aka record datastructure
in objects, what if I use : instead of , and all
## What is TypeScript?

TypeScript is a **strongly typed, object-oriented programming language** developed and maintained by Microsoft. It's a **superset of JavaScript**, meaning all valid JavaScript code is also valid TypeScript code. TypeScript adds optional static typing, classes, interfaces, and other features to JavaScript, then compiles down to plain JavaScript that runs anywhere JavaScript runs.

**Key Facts:**
- Created by Anders Hejlsberg (also creator of C#) at Microsoft
- First released in October 2012
- Open source (Apache 2.0 license)
- Compiles to clean, readable JavaScript (ES3, ES5, ES6+)
- Works with any browser, any host, any operating system

---

## Why TypeScript?

### 1. **Static Type Checking**
TypeScript catches errors at compile-time rather than runtime:

```typescript name=type-safety-example.ts
// JavaScript - error only discovered at runtime
function add(a, b) {
  return a + b;
}
add(5, "10"); // Returns "510" - unexpected!

// TypeScript - error caught during development
function add(a: number, b: number): number {
  return a + b;
}
add(5, "10"); // Error: Argument of type 'string' is not assignable to parameter of type 'number'
```

### 2. **Enhanced IDE Support**
- Intelligent code completion (IntelliSense)
- Instant error detection
- Better refactoring tools
- Accurate navigation and documentation

### 3. **Better Code Documentation**
Types serve as inline documentation that never goes out of sync:

```typescript name=self-documenting-code.ts
// The function signature tells you everything you need to know
function createUser(name: string, age: number, isAdmin: boolean): User {
  // implementation
}
```

### 4. **Scales with Your Project**
TypeScript shines in large codebases where tracking types manually becomes impossible.

---

## Core Concepts

### 1. **Basic Types**

```typescript name=basic-types.ts
// Primitives
let isDone: boolean = false;
let age: number = 25;
let firstName: string = "John";

// Arrays
let numbers: number[] = [1, 2, 3, 4];
let names: Array<string> = ["Alice", "Bob"];

// Tuple - fixed-length array with known types
let person: [string, number] = ["Alice", 30];

// Enum
enum Color {
  Red,
  Green,
  Blue
}
let favoriteColor: Color = Color.Blue;

// Any - opt-out of type checking (use sparingly!)
let notSure: any = 4;
notSure = "maybe a string";
notSure = false;

// Unknown - safer alternative to any
let value: unknown = "hello";
if (typeof value === "string") {
  console.log(value.toUpperCase()); // Type narrowing required
}

// Void - absence of a type (typically for functions that don't return)
function logMessage(message: string): void {
  console.log(message);
}

// Null and Undefined
let u: undefined = undefined;
let n: null = null;

// Never - represents values that never occur
function throwError(message: string): never {
  throw new Error(message);
}
```

### **Type Annotations**

The fundamental shift in syntax is this:

- **Java/C++:** Type comes _before_ the variable (`int age = 19;`)
    
- **TypeScript:** Type comes _after_ the variable, separated by a colon (`let age: number = 19;`)
    

The colon (`:`) is simply the syntax that tells the TS compiler, "I am about to declare the type for the thing I just named."

### 1. Variables (Primitives)

You annotate basic types directly after the variable name.

TypeScript

```
let score: number = 100;
let username: string = "Steve";
let isOnline: boolean = true;

// TS also has 'any' to bypass checking entirely (avoid when possible)
let unknownData: any = "could be anything"; 
```

### 2. Arrays

There are two ways to annotate arrays. Both do the exact same thing.

TypeScript

```
// Bracket syntax (most common)
let scores: number[] = [95, 80, 100];

// Generic syntax (identical to Java's ArrayList<Integer>)
let names: Array<string> = ["Alice", "Bob"]; 
```

### 3. Functions

You must annotate both the parameters and the return type.

- **Parameters:** Annotated exactly like variables, after the parameter name.
    
- **Return Type:** Annotated after the closing parenthesis `()` of the arguments.
    

TypeScript

```
// Java equivalent: public int calculateDamage(int base, int multiplier)
function calculateDamage(base: number, multiplier: number): number {
    return base * multiplier;
}

// If a function returns nothing, use 'void' (just like C++/Java)
function logError(message: string): void {
    console.log(message);
}
```

### 4. Objects (Interfaces)

Instead of requiring a full class instantiation like Java to enforce an object's structure, TS lets you define the "shape" of data using an `interface` or `type`, and then use that as an annotation.

TypeScript

```
interface Player {
    id: number;
    username: string;
    isBanned: boolean;
}

// Annotating the 'p1' object with the 'Player' interface
let p1: Player = {
    id: 1,
    username: "Notch",
    isBanned: false
};
```

### 2. **Interfaces**

Interfaces define the structure of objects:

```typescript name=interfaces.ts
interface User {
  id: number;
  name: string;
  email: string;u(() => {
  age?: number; // Optional property
  readonly createdAt: Date; // Read-only property
}

const user: User = {
  id: 1,
  name: "Alice",
  email: "alice@example.com",
  createdAt: new Date()
};

// user.createdAt = new Date(); // Error: Cannot assign to 'createdAt' because it is a read-only property

// Interface for functions
interface SearchFunc {
  (source: string, subString: string): boolean;
}

const mySearch: SearchFunc = (src, sub) => {
  return src.includes(sub);
};

// Extending interfaces
interface Employee extends User {
  department: string;
  salary: number;
}
```

### 3. **Type Aliases**

Create custom types with `type`:

```typescript name=type-aliases.ts
// Union types
type Status = "pending" | "approved" | "rejected";
type ID = string | number;

// Object types
type Point = {
  x: number;
  y: number;
};

// Intersection types
type Timestamped = {
  createdAt: Date;
  updatedAt: Date;
};

type User = {
  name: string;
  email: string;
};

type TimestampedUser = User & Timestamped;

const user: TimestampedUser = {
  name: "Bob",
  email: "bob@example.com",
  createdAt: new Date(),
  updatedAt: new Date()
};
```

### 4. **Functions**

```typescript name=functions.ts
// Function with typed parameters and return type
function add(a: number, b: number): number {
  return a + b;
}

// Optional parameters
function greet(name: string, greeting?: string): string {
  return `${greeting || "Hello"}, ${name}!`;
}

// Default parameters
function createUser(name: string, role: string = "user"): void {
  console.log(`Creating ${role}: ${name}`);
}

// Rest parameters
function sum(...numbers: number[]): number {
  return numbers.reduce((total, n) => total + n, 0);
}

// Function type expressions
type MathOperation = (x: number, y: number) => number;

const multiply: MathOperation = (x, y) => x * y;

// Arrow functions
const square = (n: number): number => n * n;

// Overload signatures
function getValue(key: string): string;
function getValue(key: number): number;
function getValue(key: string | number): string | number {
  if (typeof key === "string") {
    return "string value";
  }
  return 42;
}
```

### 5. **Classes**

```typescript name=classes.ts
class Animal {
  // Properties
  private name: string;
  protected age: number;
  public species: string;

  // Constructor
  constructor(name: string, age: number, species: string) {
    this.name = name;
    this.age = age;
    this.species = species;
  }

  // Methods
  public makeSound(): void {
    console.log("Some generic sound");
  }

  // Getter
  public getName(): string {
    return this.name;
  }

  // Setter
  public setName(name: string): void {
    this.name = name;
  }
}

class Dog extends Animal {
  private breed: string;

  constructor(name: string, age: number, breed: string) {
    super(name, age, "Dog");
    this.breed = breed;
  }

  // Override method
  public makeSound(): void {
    console.log("Woof! Woof!");
  }

  public getBreed(): string {
    return this.breed;
  }
}

// Abstract classes
abstract class Shape {
  abstract getArea(): number;
  
  describe(): string {
    return `Area: ${this.getArea()}`;
  }
}

class Circle extends Shape {
  constructor(private radius: number) {
    super();
  }

  getArea(): number {
    return Math.PI * this.radius ** 2;
  }
}
```

### 6. **Generics**

Generics allow you to write reusable, type-safe code:

```typescript name=generics.ts
// Generic function
function identity<T>(arg: T): T {
  return arg;
}

let output1 = identity<string>("hello");
let output2 = identity<number>(42);

// Generic interface
interface Container<T> {
  value: T;
  getValue: () => T;
}

const stringContainer: Container<string> = {
  value: "hello",
  getValue: () => "hello"
};

// Generic class
class DataStore<T> {
  private data: T[] = [];

  add(item: T): void {
    this.data.push(item);
  }

  get(index: number): T | undefined {
    return this.data[index];
  }

  getAll(): T[] {
    return this.data;
  }
}

const numberStore = new DataStore<number>();
numberStore.add(1);
numberStore.add(2);

// Generic constraints
interface HasLength {
  length: number;
}

function logLength<T extends HasLength>(arg: T): void {
  console.log(arg.length);
}

logLength("hello"); // OK
logLength([1, 2, 3]); // OK
// logLength(42); // Error: number doesn't have length property
```

### 7. **Type Guards and Narrowing**

```typescript name=type-guards.ts
// typeof guard
function printValue(value: string | number) {
  if (typeof value === "string") {
    console.log(value.toUpperCase()); // TypeScript knows it's a string
  } else {
    console.log(value.toFixed(2)); // TypeScript knows it's a number
  }
}

// instanceof guard
class Cat {
  meow() { console.log("Meow!"); }
}

class Dog {
  bark() { console.log("Woof!"); }
}

function makeSound(animal: Cat | Dog) {
  if (animal instanceof Cat) {
    animal.meow();
  } else {
    animal.bark();
  }
}

// Custom type guard
interface Fish {
  swim: () => void;
}

interface Bird {
  fly: () => void;
}

function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}

function move(pet: Fish | Bird) {
  if (isFish(pet)) {
    pet.swim();
  } else {
    pet.fly();
  }
}

// Discriminated unions
type Success = {
  type: "success";
  data: string;
};

type Error = {
  type: "error";
  message: string;
};

type Result = Success | Error;

function handleResult(result: Result) {
  switch (result.type) {
    case "success":
      console.log(result.data);
      break;
    case "error":
      console.log(result.message);
      break;
  }
}
```

### 8. **Utility Types**

TypeScript includes built-in utility types for common transformations:

```typescript name=utility-types.ts
interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}

// Partial - makes all properties optional
type PartialUser = Partial<User>;
const updateUser: PartialUser = { name: "New Name" };

// Required - makes all properties required
type RequiredUser = Required<User>;

// Readonly - makes all properties read-only
type ReadonlyUser = Readonly<User>;

// Pick - select specific properties
type UserPreview = Pick<User, "id" | "name">;

// Omit - exclude specific properties
type UserWithoutEmail = Omit<User, "email">;

// Record - construct object type with specific keys
type Roles = "admin" | "user" | "guest";
type RolePermissions = Record<Roles, string[]>;

const permissions: RolePermissions = {
  admin: ["read", "write", "delete"],
  user: ["read", "write"],
  guest: ["read"]
};

// ReturnType - extract return type of a function
function createUser() {
  return { id: 1, name: "Alice" };
}
type User2 = ReturnType<typeof createUser>;

// Awaited - unwrap Promise types
type AsyncUser = Promise<User>;
type UnwrappedUser = Awaited<AsyncUser>; // User
```

---

## Setting Up TypeScript

### Installation

```bash
# Install TypeScript globally
npm install -g typescript

# Or install in a project
npm install --save-dev typescript

# Check version
tsc --version
```

### Configuration (`tsconfig.json`)

Create a configuration file:

```bash
tsc --init
```

````json name=tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",                    // JavaScript version to compile to
    "module": "commonjs",                  // Module system
    "lib": ["ES2020"],                     // Standard library to include
    "outDir": "./dist",                    // Output directory
    "rootDir": "./src",                    // Input directory
    "strict": true,                        // Enable all strict type checking
    "esModuleInterop": true,              // Interop between CommonJS and ES modules
    "skipLibCheck": true,                 // Skip type checking of declaration files
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,            // Import JSON files
    "declaration": true,                  // Generate .d.ts files
    "sourceMap": true                     // Generate source maps for debugging
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```
````

### Basic Workflow

```typescript name=example.ts
// src/index.ts
interface Greeting {
  message: string;
  timestamp: Date;
}

function createGreeting(name: string): Greeting {
  return {
    message: `Hello, ${name}!`,
    timestamp: new Date()
  };
}

const greeting = createGreeting("World");
console.log(greeting.message);
```

```bash
# Compile TypeScript to JavaScript
tsc

# Compile and watch for changes
tsc --watch

# Run directly with ts-node
npx ts-node src/index.ts
```

---

## Best Practices

### 1. **Enable Strict Mode**
Always use `"strict": true` in your `tsconfig.json` for maximum type safety.

### 2. **Avoid `any`**
Use `unknown` instead when you're unsure of a type, and narrow it down with type guards.

### 3. **Use Interfaces for Objects, Type Aliases for Unions**
```typescript
// Prefer interface for object shapes
interface User {
  name: string;
}

// Use type for unions, primitives, tuples
type Status = "active" | "inactive";
```

### 4. **Leverage Type Inference**
```typescript
// Don't annotate unnecessarily
const name = "Alice"; // TypeScript infers string

// Do annotate when needed
function greet(name: string) {
  return `Hello, ${name}`;
}
```

### 5. **Use `const` Assertions for Literal Types**
```typescript
const config = {
  apiUrl: "https://api.example.com",
  timeout: 5000
} as const;

// config.apiUrl = "new url"; // Error: readonly
```

---

## Common Use Cases

### 1. **React with TypeScript**
```typescript name=react-example.tsx
interface Props {
  name: string;
  age?: number;
}

const UserCard: React.FC<Props> = ({ name, age }) => {
  return (
    <div>
      <h2>{name}</h2>
      {age && <p>Age: {age}</p>}
    </div>
  );
};
```

### 2. **Express API with TypeScript**
```typescript name=express-example.ts
import express, { Request, Response } from 'express';

interface CreateUserBody {
  name: string;
  email: string;
}

app.post('/users', (req: Request<{}, {}, CreateUserBody>, res: Response) => {
  const { name, email } = req.body;
  // Handle user creation
  res.json({ success: true });
});
```

### 3. **Async/Await**
```typescript name=async-example.ts
interface ApiResponse<T> {
  data: T;
  status: number;
}

async function fetchUser(id: number): Promise<ApiResponse<User>> {
  const response = await fetch(`/api/users/${id}`);
  const data = await response.json();
  return {
    data,
    status: response.status
  };
}
```

---

## Resources for Learning More

- **Official Documentation**: https://www.typescriptlang.org/docs/
- **TypeScript Playground**: https://www.typescriptlang.org/play
- **TypeScript Deep Dive** (free book): https://basarat.gitbook.io/typescript/
- **Effective TypeScript** by Dan Vanderkam
- **DefinitelyTyped**: Type definitions for JavaScript libraries

---

## Conclusion

TypeScript transforms JavaScript development by adding type safety, better tooling, and improved maintainability. While there's a learning curve, the benefits—especially for larger projects and teams—far outweigh the initial investment. Start by gradually adopting TypeScript in existing projects, and you'll quickly appreciate the confidence it brings to your codebase.