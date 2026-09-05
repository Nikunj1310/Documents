C++ is a powerful and versatile programming language that has played a significant role in the development of software as we know it today.

## Key Characteristics:
- **General-purpose:**
    - C++ can be used to develop a wide range of applications, from operating systems and game engines to embedded systems and high-performance software.
- **Object-oriented:**
    - It supports object-oriented programming (OOP), which allows developers to organize code into reusable components called objects, enhancing code modularity and maintainability.
- **Compiled language:**
    - C++ code is compiled into machine code, resulting in fast and efficient execution.
- **Powerful and flexible:**
    - It provides low-level memory manipulation capabilities, giving developers fine-grained control over system resources.
- **Standardized:**
    - C++ is standardized by the International Organization for Standardization (ISO), ensuring consistency across different compilers and platforms.

# C++ Syntax basics:
![C++ syntax](Syntax%20of%20C++)

---
# Compiling the code
 Same as that in The C Programming Language, just use `g++` instead of `gcc`.
 When you compile a C++ file, the process typically involves several stages, beginning with pre-processing and then the actual compilation, which ultimately leads to an executable program.

Here's a breakdown of what happens:
1. **Pre-processing**:
	Before any actual compilation, the **preprocessor** handles directives that begin with `#`.
	- **Macro expansion**: Replaces `#define` macros with their values.
	- **File inclusion**: Inserts the contents of headers via `#include`.
	- **Conditional compilation**: Evaluates `#ifdef`, `#ifndef`, `#if` blocks and includes or excludes code accordingly.
	
	The output of this step is a single expanded C++ translation unit.
	
2. **Compilation (Translation to Assembly)**
	The compiler proper takes the preprocessed code and performs:
	1. **Lexical analysis** – Breaks the source text into tokens (identifiers, keywords, literals).
	2. **Syntax analysis (parsing)** – Builds an abstract syntax tree (AST) to ensure your code follows C++ grammar.
	3. **Semantic analysis** – Checks types, scopes, and other language rules; resolves overloads and templates.
	4. **Optimization** – Performs intermediate transformations (inlining, dead-code elimination) to streamline performance.
	5. **Code generation** – Converts the optimized AST into assembly language for the target architecture.
	
	The result is an assembly file (e.g., `file.s`).
	
3. **Assembly (Translation to Object Code)**
	The **assembler** takes the architecture-specific assembly code and translates it into **object code** (.o or .obj files). An object file contains:	
	- **Machine instructions** in binary form.
	- **Symbol table** listing functions and global variables (but unresolved references remain).
	- **Relocation information** to let the linker adjust addresses once all code is combined.
    
4. **Linking**
	The **linker** combines one or more object files and static libraries into a single executable or shared library. Its tasks include:
	- **Symbol resolution**: Matches function calls and variable references to their definitions.
	- **Relocation**: Adjusts memory addresses so that code and data sections fit together.
	- **Library incorporation**: Pulls in code from standard libraries (`libstdc++`, `libc`) or user-provided archives.

This multi-stage process takes your human-readable C++ code and transforms it step-by-step into a runnable binary program.

The binary file that you get after compiling can be either a **library** or an **actual executable program** (also referred to as an executable binary). When setting up a project in Visual Studio, for instance, the configuration type determines the output binary, allowing you to choose between an application (.EXE) for an executable binary or a library.

---
# Elements of the language:
There are a few starting features that are required to get started with a programming language, so lets have a look at those elements:
- [Data Types](Data%20types%20in%20C++)
- [Variables](Variables%20in%20C++) 
- [Operators](Operators%20in%20C++) 
- [Input / Output and PlaceHolders](Input&Output%20in%20C++)
- [Functions](Functions%20in%20C++) 
- [Control Flow](Control%20Flow%20Statements%20in%20C++.md) 
- [Data Structures](Data%20Structures%20in%20C++.md) 

Now that all the basic stuff is covered, time to look at Pointers in C++.
As C++ is just an extension of C, the logic and syntax of pointers here is same as that in [[Pointers and Working with Memory Allocation]].

Now, lets have a look at [[Object Oriented Programming in C++]], this will only cover the syntax, so for the theory check out [[Object Oriented Programming]].

# [[STL in C++]]


# Libraries 

The standard Header of our programming language and other libraries offered
