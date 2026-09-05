
## 1. Classical Data Structures (as in C)
You can still implement all fundamental data structures in C++ as you would in C—using pointers, structs/classes, and dynamic memory. Examples include:
- [Arrays](Arrays%20in%20C++)
- Linked Lists (singly, doubly)
- Stacks and Queues (array or linked list-based)
- Trees (binary, AVL, etc.)
- Graphs (adjacency matrix/list)

C++ allows you to use `class` and `struct` for encapsulation and member functions, making your implementations cleaner and more modular4.

---
## 2. STL (Standard Template Library) Containers
C++'s STL provides ready-to-use, efficient, and type-safe implementations of many common data structures, saving you from writing them from scratch:

| STL Container                     | Description                                                                    | Typical Use                      |
| --------------------------------- | ------------------------------------------------------------------------------ | -------------------------------- |
| `std::vector`                     | Dynamic array, resizable, fast random access                                   | Lists, buffers                   |
| `std::list`                       | Doubly linked list                                                             | Frequent insert/delete in middle |
| `std::deque`                      | Double-ended queue                                                             | Fast insert/delete at both ends  |
| `std::stack`                      | LIFO stack (adaptor over other containers)                                     | Undo, parsing                    |
| `std::queue`                      | FIFO queue (adaptor)                                                           | Task scheduling                  |
| `std::priority_queue`             | Heap-based priority queue                                                      | Scheduling, simulation           |
| `std::set` / `std::unordered_set` | Unique elements, fast lookup (`set` is ordered, `unordered_set` is hash-based) | Membership tests                 |
| `std::map` / `std::unordered_map` | Key-value pairs (`map` is ordered, `unordered_map` is hash-based)              | Dictionaries, caches             |
| `std::multiset` / `std::multimap` | Like set/map, but allows duplicate keys                                        | Counting, grouping               |

# Structs in C++
### 1. What is a Struct?

A `struct` (short for structure) is a user-defined data type that lets you group variables of _different_ data types under a single name. Think of it like creating a digital ID card for a person, a car, or an enemy in a video game.

**Defining and using a basic struct:**

C++

```
#include <iostream>
#include <string>

// 1. Define the struct
struct Student {
    std::string name;
    int age;
    float gpa;
};

int main() {
    // 2. Create a variable of type Student
    Student student1;

    // 3. Access members using the dot (.) operator
    student1.name = "Alice";
    student1.age = 20;
    student1.gpa = 3.8;

    std::cout << "Name: " << student1.name << "\n";
    
    return 0;
}
```

---

### 2. The Pointer Approach to Structs

When your structs get large, passing them around to different functions by value makes a complete copy of all that data, which is slow and eats up memory. Instead, we use **pointers** to simply point to the memory address where the struct already lives.

When you use a pointer to point to a struct, accessing the data inside it changes slightly.

#### The Arrow Operator (`->`)

If you have a normal struct variable, you use the **dot operator (`.`)** to access its members.

If you have a _pointer_ to a struct, you use the **arrow operator (`->`)** to access its members.

_(Note: Under the hood, `ptr->age` is just a cleaner shortcut for `(*ptr).age`)_

**Example: Pointers pointing to existing structs**

```C++
#include <iostream>
#include <string>

struct Student {
    std::string name;
    int age;
};

int main() {
    // Create a normal struct variable
    Student student1;
    student1.name = "Bob";
    student1.age = 21;

    // Create a pointer that points to the address of student1
    Student* ptr = &student1;

    // Access the data using the arrow operator ->
    std::cout << "Original Name: " << ptr->name << "\n";

    // Modify the data via the pointer
    ptr->age = 22; 
    std::cout << "Updated Age: " << student1.age << "\n"; // Will print 22

    return 0;
}
```

---

### 3. Dynamic Memory Allocation with Structs

Often, you won't know how many structs you need until the program is running (like loading a list of users). In this case, you use pointers to dynamically allocate memory on the heap using the `new` keyword.

C++

```
#include <iostream>
#include <string>

struct Car {
    std::string model;
    int year;
};

int main() {
    // Dynamically allocate a struct in memory
    Car* myCar = new Car;

    // Initialize the values using the arrow operator
    myCar->model = "Tesla Model 3";
    myCar->year = 2024;

    std::cout << "I drive a " << myCar->year << " " << myCar->model << ".\n";

    // IMPORTANT: Always free dynamically allocated memory!
    delete myCar;
    myCar = nullptr; // Good practice to prevent dangling pointers

    return 0;
}
```

### Summary of Rules

- **Struct Variable (`Student s;`):** Use the dot to access `s.age`.
    
- **Struct Pointer (`Student* p = &s;`):** Use the arrow to access `p->age`.
    
- **Heap Allocation (`new StructName;`):** Always remember to `delete` the pointer when you are done to prevent memory leaks!
    

## Constructors in Structs 

### 1. The Anatomy of a Constructor

A constructor has two main rules:

1. It must have the **exact same name** as the struct itself.
    
2. It has **no return type** (not even `void`).
    

### 2. The Default Constructor

A default constructor is called when you create a struct without passing any arguments to it. It is perfect for setting default "fallback" values.

C++

```
#include <iostream>
#include <string>

struct Player {
    std::string name;
    int health;
    int level;

    // Default Constructor
    Player() {
        name = "Guest";
        health = 100;
        level = 1;
        std::cout << "A new player has joined the game!\n";
    }
};

int main() {
    // This automatically triggers the Player() constructor
    Player player1; 

    std::cout << player1.name << " is level " << player1.level << ".\n";
    // Output: Guest is level 1.
    
    return 0;
}
```

---

### 3. The Parameterized Constructor

What if you want to set specific values the moment you create the struct? You can create a constructor that takes arguments.

_(Note: You can have multiple constructors in the same struct. C++ will figure out which one to use based on the arguments you provide!)_

C++

```
struct Player {
    std::string name;
    int health;

    // Default Constructor
    Player() {
        name = "Guest";
        health = 100;
    }

    // Parameterized Constructor
    Player(std::string playerName, int startingHealth) {
        name = playerName;
        health = startingHealth;
    }
};

int main() {
    // Calls the default constructor
    Player p1; 

    // Calls the parameterized constructor
    Player p2("Arthur", 250); 

    std::cout << p2.name << " has " << p2.health << " HP.\n";
    
    return 0;
}
```

---

### 4. Pro-Tip: Member Initialization Lists

While the examples above work perfectly, C++ developers prefer a more efficient syntax called an **Initialization List**. Instead of assigning values inside the curly braces `{ }`, you assign them using a colon `:` right after the constructor's parameters.

This is faster because it initializes the variables directly at the moment of creation, rather than creating them empty and then overwriting them inside the curly brackets.

C++

```
struct Player {
    std::string name;
    int health;

    // Parameterized constructor using an initialization list
    Player(std::string n, int h) : name(n), health(h) {
        // The body can often be left entirely empty!
    }
};
```

---

### 5. Combining Constructors with Pointers

Since we just covered pointers and dynamic memory (`new`), here is how you call a parameterized constructor when creating a struct on the heap:

C++

```
int main() {
    // Dynamically allocating memory AND calling the constructor at the same time
    Player* boss = new Player("Bowser", 1000);

    // Using the arrow operator since 'boss' is a pointer
    std::cout << "Boss Name: " << boss->name << "\n";

    delete boss; // Always clean up!
    return 0;
}
```

---

For other STL data structures, check out : [[STL in C++]]