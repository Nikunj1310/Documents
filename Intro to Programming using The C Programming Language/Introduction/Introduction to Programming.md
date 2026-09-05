***Disclaimer: We will use the C programming language to learn programming fundamentals, as most other programming languages follow similar core principles and rules established by C***

---
#  What is Programming?

`Programme` is just a basic set of instructions telling a computer to do some stuff, and the action of creating a programme is called `Programming`.

Programming is the process of writing instructions that a computer can understand and execute. This means getting a computer do a certain task in order to achieve something without making a single mistake.

Imagine the computer is your **less than intelligent** friend who needs each and every instruction on how to do a certain thing, and what we want our friend  to do is to make a pancake, or even simpler, make a Lego set. Now we need to give it each and every detail, that which how to identify the piece, how to pick it up, in what order should he pick how, how to place them , where to place them and in what order should he place them. 
Even if a single basic step is missing, the brain of that friend will crash and that friend will not be able to achieve the task.

Same applies for programming, when we program, instead of a friend, we have a computer which is super fast and accurate, but at the same time dumb, like really dumb.

So we write a set of instructions and then it works on them using the instructions we gave like really fast and gives us the output we desired like really fast.

---
# Language of Code

When we do programming,basically we want to talk to the computer in order to make it do something. And in order to communicate, we have to speak in a certain language.

Computers cannot understand the language we speak. They were built to understand the [[Machine Code]] written in `binary language (A language which has only two things, 1 and 0, and that's it!!!)`. This creates a problem. Machine code is not something that we can work with, it is hard to understand and work with.
This is why we had to create our own language that we would use in order to work with computers, and to convert that into machine code, we created something called [[Implementation]].
This language that we use to communicate with computers is called a `programming language`.

---
## Different types of programming languages?

There are multiple types of programming languages. Each programming language has a set of rules called "[[Syntax]]" that is to be followed while writing in that language.

All languages are built for different purpose, which means some are good at some things, while other are proficient at other.

Some are built to be fast but hard to access and control, some are made so that they are easy to control but cannot be fast, etc.
Each has their own pros and cons.

---
## What are the different types of programming languages?

Since there are so many programming language, it is intuitive that we would divide them into different types depending upon the use case.
Hence, the Programming languages can broadly be classified as:
### High Level Programming languages

- These languages are designed to be easier for humans to read and write. 
- They are more abstract and focus on the logic of the problem rather than the specific details of the hardware. 
- Think of them as using more human-like words and phrases. 
- They need to be translated into machine code before a computer can understand them. 

Examples include:
	**Python, Java, JavaScript, C, C++, C#, F#, G#, PHP, Ruby, CSS, HTML, Swift, Go, etc.**

### Low Level Programming languages

These languages are closer to machine code and provide more direct control over the hardware. They are more difficult to learn and write but offer greater performance and efficiency.

- **Assembly Language:** A low-level language that uses mnemonics to represent machine code instructions. Each instruction typically corresponds to a single machine code instruction.
- **Machine Code:** The most basic level of programming language, consisting of binary instructions that the computer's CPU can directly execute. It's represented as sequences of 0's and 1's.

---
# Where do we programme or write code?
The process of writing a programme using a particular programming language is called coding.
Now that we know what is programming and programming languages, the question arises where do we write the code?

We write the code in an [[Integrated Development Environment(IDE)]] in order to write code. An IDE is what uses the implementation of the language in which our code is written and executes it.

Most common IDE's used are VS Code, NetBeans, Microsoft Visual Studio, etc,. 

Many high level languages(like python) provide their own IDE's.
IDE's are quite useful as they help us to identify errors or mistakes we have made in using the language we are writing in, as mentioned earlier, a programming language is like a humans language, it has it's own set of rules/grammar or more technically called as `Syntax`, but syntax mistakes are not the only errors one might face, but that will be looked upon later.

---
# Types of Programming
There are different programming languages and their types depending upon how closely they work with our machine, but the programming itself has a few types depending upon our main goal with it.
There are essentially three types of programming:
- Procedural Programming
- [[Object Oriented Programming]]
- Functional programming
Covering the basics of programming and its types and all, time to look at the programming itself, i.e., how code it written, what are the elements and keywords used, their meaning and techniques of programming. 


One might say that what we are doing ahead is procedural programming itself as it is the type of programming that works purely on the basis of functions and variable(meaning of which is covered ahead).

---
# Some Basic Elements of programming:

Now that we know what is programming and programming languages and how it is executed and where it is executed, it is time to look at the most basic elements of the code.

`The purpose of programming` is to solve a problem, or make the computer do some task.
Any task requires some quantitative value with which/ on which it works. 
For example, finding the sum of two numbers, or getting answer to which number is the greatest or which letter is in CAPITAL format or lowercase format?

So in order to solve a problem, we usually work with `some value`, and apply some `functions` or `operators` on those values in order to do the calculations and find the solution to the problem.

That value that we need can either be `taken from the users`  or it can `predefined` in the program itself. Whatever might be the case, we need to `store those values` somehow, and `name that place` in order to identify them.

Those `values can be of various types`, for example they could be an integer or a float, or a collection of letters or special symbols, or just a character or a symbol. They could also be complex.

We also might want to take in multiple values and store them conveniently and access them all together.

Also we might need to add description in the code about what  does what so that we don't forget anything later when we or someone else sees the programme.

Keeping all these things in mind, we have the following elements of programming:

- First let's look at the basic elements that work together to form the logic and behavior of a program.
- At the start of programme we define a [[Header]], which gives information about which [[library]] to use in order to look for the meaning of the words used in the programme.
- Also, everything that we name in the programme follows a rule of naming and is called an [[Identifier]].
- While writing the programme, we have to follow a certain rules of that programming language called the [[Syntax]] of that programming language. 
Now after covering this, it would be a good time to check out [[Implementation]], and as we are using The C Programming Language, check out [[The C Implementation]].
The basic elements of programming are the fundamental building blocks you use to create software. While the specifics might vary slightly between languages, these core concepts are generally universal:

1. **[[Data Types and Constants]]:** Data types classify the kind of data a variable can hold. Common data types include:
    - **Integers
    - **Floating-point numbers
    - **Strings
    - **Booleans
    
2. **[[Variables and their scope]]**: 
	These are named storage locations in the computer's memory that hold data. Think of them as containers that can hold different values. You give each variable a name (an identifier), and you can store and retrieve data from it. Crucially, the value stored in a variable can change during the execution of a program (hence the name "variable").
	
	(Now we will discuss the user-defined data types called ["Structs"](Structs.md))
	
3. **[[Operators]]:** These are symbols that perform operations on data. Examples:
    
    - **Arithmetic operators:** `+` (addition), `-` (subtraction), `*` (multiplication), `/` (division), `%` (modulo - remainder).
    - **Assignment operator:** `=` (assigns a value to a variable).
    - **Comparison operators:** `==` (equals), `!=` (not equals), `>` (greater than), `<` (less than), `>=` (greater than or equals), `<=` (less than or equals).
    - **Logical operators:** `&&` (AND), `||` (OR), `!` (NOT).
    - **Bitwise Operators:** 
    
4. **[[Input&Output and placeholders]]:** 
	These operations allow a program to interact with the outside world. Input involves getting data from the user (e.g., keyboard, mouse, files) or other sources. Output involves displaying data to the user (e.g., on the screen, in a file) or sending it to other systems.
	
5. **[[Functions]]:** 
	These are reusable blocks of code that perform a specific task. Functions can take input values (arguments or parameters) and can return a value. They help to organize code, make it more readable, and avoid repetition.
    
6. [[Control Flow Statements]]: 
	These statements determine the order in which instructions are executed. They allow you to create programs that can make decisions and repeat actions. Key control flow statements include:
    
    - **Conditional statements (if-else):** Execute different blocks of code based on whether a condition is true or false.
    - **Loops (for, while):** Repeat a block of code multiple times.
    
7. **[[Data Structures]]:** 
	These are ways of organizing and storing collections of data. Common data structures include:
    
    - **Arrays:** Ordered collections of elements of the same data type.
    - **Lists:** Ordered collections of elements (can be of different data types in some languages).
    - **Dictionaries (or Hash Maps):** Collections of key-value pairs.
    - **Sets:** Unordered collections of unique elements.

Now, when we want to automate a task via a programming language, we first think of **`logic`** behind it, that how it can be solved and all. *Logic behind any problem is the reasoning or the thought process behind solving it.*
Next we write down a step-by-step guide to how to solve that problem or perform that task, an abstract language independent description of how to achieve a result. This is what we call an **`Algorithm`**.
And then, we finally **`code`** it in a particular programming language of our choice.
However, everything in the world needs time, so we look at the [time complexity](Time%20Complexity) of the code you write in a programming language, and *using that concept, we come up with the best algorithm* for a given problem, as the logic would always remain the same, what would change is how it is implemented, i.e., the algorithm.

---
# Techniques of programming

Now that we know about the basics of programming, time to look at the basic techniques used in programming.

![[Introduction to Data Structures And Algorithm]]

---
# Pointers/Working with memory itself:

Until now, the basis struct of a script is done, but we haven't answered the question that where and how exactly is the programme executed? Also when we pass a variable to a function and work on it, why does the variable itself isn't changed?
Also we said earlier that every variable is allocated a space in the memory, but what is that space and how does it all works together?

Answers to these question would be clearer when we look at a concept called ["Pointers and memory allocaton"](Pointers%20and%20Working%20with%20Memory%20Allocation.md).  But before that, we first need to understand [[Memory Allocation]]

---
# Bitwise operations

After looking at how to allocate memory and work with it, we now shift the focus to more of an elementary topics essential for low-level programming and to optimize the code called [[Bitwise Operators]]. 

---
# File input and output 
[[File Input and Output]] 

---
# The C Preprocessor
8. **[[Modularity]]**

# Universal Character Encoding (`UCE`);
Universal Character Encoding (UCE) refers to systems that provide a consistent representation of characters (letters, symbols, control codes, etc.) from all writing systems around the world so computers can store, process, and exchange text reliably. It enables interoperability across platforms, languages, and applications by assigning unique code points (numbers) to each character.

## Basics of Universal Character Encoding (UCE)

- **Character Encoding:** It is a method to represent characters as numeric values (code points) in computers. Without encoding, text cannot be stored or shared digitally.
    
- **Purpose of UCE:** To unify the representation of characters from all languages, symbols, and scripts into a single standardized system.
    
- **Universal:** Implies comprehensive coverage across languages, including alphabets, symbols, punctuation, and even historic or technical characters.
    
- **Importance:** Avoids conflicts or misinterpretations caused by multiple coding systems. E.g., ASCII was limited to English letters, causing issues when handling other languages.
    

## Core Concepts

- **Characters:** Abstract symbols like 'A', '字', or '😊'.
    
- **Code Points:** Unique numbers assigned to each character in the encoding standard.
    
- **Encoding Form:** How code points are translated into bytes for storage or transmission, e.g., UTF-8 or UTF-16.
    
- **Encoding Scheme:** Defines the mapping and byte allocation rules for characters.
    

## Types of Universal Character Encodings (UCEs)

In practice, the term UCE overlaps with or is often used when referring to standards like Unicode and other comprehensive encoding systems. Key types include:

1. **Unicode**
    

- The most widely accepted universal character encoding standard.
    
- Covers almost all scripts and symbols worldwide.
    
- Code points range from 0 to over 1 million (theoretically), but practical limits used are much smaller.
    
- Defines multiple encoding forms:
    
    - **UTF-8:** Variable-length encoding (1-4 bytes). Backward compatible with ASCII; widely used on the web.
        
    - **UTF-16:** Uses 2 or 4 bytes per character; common in systems like Windows.
        
    - **UTF-32:** Fixed 4 bytes per character; simpler but more storage-heavy.
        
- Unicode Consortium maintains this.
    

2. **ISO/IEC 10646 (Universal Character Set, UCS)**
    

- An international standard closely aligned with Unicode.
    
- Defines a 21-bit code space for characters.
    
- UCS forms:
    
    - **UCS-2:** 2 bytes fixed width (limited to BMP—Basic Multilingual Plane).
        
    - **UCS-4:** 4 bytes fixed width; supports all Unicode characters.
        
- Unicode and ISO/IEC 10646 are synchronized in assigned code points, making them compatible universal character encoding standards.
    

3. **Other Notable Universal or Broad Character Sets (Overlap with UCE concept)**
    

- **GB18030 (China):** Includes all Unicode characters plus additional ones important in Chinese text processing.
    
- **Big5 and Shift-JIS:** Widely used regional encodings but not fully universal; represent subsets of characters.
    

## Summary

|Term|Description|Encoding Forms|Coverage|
|---|---|---|---|
|**Unicode**|Universal character standard maintained by Unicode Consortium|UTF-8, UTF-16, UTF-32|Global scripts, symbols, emojis|
|**ISO/IEC 10646 (UCS)**|International standard aligning with Unicode|UCS-2, UCS-4|Vast range, identical to Unicode code points|
|**UCE (General Concept)**|Universal scheme of character encoding|Various (Unicode/UCS)|Character sets covering all writing systems|

## Why UCE Matters

- Enables **multilingual computing** and consistent text exchange in an increasingly globalized digital world.
    
- Eliminates confusion caused by legacy encodings that were often region or language-specific.
    
- Fundamental for modern technologies such as the web, databases, programming languages, and operating systems.  
    In summary, "Universal Character Encoding (UCE)" usually points to the concept embodied by standards like Unicode and ISO/IEC 10646, providing a universal, uniform code system for virtually every character used in human languages and symbols worldwide. Its primary forms, like UTF-8 and UTF-16, allow flexible and efficient storage and transmission of text in different environments.

---
# **![[Assertions]]**
---
# ![[Coroutines]]**