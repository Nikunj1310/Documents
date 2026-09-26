```
Make things detaioled, and order them up accoridng to the topic and make new files for longer topics
Re-read it all as well for revision 
```

## What is a Database?

A **Database** is a collection of related data. A **Database Management System (DBMS)** is a software package designed to store and manage databases. As Ramakrishnan & Gehrke emphasize, a DBMS is crucial because it provides:

- **Data Independence:** Applications are insulated from how data is structured and stored.
- **Efficient Data Access:** Uses specialized techniques (like indexing) to retrieve data quickly.
- **Data Integrity and Security:** Enforces constraints (e.g., "salary cannot be negative").
- **Concurrent Access & Crash Recovery:** Allows multiple users to access data simultaneously while protecting against system failures.

### The Evolution: File Systems vs. DBMS

Before DBMS, data was stored in operating system files. This led to:

- **Redundancy:** Same information stored in multiple places.
- **Inconsistency:** Updating one file but forgetting another.
- **Difficulty in Access:** Writing a new C++ or Java program for every new query.

## 2. Types of Databases

Modern computing utilizes various database models based on the use case:

- **Relational Databases (RDBMS):** Data is organized into tables (relations). Examples: MySQL, PostgreSQL, Oracle. This is our primary focus.    
- **NoSQL Databases:** Non-tabular, used for big data and real-time web apps. Includes Document (MongoDB), Key-Value (Redis), and Graph databases (Neo4j).
- **Object-Oriented Databases:** Stores data as objects, similar to OOP languages.
- **Distributed Databases:** Data is stored across multiple physical locations.

### Relational Databases (RDBMS) vs. Object-Oriented (OODBMS)

> The Questions:
> What is the difference between relational database and Object-Oriented database the table defined can be thought of as a class, and all the rows as columns?

To answer your question: **Yes, the analogy is very close!** In a RDBMS, we use tables, while in OOP, we use classes.

| Feature           | Relational (RDBMS)                                     | Object-Oriented (OODBMS)                   |
| ----------------- | ------------------------------------------------------ | ------------------------------------------ |
| **Basic Unit**    | **Table (Relation)**                                   | **Class / Object**                         |
| **Structure**     | Rows (Records) and Columns (Attributes).               | Objects with attributes and methods.       |
| **Relationships** | Established via Foreign Keys.                          | Established via references/pointers.       |
| **Logic**         | Logic is in the application code (SQL).                | Logic (Methods) is stored _with_ the data. |
| **Analogy**       | **Table** $\approx$ Class; **Row** $\approx$ Instance. | **Object** is stored directly.             |

**The "Impedance Mismatch":** In your Java/C++ code, you think in objects, but to save them in MySQL, you have to "flatten" them into rows. OODBMS (like ObjectDB) attempts to solve this by storing the object directly, but RDBMS remains more popular due to its rigorous mathematical foundation (Relational Algebra).

### NoSQL Databases (Deep Dive)

NoSQL (Not Only SQL) databases are "Schema-less," meaning they don't require a fixed table structure. This makes them ideal for rapid development and massive scale.

1. **Document Databases (e.g., MongoDB):**
    
    - **Concept:** Stores data in JSON-like documents.    
    - **Example:** A "User" document might have varying fields (some have phone numbers, some don't) without needing a column for every possibility.
    
2. **Key-Value Stores (e.g., Redis):**
    
    - **Concept:** Like a giant Dictionary or Hash Map. You provide a Key and get a Value.
    - **Example:** Storing session tokens or shopping carts for a website where speed is the only priority.
    
3. **Graph Databases (e.g., Neo4j):**
    
    - **Concept:** Focuses on relationships (Edges) between entities (Nodes).    
    - **Example:** A Social Network. "User A" _is friends with_ "User B." Querying "friends of friends" is much faster here than in SQL.
    
4. **Wide-Column Stores (e.g., Cassandra):**
    
    - **Concept:** Designed for massive writes and large datasets across many servers.    
    - **Example:** Storing sensor logs from millions of IoT devices.

## Introduction to SQL

**SQL (Structured Query Language)** is the standard language for interacting with Relational Databases. It is declarative, meaning you tell the system _what_ you want, not _how_ to get it.

SQL is divided into sub-languages:

1. **DDL (Data Definition Language):** Defines the structure (schema).
    - `CREATE`, `ALTER`, `DROP`.
        
2. **DML (Data Manipulation Language):** Manages the data within the schema.
    - `INSERT`, `UPDATE`, `DELETE`.
    
3. **DQL (Data Query Language):** Used to retrieve data.
    - `SELECT`.
    
4. **DCL (Data Control Language):** Manages permissions.
    - `GRANT`, `REVOKE`.

## Getting Started with MySQL

MySQL is one of the most popular open-source RDBMS. It uses a Client-Server architecture.

### Basic Syntax & Commands

To begin, you typically interact with the MySQL server via a terminal or a GUI (like MySQL Workbench).

**Step 1: Create and Use a Database**

```MySQL
CREATE DATABASE University;
USE University;
```

**Step 2: Create a Table** Following the schema from the "Student" example in Ramakrishnan:

```MySQL
CREATE TABLE Students (
    sid INT PRIMARY KEY,
    name VARCHAR(50),
    login VARCHAR(20),
    age INT,
    gpa REAL
);
```

**Step 3: Insert Data**

```MySQL
INSERT INTO Students VALUES (53666, 'Jones', 'jones@cs', 18, 3.4);
```

**Step 4: Query Data**

```MySQL
SELECT name, gpa 
FROM Students 
WHERE age > 19;
```

(***Now follow along [[Introduction to MySQL]] to get into depth about MySQL!***)

# What is a Database Management System (DBMS)?

A **Database Management System (DBMS)** is a collection of interrelated data plus a set of programs to access that data. It provides an environment that is both convenient and efficient to store, retrieve, and manage information about a particular organization or enterprise.

## Why Do We Need DBMS?

Before DBMS, organizations used **file systems** to store data, which created serious problems:

- **Data redundancy**: Same information stored multiple times in different files. For example, if you store student records, you might repeat department name "Computer Science" and department head "Steve Rogers" for every student in that department.
- **Data inconsistency**: When the same data exists in multiple places, updating one copy but not others creates conflicts.
- **Difficulty accessing data**: You need to write a new program every time you want to retrieve data differently.
- **No integrity constraints**: Rules like "account balance must be greater than 0" had to be manually coded into every program.
- **Atomicity problems**: If a transaction fails midway (like transferring \$100 from account X to Y), the database could end up in an inconsistent state. Atomicity means a transaction either completes fully or doesn't happen at all.
- **Concurrency issues**: Multiple users accessing and updating the same data simultaneously without control leads to errors.[^1]
- **Security problems**: No systematic way to control who can access what data.[^1]

DBMS solves all these problems.[^1]

## Real-World DBMS Applications

Databases are everywhere:

- **Banking**: All transactions, account information
- **Airlines**: Flight reservations, schedules
- **Universities**: Student registration, grades
- **Sales**: Customer data, products, purchases
- **Manufacturing**: Production tracking, inventory, supply chain


## Three Levels of Data Abstraction

DBMS organizes data into three levels to separate how data is stored from how users see it:[^1]

- **Physical level**: How data is actually stored on disk (storage format, file structure)
- **Logical level**: What data is stored and relationships between data (like defining a customer has name, street, city)
- **View level**: What individual users or applications see (can hide sensitive info like salaries)


## Schema vs Instance

Think of a database like a variable in programming:

- **Schema**: The structure/design of the database (like the data type of a variable). It defines tables, columns, data types, and relationships. Schema rarely changes.
- **Instance**: The actual data stored at a specific moment (like the value of a variable). Instance changes frequently as data is inserted, updated, or deleted.

Example: The schema defines "Student table has ID, Name, Age columns." The instance is the actual student records like "1, Tony Stark, 18" at 9 AM today.

## Data Independence

- **Physical Data Independence**: You can change how data is stored (physical level) without changing the logical structure or applications.
- **Logical Data Independence**: You can modify the logical schema without affecting user views or applications.

This separation allows flexibility—you can optimize storage or add new tables without breaking existing programs.

## Database Users

Different types of users interact with DBMS:

- **Naive users**: Regular users like bank customers using ATMs or web apps
- **Application programmers**: Developers who write programs that interact with the database
- **Sophisticated users**: Data analysts who write complex queries
- **Database Administrator (DBA)**: Person responsible for schema design, security, backups, performance monitoring, and recovery


## System Architecture

Modern DBMS typically uses:

- **Two-tier**: Client programs directly communicate with database server
- **Three-tier**: Web applications with middleware between user interface and database

# Entity-Relationship (ER) Model

## Introduction to ER Model

The **Entity-Relationship (ER) Model** is a conceptual data model used to design databases. It provides a high-level, visual representation of data structure before implementation.

**Purpose:**
- Design databases systematically
- Communicate database structure to stakeholders
- Serve as a blueprint for physical database implementation

## Entities and Entity Sets

### Entity
A **real-world object** that exists independently and can be distinguished from other objects.

**Examples:**
- A specific student: "John Smith, ID: 12345"
- A particular car: "Toyota Camry, License: ABC123"
- An individual course: "DBMS101"

### Entity Set
A **collection of similar entities** sharing the same properties.

**Examples:**
- `STUDENT` (all students)
- `COURSE` (all courses)
- `EMPLOYEE` (all employees)

**Analogy:** 
- Entity Set = Table Name
- Entity = Row in that table

### ER Diagram Notation

| Element           | Symbol                                | Description           |
| ----------------- | ------------------------------------- | --------------------- |
| **Strong Entity** | ![Rectangle](Rectangle)               | Single-line rectangle |
| **Weak Entity**   | ![Double Rectangle](Double Rectangle) | Double-line rectangle |

```
┌─────────────┐
│   STUDENT   │  ← Strong Entity
└─────────────┘

╔═════════════╗
║  DEPENDENT  ║  ← Weak Entity
╚═════════════╝
```

### MySQL Implementation

```sql
-- Entity Set becomes a table
CREATE TABLE STUDENT (
    student_id INT PRIMARY KEY,
    name VARCHAR(100),
    address VARCHAR(200),
    phone_no VARCHAR(15)
);

CREATE TABLE CLASSROOM (
    lecture_hall_no VARCHAR(10) PRIMARY KEY,
    capacity INT,
    no_of_benches INT,
    no_of_boards INT
);
```


## Attributes

**Attributes** are properties that describe entities. They become columns in database tables.

### Simple vs Composite Attributes

#### Simple Attribute
**Cannot be divided** into smaller meaningful parts.

**ER Notation:** 
```
    ○
    │
   age
```

**Examples:**
- `age`
- `gender`
- `student_id`
- `salary`

```sql
CREATE TABLE EMPLOYEE (
    emp_id INT,      -- Simple
    age INT,         -- Simple
    gender CHAR(1)   -- Simple
);
```

#### Composite Attribute
**Can be broken down** into smaller sub-attributes.

**ER Notation:**
```
        ○ name
       /│\
      / │ \
     ○  ○  ○
  first middle last
```

**Examples:**

**Name:**
- `first_name`
- `middle_name`
- `last_name`

**Address:**
- `street_number`
- `street_name`
- `city`
- `state`
- `postal_code`

**Mapping Rule:** Only **leaf-level** sub-attributes become columns.

```sql
CREATE TABLE PERSON (
    person_id INT PRIMARY KEY,
    -- Don't create a 'name' column
    first_name VARCHAR(50),   -- Leaf attribute
    middle_name VARCHAR(50),  -- Leaf attribute
    last_name VARCHAR(50),    -- Leaf attribute
    -- Don't create an 'address' column
    street_number VARCHAR(10),
    street_name VARCHAR(100),
    city VARCHAR(50),
    state VARCHAR(50),
    postal_code VARCHAR(10)
);
```

### 3.2 Single-valued vs Multivalued Attributes

#### Single-valued Attribute (atomic value)
Has **only one value** per entity.

**ER Notation:**
```
    ○
    │
 birth_date
```

**Examples:**
- `birth_date` (one per person)
- `student_id` (one per student)
- `passport_number` (one per person)

```sql
CREATE TABLE STUDENT (
    student_id INT PRIMARY KEY,  -- Single-valued
    birth_date DATE              -- Single-valued
);
```

#### Multivalued Attribute
Can have **multiple values** for a single entity.

**ER Notation:**
```
   ◎◎
    │
phone_numbers
```
(Double ellipse/circle)

**Examples:**
- `phone_numbers` (mobile, home, office)
- `email_addresses`
- `hobbies`
- `degrees_earned`

**Mapping Rule:** Create a **separate table** with:
1. The entity's primary key (foreign key)
2. The multivalued attribute column(s)
3. Composite primary key of both

```sql
-- Main entity
CREATE TABLE STUDENT (
    student_id INT PRIMARY KEY,
    name VARCHAR(100)
);

-- Separate table for multivalued attribute
CREATE TABLE STUDENT_PHONE (
    student_id INT,
    phone_number VARCHAR(15),
    phone_type VARCHAR(20),  -- Optional: 'mobile', 'home', 'office'
    
    PRIMARY KEY (student_id, phone_number),
    FOREIGN KEY (student_id) REFERENCES STUDENT(student_id)
        ON DELETE CASCADE
);

-- Example: One student with multiple phones
INSERT INTO STUDENT VALUES (101, 'John Doe');
INSERT INTO STUDENT_PHONE VALUES (101, '555-1234', 'mobile');
INSERT INTO STUDENT_PHONE VALUES (101, '555-5678', 'home');
INSERT INTO STUDENT_PHONE VALUES (101, '555-9012', 'office');
```

**Another Example: Hobbies**

```sql
CREATE TABLE EMPLOYEE (
    emp_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE EMPLOYEE_HOBBY (
    emp_id INT,
    hobby VARCHAR(50),
    
    PRIMARY KEY (emp_id, hobby),
    FOREIGN KEY (emp_id) REFERENCES EMPLOYEE(emp_id)
        ON DELETE CASCADE
);
```

### 3.3 Stored vs Derived Attributes

#### Stored Attribute
**Physically stored** in the database.

```sql
CREATE TABLE STUDENT (
    student_id INT PRIMARY KEY,
    birth_date DATE  -- Stored
);
```

#### Derived Attribute
**Calculated** from other attributes, **not stored**.

**ER Notation:**
```
   (○)  ← Dashed ellipse
    │
   age
```

**Examples:**
- `age` (derived from `birth_date`)
- `total_marks` (sum of subject marks)
- `years_of_service` (derived from `hire_date`)
- `GPA` (calculated from grades)

```sql
-- Don't create a column for derived attributes
CREATE TABLE STUDENT (
    student_id INT PRIMARY KEY,
    birth_date DATE  -- Store this
    -- age INT  ← DON'T DO THIS
);

-- Calculate age when needed
SELECT 
    student_id,
    birth_date,
    TIMESTAMPDIFF(YEAR, birth_date, CURDATE()) AS age  -- Derived
FROM STUDENT;

-- Another example: Total salary
CREATE TABLE EMPLOYEE (
    emp_id INT PRIMARY KEY,
    basic_salary DECIMAL(10,2),
    allowance DECIMAL(10,2),
    bonus DECIMAL(10,2)
    -- total_salary DECIMAL(10,2)  ← DON'T store this
);

SELECT 
    emp_id,
    basic_salary,
    allowance,
    bonus,
    (basic_salary + allowance + bonus) AS total_salary  -- Derived
FROM EMPLOYEE;
```

**Using Virtual Columns (MySQL 5.7+):**

```sql
CREATE TABLE EMPLOYEE (
    emp_id INT PRIMARY KEY,
    hire_date DATE,
    years_of_service INT AS (TIMESTAMPDIFF(YEAR, hire_date, CURDATE())) VIRTUAL
);
```

### 3.4 Domain / Value Set

The **set of allowed values** for an attribute.

**Examples:**

| Attribute     | Domain                                            |
| ------------- | ------------------------------------------------- |
| `gender`      | {'M', 'F', 'Other'}                               |
| `shirt_size`  | {'S', 'M', 'L', 'XL', 'XXL'}                      |
| `grade`       | {'A', 'B', 'C', 'D', 'F'}                         |
| `age`         | {0, 1, 2, ..., 150}                               |
| `day_of_week` | {'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun'} |

```sql
CREATE TABLE STUDENT (
    student_id INT PRIMARY KEY,
    name VARCHAR(100),
    gender ENUM('M', 'F', 'Other'),  -- Domain constraint
    shirt_size ENUM('S', 'M', 'L', 'XL', 'XXL'),
    age INT CHECK (age >= 0 AND age <= 150),
    
    CONSTRAINT chk_gender CHECK (gender IN ('M', 'F', 'Other'))
);

CREATE TABLE ENROLLMENT (
    student_id INT,
    course_id INT,
    grade CHAR(1) CHECK (grade IN ('A', 'B', 'C', 'D', 'F')),
    PRIMARY KEY (student_id, course_id)
);
```

### Attribute Notation Summary

| Attribute Type | ER Notation | Example |
|----------------|-------------|---------|
| **Simple** | `○` (Single ellipse) | age, id |
| **Composite** | `○` with branches | name (first, middle, last) |
| **Single-valued** | `○` (Single ellipse) | birth_date |
| **Multivalued** | `◎◎` (Double ellipse) | phone_numbers |
| **Derived** | `(○)` (Dashed ellipse) | age from birth_date |
| **Key Attribute** | `○` with underline | student_id |

---

## Relationships and Relationship Sets 

### Relationship
An **association** between two or more entities.

**Examples:**
- Student **enrolls in** Course
- Employee **works for** Department
- Author **writes** Book

### Relationship Set
A **collection of similar relationships** between entity sets.

**ER Notation:**
```
┌─────────┐         ◇         ┌─────────┐
│ STUDENT │─────enrolls_in────│ COURSE  │
└─────────┘                   └─────────┘
```
(Diamond shape for relationship)

### 4.1 Degree of Relationships

#### Unary (Recursive) Relationship
Relationship **within the same entity set**.

**ER Diagram:**
```
        ┌─────────────┐
    ┌───│  EMPLOYEE   │◄──┐
    │   └─────────────┘   │
    │          │          │
    │       manages       │
    │          │          │
    └──────────◇──────────┘
          (manager)  (worker)
          ↑ Role indicators ↑
```

**Examples:**
1. **Employee manages Employee**
   - manager → worker relationship
   
2. **Person is_married_to Person**
   - spouse1 → spouse2 relationship

3. **Course is_prerequisite_for Course**
   - prerequisite → dependent course

```sql
-- Example 1: Employee manages Employee
CREATE TABLE EMPLOYEE (
    emp_id INT PRIMARY KEY,
    name VARCHAR(100),
    manager_id INT,  -- Foreign key to same table
    
    FOREIGN KEY (manager_id) REFERENCES EMPLOYEE(emp_id)
);

-- Data example
INSERT INTO EMPLOYEE VALUES (1, 'CEO', NULL);        -- Top manager
INSERT INTO EMPLOYEE VALUES (2, 'Manager A', 1);     -- Reports to CEO
INSERT INTO EMPLOYEE VALUES (3, 'Employee B', 2);    -- Reports to Manager A

-- Find manager of an employee
SELECT 
    e.name AS employee,
    m.name AS manager
FROM EMPLOYEE e
LEFT JOIN EMPLOYEE m ON e.manager_id = m.emp_id;

-- Example 2: Course prerequisites
CREATE TABLE COURSE (
    course_id VARCHAR(10) PRIMARY KEY,
    course_name VARCHAR(100)
);

CREATE TABLE PREREQUISITE (
    course_id VARCHAR(10),
    prerequisite_course_id VARCHAR(10),
    
    PRIMARY KEY (course_id, prerequisite_course_id),
    FOREIGN KEY (course_id) REFERENCES COURSE(course_id),
    FOREIGN KEY (prerequisite_course_id) REFERENCES COURSE(course_id)
);

-- Data: DBMS requires Data Structures
INSERT INTO COURSE VALUES ('CS101', 'Data Structures');
INSERT INTO COURSE VALUES ('CS201', 'DBMS');
INSERT INTO PREREQUISITE VALUES ('CS201', 'CS101');
```

#### Binary Relationship
Relationship between **two different entity sets**.

**ER Diagram:**
```
┌─────────┐         ◇         ┌─────────┐
│ STUDENT │────enrolls_in─────│ COURSE  │
└─────────┘                   └─────────┘
```

**Examples:**
- Student enrolls in Course
- Employee works in Department
- Customer borrows Loan

```sql
-- Example: Student enrolls in Course
CREATE TABLE STUDENT (
    student_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE COURSE (
    course_id VARCHAR(10) PRIMARY KEY,
    course_name VARCHAR(100)
);

CREATE TABLE ENROLLS_IN (
    student_id INT,
    course_id VARCHAR(10),
    enrollment_date DATE,
    
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES STUDENT(student_id),
    FOREIGN KEY (course_id) REFERENCES COURSE(course_id)
);
```

#### Ternary Relationship
**Simultaneous relationship** among three entity sets.

**ER Diagram:**
```
┌────────────┐
│ INSTRUCTOR │──┐
└────────────┘  │
                │    ◇
                ├──teaches──┐
                │           │
┌─────────┐     │           │    ┌─────────┐
│ STUDENT │─────┘           └────│ COURSE  │
└─────────┘                      └─────────┘
```

**Example:** Instructor teaches Student in Course (which instructor teaches which students in which course)

**Real-world scenario:** 
- Dr. Smith teaches John and Mary in DBMS
- Dr. Johnson teaches John and Peter in Algorithms

```sql
CREATE TABLE INSTRUCTOR (
    instructor_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE STUDENT (
    student_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE COURSE (
    course_id VARCHAR(10) PRIMARY KEY,
    course_name VARCHAR(100)
);

-- Ternary relationship
CREATE TABLE TEACHES (
    instructor_id INT,
    student_id INT,
    course_id VARCHAR(10),
    semester VARCHAR(20),
    
    PRIMARY KEY (instructor_id, student_id, course_id, semester),
    FOREIGN KEY (instructor_id) REFERENCES INSTRUCTOR(instructor_id),
    FOREIGN KEY (student_id) REFERENCES STUDENT(student_id),
    FOREIGN KEY (course_id) REFERENCES COURSE(course_id)
);

-- Data example
INSERT INTO TEACHES VALUES (101, 201, 'CS301', 'Fall 2026');
-- Instructor 101 teaches Student 201 in Course CS301 in Fall 2026
```

**Another Example:** Supplier supplies Part to Project

```sql
CREATE TABLE SUPPLIER (
    supplier_id INT PRIMARY KEY,
    supplier_name VARCHAR(100)
);

CREATE TABLE PART (
    part_id INT PRIMARY KEY,
    part_name VARCHAR(100)
);

CREATE TABLE PROJECT (
    project_id INT PRIMARY KEY,
    project_name VARCHAR(100)
);

CREATE TABLE SUPPLIES (
    supplier_id INT,
    part_id INT,
    project_id INT,
    quantity INT,
    supply_date DATE,
    
    PRIMARY KEY (supplier_id, part_id, project_id),
    FOREIGN KEY (supplier_id) REFERENCES SUPPLIER(supplier_id),
    FOREIGN KEY (part_id) REFERENCES PART(part_id),
    FOREIGN KEY (project_id) REFERENCES PROJECT(project_id)
);
```

### 4.2 Attributes on Relationships

Relationships can have **descriptive attributes**.

**Example:** Student enrolls in Course

**Relationship Attributes:**
- `enrollment_date`
- `semester`
- `grade`

```sql
CREATE TABLE ENROLLS_IN (
    student_id INT,
    course_id VARCHAR(10),
    
    -- Relationship attributes
    enrollment_date DATE,
    semester VARCHAR(20),
    year INT,
    grade CHAR(1),
    
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES STUDENT(student_id),
    FOREIGN KEY (course_id) REFERENCES COURSE(course_id)
);
```

**Another Example:** Employee works in Department

```sql
CREATE TABLE WORKS_IN (
    emp_id INT,
    dept_id INT,
    
    -- Relationship attributes
    start_date DATE,
    role VARCHAR(50),
    
    PRIMARY KEY (emp_id, dept_id),
    FOREIGN KEY (emp_id) REFERENCES EMPLOYEE(emp_id),
    FOREIGN KEY (dept_id) REFERENCES DEPARTMENT(dept_id)
);
```

---

## 5. Cardinality and Participation Constraints 

### 5.1 Cardinality Constraints

Defines **how many instances** of one entity can be associated with instances of another entity.

#### One-to-One (1:1)

Each entity in A relates to **at most one** entity in B, and vice versa.

**ER Notation:**
```
┌─────────┐     1      ◇      1     ┌──────────────┐
│ STUDENT │────────allocates────────│ HOSTEL_ROOM  │
└─────────┘                         └──────────────┘
```

**Real-world Examples:**
- One student allocated to one hostel room
- One person has one passport
- One employee has one company ID card

```sql
-- Method 1: Foreign key in either table
CREATE TABLE STUDENT (
    student_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE HOSTEL_ROOM (
    room_id INT PRIMARY KEY,
    room_number VARCHAR(10),
    student_id INT UNIQUE,  -- UNIQUE ensures 1:1
    
    FOREIGN KEY (student_id) REFERENCES STUDENT(student_id)
);

-- Method 2: Share the same primary key
CREATE TABLE PERSON (
    person_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE PASSPORT (
    person_id INT PRIMARY KEY,  -- Same as PERSON's PK
    passport_number VARCHAR(20),
    issue_date DATE,
    
    FOREIGN KEY (person_id) REFERENCES PERSON(person_id)
);
```

#### One-to-Many (1:M)

Each entity in A relates to **multiple** entities in B, but each entity in B relates to **at most one** in A.

**ER Notation:**
```
┌────────────┐    1      ◇      M    ┌─────────┐
│ INSTRUCTOR │─────────teaches───────│ COURSE  │
└────────────┘                       └─────────┘
```

**Real-world Examples:**
- One instructor teaches many courses
- One department has many employees
- One customer places many orders

```sql
-- Foreign key goes in the "Many" side
CREATE TABLE INSTRUCTOR (
    instructor_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE COURSE (
    course_id VARCHAR(10) PRIMARY KEY,
    course_name VARCHAR(100),
    instructor_id INT,  -- Foreign key on "Many" side
    
    FOREIGN KEY (instructor_id) REFERENCES INSTRUCTOR(instructor_id)
);

-- Data example
INSERT INTO INSTRUCTOR VALUES (101, 'Dr. Smith');
INSERT INTO COURSE VALUES ('CS101', 'DBMS', 101);
INSERT INTO COURSE VALUES ('CS102', 'Algorithms', 101);
INSERT INTO COURSE VALUES ('CS103', 'Networks', 101);
```

#### Many-to-One (M:1)

**Multiple** entities in A relate to **one** entity in B.

**ER Notation:**
```
┌─────────┐     M      ◇      1     ┌──────────┐
│ STUDENT │──────────does───────────│ PROJECT  │
└─────────┘                         └──────────┘
```

**Real-world Examples:**
- Many students work on one major project
- Many employees belong to one department
- Many books written by one author

```sql
CREATE TABLE PROJECT (
    project_id INT PRIMARY KEY,
    project_name VARCHAR(100)
);

CREATE TABLE STUDENT (
    student_id INT PRIMARY KEY,
    name VARCHAR(100),
    project_id INT,  -- Foreign key on "Many" side
    
    FOREIGN KEY (project_id) REFERENCES PROJECT(project_id)
);

-- Data example: Multiple students on same project
INSERT INTO PROJECT VALUES (1, 'Library Management System');
INSERT INTO STUDENT VALUES (101, 'John', 1);
INSERT INTO STUDENT VALUES (102, 'Mary', 1);
INSERT INTO STUDENT VALUES (103, 'Peter', 1);
```

#### Many-to-Many (M:M)

Entities in both sets can relate to **multiple** entities in the other set.

**ER Notation:**
```
┌─────────┐     M      ◇      M     ┌─────────┐
│ STUDENT │─────────enrolls─────────│ COURSE  │
└─────────┘                         └─────────┘
```

**Real-world Examples:**
- Students enroll in many courses; courses have many students
- Actors appear in many movies; movies have many actors
- Authors write many books; books have many authors

```sql
-- Requires a separate "junction" table
CREATE TABLE STUDENT (
    student_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE COURSE (
    course_id VARCHAR(10) PRIMARY KEY,
    course_name VARCHAR(100)
);

-- Junction table
CREATE TABLE ENROLLS_IN (
    student_id INT,
    course_id VARCHAR(10),
    enrollment_date DATE,
    
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES STUDENT(student_id),
    FOREIGN KEY (course_id) REFERENCES COURSE(course_id)
);

-- Data example
INSERT INTO STUDENT VALUES (101, 'John'), (102, 'Mary');
INSERT INTO COURSE VALUES ('CS101', 'DBMS'), ('CS102', 'Algorithms');

-- John enrolls in both courses
INSERT INTO ENROLLS_IN VALUES (101, 'CS101', '2026-01-15');
INSERT INTO ENROLLS_IN VALUES (101, 'CS102', '2026-01-15');

-- Mary enrolls in CS101
INSERT INTO ENROLLS_IN VALUES (102, 'CS101', '2026-01-16');
```

### 5.2 Min-Max Notation

Specifies **minimum and maximum** number of relationship instances.

**Notation:** `(min, max)`

**ER Diagram:**
```
┌──────────┐  (0,2)     ◇     (1,M)   ┌──────┐
│ CUSTOMER │───────────borrows────────│ LOAN │
└──────────┘                          └──────┘
```

**Interpretation:**
- `(0, 2)`: Customer can have **0 to 2** loans
- `(1, M)`: Each loan must have **at least 1** customer, up to many

```sql
-- Enforce min-max using triggers (simplified example)
CREATE TABLE CUSTOMER (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE LOAN (
    loan_id INT PRIMARY KEY,
    amount DECIMAL(10,2)
);

CREATE TABLE BORROWS (
    customer_id INT,
    loan_id INT,
    
    PRIMARY KEY (customer_id, loan_id),
    FOREIGN KEY (customer_id) REFERENCES CUSTOMER(customer_id),
    FOREIGN KEY (loan_id) REFERENCES LOAN(loan_id)
);

-- Trigger to enforce max 2 loans per customer
DELIMITER $$
CREATE TRIGGER check_max_loans
BEFORE INSERT ON BORROWS
FOR EACH ROW
BEGIN
    DECLARE loan_count INT;
    
    SELECT COUNT(*) INTO loan_count
    FROM BORROWS
    WHERE customer_id = NEW.customer_id;
    
    IF loan_count >= 2 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Customer cannot have more than 2 loans';
    END IF;
END$$
DELIMITER ;
```

**Common Min-Max Patterns:**

| Pattern  | Meaning                 |
| -------- | ----------------------- |
| `(0, 1)` | Optional, at most one   |
| `(1, 1)` | Exactly one (mandatory) |
| `(0, M)` | Zero or more (optional) |
| `(1, M)` | One or more (mandatory) |
| `(0, N)` | Up to N instances       |

### 5.3 Participation Constraints

Defines whether **all entities must participate** in a relationship.

#### Total Participation

**Every entity must participate** in at least one relationship.

**ER Notation:** Double line `═══`

**ER Diagram:**
```
┌──────┐            ◇           ┌──────────┐
│ LOAN ╞══════must_have════════│ CUSTOMER │
└──────┘                        └──────────┘
  (Total participation)
```

**Meaning:** Every loan **must** have a customer (no loan exists without a customer).

```sql
-- Enforce with NOT NULL foreign key
CREATE TABLE CUSTOMER (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE LOAN (
    loan_id INT PRIMARY KEY,
    amount DECIMAL(10,2),
    customer_id INT NOT NULL,  -- NOT NULL enforces total participation
    
    FOREIGN KEY (customer_id) REFERENCES CUSTOMER(customer_id)
);

-- This will FAIL (total participation violated)
-- INSERT INTO LOAN VALUES (1, 10000, NULL);  ← Error!
```

#### Partial Participation

**Some entities may not participate** in any relationship.

**ER Notation:** Single line `───`

**ER Diagram:**
```
┌──────────┐          ◇          ┌──────┐
│ CUSTOMER │────may_borrow───────│ LOAN │
└──────────┘                     └──────┘
  (Partial participation)
```

**Meaning:** Not all customers have loans (some customers may have zero loans).

```sql
-- Foreign key allows NULL
CREATE TABLE CUSTOMER (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE LOAN (
    loan_id INT PRIMARY KEY,
    amount DECIMAL(10,2)
);

CREATE TABLE BORROWS (
    customer_id INT,
    loan_id INT,
    
    PRIMARY KEY (customer_id, loan_id),
    FOREIGN KEY (customer_id) REFERENCES CUSTOMER(customer_id),
    FOREIGN KEY (loan_id) REFERENCES LOAN(loan_id)
);

-- Valid: Customer with no loans
INSERT INTO CUSTOMER VALUES (101, 'John');
-- No entry in BORROWS for customer 101 → Partial participation
```

### Complete Constraint Example

```
┌─────────┐  (1,1)     ◇     (1,M)   ┌──────────┐
│ STUDENT ╞═══════enrolled_in═══════│ COLLEGE  │
└─────────┘  Total              Partial └──────────┘
```

**Interpretation:**
- Each student must be enrolled in exactly one college (total participation, 1:1)
- Each college can have one or many students (partial participation, 1:M)

```sql
CREATE TABLE COLLEGE (
    college_id INT PRIMARY KEY,
    college_name VARCHAR(100)
);

CREATE TABLE STUDENT (
    student_id INT PRIMARY KEY,
    name VARCHAR(100),
    college_id INT NOT NULL,  -- Total participation
    
    FOREIGN KEY (college_id) REFERENCES COLLEGE(college_id)
);
```

---

## Keys

Keys are attributes that **uniquely identify** entities in an entity set.

### 6.1 Super Key

**Any set of attributes** that can uniquely identify a row.

**Definition:** A super key is any combination of attributes whose values uniquely identify a tuple (row).

**Examples:**

Given `STUDENT(student_id, email, name, phone)`:

**Super Keys:**
- `{student_id}` 
- `{email}` 
- `{student_id, name}`  (still unique)
- `{student_id, email, phone}`  (still unique)
- `{email, name}`  (if email is unique)

**Not a Super Key:**
- `{name}` ✗ (not unique - many students can have same name)
- `{phone}` ✗ (not unique if multiple students share a phone)

### 6.2 Candidate Key

A **minimal super key** - no redundant attributes.

**Definition:** A super key with no unnecessary attributes. If you remove any attribute, it loses uniqueness.

**Test for Candidate Key:**
1. Start with a super key
2. Try removing each attribute one by one
3. If it stays unique after removal → original was NOT a candidate key
4. If it loses uniqueness → original WAS a candidate key

**Examples:**

Given `STUDENT(student_id, email, name, phone)`:

| Set                   | Super Key? | Candidate Key? | Why?                               |
| --------------------- | ---------- | -------------- | ---------------------------------- |
| `{student_id}`        | ✓          | ✓              | Minimal - can't remove anything    |
| `{email}`             | ✓          | ✓              | Minimal - can't remove anything    |
| `{student_id, name}`  | ✓          | ✗              | Can remove `name` and still unique |
| `{student_id, email}` | ✓          | ✗              | Can remove either and still unique |

**Another Example:**

`EMPLOYEE(emp_id, ssn, email, name, dept)`

**Candidate Keys:**
- `{emp_id}` - unique employee ID
- `{ssn}` - unique social security number  
- `{email}` - unique email address

**Not Candidate Keys:**
- `{emp_id, name}` - redundant `name`
- `{ssn, email}` - redundant (either alone is enough)

### 6.3 Primary Key

The **chosen candidate key** to be the main identifier.

**Rules:**
- Must be **unique**
- Must be **NOT NULL**
- Only **one per table**
- Should be **stable** (doesn't change often)
- Preferably **simple** (fewer attributes)

```sql
-- Single attribute primary key
CREATE TABLE STUDENT (
    student_id INT PRIMARY KEY,  -- Primary key
    email VARCHAR(100) UNIQUE,   -- Candidate key (not chosen as primary)
    name VARCHAR(100)
);

-- Named constraint
CREATE TABLE EMPLOYEE (
    emp_id INT,
    ssn VARCHAR(11),
    email VARCHAR(100),
    name VARCHAR(100),
    
    CONSTRAINT pk_employee PRIMARY KEY (emp_id)
);
```

### 6.4 Composite Primary Key

A primary key consisting of **multiple attributes**.

**When to use:**
- Natural combination uniquely identifies entity
- Junction tables (M:M relationships)
- Weak entity sets

```sql
-- Example 1: Student enrollment (M:M relationship)
CREATE TABLE ENROLLMENT (
    student_id INT,
    course_id VARCHAR(10),
    semester VARCHAR(20),
    grade CHAR(1),
    
    PRIMARY KEY (student_id, course_id),  -- Composite key
    FOREIGN KEY (student_id) REFERENCES STUDENT(student_id),
    FOREIGN KEY (course_id) REFERENCES COURSE(course_id)
);

-- Example 2: Flight seat reservation
CREATE TABLE SEAT_RESERVATION (
    flight_number VARCHAR(10),
    seat_number VARCHAR(5),
    passenger_name VARCHAR(100),
    
    PRIMARY KEY (flight_number, seat_number)  -- Composite key
);

-- Example 3: Time slots
CREATE TABLE CLASS_SCHEDULE (
    room_id VARCHAR(10),
    day VARCHAR(10),
    time_slot VARCHAR(20),
    course_id VARCHAR(10),
    
    PRIMARY KEY (room_id, day, time_slot)  -- Three-attribute composite key
);
```

### 6.5 Alternate Key

**Candidate keys that were NOT chosen** as the primary key.

**Implementation:** Use `UNIQUE` constraint

```sql
CREATE TABLE EMPLOYEE (
    emp_id INT PRIMARY KEY,      -- Primary key
    ssn VARCHAR(11) UNIQUE,      -- Alternate key
    email VARCHAR(100) UNIQUE,   -- Alternate key
    name VARCHAR(100)
);

-- Both SSN and email are candidate keys, but emp_id was chosen as primary
```

### 6.6 Foreign Key

An attribute that **references the primary key** of another table.

**Purpose:**
- Establish relationships between tables
- Maintain referential integrity
- Link parent-child tables

```sql
-- Parent table
CREATE TABLE DEPARTMENT (
    dept_id INT PRIMARY KEY,
    dept_name VARCHAR(100)
);

-- Child table
CREATE TABLE EMPLOYEE (
    emp_id INT PRIMARY KEY,
    name VARCHAR(100),
    dept_id INT,  -- Foreign key
    
    FOREIGN KEY (dept_id) REFERENCES DEPARTMENT(dept_id)
);

-- Referential integrity enforced
INSERT INTO DEPARTMENT VALUES (1, 'IT');
INSERT INTO EMPLOYEE VALUES (101, 'John', 1);  -- ✓ Valid

-- This will FAIL (dept_id 99 doesn't exist)
-- INSERT INTO EMPLOYEE VALUES (102, 'Mary', 99);  ← Error!
```

**Foreign Key Actions:**

```sql
CREATE TABLE EMPLOYEE (
    emp_id INT PRIMARY KEY,
    name VARCHAR(100),
    dept_id INT,
    
    FOREIGN KEY (dept_id) REFERENCES DEPARTMENT(dept_id)
        ON DELETE CASCADE      -- Delete employees when department is deleted
        ON UPDATE CASCADE      -- Update emp.dept_id when dept.dept_id changes
);

-- Other options:
-- ON DELETE SET NULL     → Set foreign key to NULL
-- ON DELETE RESTRICT     → Prevent deletion if referenced
-- ON DELETE NO ACTION    → Same as RESTRICT
-- ON UPDATE SET NULL
-- ON UPDATE RESTRICT
```

### Keys in Relationships

For a relationship set, the **combination of primary keys** from participating entities forms the super key.

**Example:** `ENROLLS_IN(student_id, course_id, semester, grade)`

**Super Key:** `{student_id, course_id}`

```sql
CREATE TABLE ENROLLS_IN (
    student_id INT,
    course_id VARCHAR(10),
    semester VARCHAR(20),
    grade CHAR(1),
    
    PRIMARY KEY (student_id, course_id),  -- Super key of relationship
    FOREIGN KEY (student_id) REFERENCES STUDENT(student_id),
    FOREIGN KEY (course_id) REFERENCES COURSE(course_id)
);
```

### MySQL Key Constraint Summary

| Concept | MySQL Constraint | Count per Table | Allows NULL? |
|---------|------------------|-----------------|--------------|
| **Primary Key** | `PRIMARY KEY` | 1 | ✗ No |
| **Candidate Key** | `UNIQUE` | Multiple | ✓ Yes (unless `NOT NULL`) |
| **Alternate Key** | `UNIQUE` | Multiple | ✓ Yes |
| **Foreign Key** | `FOREIGN KEY` | Multiple | ✓ Yes (unless `NOT NULL`) |
| **Super Key** | _(Conceptual)_ | N/A | N/A |

**Complete Example:**

```sql
CREATE TABLE STUDENT (
    -- Primary key
    student_id INT PRIMARY KEY,
    
    -- Alternate keys (candidate keys not chosen as primary)
    email VARCHAR(100) UNIQUE NOT NULL,
    ssn VARCHAR(11) UNIQUE,
    
    -- Regular attributes
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    birth_date DATE,
    
    -- Foreign key
    major_dept_id INT,
    
    FOREIGN KEY (major_dept_id) REFERENCES DEPARTMENT(dept_id)
        ON DELETE SET NULL
        ON UPDATE CASCADE,
    
    -- Composite unique constraint (another candidate key)
    CONSTRAINT uc_name_dob UNIQUE (first_name, last_name, birth_date)
);
```


## Weak Entities

### Definition

A **weak entity** is an entity that:
1. **Cannot be uniquely identified** by its own attributes alone
2. **Depends on a strong entity** (owner) for its existence
3. Has a **partial key (discriminator)** that identifies entities within the owner

### Strong vs Weak Entity

| Aspect | Strong Entity | Weak Entity |
|--------|---------------|-------------|
| **Primary Key** | Has its own | Borrows from owner |
| **Existence** | Independent | Dependent |
| **ER Notation** | Single rectangle `┌───┐` | Double rectangle `╔═══╗` |
| **Example** | `LOAN` | `PAYMENT` |

### ER Diagram Notation

```
┌──────────┐                     ╔═══════════╗
│   LOAN   │                     ║  PAYMENT  ║
│ (Strong) │                     ║  (Weak)   ║
└──────────┘                     ╚═══════════╝
     │                                 │
     │           ◇◇                    │
     └──────has_payment────────────────┘
           (Identifying                 
            Relationship)               
                                  payment_number
                                  (Discriminator)
```

**Notation Summary:**

| Element | Symbol | Description |
|---------|--------|-------------|
| **Weak Entity** | `╔═══╗` | Double rectangle |
| **Identifying Relationship** | `◇◇` | Double diamond |
| **Discriminator (Partial Key)** | Underline with dashes | `payment_number` |
| **Total Participation** | `═══` | Double line (weak entity side) |

### Example 1: Loan and Payment

**Scenario:** 
- A `LOAN` is a strong entity
- A `PAYMENT` is a weak entity (payments exist only for a loan)
- `payment_number` alone doesn't uniquely identify a payment across all loans
- Combination `(loan_number, payment_number)` uniquely identifies a payment

**ER Diagram:**
```
┌────────────────┐                    ╔══════════════════╗
│     LOAN       │                    ║    PAYMENT       ║
├────────────────┤                    ╠══════════════════╣
│ loan_number PK │◇◇───has_payment──◇◇║ payment_number   ║ ← Discriminator
│ amount         │                    ║ payment_date     ║
│ interest_rate  │                    ║ payment_amount   ║
└────────────────┘                    ╚══════════════════╝
      (Strong)                              (Weak)
```

```sql
-- Strong entity
CREATE TABLE LOAN (
    loan_number INT PRIMARY KEY,
    amount DECIMAL(10,2),
    interest_rate DECIMAL(5,2)
);

-- Weak entity
CREATE TABLE PAYMENT (
    payment_number INT NOT NULL,       -- Discriminator (partial key)
    loan_number INT NOT NULL,          -- Foreign key to owner
    payment_date DATE NOT NULL,
    payment_amount DECIMAL(10,2) NOT NULL,
    
    -- Composite primary key: owner's key + discriminator
    PRIMARY KEY (loan_number, payment_number),
    
    -- Foreign key with CASCADE (weak entity depends on strong entity)
    FOREIGN KEY (loan_number) REFERENCES LOAN(loan_number)
        ON DELETE CASCADE  -- If loan deleted, all payments deleted
        ON UPDATE CASCADE
);

-- Data example
INSERT INTO LOAN VALUES (1001, 100000, 8.5);

INSERT INTO PAYMENT VALUES (1, 1001, '2026-01-15', 5000);  -- First payment
INSERT INTO PAYMENT VALUES (2, 1001, '2026-02-15', 5000);  -- Second payment
INSERT INTO PAYMENT VALUES (3, 1001, '2026-03-15', 5000);  -- Third payment

-- payment_number 1, 2, 3 are only unique within loan 1001
-- Another loan can also have payment_number 1, 2, 3
INSERT INTO LOAN VALUES (1002, 50000, 7.5);
INSERT INTO PAYMENT VALUES (1, 1002, '2026-01-20', 2500);  -- payment_number 1 again!
```

### Example 2: Employee and Dependent

**Scenario:**
- `EMPLOYEE` is a strong entity
- `DEPENDENT` is a weak entity (dependents exist only for an employee)
- `dependent_name` alone may not be unique (two employees might have dependents named "John")

**ER Diagram:**
```
┌────────────────┐                     ╔══════════════════╗
│   EMPLOYEE     │                     ║    DEPENDENT     ║
├────────────────┤                     ╠══════════════════╣
│ emp_id PK      │◇◇──has_dependent──◇◇║ dependent_name   ║ ← Discriminator
│ name           │                     ║ birth_date       ║
│ salary         │                     ║ relationship     ║
└────────────────┘                     ╚══════════════════╝
      (Strong)                              (Weak)
```

```sql
-- Strong entity
CREATE TABLE EMPLOYEE (
    emp_id INT PRIMARY KEY,
    name VARCHAR(100),
    salary DECIMAL(10,2)
);

-- Weak entity
CREATE TABLE DEPENDENT (
    emp_id INT NOT NULL,               -- Foreign key to owner
    dependent_name VARCHAR(100) NOT NULL,  -- Discriminator
    birth_date DATE,
    relationship VARCHAR(20),
    
    -- Composite primary key
    PRIMARY KEY (emp_id, dependent_name),
    
    FOREIGN KEY (emp_id) REFERENCES EMPLOYEE(emp_id)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);

-- Data example
INSERT INTO EMPLOYEE VALUES (101, 'John Smith', 75000);
INSERT INTO EMPLOYEE VALUES (102, 'Mary Johnson', 80000);

-- John's dependents
INSERT INTO DEPENDENT VALUES (101, 'Alice Smith', '2010-05-15', 'Daughter');
INSERT INTO DEPENDENT VALUES (101, 'Bob Smith', '2012-08-20', 'Son');

-- Mary's dependent (also named 'Alice' - no conflict!)
INSERT INTO DEPENDENT VALUES (102, 'Alice Johnson', '2015-03-10', 'Daughter');
```

### Example 3: Building and Apartment

```sql
-- Strong entity
CREATE TABLE BUILDING (
    building_id INT PRIMARY KEY,
    building_name VARCHAR(100),
    address VARCHAR(200)
);

-- Weak entity
CREATE TABLE APARTMENT (
    building_id INT NOT NULL,
    apartment_number VARCHAR(10) NOT NULL,  -- Discriminator (e.g., '101', '102')
    bedrooms INT,
    rent DECIMAL(10,2),
    
    PRIMARY KEY (building_id, apartment_number),
    
    FOREIGN KEY (building_id) REFERENCES BUILDING(building_id)
        ON DELETE CASCADE
);

-- Data
INSERT INTO BUILDING VALUES (1, 'Sunset Apartments', '123 Main St');
INSERT INTO BUILDING VALUES (2, 'Ocean View', '456 Beach Rd');

-- Building 1 apartments
INSERT INTO APARTMENT VALUES (1, '101', 2, 1200);
INSERT INTO APARTMENT VALUES (1, '102', 2, 1200);
INSERT INTO APARTMENT VALUES (1, '201', 3, 1500);

-- Building 2 apartments (apartment_number '101' again - no conflict!)
INSERT INTO APARTMENT VALUES (2, '101', 1, 1000);
```

### Example 4: University - Department - Course

**Multi-level weak entity hierarchy**

```sql
-- Strong entity
CREATE TABLE UNIVERSITY (
    university_id INT PRIMARY KEY,
    university_name VARCHAR(100)
);

-- Weak entity (depends on UNIVERSITY)
CREATE TABLE DEPARTMENT (
    university_id INT NOT NULL,
    dept_code VARCHAR(10) NOT NULL,  -- Discriminator
    dept_name VARCHAR(100),
    
    PRIMARY KEY (university_id, dept_code),
    FOREIGN KEY (university_id) REFERENCES UNIVERSITY(university_id)
        ON DELETE CASCADE
);

-- Weak entity (depends on DEPARTMENT, which depends on UNIVERSITY)
CREATE TABLE COURSE (
    university_id INT NOT NULL,
    dept_code VARCHAR(10) NOT NULL,
    course_number VARCHAR(10) NOT NULL,  -- Discriminator
    course_name VARCHAR(100),
    credits INT,
    
    PRIMARY KEY (university_id, dept_code, course_number),
    FOREIGN KEY (university_id, dept_code) 
        REFERENCES DEPARTMENT(university_id, dept_code)
        ON DELETE CASCADE
);
```

### Why Use Weak Entities?

**1. Natural Modeling:**
- Some entities naturally depend on others
- Payments only exist for loans
- Apartment numbers only make sense within a building

**2. Prevent Orphaned Records:**
- CASCADE delete ensures consistency
- When loan deleted, all its payments deleted automatically

**3. Simplify Keys:**
- No need for globally unique payment IDs
- Use simple sequential numbers within each loan

**4. Enforce Business Rules:**
- Payment cannot exist without a loan
- Dependent cannot exist without an employee

### Weak Entity Rules

1. **Must have total participation** in identifying relationship
2. **Must have one-to-many** relationship with owner (one loan → many payments)
3. **Primary key = owner's key + discriminator**
4. **Use CASCADE for referential integrity**
5. **Discriminator must be underlined** with dashes in ER diagram

---

## Aggregation

**Aggregation** treats a **relationship as an entity** to participate in another relationship.

**When to use:**
- When a relationship itself needs to relate to another entity
- Modeling ternary relationships that need additional associations

**ER Diagram:**
```
         ┌──────────────────────────────────┐
         │        Aggregation               │
         │                                  │
         │  ┌─────────┐       ┌─────────┐   │
         │  │ EMPLOYEE│───◇───│ PROJECT │   │         ┌──────────┐
         │  └─────────┘ works └─────────┘   │────◇────│ MANAGER  │
         │               on                 │ manages └──────────┘
         └──────────────────────────────────┘
```

**Example Scenario:** 
- Employees work on Projects
- Managers manage which employees work on which projects

```sql
-- Entities
CREATE TABLE EMPLOYEE (
    emp_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE PROJECT (
    project_id INT PRIMARY KEY,
    project_name VARCHAR(100)
);

CREATE TABLE MANAGER (
    manager_id INT PRIMARY KEY,
    name VARCHAR(100)
);

-- Base relationship: Employee works on Project
CREATE TABLE WORKS_ON (
    emp_id INT,
    project_id INT,
    hours_per_week INT,
    
    PRIMARY KEY (emp_id, project_id),
    FOREIGN KEY (emp_id) REFERENCES EMPLOYEE(emp_id),
    FOREIGN KEY (project_id) REFERENCES PROJECT(project_id)
);

-- Aggregation: Manager manages the "works_on" relationship
CREATE TABLE MANAGES_WORK (
    manager_id INT,
    emp_id INT,
    project_id INT,
    assignment_date DATE,
    
    PRIMARY KEY (manager_id, emp_id, project_id),
    FOREIGN KEY (manager_id) REFERENCES MANAGER(manager_id),
    -- This foreign key references the aggregated relationship
	    FOREIGN KEY (emp_id, project_id) REFERENCES WORKS_ON(emp_id, Q5G3 BHJMNRWY16CFTV241)
);
-- So we reference the relation we are refering to in aggregates
```


---
# Enhanced Entity-Relationship (EER) Model

## Introduction to EER Model 

### What is the EER Model?

The **Enhanced Entity-Relationship (EER) Model** extends the basic ER model by adding:
- **Subclasses and Superclasses** (specialization/generalization)
- **Attribute Inheritance**
- **Constraints** on relationships between superclasses and subclasses
- **Union Types (Categories)**

### Why Do We Need EER?

The standard ER model couldn't represent:
- **Hierarchical classifications** (e.g., Employees → Managers, Engineers, Secretaries)
- **Shared properties** across entity types
- **Complex real-world scenarios** requiring object-oriented concepts

**Real-World Example:**

In a company database:
- All employees share common attributes (name, SSN, birth_date)
- But **Managers** have unique attributes (bonus, dept_managed)
- **Engineers** have unique attributes (engineering_type, certifications)
- **Secretaries** have unique attributes (typing_speed, office_hours)

The EER model allows us to model this hierarchy efficiently without duplicating the common attributes.

---

## Superclasses and Subclasses {#superclass-subclass}

### The IS-A Relationship

A **subclass** IS-A specialized version of a **superclass**.

**Definition:**
- **Superclass (Supertype):** A general entity type
- **Subclass (Subtype):** A specialized entity type that inherits from the superclass

**Examples:**
- `SECRETARY` IS-A `EMPLOYEE`
- `ENGINEER` IS-A `EMPLOYEE`
- `MANAGER` IS-A `EMPLOYEE`
- `CAR` IS-A `VEHICLE`
- `CHECKING_ACCOUNT` IS-A `ACCOUNT`

### EER Diagram Notation

```
                ┌─────────────┐
                │  EMPLOYEE   │  ← Superclass
                │ (SSN, Name) │
                └──────┬──────┘
                       │
                       ○  ← Circle (specialization symbol)
                      ╱│╲
                     ╱ │ ╲
                    ╱  │  ╲
        ┌──────────┐ ┌┴────────┐ ┌──────────┐
        │SECRETARY │ │ENGINEER │ │TECHNICIAN│  ← Subclasses
        └──────────┘ └─────────┘ └──────────┘
```

**Symbol Breakdown:**

| Symbol               | Meaning                                 |
| -------------------- | --------------------------------------- |
| **Rectangle**        | Entity (superclass or subclass)         |
| **Circle ○**         | Specialization/Generalization connector |
| **Line from circle** | Connects superclass to subclasses       |
| **`d` in circle**    | Disjoint constraint                     |
| **`o` in circle**    | Overlapping constraint                  |
| **Double line**      | Total participation (completeness)      |
| **Single line**      | Partial participation                   |

### Existence Dependency

**Critical Rule:** An entity **cannot exist only in a subclass**; it must also exist in the superclass.

**Example:**
- You cannot be a `SECRETARY` without first being an `EMPLOYEE`
- Every secretary has all employee attributes (SSN, name, birth_date) PLUS secretary-specific attributes (typing_speed)

---

## 3. Specialization (Top-Down)

### Definition

**Specialization** is the process of defining **subclasses** from a **superclass** based on distinguishing characteristics.

**Direction:** Top → Down (from general to specific)

![[Pasted image 20260313005740.png]]

### Process
1. Start with a general superclass
2. Identify distinguishing characteristics
3. Define subclasses based on these characteristics
4. Add subclass-specific attributes

### Example 1: Company Employee Database

**Superclass:** `EMPLOYEE`
- Common attributes: `SSN`, `Name`, `Birth_date`, `Address`, `Salary`

**Specialization based on job type:**

```
                    ┌─────────────────────────┐
                    │       EMPLOYEE          │
                    │ SSN, Name, Birth_date   │
                    │ Address, Salary         │
                    └───────────┬─────────────┘
                                │
                                ○ d  ← Disjoint
                               ╱│╲
                              ╱ │ ╲
                             ╱  │  ╲
              ┌────────────┐  ┌┴──────────┐  ┌───────────┐
              │ SECRETARY  │  │ ENGINEER  │  │ TECHNICIAN│
              │Typing_speed│  │Eng_type   │  │TGrade     │
              │Office_hours│  │Certs      │  │           │
              └────────────┘  └───────────┘  └───────────┘
```

**MySQL Implementation:**

```sql
-- Superclass
CREATE TABLE EMPLOYEE (
    ssn VARCHAR(11) PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    birth_date DATE,
    address VARCHAR(200),
    salary DECIMAL(10,2),
    job_type VARCHAR(20)  -- Attribute to determine subclass
);

-- Subclass: Secretary
CREATE TABLE SECRETARY (
    ssn VARCHAR(11) PRIMARY KEY,
    typing_speed INT,  -- Words per minute
    office_hours VARCHAR(50),
    
    FOREIGN KEY (ssn) REFERENCES EMPLOYEE(ssn)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);

-- Subclass: Engineer
CREATE TABLE ENGINEER (
    ssn VARCHAR(11) PRIMARY KEY,
    engineering_type VARCHAR(50),  -- 'Software', 'Mechanical', 'Electrical'
    certifications TEXT,
    
    FOREIGN KEY (ssn) REFERENCES EMPLOYEE(ssn)
        ON DELETE CASCADE
);

-- Subclass: Technician
CREATE TABLE TECHNICIAN (
    ssn VARCHAR(11) PRIMARY KEY,
    tgrade INT,  -- Technical grade level
    
    FOREIGN KEY (ssn) REFERENCES EMPLOYEE(ssn)
        ON DELETE CASCADE
);

-- Sample data
INSERT INTO EMPLOYEE VALUES 
    ('123-45-6789', 'Alice Johnson', '1990-05-15', '123 Main St', 45000, 'Secretary'),
    ('234-56-7890', 'Bob Smith', '1985-08-22', '456 Oak Ave', 75000, 'Engineer'),
    ('345-67-8901', 'Carol White', '1992-11-30', '789 Elm St', 55000, 'Technician');

INSERT INTO SECRETARY VALUES ('123-45-6789', 80, '9AM-5PM');
INSERT INTO ENGINEER VALUES ('234-56-7890', 'Software', 'AWS Certified, Oracle DBA');
INSERT INTO TECHNICIAN VALUES ('345-67-8901', 3);
```

### Example 2: University Student Specialization

```
                    ┌─────────────────────────┐
                    │       STUDENT           │
                    │ Student_ID, Name        │
                    │ Email, DOB              │
                    └───────────┬─────────────┘
                                │
                                ○ d  ← Disjoint (can't be both)
                               ╱│╲
                              ╱ │ ╲
                             ╱  │  ╲
              ┌─────────────┐  ┌┴──────────────┐
              │UNDERGRADUATE│  │   GRADUATE    │
              │Major        │  │Thesis_topic   │
              │Class_year   │  │Advisor_name   │
              └─────────────┘  └───────────────┘
```

```sql
CREATE TABLE STUDENT (
    student_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100) UNIQUE,
    dob DATE,
    student_type VARCHAR(20)  -- 'Undergraduate' or 'Graduate'
);

CREATE TABLE UNDERGRADUATE (
    student_id INT PRIMARY KEY,
    major VARCHAR(50),
    class_year INT,  -- 1, 2, 3, 4
    
    FOREIGN KEY (student_id) REFERENCES STUDENT(student_id)
        ON DELETE CASCADE
);

CREATE TABLE GRADUATE (
    student_id INT PRIMARY KEY,
    thesis_topic VARCHAR(200),
    advisor_name VARCHAR(100),
    degree_type ENUM('Masters', 'PhD'),
    
    FOREIGN KEY (student_id) REFERENCES STUDENT(student_id)
        ON DELETE CASCADE
);
```

### Example 3: Bank Account Specialization

```
                    ┌─────────────────────────┐
                    │       ACCOUNT           │
                    │ Account_no, Balance     │
                    │ Date_opened             │
                    └───────────┬─────────────┘
                                │
                                ○ o  ← Overlapping (can have both)
                               ╱│╲
                              ╱ │ ╲
                             ╱  │  ╲
              ┌─────────────┐  ┌┴──────────────┐
              │  SAVINGS    │  │   CHECKING    │
              │Interest_rate│  │Overdraft_limit│
              │Min_balance  │  │Fee_per_check  │
              └─────────────┘  └───────────────┘
```

```sql
CREATE TABLE ACCOUNT (
    account_no VARCHAR(20) PRIMARY KEY,
    balance DECIMAL(12,2),
    date_opened DATE,
    customer_id INT
);

CREATE TABLE SAVINGS (
    account_no VARCHAR(20) PRIMARY KEY,
    interest_rate DECIMAL(5,2),  -- e.g., 3.5 for 3.5%
    min_balance DECIMAL(10,2),
    
    FOREIGN KEY (account_no) REFERENCES ACCOUNT(account_no)
);

CREATE TABLE CHECKING (
    account_no VARCHAR(20) PRIMARY KEY,
    overdraft_limit DECIMAL(10,2),
    fee_per_check DECIMAL(5,2),
    
    FOREIGN KEY (account_no) REFERENCES ACCOUNT(account_no)
);

-- Example: Account that is BOTH savings and checking
INSERT INTO ACCOUNT VALUES ('ACC001', 5000.00, '2025-01-15', 101);
INSERT INTO SAVINGS VALUES ('ACC001', 3.5, 1000.00);
INSERT INTO CHECKING VALUES ('ACC001', 500.00, 0.25);
```

---
## 4. Generalization (Bottom-Up) 

### Definition
**Generalization** is the **reverse** of specialization. It's the process of combining multiple entity types with common features into a single **superclass**.
**Direction:** Bottom → Up (from specific to general)

### Process

1. Identify multiple entity types with common attributes
2. Extract common attributes
3. Create a superclass with these common attributes
4. Make original entities subclasses

![[Pasted image 20260313005817.png]]

### Example 1: Vehicle Generalization

**Before Generalization:**

```
┌─────────┐       ┌─────────┐       ┌─────────┐
│   CAR   │       │  TRUCK  │       │   BUS   │
│ VIN     │       │ VIN     │       │ VIN     │
│ Make    │       │ Make    │       │ Make    │
│ Model   │       │ Model   │       │ Model   │
│ NumDoors│       │ Tonnage │       │ Capacity│
└─────────┘       └─────────┘       └─────────┘
```

Common attributes: `VIN`, `Make`, `Model`, `Year`, `Color`

**After Generalization:**

```
                    ┌─────────────────────────┐
                    │       VEHICLE           │  ← Generalized superclass
                    │ VIN, Make, Model        │
                    │ Year, Color             │
                    └───────────┬─────────────┘
                                │
                                ○ d
                               ╱│╲
                              ╱ │ ╲
                             ╱  │  ╲
              ┌───────────┐  ┌┴────────┐  ┌─────────┐
              │    CAR    │  │  TRUCK  │  │   BUS   │
              │ NumDoors  │  │ Tonnage │  │Capacity │
              │ BodyStyle │  │BedLength│  │RouteNum │
              └───────────┘  └─────────┘  └─────────┘
```

**MySQL Implementation:**

```sql
-- Generalized superclass
CREATE TABLE VEHICLE (
    vin VARCHAR(17) PRIMARY KEY,
    make VARCHAR(50),
    model VARCHAR(50),
    year INT,
    color VARCHAR(30),
    vehicle_type VARCHAR(20)  -- 'Car', 'Truck', 'Bus'
);

-- Subclass: Car
CREATE TABLE CAR (
    vin VARCHAR(17) PRIMARY KEY,
    num_doors INT,
    body_style VARCHAR(30),  -- 'Sedan', 'Coupe', 'Hatchback', 'SUV'
    
    FOREIGN KEY (vin) REFERENCES VEHICLE(vin)
        ON DELETE CASCADE
);

-- Subclass: Truck
CREATE TABLE TRUCK (
    vin VARCHAR(17) PRIMARY KEY,
    tonnage DECIMAL(8,2),  -- Cargo capacity in tons
    bed_length DECIMAL(5,2),  -- In feet
    four_wheel_drive BOOLEAN,
    
    FOREIGN KEY (vin) REFERENCES VEHICLE(vin)
        ON DELETE CASCADE
);

-- Subclass: Bus
CREATE TABLE BUS (
    vin VARCHAR(17) PRIMARY KEY,
    passenger_capacity INT,
    route_number VARCHAR(20),
    wheelchair_accessible BOOLEAN,
    
    FOREIGN KEY (vin) REFERENCES VEHICLE(vin)
        ON DELETE CASCADE
);

-- Sample data
INSERT INTO VEHICLE VALUES 
    ('1HGBH41JXMN109186', 'Honda', 'Accord', 2022, 'Silver', 'Car'),
    ('1FTFW1ET5DFC10312', 'Ford', 'F-150', 2021, 'Blue', 'Truck'),
    ('5TDKK3DC2ES123456', 'Toyota', 'School Bus', 2020, 'Yellow', 'Bus');

INSERT INTO CAR VALUES ('1HGBH41JXMN109186', 4, 'Sedan');
INSERT INTO TRUCK VALUES ('1FTFW1ET5DFC10312', 2.5, 6.5, TRUE);
INSERT INTO BUS VALUES ('5TDKK3DC2ES123456', 45, 'Route 12', TRUE);
```

### Example 2: Person Generalization

**Scenario:** University database has separate entities for `FACULTY`, `STAFF`, and `STUDENT`. All have common attributes.

**After Generalization:**

```
                    ┌─────────────────────────┐
                    │       PERSON            │
                    │ Person_ID, Name         │
                    │ DOB, Address, Phone     │
                    └───────────┬─────────────┘
                                │
                                ○ o  ← Overlapping
                               ╱│╲
                              ╱ │ ╲
                             ╱  │  ╲
              ┌──────────┐  ┌┴────────┐  ┌─────────┐
              │ FACULTY  │  │  STAFF  │  │ STUDENT │
              │ Rank     │  │Position │  │Major    │
              │ Office   │  │Dept     │  │GPA      │
              └──────────┘  └─────────┘  └─────────┘
```

```sql
CREATE TABLE PERSON (
    person_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    dob DATE,
    address VARCHAR(200),
    phone VARCHAR(15),
    email VARCHAR(100) UNIQUE
);

CREATE TABLE FACULTY (
    person_id INT PRIMARY KEY,
    rank VARCHAR(50),  -- 'Assistant Professor', 'Associate Professor', 'Professor'
    office VARCHAR(20),
    research_area VARCHAR(100),
    
    FOREIGN KEY (person_id) REFERENCES PERSON(person_id)
);

CREATE TABLE STAFF (
    person_id INT PRIMARY KEY,
    position VARCHAR(50),
    department VARCHAR(50),
    hire_date DATE,
    
    FOREIGN KEY (person_id) REFERENCES PERSON(person_id)
);

CREATE TABLE STUDENT (
    person_id INT PRIMARY KEY,
    major VARCHAR(50),
    gpa DECIMAL(3,2),
    enrollment_date DATE,
    
    FOREIGN KEY (person_id) REFERENCES PERSON(person_id)
);

-- Example: Person who is both Faculty and Student (PhD student teaching)
INSERT INTO PERSON VALUES (NULL, 'Dr. John Smith', '1985-03-15', '123 College Ave', '555-1234', 'john@univ.edu');
SET @person_id = LAST_INSERT_ID();

INSERT INTO FACULTY VALUES (@person_id, 'Assistant Professor', 'SB-301', 'Database Systems');
INSERT INTO STUDENT VALUES (@person_id, 'Computer Science PhD', 3.95, '2020-09-01');
```

---
## Constraints in EER 

EER models use **three types of constraints** to define the rules governing superclass/subclass relationships.

### Disjointness Constraint

**Question:** Can an entity belong to **multiple subclasses** at the same time?

#### Disjoint (`d`)

An entity can be a member of **at most ONE** subclass.
**Symbol:** `d` inside the circle

**Real-World Examples:**

1. **Student Classification:**
   ```
   STUDENT → {UNDERGRADUATE, GRADUATE}  [Disjoint]
   ```
   A student cannot be both undergraduate AND graduate simultaneously.

2. **Employee Job Type:**
   ```
   EMPLOYEE → {SECRETARY, ENGINEER, TECHNICIAN}  [Disjoint]
   ```
   An employee has ONE job classification.

**MySQL Enforcement:**
```mysql
-- Method 1: Use CHECK constraint (MySQL 8.0+)
CREATE TABLE EMPLOYEE (
    ssn VARCHAR(11) PRIMARY KEY,
    name VARCHAR(100),
    job_type VARCHAR(20),
    
    CHECK (job_type IN ('Secretary', 'Engineer', 'Technician'))
);

-- Method 2: Use triggers to enforce disjointness
DELIMITER $$

CREATE TRIGGER check_employee_disjoint_insert
BEFORE INSERT ON SECRETARY
FOR EACH ROW
BEGIN
    IF EXISTS (SELECT 1 FROM ENGINEER WHERE ssn = NEW.ssn) 
       OR EXISTS (SELECT 1 FROM TECHNICIAN WHERE ssn = NEW.ssn) THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Employee cannot be in multiple job categories';
    END IF;
END$$

DELIMITER ;
```

#### Overlapping (`o`)

An entity can belong to **multiple subclasses** simultaneously.

**Symbol:** `o` inside the circle

**Real-World Examples:**

1. **Person Roles:**
   ```
   PERSON → {EMPLOYEE, ALUMNUS, STUDENT}  [Overlapping]
   ```
   A person can be an employee (professor) AND an alumnus (graduated from there) simultaneously.

2. **Parts Classification:**
   ```
   PART → {MANUFACTURED_PART, PURCHASED_PART}  [Overlapping]
   ```
   A company might manufacture a part in-house AND purchase it from suppliers as backup.

3. **Bank Account:**
   ```
   ACCOUNT → {SAVINGS, CHECKING}  [Overlapping]
   ```
   An account can be BOTH savings (earns interest) and checking (has checks).

**MySQL Implementation:**

```mysql
CREATE TABLE PERSON (
    person_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE EMPLOYEE (
    person_id INT PRIMARY KEY,
    employee_id VARCHAR(20),
    hire_date DATE,
    
    FOREIGN KEY (person_id) REFERENCES PERSON(person_id)
);

CREATE TABLE ALUMNUS (
    person_id INT PRIMARY KEY,
    graduation_year INT,
    degree VARCHAR(50),
    
    FOREIGN KEY (person_id) REFERENCES PERSON(person_id)
);

CREATE TABLE STUDENT (
    person_id INT PRIMARY KEY,
    student_id VARCHAR(20),
    major VARCHAR(50),
    
    FOREIGN KEY (person_id) REFERENCES PERSON(person_id)
);

-- Example: Person who is BOTH employee and alumnus
INSERT INTO PERSON VALUES (101, 'Dr. Jane Smith');
INSERT INTO EMPLOYEE VALUES (101, 'E12345', '2015-08-15');
INSERT INTO ALUMNUS VALUES (101, 2010, 'PhD Computer Science');
-- Same person (101) exists in both EMPLOYEE and ALUMNUS subclasses
```

#### But then what would be difference between Overlapping Specialization and Generalization EER?

NO. Overlapping is a **constraint type**, not a direction indicator. 
**The direction (specialization vs generalization) is about the DESIGN PROCESS, not the constraint.** 
The overlapping constraint describes **membership rules**, not the **design process**. Whether you designed it top-down (specialization) or bottom-up (generalization), you can end up with the same overlapping constraint. The final EER diagram and SQL implementation are identical regardless of which approach you used. 

### 5.2 Completeness Constraint (Participation)

**Question:** Must **every entity** in the superclass belong to **at least one** subclass?

#### Total Participation (Mandatory)
**Every** entity in the superclass **must** belong to at least one subclass.
**Symbol:** Double line from superclass to circle `═══`

**Real-World Examples:**

1. **Hospital Patients:**
   ```
   PATIENT → {OUTPATIENT, RESIDENT_PATIENT}  [Total]
   ```
   Every patient MUST be classified as either outpatient OR resident.

2. **Vehicle Registration:**
   ```
   VEHICLE → {CAR, TRUCK, MOTORCYCLE, BUS}  [Total]
   ```
   Every registered vehicle must fall into one of these categories.

**EER Diagram:**

```
                    ┌─────────────────────┐
                    │      PATIENT        │
                    └══════════┬┬═════════┘  ← Double line
                               ││
                               ○ d  ← Total, Disjoint
                              ╱ ╲
                             ╱   ╲
                ┌───────────┴┐   ┌┴───────────────┐
                │ OUTPATIENT │   │RESIDENT_PATIENT│
                └────────────┘   └────────────────┘
```

**MySQL Enforcement:**
```sql
CREATE TABLE PATIENT (
    patient_id INT PRIMARY KEY,
    name VARCHAR(100),
    patient_type VARCHAR(20) NOT NULL,  -- Force classification
    
    CHECK (patient_type IN ('Outpatient', 'Resident'))
);

-- Trigger to ensure total participation
DELIMITER $$

CREATE TRIGGER enforce_total_participation
AFTER INSERT ON PATIENT
FOR EACH ROW
BEGIN
    DECLARE subclass_count INT;
    
    SELECT (
        (SELECT COUNT(*) FROM OUTPATIENT WHERE patient_id = NEW.patient_id) +
        (SELECT COUNT(*) FROM RESIDENT_PATIENT WHERE patient_id = NEW.patient_id)
    ) INTO subclass_count;
    
    IF subclass_count = 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Patient must belong to at least one subclass';
    END IF;
END$$

DELIMITER ;
```

#### Partial Participation (Optional)

Some entities in the superclass **may NOT** belong to any subclass.
**Symbol:** Single line from superclass to circle `───`

**Real-World Examples:**

1. **Company Employees:**
   ```
   EMPLOYEE → {MANAGER, ENGINEER}  [Partial]
   ```
   Not all employees are managers or engineers (janitors, HR, security exist as just employees).

2. **University Courses:**
   ```
   COURSE → {ONLINE_COURSE, LAB_COURSE}  [Partial]
   ```
   Some courses are regular lecture courses (neither online nor lab).

**EER Diagram:**
```
                    ┌─────────────────────┐
                    │      EMPLOYEE       │
                    └───────────┬─────────┘  ← Single line
                                │
                                ○ d  ← Partial, Disjoint
                               ╱│╲
                              ╱ │ ╲
              ┌────────────┐  ┌┴────────┐
              │  MANAGER   │  │ ENGINEER│
              └────────────┘  └─────────┘
```

**MySQL Implementation:**
```sql
CREATE TABLE EMPLOYEE (
    ssn VARCHAR(11) PRIMARY KEY,
    name VARCHAR(100),
    salary DECIMAL(10,2)
    -- No mandatory job_type field
);

CREATE TABLE MANAGER (
    ssn VARCHAR(11) PRIMARY KEY,
    bonus DECIMAL(10,2),
    dept_managed VARCHAR(50),
    
    FOREIGN KEY (ssn) REFERENCES EMPLOYEE(ssn)
);

CREATE TABLE ENGINEER (
    ssn VARCHAR(11) PRIMARY KEY,
    engineering_type VARCHAR(50),
    
    FOREIGN KEY (ssn) REFERENCES EMPLOYEE(ssn)
);

-- Valid: Employee who is neither manager nor engineer
INSERT INTO EMPLOYEE VALUES ('111-22-3333', 'Mike Johnson', 40000);
-- No corresponding entry in MANAGER or ENGINEER tables
```

![[Pasted image 20260313010515.png]]
![[Pasted image 20260313011426.png]]

### [[So, is Generalization always have to be total participation and can never be partial participation]]

### 5.3 Membership Constraint (Defining Predicates)

**Question:** **How** is membership in a subclass determined?

#### Attribute-Defined (Condition-Defined)

Membership is **automatically determined** by the value of a specific attribute. 
**Notation:** Attribute name shown next to the circle 

**Real-World Examples:**

1. **Bank Accounts by Type:**
   ```
   ACCOUNT → {SAVINGS, CHECKING}  [Based on Account_Type]
   ```
   If `Account_Type = 'Savings'` → goes to SAVINGS subclass
   If `Account_Type = 'Checking'` → goes to CHECKING subclass

2. **Students by Level:**
   ```
   STUDENT → {FRESHMAN, SOPHOMORE, JUNIOR, SENIOR}  [Based on Class_Year]
   ```

**EER Diagram:**

```
                    ┌─────────────────────┐
                    │      ACCOUNT        │
                    │ Account_Type        │  ← Defining attribute
                    └───────────┬─────────┘
                                │
                        Account_Type  ← Label on line
                                ○ d
                               ╱│╲
              ┌──────────┐    ┌┴──────────┐
              │ SAVINGS  │    │ CHECKING  │
              └──────────┘    └───────────┘
```

**MySQL Implementation:**

```sql
CREATE TABLE ACCOUNT (
    account_no VARCHAR(20) PRIMARY KEY,
    balance DECIMAL(12,2),
    account_type VARCHAR(20) NOT NULL,  -- Defining attribute
    
    CHECK (account_type IN ('Savings', 'Checking'))
);

-- Use views to represent subclasses
CREATE VIEW SAVINGS AS
SELECT * FROM ACCOUNT WHERE account_type = 'Savings';

CREATE VIEW CHECKING AS
SELECT * FROM ACCOUNT WHERE account_type = 'Checking';

-- Or use tables with automatic insertion
CREATE TABLE SAVINGS (
    account_no VARCHAR(20) PRIMARY KEY,
    interest_rate DECIMAL(5,2),
    
    FOREIGN KEY (account_no) REFERENCES ACCOUNT(account_no)
);

-- Trigger to auto-insert based on account_type
DELIMITER $$

CREATE TRIGGER auto_classify_account
AFTER INSERT ON ACCOUNT
FOR EACH ROW
BEGIN
    IF NEW.account_type = 'Savings' THEN
        INSERT INTO SAVINGS (account_no, interest_rate) 
        VALUES (NEW.account_no, 2.5);  -- Default interest rate
    END IF;
END$$

DELIMITER ;
```

#### User-Defined

Membership is **manually specified** by a user or application; no automatic rule exists.

**Real-World Examples:**

1. **VIP Customers:**
   ```
   CUSTOMER → {VIP_CUSTOMER}  [User-Defined]
   ```
   Store manager manually designates certain customers as VIP based on loyalty, spending, or personal relationship.

2. **Featured Products:**
   ```
   PRODUCT → {FEATURED_PRODUCT}  [User-Defined]
   ```
   Marketing team manually selects which products to feature.

**MySQL Implementation:**

```sql
CREATE TABLE CUSTOMER (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
);

CREATE TABLE VIP_CUSTOMER (
    customer_id INT PRIMARY KEY,
    vip_since DATE,
    assigned_by VARCHAR(100),  -- Who made them VIP
    notes TEXT,
    
    FOREIGN KEY (customer_id) REFERENCES CUSTOMER(customer_id)
);

-- Manager manually adds VIP status
INSERT INTO CUSTOMER VALUES (101, 'Alice Brown', 'alice@email.com');
-- Later, manager decides to make her VIP
INSERT INTO VIP_CUSTOMER VALUES (101, '2026-03-11', 'Manager John', 'Long-time loyal customer');
```

### Constraint Combination Examples

#### Example 1: Disjoint + Total

**Scenario:** Every vehicle must be exactly one type.

```
                    VEHICLE
                   (VIN, Make)
                       ═══  ← Total (double line)
                        ○ d  ← Disjoint
                       ╱│╲
                 CAR  TRUCK  MOTORCYCLE
```

**Interpretation:** Every vehicle MUST be classified, and can be ONLY one type.

```sql
CREATE TABLE VEHICLE (
    vin VARCHAR(17) PRIMARY KEY,
    make VARCHAR(50),
    vehicle_type VARCHAR(20) NOT NULL,  -- Total: NOT NULL
    
    CHECK (vehicle_type IN ('Car', 'Truck', 'Motorcycle'))  -- Disjoint
);
```

#### Example 2: Overlapping + Partial

**Scenario:** Employees may have multiple roles, but not required.

```
                    EMPLOYEE
                   (SSN, Name)
                       ───  ← Partial (single line)
                        ○ o  ← Overlapping
                       ╱│╲
              MANAGER  ENGINEER  SALESPERSON
```

**Interpretation:** An employee may be none, one, or multiple roles.

```sql
CREATE TABLE EMPLOYEE (
    ssn VARCHAR(11) PRIMARY KEY,
    name VARCHAR(100)
);

-- Can exist in multiple subclasses or none
CREATE TABLE MANAGER (
    ssn VARCHAR(11) PRIMARY KEY,
    FOREIGN KEY (ssn) REFERENCES EMPLOYEE(ssn)
);

CREATE TABLE ENGINEER (
    ssn VARCHAR(11) PRIMARY KEY,
    FOREIGN KEY (ssn) REFERENCES EMPLOYEE(ssn)
);

CREATE TABLE SALESPERSON (
    ssn VARCHAR(11) PRIMARY KEY,
    FOREIGN KEY (ssn) REFERENCES EMPLOYEE(ssn)
);

-- Valid scenarios:
-- 1. Employee in no subclass
-- 2. Employee only in MANAGER
-- 3. Employee in both MANAGER and ENGINEER
```

#### Example 3: Disjoint + Partial + Attribute-Defined

**Scenario:** Some courses have special types, determined by course format.

```
                    COURSE
                (Course_ID, Format)
                       ───  ← Partial
                     Format ← Defining attribute
                        ○ d  ← Disjoint
                       ╱│╲
              ONLINE  LAB  HYBRID
```

```sql
CREATE TABLE COURSE (
    course_id VARCHAR(10) PRIMARY KEY,
    course_name VARCHAR(100),
    format VARCHAR(20),  -- Can be NULL (partial)
    
    CHECK (format IS NULL OR format IN ('Online', 'Lab', 'Hybrid'))
);

-- Regular course with no special format
INSERT INTO COURSE VALUES ('CS101', 'Intro to CS', NULL);

-- Special format courses
INSERT INTO COURSE VALUES ('CS102', 'Programming Lab', 'Lab');
INSERT INTO COURSE VALUES ('CS103', 'Web Development', 'Online');
```

---
## 6. Attribute Inheritance

### Definition
**Attribute Inheritance:** An entity in a subclass **automatically inherits ALL attributes and relationships** from its superclass.

### Rules

1. **Inherited attributes** don't need to be repeated in the subclass
2. **Subclass-specific attributes** are added to the inherited ones
3. **Primary key** of the superclass becomes the primary key of the subclass
4. Changes to superclass attributes affect all subclasses

### Example: Employee Hierarchy

**Superclass:** `EMPLOYEE (SSN, Name, Birth_date, Address, Salary)`

**Subclass:** `SECRETARY`
- **Inherited:** SSN, Name, Birth_date, Address, Salary
- **Own attributes:** Typing_speed, Office_hours

**Complete attribute set for SECRETARY:**
```
{SSN, Name, Birth_date, Address, Salary, Typing_speed, Office_hours}
```

**MySQL Query to Get Complete Information:**

```sql
-- Get all attributes for a secretary (inherited + own)
SELECT 
    e.ssn,
    e.name,
    e.birth_date,
    e.address,
    e.salary,          -- All inherited from EMPLOYEE
    s.typing_speed,
    s.office_hours     -- Own attributes
FROM EMPLOYEE e
JOIN SECRETARY s ON e.ssn = s.ssn
WHERE e.ssn = '123-45-6789';
```

### Inheritance with Relationships

Subclasses also inherit **relationships** from the superclass.

**Example:**

```
EMPLOYEE ──works_for→ DEPARTMENT

SECRETARY (subclass of EMPLOYEE) automatically has works_for relationship
```

```sql
-- Employee works for Department
CREATE TABLE DEPARTMENT (
    dept_id INT PRIMARY KEY,
    dept_name VARCHAR(100)
);

CREATE TABLE EMPLOYEE (
    ssn VARCHAR(11) PRIMARY KEY,
    name VARCHAR(100),
    dept_id INT,  -- Relationship: works_for
    
    FOREIGN KEY (dept_id) REFERENCES DEPARTMENT(dept_id)
);

CREATE TABLE SECRETARY (
    ssn VARCHAR(11) PRIMARY KEY,
    typing_speed INT,
    
    FOREIGN KEY (ssn) REFERENCES EMPLOYEE(ssn)
);

-- Secretary automatically inherits the works_for relationship
SELECT 
    s.ssn,
    e.name,
    d.dept_name  -- Inherited relationship
FROM SECRETARY s
JOIN EMPLOYEE e ON s.ssn = e.ssn
JOIN DEPARTMENT d ON e.dept_id = d.dept_id;
```

---

## 7. Multiple Inheritance and Lattices

### Hierarchy vs Lattice

**Hierarchy (Single Inheritance):**
- Each subclass has **exactly ONE** direct superclass
- Forms a tree structure

**Lattice (Multiple Inheritance):**
- A subclass has **MORE THAN ONE** direct superclass
- Forms a graph structure

### Multiple Inheritance Example

**Scenario:** University has people who are both students and employees.

```
         ┌──────────┐          ┌──────────┐
         │ EMPLOYEE │          │ STUDENT  │
         │ Salary   │          │ Major    │
         └─────┬────┘          └────┬─────┘
               │                    │
               └────────┬───────────┘
                        │
                  ┌─────▼──────┐
                  │ TEACHING_  │
                  │ ASSISTANT  │  ← Shared subclass
                  │ Hours_week │
                  └────────────┘
```

**TEACHING_ASSISTANT** inherits from **both** EMPLOYEE and STUDENT.

**Inherited attributes:**
- From EMPLOYEE: Salary, Hire_date
- From STUDENT: Major, GPA
- Own attributes: Hours_week, Course_assigned

**MySQL Implementation:**

```sql
CREATE TABLE PERSON (
    person_id INT PRIMARY KEY,
    name VARCHAR(100),
    dob DATE,
    address VARCHAR(200)
);

CREATE TABLE EMPLOYEE (
    person_id INT PRIMARY KEY,
    employee_id VARCHAR(20),
    salary DECIMAL(10,2),
    hire_date DATE,
    
    FOREIGN KEY (person_id) REFERENCES PERSON(person_id)
);

CREATE TABLE STUDENT (
    person_id INT PRIMARY KEY,
    student_id VARCHAR(20),
    major VARCHAR(50),
    gpa DECIMAL(3,2),
    
    FOREIGN KEY (person_id) REFERENCES PERSON(person_id)
);

-- Shared subclass: Multiple inheritance
CREATE TABLE TEACHING_ASSISTANT (
    person_id INT PRIMARY KEY,
    hours_per_week INT,
    course_assigned VARCHAR(50),
    
    FOREIGN KEY (person_id) REFERENCES PERSON(person_id)
    -- Must also exist in both EMPLOYEE and STUDENT
);

-- Trigger to enforce multiple inheritance
DELIMITER $$

CREATE TRIGGER check_ta_inheritance
BEFORE INSERT ON TEACHING_ASSISTANT
FOR EACH ROW
BEGIN
    IF NOT EXISTS (SELECT 1 FROM EMPLOYEE WHERE person_id = NEW.person_id) THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'TA must be an employee';
    END IF;
    
    IF NOT EXISTS (SELECT 1 FROM STUDENT WHERE person_id = NEW.person_id) THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'TA must be a student';
    END IF;
END$$

DELIMITER ;

-- Sample data
INSERT INTO PERSON VALUES (1, 'John Smith', '1995-05-15', '123 Campus Dr');
INSERT INTO EMPLOYEE VALUES (1, 'E1001', 15000, '2024-08-20');
INSERT INTO STUDENT VALUES (1, 'S5001', 'Computer Science', 3.8);
INSERT INTO TEACHING_ASSISTANT VALUES (1, 20, 'CS101');

-- Query to get all inherited attributes
SELECT 
    p.name,
    p.dob,
    e.employee_id,
    e.salary,
    s.student_id,
    s.major,
    s.gpa,
    ta.hours_per_week,
    ta.course_assigned
FROM TEACHING_ASSISTANT ta
JOIN PERSON p ON ta.person_id = p.person_id
JOIN EMPLOYEE e ON ta.person_id = e.person_id
JOIN STUDENT s ON ta.person_id = s.person_id;
```

### Complex Lattice Example

```
    ┌──────────┐        ┌──────────┐
    │ STUDENT  ��        │ EMPLOYEE │
    └────┬─────┘        └────┬─────┘
         │                   │
         │  ┌────────────┐   │
         └──│GRAD_STUDENT│───┘
            │            │
            └──────┬─────┘
                   │
            ┌──────▼──────┐
            │ RESEARCH_   │
            │ ASSISTANT   │
            └─────────────┘
```

**RESEARCH_ASSISTANT** inherits from GRAD_STUDENT, which inherits from both STUDENT and EMPLOYEE.

```sql
CREATE TABLE GRAD_STUDENT (
    person_id INT PRIMARY KEY,
    thesis_topic VARCHAR(200),
    advisor_id INT,
    
    FOREIGN KEY (person_id) REFERENCES PERSON(person_id)
);

CREATE TABLE RESEARCH_ASSISTANT (
    person_id INT PRIMARY KEY,
    research_area VARCHAR(100),
    grant_number VARCHAR(50),
    
    FOREIGN KEY (person_id) REFERENCES GRAD_STUDENT(person_id)
);

-- Must exist in PERSON, EMPLOYEE, STUDENT, and GRAD_STUDENT
```

![[Pasted image 20260313011949.png]]

---

## 8. Union Types (Categories) 
### Definition

A **Category (Union Type)** is a subclass that represents a **subset of the UNION** of multiple distinct superclasses.

**Key difference from shared subclass:**
- **Shared subclass:** Entity must exist in ALL superclasses
- **Category:** Entity exists in **ONE OR MORE** (but not necessarily all) superclasses

### Notation

**Symbol:** `U` in a circle, with lines to multiple superclasses

```
┌─────────┐    ┌──────┐    ┌─────────┐
│ PERSON  │    │ BANK │    │ COMPANY │
└────┬────┘    └───┬──┘    └────┬────┘
     │             │             │
     └─────────────┴─────────────┘
                   U  ← Union symbol
                   │
             ┌─────▼──────┐
             │   OWNER    │
             └────────────┘
```

![[Pasted image 20260313012212.png]]

### Example 1: Vehicle Owner

**Scenario:** A vehicle owner can be a PERSON, a BANK (leasing), or a COMPANY (fleet vehicle).

**Interpretation:** An OWNER is either:
- A person who owns a vehicle, OR
- A bank that owns a vehicle, OR
- A company that owns a vehicle

```
┌─────────┐    ┌──────┐    ┌─────────┐
│ PERSON  │    │ BANK │    │ COMPANY │
│ SSN     │    │ Code │    │ Tax_ID  │
└────┬────┘    └───┬──┘    └────┬────┘
     │             │             │
     └─────────────┴─────────────┘
                   U
                   │
             ┌─────▼──────┐
             │   OWNER    │
             │ Owner_ID   │
             └─────┬──────┘
                   │
             ┌─────▼──────┐
             │  VEHICLE   │
             │    VIN     │
             └────────────┘
```

**MySQL Implementation:**

```sql
CREATE TABLE PERSON (
    ssn VARCHAR(11) PRIMARY KEY,
    name VARCHAR(100),
    address VARCHAR(200)
);

CREATE TABLE BANK (
    bank_code VARCHAR(20) PRIMARY KEY,
    bank_name VARCHAR(100),
    branch_address VARCHAR(200)
);

CREATE TABLE COMPANY (
    tax_id VARCHAR(20) PRIMARY KEY,
    company_name VARCHAR(100),
    industry VARCHAR(50)
);

-- Category: OWNER
CREATE TABLE OWNER (
    owner_id INT PRIMARY KEY AUTO_INCREMENT,
    owner_type VARCHAR(20) NOT NULL,  -- 'Person', 'Bank', or 'Company'
    person_ssn VARCHAR(11),
    bank_code VARCHAR(20),
    company_tax_id VARCHAR(20),
    
    -- Exactly ONE foreign key must be NOT NULL
    CHECK (
        (owner_type = 'Person' AND person_ssn IS NOT NULL AND bank_code IS NULL AND company_tax_id IS NULL) OR
        (owner_type = 'Bank' AND bank_code IS NOT NULL AND person_ssn IS NULL AND company_tax_id IS NULL) OR
        (owner_type = 'Company' AND company_tax_id IS NOT NULL AND person_ssn IS NULL AND bank_code IS NULL)
    ),
    
    FOREIGN KEY (person_ssn) REFERENCES PERSON(ssn),
    FOREIGN KEY (bank_code) REFERENCES BANK(bank_code),
    FOREIGN KEY (company_tax_id) REFERENCES COMPANY(tax_id)
);

CREATE TABLE VEHICLE (
    vin VARCHAR(17) PRIMARY KEY,
    make VARCHAR(50),
    model VARCHAR(50),
    year INT,
    owner_id INT,
    
    FOREIGN KEY (owner_id) REFERENCES OWNER(owner_id)
);

-- Sample data
INSERT INTO PERSON VALUES ('123-45-6789', 'John Doe', '123 Main St');
INSERT INTO BANK VALUES ('BNK001', 'First National Bank', '456 Bank Ave');
INSERT INTO COMPANY VALUES ('TX123456', 'ABC Corporation', 'Technology');

-- Owner who is a person
INSERT INTO OWNER (owner_type, person_ssn) 
VALUES ('Person', '123-45-6789');

-- Owner who is a bank
INSERT INTO OWNER (owner_type, bank_code) 
VALUES ('Bank', 'BNK001');

-- Owner who is a company
INSERT INTO OWNER (owner_type, company_tax_id) 
VALUES ('Company', 'TX123456');

-- Query to get owner details
SELECT 
    o.owner_id,
    o.owner_type,
    COALESCE(p.name, b.bank_name, c.company_name) AS owner_name,
    v.vin,
    v.make,
    v.model
FROM OWNER o
LEFT JOIN PERSON p ON o.person_ssn = p.ssn
LEFT JOIN BANK b ON o.bank_code = b.bank_code
LEFT JOIN COMPANY c ON o.company_tax_id = c.tax_id
JOIN VEHICLE v ON o.owner_id = v.owner_id;
```

### Example 2: Account Holder

**Scenario:** Bank account holders can be individuals or businesses.

```
┌─────────────┐          ┌──────────┐
│ INDIVIDUAL  │          │ BUSINESS │
│ SSN, Name   │          │ Tax_ID   │
└──────┬──────┘          └─────┬────┘
       │                       │
       └───────────┬───────────┘
                   U
                   │
          ┌────────▼─────────┐
          │ ACCOUNT_HOLDER   │
          │ Holder_ID        │
          └────────┬─────────┘
                   │
          ┌────────▼─────────┐
          │ BANK_ACCOUNT     │
          │ Account_No       │
          └──────────────────┘
```

```sql
CREATE TABLE INDIVIDUAL (
    ssn VARCHAR(11) PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    dob DATE
);

CREATE TABLE BUSINESS (
    tax_id VARCHAR(20) PRIMARY KEY,
    business_name VARCHAR(100),
    business_type VARCHAR(50)
);

CREATE TABLE ACCOUNT_HOLDER (
    holder_id INT PRIMARY KEY AUTO_INCREMENT,
    holder_type ENUM('Individual', 'Business') NOT NULL,
    individual_ssn VARCHAR(11),
    business_tax_id VARCHAR(20),
    
    CHECK (
        (holder_type = 'Individual' AND individual_ssn IS NOT NULL AND business_tax_id IS NULL) OR
        (holder_type = 'Business' AND business_tax_id IS NOT NULL AND individual_ssn IS NULL)
    ),
    
    FOREIGN KEY (individual_ssn) REFERENCES INDIVIDUAL(ssn),
    FOREIGN KEY (business_tax_id) REFERENCES BUSINESS(tax_id)
);

CREATE TABLE BANK_ACCOUNT (
    account_no VARCHAR(20) PRIMARY KEY,
    balance DECIMAL(12,2),
    account_type VARCHAR(20),
    holder_id INT NOT NULL,
    
    FOREIGN KEY (holder_id) REFERENCES ACCOUNT_HOLDER(holder_id)
);
```

### Example 3: Registered User (Multiple Partial Participation)

**Scenario:** A registered user can be a customer, vendor, or both.

```
┌──────────┐          ┌────────┐
│ CUSTOMER │          │ VENDOR │
└─────┬────┘          └────┬───┘
      │                    │
      └────────┬───────────┘
               U
               │
        ┌──────▼────────┐
        │ REGISTERED_   │
        │ USER          │
        └───────────────┘
```

```sql
CREATE TABLE CUSTOMER (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    loyalty_points INT
);

CREATE TABLE VENDOR (
    vendor_id INT PRIMARY KEY,
    vendor_name VARCHAR(100),
    commission_rate DECIMAL(5,2)
);

CREATE TABLE REGISTERED_USER (
    user_id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) UNIQUE,
    email VARCHAR(100) UNIQUE,
    password_hash VARCHAR(255),
    customer_id INT,
    vendor_id INT,
    
    -- Must be at least customer OR vendor (or both)
    CHECK (customer_id IS NOT NULL OR vendor_id IS NOT NULL),
    
    FOREIGN KEY (customer_id) REFERENCES CUSTOMER(customer_id),
    FOREIGN KEY (vendor_id) REFERENCES VENDOR(vendor_id)
);

-- User who is only a customer
INSERT INTO CUSTOMER VALUES (1, 'John Doe', 500);
INSERT INTO REGISTERED_USER (username, email, customer_id, vendor_id)
VALUES ('johndoe', 'john@email.com', 1, NULL);

-- User who is only a vendor
INSERT INTO VENDOR VALUES (1, 'ABC Supplies', 10.5);
INSERT INTO REGISTERED_USER (username, email, customer_id, vendor_id)
VALUES ('abcsupplies', 'abc@supplies.com', NULL, 1);

-- User who is BOTH customer and vendor
INSERT INTO CUSTOMER VALUES (2, 'Jane Smith', 1000);
INSERT INTO VENDOR VALUES (2, 'Janes Shop', 8.0);
INSERT INTO REGISTERED_USER (username, email, customer_id, vendor_id)
VALUES ('janesmith', 'jane@email.com', 2, 2);
```

### [[Ok, but can it be said that Category is just like a total participation overlapping Generalization]]

---

## 9. EER to Relational Mapping

### Strategy 1: Single Table (Table per Hierarchy)

**Create ONE table** for the superclass with all attributes from all subclasses.

**Pros:**
- Simple queries (no joins)
- Fast access

**Cons:**
- Many NULL values (wasted space)
- Can violate normalization

**When to use:** 
- Few subclasses
- Subclasses have few unique attributes
- Queries mostly access superclass attributes

```sql
-- Single table for entire hierarchy
CREATE TABLE EMPLOYEE (
    ssn VARCHAR(11) PRIMARY KEY,
    name VARCHAR(100),
    birth_date DATE,
    salary DECIMAL(10,2),
    
    -- Discriminator
    employee_type VARCHAR(20),
    
    -- Secretary attributes (NULL for non-secretaries)
    typing_speed INT,
    office_hours VARCHAR(50),
    
    -- Engineer attributes (NULL for non-engineers)
    engineering_type VARCHAR(50),
    certifications TEXT,
    
    -- Technician attributes (NULL for non-technicians)
    tgrade INT
);
```

### Strategy 2: Table per Type (Class Table Inheritance)

**Create separate tables** for superclass and each subclass, linked by foreign keys.

**Pros:**
- No wasted space
- Clear structure
- Easy to add new subclasses

**Cons:**
- Requires joins for complete information
- More complex queries

**When to use:**
- Many subclasses
- Subclasses have many unique attributes
- Need to query subclasses independently

```sql
-- Superclass table
CREATE TABLE EMPLOYEE (
    ssn VARCHAR(11) PRIMARY KEY,
    name VARCHAR(100),
    birth_date DATE,
    salary DECIMAL(10,2)
);

-- Subclass tables
CREATE TABLE SECRETARY (
    ssn VARCHAR(11) PRIMARY KEY,
    typing_speed INT,
    office_hours VARCHAR(50),
    
    FOREIGN KEY (ssn) REFERENCES EMPLOYEE(ssn)
        ON DELETE CASCADE
);

CREATE TABLE ENGINEER (
    ssn VARCHAR(11) PRIMARY KEY,
    engineering_type VARCHAR(50),
    certifications TEXT,
    
    FOREIGN KEY (ssn) REFERENCES EMPLOYEE(ssn)
        ON DELETE CASCADE
);

CREATE TABLE TECHNICIAN (
    ssn VARCHAR(11) PRIMARY KEY,
    tgrade INT,
    
    FOREIGN KEY (ssn) REFERENCES EMPLOYEE(ssn)
        ON DELETE CASCADE
);
```

### Strategy 3: Table per Concrete Class

**Create separate tables** for each subclass ONLY (no superclass table).

**Pros:**
- No joins needed
- Fast queries on individual subclasses

**Cons:**
- Duplicate attributes
- Hard to query all employees
- Difficult to maintain

**When to use:**
- Rarely need to query across all subclasses
- Subclasses are very different

```sql
-- No EMPLOYEE table

CREATE TABLE SECRETARY (
    ssn VARCHAR(11) PRIMARY KEY,
    -- Repeated superclass attributes
    name VARCHAR(100),
    birth_date DATE,
    salary DECIMAL(10,2),
    -- Subclass attributes
    typing_speed INT,
    office_hours VARCHAR(50)
);

CREATE TABLE ENGINEER (
    ssn VARCHAR(11) PRIMARY KEY,
    -- Repeated superclass attributes
    name VARCHAR(100),
    birth_date DATE,
    salary DECIMAL(10,2),
    -- Subclass attributes
    engineering_type VARCHAR(50),
    certifications TEXT
);

CREATE TABLE TECHNICIAN (
    ssn VARCHAR(11) PRIMARY KEY,
    -- Repeated superclass attributes
    name VARCHAR(100),
    birth_date DATE,
    salary DECIMAL(10,2),
    -- Subclass attributes
    tgrade INT
);

-- Query all employees requires UNION
SELECT ssn, name, 'Secretary' AS type FROM SECRETARY
UNION
SELECT ssn, name, 'Engineer' AS type FROM ENGINEER
UNION
SELECT ssn, name, 'Technician' AS type FROM TECHNICIAN;
```

### Mapping Decision Tree

```
Is there a superclass with many shared attributes?
│
├─ YES → Use Strategy 2 (Table per Type)
│        Are queries mostly on individual subclasses?
│        │
│        ├─ YES → Strategy 2 is good
│        └─ NO → Consider Strategy 1 if few subclasses
│
└─ NO → Use Strategy 3 (Table per Concrete Class)
```

---

## 10. Complete Examples 
### Example 1: University Database with EER

**Requirements:**
- University has people: students, faculty, staff
- Students can be undergraduate or graduate
- Faculty can teach courses
- Some graduate students are teaching assistants (both student and employee)

**EER Diagram:**

```
                    ┌─────────────────────────┐
                    │       PERSON            │
                    │ Person_ID, Name, DOB    │
                    └───────────┬─────────────┘
                                │
                                ○ o,∂  ← Overlapping, Partial
                               ╱│╲
                              ╱ │ ╲
             ┌──────────┐    ┌┴────────┐    ┌──────────┐
             │ STUDENT  │    │ FACULTY │    │  STAFF   │
             └─────┬────┘    └─────────┘    └──────────┘
                   │
                   ○ d,T  ← Disjoint, Total
                  ╱│╲
        ┌────────┐ ┌┴────────────┐
        │UNDERGRAD│ │  GRADUATE   │
        └─────────┘ └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │   TEACHING  │
                    │  ASSISTANT  │
                    └─────────────┘
```

**Complete MySQL Implementation:**

```sql
-- Superclass
CREATE TABLE PERSON (
    person_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    dob DATE,
    address VARCHAR(200),
    phone VARCHAR(15),
    email VARCHAR(100) UNIQUE
);

-- Subclass: Student (overlapping with others)
CREATE TABLE STUDENT (
    person_id INT PRIMARY KEY,
    student_id VARCHAR(20) UNIQUE NOT NULL,
    major VARCHAR(50),
    enrollment_date DATE,
    student_level VARCHAR(20) NOT NULL,  -- 'Undergraduate' or 'Graduate'
    
    FOREIGN KEY (person_id) REFERENCES PERSON(person_id)
        ON DELETE CASCADE,
    
    CHECK (student_level IN ('Undergraduate', 'Graduate'))
);

-- Subclass: Faculty
CREATE TABLE FACULTY (
    person_id INT PRIMARY KEY,
    faculty_id VARCHAR(20) UNIQUE NOT NULL,
    rank VARCHAR(50),  -- 'Assistant Prof', 'Associate Prof', 'Professor'
    department VARCHAR(50),
    office VARCHAR(20),
    research_area VARCHAR(100),
    
    FOREIGN KEY (person_id) REFERENCES PERSON(person_id)
        ON DELETE CASCADE
);

-- Subclass: Staff
CREATE TABLE STAFF (
    person_id INT PRIMARY KEY,
    staff_id VARCHAR(20) UNIQUE NOT NULL,
    position VARCHAR(50),
    department VARCHAR(50),
    hire_date DATE,
    
    FOREIGN KEY (person_id) REFERENCES PERSON(person_id)
        ON DELETE CASCADE
);

-- Specialization of Student: Undergraduate (Disjoint, Total)
CREATE TABLE UNDERGRADUATE (
    person_id INT PRIMARY KEY,
    class_year INT,  -- 1, 2, 3, 4
    advisor_id INT,
    gpa DECIMAL(3,2),
    
    FOREIGN KEY (person_id) REFERENCES STUDENT(person_id)
        ON DELETE CASCADE,
    FOREIGN KEY (advisor_id) REFERENCES FACULTY(person_id)
);

-- Specialization of Student: Graduate (Disjoint, Total)
CREATE TABLE GRADUATE (
    person_id INT PRIMARY KEY,
    thesis_topic VARCHAR(200),
    advisor_id INT,
    degree_type ENUM('Masters', 'PhD'),
    qualifying_exam_passed BOOLEAN,
    
    FOREIGN KEY (person_id) REFERENCES STUDENT(person_id)
        ON DELETE CASCADE,
    FOREIGN KEY (advisor_id) REFERENCES FACULTY(person_id)
);

-- Multiple Inheritance: Teaching Assistant (both Graduate Student and Employee)
CREATE TABLE TEACHING_ASSISTANT (
    person_id INT PRIMARY KEY,
    ta_id VARCHAR(20) UNIQUE,
    course_assigned VARCHAR(50),
    hours_per_week INT,
    hourly_rate DECIMAL(8,2),
    
    FOREIGN KEY (person_id) REFERENCES GRADUATE(person_id)
        ON DELETE CASCADE
);

-- Sample Data
INSERT INTO PERSON VALUES 
    (NULL, 'Alice Johnson', '2003-05-15', '123 Dorm A', '555-0001', 'alice@univ.edu'),
    (NULL, 'Bob Smith', '1985-08-22', '456 Faculty Row', '555-0002', 'bob@univ.edu'),
    (NULL, 'Carol White', '1992-11-30', '789 Admin Bldg', '555-0003', 'carol@univ.edu'),
    (NULL, 'David Lee', '2000-03-10', '321 Grad Housing', '555-0004', 'david@univ.edu');

SET @alice_id = 1;
SET @bob_id = 2;
SET @carol_id = 3;
SET @david_id = 4;

-- Alice is an undergraduate student
INSERT INTO STUDENT VALUES (@alice_id, 'S2023001', 'Computer Science', '2023-08-20', 'Undergraduate');
INSERT INTO UNDERGRADUATE VALUES (@alice_id, 2, @bob_id, 3.7);

-- Bob is faculty
INSERT INTO FACULTY VALUES (@bob_id, 'F001', 'Associate Professor', 'Computer Science', 'TB-301', 'Database Systems');

-- Carol is staff
INSERT INTO STAFF VALUES (@carol_id, 'ST001', 'Administrative Assistant', 'Registrar', '2015-03-01');

-- David is a graduate student AND teaching assistant (multiple roles)
INSERT INTO STUDENT VALUES (@david_id, 'S2023002', 'Computer Science', '2023-08-20', 'Graduate');
INSERT INTO GRADUATE VALUES (@david_id, 'Query Optimization in Distributed Databases', @bob_id, 'PhD', TRUE);
INSERT INTO TEACHING_ASSISTANT VALUES (@david_id, 'TA001', 'CS101', 20, 25.00);

-- Queries

-- 1. Get all information about a teaching assistant (multiple inheritance)
SELECT 
    p.name,
    p.email,
    s.student_id,
    s.major,
    g.thesis_topic,
    g.degree_type,
    f.name AS advisor_name,
    ta.ta_id,
    ta.course_assigned,
    ta.hours_per_week,
    (ta.hours_per_week * ta.hourly_rate * 4) AS monthly_salary
FROM TEACHING_ASSISTANT ta
JOIN GRADUATE g ON ta.person_id = g.person_id
JOIN STUDENT s ON ta.person_id = s.person_id
JOIN PERSON p ON ta.person_id = p.person_id
LEFT JOIN FACULTY f ON g.advisor_id = f.person_id
LEFT JOIN PERSON fp ON f.person_id = fp.person_id;

-- 2. Get all students (both undergrad and grad)
SELECT 
    p.name,
    s.student_id,
    s.student_level,
    s.major,
    CASE 
        WHEN u.person_id IS NOT NULL THEN CONCAT('Year ', u.class_year)
        WHEN g.person_id IS NOT NULL THEN g.degree_type
    END AS details
FROM STUDENT s
JOIN PERSON p ON s.person_id = p.person_id
LEFT JOIN UNDERGRADUATE u ON s.person_id = u.person_id
LEFT JOIN GRADUATE g ON s.person_id = g.person_id;

-- 3. Find people with multiple roles
SELECT 
    p.person_id,
    p.name,
    COUNT(DISTINCT role) AS num_roles,
    GROUP_CONCAT(DISTINCT role) AS roles
FROM (
    SELECT person_id, 'Student' AS role FROM STUDENT
    UNION ALL
    SELECT person_id, 'Faculty' AS role FROM FACULTY
    UNION ALL
    SELECT person_id, 'Staff' AS role FROM STAFF
    UNION ALL
    SELECT person_id, 'Teaching Assistant' AS role FROM TEACHING_ASSISTANT
) AS roles
JOIN PERSON p ON roles.person_id = p.person_id
GROUP BY p.person_id, p.name
HAVING num_roles > 1;
```

### Example 2: E-Commerce with Union Types

**Scenario:** Product suppliers can be manufacturers, distributors, or individual sellers.

**EER Diagram:**

```
┌────────────┐  ┌────────────┐  ┌──────────────┐
│MANUFACTURER│  │DISTRIBUTOR │  │INDIVIDUAL_   │
│            │  │            │  │SELLER        │
└──────┬─────┘  └──────┬─────┘  └──────┬───────┘
       │               │               │
       └───────────────┴───────────────┘
                       U
                       │
                ┌──────▼───────┐
                │   SUPPLIER   │
                └──────┬───────┘
                       │
                       │ supplies
                       │
                ┌──────▼───────┐
                │   PRODUCT    │
                └──────────────┘
```

**MySQL Implementation:**

```sql
CREATE TABLE MANUFACTURER (
    manufacturer_id INT PRIMARY KEY AUTO_INCREMENT,
    company_name VARCHAR(100),
    factory_location VARCHAR(100),
    production_capacity INT,
    certification VARCHAR(100)
);

CREATE TABLE DISTRIBUTOR (
    distributor_id INT PRIMARY KEY AUTO_INCREMENT,
    company_name VARCHAR(100),
    warehouse_locations TEXT,
    distribution_network VARCHAR(50)
);

CREATE TABLE INDIVIDUAL_SELLER (
    seller_id INT PRIMARY KEY AUTO_INCREMENT,
    seller_name VARCHAR(100),
    contact_phone VARCHAR(15),
    seller_rating DECIMAL(3,2)
);

-- Union type: Supplier
CREATE TABLE SUPPLIER (
    supplier_id INT PRIMARY KEY AUTO_INCREMENT,
    supplier_type ENUM('Manufacturer', 'Distributor', 'Individual') NOT NULL,
    manufacturer_id INT,
    distributor_id INT,
    seller_id INT,
    registration_date DATE,
    
    -- Enforce union constraint
    CHECK (
        (supplier_type = 'Manufacturer' AND manufacturer_id IS NOT NULL 
         AND distributor_id IS NULL AND seller_id IS NULL) OR
        (supplier_type = 'Distributor' AND distributor_id IS NOT NULL 
         AND manufacturer_id IS NULL AND seller_id IS NULL) OR
        (supplier_type = 'Individual' AND seller_id IS NOT NULL 
         AND manufacturer_id IS NULL AND distributor_id IS NULL)
    ),
    
    FOREIGN KEY (manufacturer_id) REFERENCES MANUFACTURER(manufacturer_id),
    FOREIGN KEY (distributor_id) REFERENCES DISTRIBUTOR(distributor_id),
    FOREIGN KEY (seller_id) REFERENCES INDIVIDUAL_SELLER(seller_id)
);

CREATE TABLE PRODUCT (
    product_id INT PRIMARY KEY AUTO_INCREMENT,
    product_name VARCHAR(200),
    category VARCHAR(50),
    price DECIMAL(10,2),
    supplier_id INT NOT NULL,
    stock_quantity INT,
    
    FOREIGN KEY (supplier_id) REFERENCES SUPPLIER(supplier_id)
);

-- Sample data
INSERT INTO MANUFACTURER VALUES 
    (NULL, 'TechCorp Manufacturing', 'China', 10000, 'ISO 9001');
    
INSERT INTO DISTRIBUTOR VALUES 
    (NULL, 'Global Distributors Inc', 'USA, Canada, Mexico', 'North America');
    
INSERT INTO INDIVIDUAL_SELLER VALUES 
    (NULL, 'John Doe', '555-1234', 4.5);

INSERT INTO SUPPLIER (supplier_type, manufacturer_id) 
VALUES ('Manufacturer', 1);

INSERT INTO SUPPLIER (supplier_type, distributor_id) 
VALUES ('Distributor', 1);

INSERT INTO SUPPLIER (supplier_type, seller_id) 
VALUES ('Individual', 1);

-- Query to get supplier details
SELECT 
    s.supplier_id,
    s.supplier_type,
    COALESCE(m.company_name, d.company_name, i.seller_name) AS name,
    COUNT(p.product_id) AS num_products
FROM SUPPLIER s
LEFT JOIN MANUFACTURER m ON s.manufacturer_id = m.manufacturer_id
LEFT JOIN DISTRIBUTOR d ON s.distributor_id = d.distributor_id
LEFT JOIN INDIVIDUAL_SELLER i ON s.seller_id = i.seller_id
LEFT JOIN PRODUCT p ON s.supplier_id = p.supplier_id
GROUP BY s.supplier_id, s.supplier_type, name;
```

### Example 3: Hospital Management with Complex Hierarchy

```sql
-- Superclass: Person
CREATE TABLE PERSON (
    person_id INT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    dob DATE,
    gender CHAR(1),
    address VARCHAR(200),
    phone VARCHAR(15)
);

-- Subclass: Patient
CREATE TABLE PATIENT (
    person_id INT PRIMARY KEY,
    patient_id VARCHAR(20) UNIQUE,
    blood_group VARCHAR(5),
    allergies TEXT,
    insurance_id VARCHAR(50),
    patient_type VARCHAR(20) NOT NULL,  -- 'Outpatient' or 'Inpatient'
    
    FOREIGN KEY (person_id) REFERENCES PERSON(person_id),
    CHECK (patient_type IN ('Outpatient', 'Inpatient'))
);

-- Specialization: Outpatient (Total, Disjoint)
CREATE TABLE OUTPATIENT (
    person_id INT PRIMARY KEY,
    appointment_date DATE,
    referring_doctor INT,
    
    FOREIGN KEY (person_id) REFERENCES PATIENT(person_id)
);

-- Specialization: Inpatient (Total, Disjoint)
CREATE TABLE INPATIENT (
    person_id INT PRIMARY KEY,
    admission_date DATE,
    room_number VARCHAR(10),
    expected_discharge DATE,
    
    FOREIGN KEY (person_id) REFERENCES PATIENT(person_id)
);

-- Subclass: Medical Staff
CREATE TABLE MEDICAL_STAFF (
    person_id INT PRIMARY KEY,
    staff_id VARCHAR(20) UNIQUE,
    hire_date DATE,
    department VARCHAR(50),
    shift VARCHAR(20),
    
    FOREIGN KEY (person_id) REFERENCES PERSON(person_id)
);

-- Specialization: Doctor
CREATE TABLE DOCTOR (
    person_id INT PRIMARY KEY,
    specialization VARCHAR(50),
    license_number VARCHAR(50) UNIQUE,
    years_experience INT,
    
    FOREIGN KEY (person_id) REFERENCES MEDICAL_STAFF(person_id)
);

-- Specialization: Nurse
CREATE TABLE NURSE (
    person_id INT PRIMARY KEY,
    nurse_level VARCHAR(20),  -- 'RN', 'LPN', 'Nurse Practitioner'
    certification VARCHAR(100),
    
    FOREIGN KEY (person_id) REFERENCES MEDICAL_STAFF(person_id)
);

-- Relationship: Treatment
CREATE TABLE TREATMENT (
    treatment_id INT PRIMARY KEY AUTO_INCREMENT,
    patient_id INT,
    doctor_id INT,
    treatment_date DATE,
    diagnosis VARCHAR(500),
    prescription TEXT,
    
    FOREIGN KEY (patient_id) REFERENCES PATIENT(person_id),
    FOREIGN KEY (doctor_id) REFERENCES DOCTOR(person_id)
);
```

---

## Summary Tables

### EER Constraint Summary

| Constraint Type | Question | Values | Symbol | MySQL Enforcement |
|-----------------|----------|--------|--------|-------------------|
| **Disjointness** | Multiple subclasses? | Disjoint `d` / Overlapping `o` | `d` or `o` in circle | CHECK constraint, Triggers |
| **Completeness** | Must be in subclass? | Total (double line) / Partial (single line) | `═══` or `───` | NOT NULL, Triggers |
| **Membership** | How determined? | Attribute-defined / User-defined | Attribute label | Automatic or Manual |

### Mapping Strategies Comparison

| Strategy | Tables | Space | Query Speed | Use When |
|----------|--------|-------|-------------|----------|
| **Single Table** | 1 | Wasted (NULLs) | Fast | Few subclasses, simple attributes |
| **Table per Type** | N+1 | Efficient | Slow (joins) | Many subclasses, complex attributes |
| **Table per Concrete** | N | Duplicated | Fast | Rarely query all together |

### EER vs ER Comparison

| Feature | ER Model | EER Model |
|---------|----------|-----------|
| **Subclasses** | ✗ Not supported | ✓ Supported |
| **Inheritance** | ✗ Not supported | ✓ Supported |
| **Specialization** | ✗ Not supported | ✓ Supported |
| **Generalization** | ✗ Not supported | ✓ Supported |
| **Union Types** | ✗ Not supported | ✓ Supported |
| **Complexity** | Simple | Complex |
| **Use Case** | Simple databases | Complex, hierarchical databases |

---

## Best Practices

### 1. Choose Appropriate Constraints

✅ **Do:**
- Use **Disjoint** when categories are mutually exclusive
- Use **Overlapping** when entities can have multiple roles
- Use **Total** when every entity must be classified
- Use **Partial** when classification is optional

❌ **Don't:**
- Overuse overlapping (can cause complexity)
- Force total participation when not needed

### 2. Select Right Mapping Strategy

✅ **Do:**
- Use **Table per Type** for most scenarios (balance of flexibility and efficiency)
- Use **Single Table** for simple hierarchies with few attributes
- Document your choice

❌ **Don't:**
- Use **Table per Concrete Class** unless absolutely necessary
- Mix strategies in the same hierarchy

### 3. Enforce Constraints

✅ **Do:**
- Use CHECK constraints for disjointness
- Use triggers for complex constraints
- Use NOT NULL for total participation

❌ **Don't:**
- Rely solely on application logic
- Forget to test constraint enforcement

### 4. Design for Querying

✅ **Do:**
- Create views for common queries
- Index foreign keys
- Consider query patterns when choosing mapping strategy

❌ **Don't:**
- Over-normalize at the expense of performance
- Create unnecessary joins

---

## Common Mistakes to Avoid

❌ **Mistake 1:** Confusing shared subclass with union type
```
Shared subclass: Must exist in ALL superclasses
Union type: Exists in ONE OR MORE superclasses
```

❌ **Mistake 2:** Not enforcing disjointness in database
```sql
-- BAD: No constraint
CREATE TABLE SECRETARY (ssn VARCHAR(11) PRIMARY KEY);
CREATE TABLE ENGINEER (ssn VARCHAR(11) PRIMARY KEY);
-- Nothing prevents same SSN in both!

-- GOOD: Use trigger to enforce
```

❌ **Mistake 3:** Using wrong mapping strategy
```
Don't use single table if subclasses have 20+ unique attributes each
```

❌ **Mistake 4:** Forgetting inheritance in queries
```sql
-- BAD: Only query subclass
SELECT * FROM SECRETARY WHERE ssn = '123-45-6789';
-- Missing inherited EMPLOYEE attributes!

-- GOOD: Join with superclass
SELECT * FROM SECRETARY s 
JOIN EMPLOYEE e ON s.ssn = e.ssn 
WHERE s.ssn = '123-45-6789';
```

---
# [[Relational Algebra in DBMS]]

---

# Views, Assertions, and Bitmap Indexing in DBMS

## 1. Views

### What is a View?

A **view** is a **virtual table** based on the result of a SQL query. It doesn't store data physically; instead, it stores the query definition.

**Key Characteristics:**
- **Virtual relation** - exists logically, not physically
- **Derived data** - computed from base tables when queried
- **Dynamic** - automatically reflects changes in underlying tables
- **Security layer** - hides sensitive data
- **Simplification** - provides simplified interface to complex queries

### Why Use Views?

| Purpose | Example |
|---------|---------|
| **Security** | Hide salary information from certain users |
| **Simplification** | Present complex joins as simple tables |
| **Data Independence** | Change underlying schema without affecting applications |
| **Customization** | Different users see different data representations |
| **Reusability** | Encapsulate frequently used queries |

Follow Along [[Views in MySQL]] in [[Introduction to MySQL]].

---

## 2. Assertions

### What is an Assertion?

An **assertion** is a **database constraint** that expresses a condition the database must **always** satisfy.

**Key Characteristics:**
- **Predicate** - a boolean expression
- **Global constraint** - can span multiple tables
- **Always enforced** - checked on every relevant update
- **Declarative** - you specify WHAT, not HOW

### Syntax

**Standard SQL:**
```sql
CREATE ASSERTION assertion_name
CHECK (predicate);

DROP ASSERTION assertion_name;
```

**IMPORTANT:** MySQL **does NOT support assertions** natively. However, we can achieve similar functionality using:
1. **CHECK constraints** (MySQL 8.0.16+)
2. **Triggers**
3. **Stored procedures**

---

### Why Use Assertions?

| Purpose | Example |
|---------|---------|
| **Business Rules** | Total branch loans < total branch deposits |
| **Data Integrity** | Every loan must have a qualified borrower |
| **Complex Constraints** | Manager salary > all subordinates |
| **Cross-Table Rules** | Inventory sum = sales sum + stock |

---

### Assertion Logic Pattern: NOT EXISTS

**Common Pattern:** "There should NOT exist any violation"

```sql
CREATE ASSERTION assertion_name
CHECK (NOT EXISTS (
    SELECT *
    FROM tables
    WHERE violation_condition
));
```

**Translation:** "Assert that no rows exist where the violation occurs"

---

### Example 1: Branch Loan Limit

**Business Rule:** The sum of all loan amounts at a branch must be less than the sum of all account balances at that branch.

**Logical Statement:**
```
For every branch:
  SUM(loan.amount) < SUM(account.balance)
```

**Assertion (Standard SQL):**
```sql
CREATE ASSERTION BRANCH_LOAN_LIMIT
CHECK (NOT EXISTS (
    SELECT branch_name
    FROM (
        SELECT branch_name, SUM(amount) AS total_loans
        FROM LOAN
        GROUP BY branch_name
    ) AS loan_totals
    JOIN (
        SELECT branch_name, SUM(balance) AS total_deposits
        FROM ACCOUNT
        GROUP BY branch_name
    ) AS deposit_totals USING (branch_name)
    WHERE total_loans >= total_deposits
));
```

**MySQL Implementation (Using Trigger):**

```mysql
DELIMITER $$

-- Trigger for INSERT on LOAN
CREATE TRIGGER check_branch_loan_limit_insert
BEFORE INSERT ON LOAN
FOR EACH ROW
BEGIN
    DECLARE total_loans DECIMAL(12,2);
    DECLARE total_deposits DECIMAL(12,2);
    
    -- Calculate total loans for the branch (including new loan)
    SELECT COALESCE(SUM(amount), 0) + NEW.amount INTO total_loans
    FROM LOAN
    WHERE branch_name = NEW.branch_name;
    
    -- Calculate total deposits for the branch
    SELECT COALESCE(SUM(balance), 0) INTO total_deposits
    FROM ACCOUNT
    WHERE branch_name = NEW.branch_name;
    
    -- Check assertion
    IF total_loans >= total_deposits THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Branch loan limit exceeded: total loans must be less than total deposits';
    END IF;
END$$

-- Trigger for UPDATE on LOAN
CREATE TRIGGER check_branch_loan_limit_update
BEFORE UPDATE ON LOAN
FOR EACH ROW
BEGIN
    DECLARE total_loans DECIMAL(12,2);
    DECLARE total_deposits DECIMAL(12,2);
    
    SELECT COALESCE(SUM(amount), 0) - OLD.amount + NEW.amount INTO total_loans
    FROM LOAN
    WHERE branch_name = NEW.branch_name;
    
    SELECT COALESCE(SUM(balance), 0) INTO total_deposits
    FROM ACCOUNT
    WHERE branch_name = NEW.branch_name;
    
    IF total_loans >= total_deposits THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Branch loan limit exceeded';
    END IF;
END$$

-- Trigger for UPDATE on ACCOUNT (deposits decrease)
CREATE TRIGGER check_branch_loan_limit_account_update
BEFORE UPDATE ON ACCOUNT
FOR EACH ROW
BEGIN
    DECLARE total_loans DECIMAL(12,2);
    DECLARE total_deposits DECIMAL(12,2);
    
    SELECT COALESCE(SUM(amount), 0) INTO total_loans
    FROM LOAN
    WHERE branch_name = NEW.branch_name;
    
    SELECT COALESCE(SUM(balance), 0) - OLD.balance + NEW.balance INTO total_deposits
    FROM ACCOUNT
    WHERE branch_name = NEW.branch_name;
    
    IF total_loans >= total_deposits THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Cannot reduce deposits: would violate branch loan limit';
    END IF;
END$$

DELIMITER ;
```

**Test the Assertion:**

```sql
-- Current state
SELECT 
    branch_name,
    (SELECT SUM(amount) FROM LOAN WHERE branch_name = 'Downtown') AS total_loans,
    (SELECT SUM(balance) FROM ACCOUNT WHERE branch_name = 'Downtown') AS total_deposits;
```

**Result:**
```
┌─────────────┬─────────────┬────────────────┐
│ branch_name │ total_loans │ total_deposits │
├─────────────┼─────────────┼────────────────┤
│ Downtown    │ 125000      │ 6200           │
└─────────────┴─────────────┴────────────────┘
```

**Try to violate:**
```sql
-- This should FAIL (125000 >= 6200 already!)
INSERT INTO LOAN VALUES ('L-104', 'Downtown', 10000, 'Personal');
```

**Error:** Branch loan limit exceeded

---

### Example 2: Minimum Borrower Balance

**Business Rule:** Every loan must have at least one borrower with an account balance of at least $1000.

**Logical Statement:**
```
For every loan:
  EXISTS at least one borrower with balance >= 1000
```

**Assertion (Standard SQL):**
```sql
CREATE ASSERTION MIN_BORROWER_BALANCE
CHECK (NOT EXISTS (
    SELECT loan_number
    FROM LOAN L
    WHERE NOT EXISTS (
        SELECT *
        FROM BORROWER B
        JOIN DEPOSITOR D ON B.customer_id = D.customer_id
        JOIN ACCOUNT A ON D.account_number = A.account_number
        WHERE B.loan_number = L.loan_number
          AND A.balance >= 1000
    )
));
```

**Translation:** "There should NOT exist any loan for which there does NOT exist a borrower with balance >= 1000"

**MySQL Implementation (Using Trigger):**

```sql
DELIMITER $$

-- Trigger for INSERT on LOAN
CREATE TRIGGER check_min_borrower_balance_loan
AFTER INSERT ON LOAN
FOR EACH ROW
BEGIN
    DECLARE qualified_borrowers INT;
    
    -- Count borrowers with sufficient balance
    SELECT COUNT(*) INTO qualified_borrowers
    FROM BORROWER B
    JOIN DEPOSITOR D ON B.customer_id = D.customer_id
    JOIN ACCOUNT A ON D.account_number = A.account_number
    WHERE B.loan_number = NEW.loan_number
      AND A.balance >= 1000;
    
    IF qualified_borrowers = 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Loan must have at least one borrower with account balance >= 1000';
    END IF;
END$$

-- Trigger for INSERT on BORROWER
CREATE TRIGGER check_min_borrower_balance_borrower
AFTER INSERT ON BORROWER
FOR EACH ROW
BEGIN
    DECLARE qualified_borrowers INT;
    
    SELECT COUNT(*) INTO qualified_borrowers
    FROM BORROWER B
    JOIN DEPOSITOR D ON B.customer_id = D.customer_id
    JOIN ACCOUNT A ON D.account_number = A.account_number
    WHERE B.loan_number = NEW.loan_number
      AND A.balance >= 1000;
    
    IF qualified_borrowers = 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Loan must have at least one borrower with account balance >= 1000';
    END IF;
END$$

-- Trigger for UPDATE/DELETE on ACCOUNT (balance changes)
CREATE TRIGGER check_min_borrower_balance_account
BEFORE UPDATE ON ACCOUNT
FOR EACH ROW
BEGIN
    -- Check if this account is associated with any loan borrowers
    IF EXISTS (
        SELECT 1
        FROM DEPOSITOR D
        JOIN BORROWER B ON D.customer_id = B.customer_id
        WHERE D.account_number = NEW.account_number
          AND OLD.balance >= 1000
          AND NEW.balance < 1000
    ) THEN
        -- Check if removing this qualified borrower would violate the constraint
        DECLARE affected_loans CURSOR FOR
            SELECT DISTINCT B.loan_number
            FROM DEPOSITOR D
            JOIN BORROWER B ON D.customer_id = B.customer_id
            WHERE D.account_number = NEW.account_number;
        
        -- Would need to check each affected loan
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Cannot reduce balance below 1000: would violate loan borrower requirement';
    END IF;
END$$

DELIMITER ;
```

**Test:**

```sql
-- Add a loan
INSERT INTO LOAN VALUES ('L-105', 'Downtown', 20000, 'Personal');

-- Try to add borrower with insufficient balance
INSERT INTO BORROWER VALUES (202, 'L-105');  -- Sarah has balance 8000 (>= 1000) ✓

-- But if we try with a customer with < 1000 balance and no other borrowers
-- It would fail ❌
```

---

### Example 3: Manager Salary Constraint

**Business Rule:** An employee's salary cannot exceed their manager's salary.

**Logical Statement:**
```
For every employee with a manager:
  employee.salary <= manager.salary
```

**Assertion (Standard SQL):**
```sql
CREATE ASSERTION MANAGER_SALARY_CONSTRAINT
CHECK (NOT EXISTS (
    SELECT *
    FROM EMPLOYEE E
    JOIN EMPLOYEE M ON E.manager_id = M.emp_id
    WHERE E.salary > M.salary
));
```

**MySQL Implementation (Using Trigger):**

```sql
DELIMITER $$

-- Trigger for INSERT on EMPLOYEE
CREATE TRIGGER check_manager_salary_insert
BEFORE INSERT ON EMPLOYEE
FOR EACH ROW
BEGIN
    DECLARE manager_salary DECIMAL(10,2);
    
    IF NEW.manager_id IS NOT NULL THEN
        SELECT salary INTO manager_salary
        FROM EMPLOYEE
        WHERE emp_id = NEW.manager_id;
        
        IF NEW.salary > manager_salary THEN
            SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'Employee salary cannot exceed manager salary';
        END IF;
    END IF;
END$$

-- Trigger for UPDATE on EMPLOYEE (salary increase)
CREATE TRIGGER check_manager_salary_update
BEFORE UPDATE ON EMPLOYEE
FOR EACH ROW
BEGIN
    DECLARE manager_salary DECIMAL(10,2);
    
    -- Check if employee's new salary exceeds their manager's
    IF NEW.manager_id IS NOT NULL THEN
        SELECT salary INTO manager_salary
        FROM EMPLOYEE
        WHERE emp_id = NEW.manager_id;
        
        IF NEW.salary > manager_salary THEN
            SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'Employee salary cannot exceed manager salary';
        END IF;
    END IF;
    
    -- Check if this employee is a manager and new salary < subordinate salary
    IF EXISTS (
        SELECT 1
        FROM EMPLOYEE
        WHERE manager_id = NEW.emp_id
          AND salary > NEW.salary
    ) THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Manager salary cannot be less than subordinate salary';
    END IF;
END$$

DELIMITER ;
```

**Test:**

```sql
-- Current: John (manager, 75000), Jane (subordinate, 65000)

-- Try to give Jane higher salary than John (❌ Should FAIL)
UPDATE EMPLOYEE SET salary = 80000 WHERE emp_id = 102;

-- Error: Employee salary cannot exceed manager salary

-- Try to reduce John's salary below Jane's (❌ Should FAIL)
UPDATE EMPLOYEE SET salary = 60000 WHERE emp_id = 101;

-- Error: Manager salary cannot be less than subordinate salary
```

---

### Example 4: Referential Integrity (Alternative to FK)

**Business Rule:** Every borrower must reference an existing customer.

**Assertion (Standard SQL):**
```sql
CREATE ASSERTION BORROWER_CUSTOMER_INTEGRITY
CHECK (NOT EXISTS (
    SELECT *
    FROM BORROWER B
    WHERE B.customer_id NOT IN (SELECT customer_id FROM CUSTOMER)
));
```

**Note:** This is already handled by FOREIGN KEY constraints, but shows assertion capability.

---

### Example 5: Account Balance Non-Negative

**Business Rule:** No account can have negative balance.

**CHECK Constraint (MySQL 8.0.16+):**
```sql
ALTER TABLE ACCOUNT
ADD CONSTRAINT chk_balance_positive
CHECK (balance >= 0);
```

**Test:**
```sql
-- This should FAIL
UPDATE ACCOUNT SET balance = -100 WHERE account_number = 'A-101';

-- Error: Check constraint violated
```

---

### Assertion Performance Considerations

**Problem:** Assertions are checked on **every relevant update**

**Example:** Branch loan limit assertion requires:
- Query all loans for branch
- Query all accounts for branch
- Compute sums
- Compare

**Performance Impact:**
- ⚠️ Slow updates if many rows affected
- ⚠️ Complex predicates = expensive checks
- ⚠️ Can cause bottlenecks in high-transaction systems

**Best Practices:**
1. ✓ Use assertions for **critical business rules** only
2. ✓ Prefer **simpler constraints** (CHECK, FK) when possible
3. ✓ Consider **periodic validation** instead of real-time
4. ✓ Optimize assertion predicates with indexes
5. ✓ Document performance impact

---

### Alternative: Periodic Validation

Instead of enforcing on every transaction:

```sql
-- Validation stored procedure (run periodically)
DELIMITER $$

CREATE PROCEDURE VALIDATE_BRANCH_LOAN_LIMITS()
BEGIN
    SELECT 
        branch_name,
        total_loans,
        total_deposits,
        'VIOLATION' AS status
    FROM (
        SELECT 
            L.branch_name,
            COALESCE(SUM(L.amount), 0) AS total_loans,
            COALESCE(SUM(A.balance), 0) AS total_deposits
        FROM LOAN L
        LEFT JOIN ACCOUNT A ON L.branch_name = A.branch_name
        GROUP BY L.branch_name
    ) AS branch_totals
    WHERE total_loans >= total_deposits;
END$$

DELIMITER ;

-- Run validation
CALL VALIDATE_BRANCH_LOAN_LIMITS();
```

---

## 3. Bitmap Indexing {#bitmap-indexing}

### What is Bitmap Indexing?

A **bitmap index** is a specialized indexing technique using **bit arrays (bitmaps)** to represent data.

**Key Characteristics:**
- **Compact** - uses bits (0 or 1) instead of traditional pointers
- **Efficient** - excellent for low-cardinality columns (few distinct values)
- **Fast queries** - uses bitwise operations (AND, OR, NOT)
- **Space-saving** - especially for sparse data

---

### When to Use Bitmap Indexes?

| Use Case | Example |
|----------|---------|
| **Low cardinality** | Gender (M/F), Status (Active/Inactive) |
| **Read-heavy workloads** | Data warehouses, reporting |
| **Multiple column queries** | WHERE gender='F' AND status='Active' |
| **Large tables** | Millions of rows with few distinct values |

**❌ Not suitable for:**
- High-cardinality columns (email, SSN, etc.)
- Frequent updates (expensive to maintain)
- OLTP systems (better suited for OLAP)

---

### Bitmap Index Structure

**Example Table:**
```sql
CREATE TABLE EMPLOYEE_STATUS (
    emp_id INT PRIMARY KEY,
    name VARCHAR(100),
    gender CHAR(1),
    status VARCHAR(20),
    department VARCHAR(50)
);

INSERT INTO EMPLOYEE_STATUS VALUES
    (1, 'Alice', 'F', 'Active', 'Sales'),
    (2, 'Bob', 'M', 'Active', 'IT'),
    (3, 'Carol', 'F', 'Inactive', 'HR'),
    (4, 'David', 'M', 'Active', 'Sales'),
    (5, 'Emma', 'F', 'Active', 'IT'),
    (6, 'Frank', 'M', 'Inactive', 'Sales'),
    (7, 'Grace', 'F', 'Active', 'HR'),
    (8, 'Henry', 'M', 'Active', 'IT');
```

---

### Bitmap Index for Gender Column

**Distinct values:** M, F

**Bitmap Representation:**

| ROWID | Name  | Gender | Bitmap 'M' | Bitmap 'F' |
|-------|-------|--------|------------|------------|
| 1     | Alice | F      | 0          | 1          |
| 2     | Bob   | M      | 1          | 0          |
| 3     | Carol | F      | 0          | 1          |
| 4     | David | M      | 1          | 0          |
| 5     | Emma  | F      | 0          | 1          |
| 6     | Frank | M      | 1          | 0          |
| 7     | Grace | F      | 0          | 1          |
| 8     | Henry | M      | 1          | 0          |

**Bitmap for 'M':** `01010101` (binary)
**Bitmap for 'F':** `10101010` (binary)

---

### Bitmap Index Storage

**Typical structure:**

| Column Value | Starting ROWID | Ending ROWID | Bitmap     |
|--------------|----------------|--------------|------------|
| M            | 1              | 8            | 01010101   |
| F            | 1              | 8            | 10101010   |

---

### Querying with Bitmap Indexes

#### Query 1: Simple Equality

**Query:** Find all female employees

**SQL:**
```sql
SELECT * FROM EMPLOYEE_STATUS WHERE gender = 'F';
```

**Bitmap Operation:**
```
Bitmap for 'F': 10101010
ROWIDs with bit=1: 1, 3, 5, 7
```

**Result:** Retrieve rows 1, 3, 5, 7

---

#### Query 2: Multiple Conditions with AND

**Query:** Find female employees who are active

**SQL:**
```sql
SELECT * FROM EMPLOYEE_STATUS 
WHERE gender = 'F' AND status = 'Active';
```

**Bitmap Representation:**

**Gender Bitmaps:**
- 'M': `01010101`
- 'F': `10101010`

**Status Bitmaps:**
- 'Active': `11011011`
- 'Inactive': `00100100`

**Bitwise AND Operation:**
```
Bitmap for 'F':      10101010
Bitmap for 'Active': 11011011
                     --------
Result (AND):        10001010
```

**ROWIDs with bit=1:** 1, 5, 7

**Result:** Alice, Emma, Grace

---

#### Query 3: OR Operation

**Query:** Find employees who are female OR in Sales

**SQL:**
```sql
SELECT * FROM EMPLOYEE_STATUS 
WHERE gender = 'F' OR department = 'Sales';
```

**Bitmaps:**

**Gender 'F':** `10101010`
**Department 'Sales':** `10010100`

**Bitwise OR:**
```
Gender 'F':    10101010
Dept 'Sales':  10010100
               --------
Result (OR):   10111110
```

**ROWIDs:** 1, 2, 4, 5, 6, 7

---

#### Query 4: NOT Operation

**Query:** Find employees who are NOT in IT

**SQL:**
```sql
SELECT * FROM EMPLOYEE_STATUS 
WHERE department != 'IT';
```

**Bitmaps:**

**Department 'IT':** `01001001`

**Bitwise NOT:**
```
Dept 'IT':   01001001
             --------
NOT IT:      10110110
```

**ROWIDs:** 1, 3, 4, 6, 7

---

#### Query 5: Complex Combination

**Query:** Find active male employees NOT in HR

**SQL:**
```sql
SELECT * FROM EMPLOYEE_STATUS 
WHERE gender = 'M' AND status = 'Active' AND department != 'HR';
```

**Bitmaps:**
- Gender 'M': `01010101`
- Status 'Active': `11011011`
- Department 'HR': `00100010`

**Operations:**
```
Step 1: M AND Active
01010101
11011011
--------
01010001

Step 2: Result AND (NOT HR)
NOT HR:  11011101
Result:  01010001
         --------
Final:   01010001
```

**ROWIDs:** 2, 8 (Bob, Henry)

---

### Bitmap Index Creation (MySQL Simulation)

**Note:** MySQL doesn't support native bitmap indexes, but we can simulate the concept.

```sql
-- Create bitmap table for gender
CREATE TABLE BITMAP_GENDER (
    gender_value CHAR(1),
    bitmap_data BINARY(1000)  -- Supports up to 8000 rows (1000 bytes * 8 bits)
);

-- Manually populate bitmap (conceptual example)
-- In practice, this would be done by specialized database systems
```

**Oracle Example (has native bitmap indexes):**
```sql
-- Oracle syntax
CREATE BITMAP INDEX idx_gender ON EMPLOYEE_STATUS(gender);
CREATE BITMAP INDEX idx_status ON EMPLOYEE_STATUS(status);
CREATE BITMAP INDEX idx_department ON EMPLOYEE_STATUS(department);
```

---

### Advantages of Bitmap Indexes

| Advantage | Explanation |
|-----------|-------------|
| **Space efficient** | Bits use minimal storage vs traditional B-tree |
| **Fast queries** | Bitwise operations are extremely fast |
| **Multiple conditions** | AND/OR/NOT operations trivial with bitmaps |
| **Good for DW** | Perfect for read-heavy analytical queries |
| **Compression** | Bitmaps compress well (run-length encoding) |

---

### Disadvantages of Bitmap Indexes

| Disadvantage | Explanation |
|--------------|-------------|
| **Update overhead** | Updating bitmap requires recomputing bits |
| **Lock contention** | Updates lock entire bitmap (not row-level) |
| **Not for OLTP** | Frequent updates make them impractical |
| **High cardinality** | Inefficient for unique/near-unique values |
| **Space waste** | Large bitmaps if cardinality is high |

---

### Bitmap vs B-tree Index

| Aspect | B-tree Index | Bitmap Index |
|--------|--------------|--------------|
| **Cardinality** | High (unique values) | Low (few values) |
| **Workload** | OLTP (read/write) | OLAP (read-heavy) |
| **Multiple conditions** | Slower (merge indexes) | Fast (bitwise ops) |
| **Updates** | Fast (row-level) | Slow (bitmap rebuild) |
| **Space** | More space | Less space |
| **Example** | Email, SSN | Gender, Status |

---

### Bitmap Compression (Run-Length Encoding)

**Sparse bitmap:**
```
00000000 00000000 11111111 00000000
```

**Compressed (RLE):**
```
[0: 16 times] [1: 8 times] [0: 8 times]
```

**Benefit:** Saves significant space for sparse data

---

### Practical Example: Data Warehouse Query

**Scenario:** Sales data warehouse

```sql
CREATE TABLE SALES (
    sale_id INT,
    product_category VARCHAR(50),
    customer_gender CHAR(1),
    sale_region VARCHAR(50),
    sale_status VARCHAR(20),
    amount DECIMAL(10,2)
);
```

**Query:** Find total sales for active female customers in Electronics in West region

**SQL:**
```sql
SELECT SUM(amount)
FROM SALES
WHERE customer_gender = 'F'
  AND product_category = 'Electronics'
  AND sale_region = 'West'
  AND sale_status = 'Active';
```

**With Bitmap Indexes:**
1. Get bitmap for Gender='F'
2. Get bitmap for Category='Electronics'
3. Get bitmap for Region='West'
4. Get bitmap for Status='Active'
5. AND all bitmaps
6. Retrieve matching rows
7. Sum amounts

**Performance:** Extremely fast (bitwise operations on compressed bitmaps)

---

## 4. Materialized Views {#materialized-views}

### What is a Materialized View?

A **materialized view** is a view whose results are **physically stored** (like a table).

**Difference from Regular View:**

| Regular View | Materialized View |
|--------------|-------------------|
| Virtual (no storage) | Physical (stored) |
| Computed on query | Pre-computed |
| Always current | Needs refresh |
| Slower (recomputes) | Faster (precomputed) |

---

### MySQL Simulation of Materialized View

MySQL doesn't have native materialized views, but we can simulate:

```sql
-- Create table to store materialized view
CREATE TABLE DEPT_SALARY_STATS_MV (
    dept_name VARCHAR(50),
    num_employees INT,
    avg_salary DECIMAL(10,2),
    total_salary DECIMAL(12,2),
    last_refreshed TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Populate materialized view
INSERT INTO DEPT_SALARY_STATS_MV (dept_name, num_employees, avg_salary, total_salary)
SELECT 
    d.dept_name,
    COUNT(e.emp_id),
    AVG(e.salary),
    SUM(e.salary)
FROM DEPARTMENT d
LEFT JOIN EMPLOYEE e ON d.dept_id = e.dept_id
GROUP BY d.dept_id, d.dept_name;

-- Refresh materialized view (manual)
TRUNCATE TABLE DEPT_SALARY_STATS_MV;
INSERT INTO DEPT_SALARY_STATS_MV (dept_name, num_employees, avg_salary, total_salary)
SELECT 
    d.dept_name,
    COUNT(e.emp_id),
    AVG(e.salary),
    SUM(e.salary)
FROM DEPARTMENT d
LEFT JOIN EMPLOYEE e ON d.dept_id = e.dept_id
GROUP BY d.dept_id, d.dept_name;
```

---

### Automatic Refresh with Event Scheduler

```sql
-- Enable event scheduler
SET GLOBAL event_scheduler = ON;

-- Create refresh event
DELIMITER $$

CREATE EVENT refresh_dept_salary_stats
ON SCHEDULE EVERY 1 HOUR
DO
BEGIN
    TRUNCATE TABLE DEPT_SALARY_STATS_MV;
    INSERT INTO DEPT_SALARY_STATS_MV (dept_name, num_employees, avg_salary, total_salary)
    SELECT 
        d.dept_name,
        COUNT(e.emp_id),
        AVG(e.salary),
        SUM(e.salary)
    FROM DEPARTMENT d
    LEFT JOIN EMPLOYEE e ON d.dept_id = e.dept_id
    GROUP BY d.dept_id, d.dept_name;
END$$

DELIMITER ;
```

---

## 5. Complete Examples {#examples}

### Example: Complete Database with Views and Assertions

```sql
-- ========================================
-- Complete University Database Example
-- ========================================

-- Base Tables
CREATE TABLE STUDENT (
    student_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    gpa DECIMAL(3,2),
    credits_completed INT
);

CREATE TABLE COURSE (
    course_id VARCHAR(10) PRIMARY KEY,
    course_name VARCHAR(100),
    credits INT,
    max_enrollment INT
);

CREATE TABLE ENROLLMENT (
    student_id INT,
    course_id VARCHAR(10),
    semester VARCHAR(20),
    grade CHAR(1),
    PRIMARY KEY (student_id, course_id, semester),
    FOREIGN KEY (student_id) REFERENCES STUDENT(student_id),
    FOREIGN KEY (course_id) REFERENCES COURSE(course_id)
);

-- Sample Data
INSERT INTO STUDENT VALUES
    (1001, 'Alice Johnson', 'alice@univ.edu', 3.8, 90),
    (1002, 'Bob Smith', 'bob@univ.edu', 3.5, 75),
    (1003, 'Carol White', 'carol@univ.edu', 3.9, 105);

INSERT INTO COURSE VALUES
    ('CS101', 'Intro to CS', 3, 50),
    ('CS201', 'Data Structures', 4, 40),
    ('MATH101', 'Calculus', 4, 60);

INSERT INTO ENROLLMENT VALUES
    (1001, 'CS101', 'Fall 2025', 'A'),
    (1001, 'MATH101', 'Fall 2025', 'B'),
    (1002, 'CS101', 'Fall 2025', 'A'),
    (1003, 'CS201', 'Fall 2025', 'A');

-- ========================================
-- VIEWS
-- ========================================

-- View 1: Honor Students (GPA >= 3.5)
CREATE VIEW HONOR_STUDENTS AS
SELECT student_id, name, email, gpa
FROM STUDENT
WHERE gpa >= 3.5;

-- View 2: Course Enrollment Summary
CREATE VIEW COURSE_ENROLLMENT_SUMMARY AS
SELECT 
    c.course_id,
    c.course_name,
    c.max_enrollment,
    COUNT(e.student_id) AS current_enrollment,
    c.max_enrollment - COUNT(e.student_id) AS available_seats
FROM COURSE c
LEFT JOIN ENROLLMENT e ON c.course_id = e.course_id
GROUP BY c.course_id, c.course_name, c.max_enrollment;

-- View 3: Student Transcript
CREATE VIEW STUDENT_TRANSCRIPT AS
SELECT 
    s.student_id,
    s.name,
    s.email,
    e.course_id,
    c.course_name,
    e.semester,
    e.grade,
    c.credits
FROM STUDENT s
JOIN ENROLLMENT e ON s.student_id = e.student_id
JOIN COURSE c ON e.course_id = c.course_id;

-- ========================================
-- ASSERTIONS (via Triggers)
-- ========================================

-- Assertion 1: Course cannot exceed max enrollment
DELIMITER $$

CREATE TRIGGER check_max_enrollment
BEFORE INSERT ON ENROLLMENT
FOR EACH ROW
BEGIN
    DECLARE current_count INT;
    DECLARE max_count INT;
    
    SELECT COUNT(*) INTO current_count
    FROM ENROLLMENT
    WHERE course_id = NEW.course_id;
    
    SELECT max_enrollment INTO max_count
    FROM COURSE
    WHERE course_id = NEW.course_id;
    
    IF current_count >= max_count THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Course enrollment limit reached';
    END IF;
END$$

DELIMITER ;

-- Assertion 2: GPA must be between 0.0 and 4.0
ALTER TABLE STUDENT
ADD CONSTRAINT chk_gpa_range
CHECK (gpa >= 0.0 AND gpa <= 4.0);

-- Query Examples
SELECT * FROM HONOR_STUDENTS;
SELECT * FROM COURSE_ENROLLMENT_SUMMARY;
SELECT * FROM STUDENT_TRANSCRIPT WHERE student_id = 1001;
```

---

## Summary Comparison Table

| Feature | Views | Assertions | Bitmap Indexes |
|---------|-------|------------|----------------|
| **Purpose** | Virtual tables | Enforce constraints | Fast querying |
| **Storage** | None (virtual) | None (logic) | Bitmaps stored |
| **Performance** | Depends on query | Overhead on updates | Fast reads |
| **MySQL Support** | ✓ Native | ⚠️ Via triggers | ✗ Not native |
| **Use Case** | Simplify, secure | Business rules | Low-cardinality queries |
| **Best For** | OLTP/OLAP | Critical rules | OLAP/Data warehouses |

---

## Best Practices

### Views
1. ✓ Use for security (hide sensitive columns)
2. ✓ Use for simplification (complex joins)
3. ✓ Document view dependencies
4. ✓ Use WITH CHECK OPTION for updatable views
5. ⚠️ Avoid deep nesting (performance)

### Assertions
1. ✓ Use for critical business rules
2. ✓ Optimize predicates (add indexes)
3. ✓ Consider periodic validation for non-critical rules
4. ⚠️ Be aware of performance overhead
5. ✓ Document all assertion logic

### Bitmap Indexes
1. ✓ Use for low-cardinality columns
2. ✓ Use in read-heavy workloads (OLAP)
3. ✗ Avoid in OLTP systems
4. ✓ Combine with compression
5. ✓ Monitor update performance


# Database Internals

![[11 Storage and Indexing.pdf]]

## 1. The Hierarchy of Data: From Bits to Tables

Before looking at hardware, we must understand how data is bundled. Think of this like a Russian nesting doll.

- **Field (Attribute):** The smallest unit. A single value (e.g., `Price: 19.99`). Corresponds to a **Column**.
    
- **Record (Tuple):** A collection of fields describing one entity. Corresponds to a **Row**.
    
    - _Fixed-Length:_ Every field has a set size. Finding the $i^{th}$ field is easy math: $Offset = \sum Sizes$.
        
    - _Variable-Length:_ Uses an array of "field offsets" at the start of the record to tell the DB where each field begins.
        
- **Page (Block):** The fundamental unit of I/O. Databases don't read one row; they read a **Page** (usually 16KB).
    
- **Extent:** A group of contiguous pages (usually 64). Used to keep related data physically close to minimize disk arm movement.
    
- **File:** A collection of pages making up a Table or an Index.
    

## 2. Physical Storage: The "Hard" in Hardware

DBMS performance is dictated by the physical limitations of the disk.

### Disk Anatomy & Performance

- **Platter:** The physical disk spinning at high speeds.
    
- **Spindle:** The axis that holds the platters.
    
- **Read-Write Head:** The "needle" that reads data.
    
- **Track:** A ring on the platter.
    
- **Cylinder:** The set of tracks at a given arm position across all platters.
    

### The Math of Waiting (Disk Access Metrics)

1. **Seek Time (1–20ms):** Moving the arm to the correct track. **(Dominant Cost)**.
2. **Rotational Delay (0–10ms):** Waiting for the disk to spin to the right block.
3. **Transfer Time (~1ms per 4KB):** Moving data to RAM.

**The Trend:** Disk capacity grows 50% yearly, but access speed (Seek/Rotate) stays almost flat. This is why software optimization (Indexing) is mandatory.

## 3. Buffer Management: The "Waiting Room"

The **Buffer Manager** allocates RAM to hold pages. It acts as a middleman between the disk and the query engine.

### Why not use the Operating System (OS)?

OS memory management is generic. A DBMS knows _more_ about the data:

- **Prefetching:** If you are scanning a table, the DBMS knows you'll need Page 2 after Page 1 and can load it ahead of time.
- **Force-Writing:** For security/logging (WAL), the DBMS must ensure data is _actually_ on disk, which OS caches might delay.

### Buffer Replacement Policies

When RAM is full, which page gets kicked out?

- **LRU (Least Recently Used):** Kick out the page untouched for the longest time.
    
- **MRU (Most Recently Used):** Kicked out the page used _last_. (Best for "Big Scans" where you won't repeat the read soon).
    
- **Clock:** Uses a "use bit." If a page is hit, bit=1. The "hand" moves; if bit=1, it sets it to 0 and moves on. if bit=0, it evicts.
    
- **Dirty Pages:** If a page was modified, it must be written to disk before being evicted.
    
- **Pinning:** Prevents a page from being evicted while a transaction is active.
    

## 4. File and Record Organization

How are records packed into a Page?

### Page Formats for Fixed-Length Records

1. **Packed:** Records are stored side-by-side. Deleting a record requires moving all subsequent records up to fill the hole.
    
2. **Unpacked (Bitmap):** A header tracks which "slots" are full or empty using bits (1 or 0). Deletion just flips a bit.
    

### File Organization Types

- **Heap:** Records are placed anywhere there is space. No order. Search = $O(N)$.
    
- **Sequential:** Records are stored in a specific order. Great for ranges, but $O(N)$ for insertions because you have to "shift" data.
    
- **Hashing:** A hash function determines the "Bucket." Search = $O(1)$ ideally.
    
- **Clustering:** Storing records from _different_ tables (e.g., Orders and OrderItems) in the same physical file to minimize I/O for Joins.
    

## 5. Indexing: The "Library Catalog"

An index is an "Access Path" to data.

### Alternatives for Data Entries ($k^*$)

- **Alternative 1:** The index entry _is_ the data record. (Clustered Index).
    
- **Alternative 2:** Entry is `<Key, RID>` (Record ID).
    
- **Alternative 3:** Entry is `<Key, List of RIDs>`.
    

### Index Classifications

- **Primary vs. Secondary:** Primary indexes use the Primary Key. Secondary indexes use any other field.
    
- **Clustered vs. Unclustered:** * **Clustered:** Data is stored in the same order as the index. (Fastest).
    
    - **Unclustered:** Index is sorted, but data is scattered. Requires one "Seek" per row found.
        
- **Sparse vs. Dense:** * **Dense:** Every record has an entry in the index.
    
    - **Sparse:** Only one entry per _page_ of the data file. (Saves space).
        
- **Bitmap Index:** Uses a string of bits (001010) to represent values (e.g., "Color: Blue"). Extremely fast for complex `AND/OR` queries.
    

## 6. MySQL Implementation & SQL Commands

```
-- Create a table with a Primary Key (Clustered Index in InnoDB)
CREATE TABLE Products (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    category_id INT,
    price DECIMAL(10,2)
);

-- Create a Unique Index (Prevents duplicate barcodes)
CREATE UNIQUE INDEX idx_barcode ON Products(barcode);

-- Create a Composite Index (Order: Category then Price)
-- Optimizes: WHERE category_id = 5 AND price < 100
CREATE INDEX idx_cat_price ON Products(category_id, price);

-- Drop an index to save write performance
DROP INDEX idx_name ON Products;
```

## 7. Optimization Tricks & Tips

1. **The Fill Factor:** Leave ~10-20% of your pages empty. This allows for new record inserts without causing "Page Splits" (where the DB has to reorganize the disk).
    
2. **Index Selectivity:** Don't index a column like `is_active` (Boolean). If the index doesn't filter out at least 80-90% of the table, the DBMS will ignore it and just scan the whole file.
    
3. **Covering Index:** Try to include all columns from a `SELECT` in your index.
    
    - _Bad:_ `SELECT name FROM Users WHERE id = 1` (Hits Index, then hits Data File).
        
    - _Good:_ Create index on `(id, name)`. (Hits Index only).
        
4. **Avoid Functions on Indexed Columns:** `WHERE YEAR(created_at) = 2023` will **not** use an index on `created_at`. Use `WHERE created_at >= '2023-01-01'` instead.
    
5. **Sequential UUIDs:** If using UUIDs as Primary Keys, use "ordered" UUIDs. Random UUIDs cause massive fragmentation and slow down every insert.
    

## Visual Summary: Performance Comparison

|                    |               |                     |               |                     |
| ------------------ | ------------- | ------------------- | ------------- | ------------------- |
| **Operation**      | **Heap File** | **Sequential File** | **Hash File** | **Clustered Index** |
| **Insert**         | $O(1)$        | $O(N)$              | $O(1)$        | $O(\log N)$         |
| **Search (Exact)** | $O(N)$        | $O(\log N)$         | $O(1)$        | $O(\log N)$         |
| **Search (Range)** | $O(N)$        | $O(\log N)$         | $O(N)$        | $O(\log N)$         |
| **Delete**         | $O(N)$        | $O(N)$              | $O(1)$        | $O(\log N)$         |

# Normalization in DBMS

## **Part 1: Functional Dependencies (FDs) - The Foundation**

### **What is a Functional Dependency?**

A **Functional Dependency** is a constraint that says: *If you know the value of attribute X, you can always determine the value of attribute Y with 100% certainty.*

**Notation:** `X → Y` (X determines Y)

**Real example:**
```
Student (StudentID, Name, Email)
```

Here: `StudentID → Name` (knowing the StudentID tells you the Name)
And: `StudentID → Email` (knowing the StudentID tells you the Email)

But: `Name ↛ Email` (knowing a name doesn't uniquely tell you the email—multiple students could have the same name)

### **Why does this matter?**

When you violate FDs, you create **update anomalies**—situations where changing data causes problems.

---

### **Attribute Closure (X⁺) - Finding What You Can Determine**

**Definition:** X⁺ is the set of ALL attributes that can be determined by knowing X.

**Algorithm to calculate X⁺:**

1. Start with X in your set
2. Look at every FD rule
3. If the left side is completely in your set, add the right side
4. Repeat until nothing new is added

**MySQL Example:**

Suppose you have these FDs:
- `StudentID → Name`
- `StudentID → Email`
- `Email → Department`

**Calculate StudentID⁺:**

```
Step 1: Start = {StudentID}
Step 2: StudentID → Name matches, add Name = {StudentID, Name}
Step 3: StudentID → Email matches, add Email = {StudentID, Name, Email}
Step 4: Email → Department matches, add Department = {StudentID, Name, Email, Department}
Step 5: No more FDs match
Result: StudentID⁺ = {StudentID, Name, Email, Department}
```

**Why this matters:** If StudentID⁺ includes ALL attributes in the table, then StudentID is a **Candidate Key** (a unique identifier).

**Real MySQL example:**
```sql
-- Table: Student
-- Attributes: StudentID, Name, Email, Department, Phone

-- FDs:
-- StudentID → Name, Email, Department, Phone
-- Email → StudentID, Name, Department, Phone

-- Calculate StudentID⁺
-- Start: {StudentID}
-- Apply StudentID → Name, Email, Department, Phone
-- Result: {StudentID, Name, Email, Department, Phone}
-- This is ALL attributes! So StudentID is a Candidate Key.
```

---

### **Armstrong's Axioms - The Rules of Inference**

These are formal rules to prove that an FD logically follows from other FDs.

**1. Reflexivity:** If Y ⊆ X, then X → Y
```
If you know StudentID and Name, you can always determine StudentID.
StudentID, Name → StudentID
```

**2. Augmentation:** If X → Y, then XZ → YZ
```
If StudentID → Name, then:
StudentID, Email → Name, Email
(Add the same attributes to both sides)
```

**3. Transitivity:** If X → Y and Y → Z, then X → Z
```
If StudentID → Email and Email → Department, then:
StudentID → Department
```

**Secondary Rules (derived from the above):**

**4. Union:** If X → Y and X → Z, then X → YZ
```
StudentID → Name AND StudentID → Email
Therefore: StudentID → Name, Email
```

**5. Decomposition:** If X → YZ, then X → Y and X → Z
```
StudentID → Name, Email
Therefore: StudentID → Name AND StudentID → Email
```

**6. Pseudotransitivity:** If X → Y and YZ → W, then XZ → W
```
StudentID → Email AND Email, Department → Phone
Therefore: StudentID, Department → Phone
```

**Practical example - proving an FD:**

```
Given FDs:
1. StudentID → Name
2. StudentID → Email
3. Email → Department

Prove: StudentID → Department

Proof:
- From FD1: StudentID → Name (given)
- From FD2: StudentID → Email (given)
- From FD3: Email → Department (given)
- Apply Transitivity on FD2 and FD3:
  StudentID → Email AND Email → Department
  Therefore: StudentID → Department ✓
```

---

### **Minimal Cover (Canonical Cover)**

A **Minimal Cover** is the "cleanest" version of your FD set: no redundancy, minimum complexity.

**Steps to find it:**

1. **Remove extraneous attributes from left sides**
2. **Remove extraneous attributes from right sides**
3. **Remove redundant FDs**

**Example:**

```
Original FDs:
1. StudentID, Name → Email
2. StudentID → Email
3. StudentID → Name

Step 1 & 2: Are there extra attributes?
FD1 has "Name" on the left but StudentID alone is enough (from FD2)
So remove FD1 entirely

Minimal Cover:
- StudentID → Email
- StudentID → Name
(FD3 is kept because it's needed)
```

**Why minimal? Because:**
- Easier to understand
- Easier to enforce in database
- Shows exactly what you need, nothing more

---

## **Part 2: Normalization & Normal Forms**

Now that you understand FDs, let's use them to **clean up bad tables**.

### **What are Anomalies?**

When your table violates FD rules, you get three types of problems:

**1. Update Anomaly:** Changing one thing requires changing multiple rows

**2. Insertion Anomaly:** You can't insert data without redundancy

**3. Deletion Anomaly:** Deleting one fact removes unrelated facts

### **Example - A Badly Designed Table:**

```SQL
CREATE TABLE StudentCourses (
    StudentID INT,
    Name VARCHAR(100),
    Email VARCHAR(100),
    CourseID INT,
    CourseName VARCHAR(100),
    Instructor VARCHAR(100),
    PRIMARY KEY (StudentID, CourseID)
);

INSERT INTO StudentCourses VALUES
(101, 'Alice', 'alice@uni.com', 'C001', 'Database', 'Dr. Smith'),
(101, 'Alice', 'alice@uni.com', 'C002', 'Algorithms', 'Dr. Jones'),
(102, 'Bob', 'bob@uni.com', 'C001', 'Database', 'Dr. Smith');
```

**Problems here:**
- **Update Anomaly:** If Alice changes her email, you must update TWO rows
- **Insertion Anomaly:** You can't add a new course (no StudentID) or a student without a course
- **Deletion Anomaly:** If Alice drops all courses, you lose her email!

---

## **1NF (First Normal Form) - Atomic Values**

**Rule:** Every attribute must contain only **atomic (indivisible) values**, i.e., shouldn't have multi-valued attribute
There shouldn't be any value that can lead us to create a same entry multiple times

**Bad (violates 1NF):**
```sql
CREATE TABLE Student_Bad (
    StudentID INT PRIMARY KEY,
    Name VARCHAR(100),
    Courses VARCHAR(255)  -- Contains 'Database, Algorithms, OS' (NOT atomic)
);
```

**Good (1NF compliant):**
```sql
CREATE TABLE Student (
    StudentID INT PRIMARY KEY,
    Name VARCHAR(100)
);

CREATE TABLE Enrollment (
    StudentID INT,
    CourseID INT,
    PRIMARY KEY (StudentID, CourseID),
    FOREIGN KEY (StudentID) REFERENCES Student(StudentID)
);
```

**Why it matters:** Atomic values prevent awkward string parsing and enable proper relationships.

---

## **2NF (Second Normal Form) - No Partial Dependencies**

**Rule:** Every non-key attribute must depend on the **ENTIRE primary key**, not just part of it.

**Problem - Partial Dependency:**
```sql
CREATE TABLE StudentCourse_Bad (
    StudentID INT,
    CourseID INT,
    Name VARCHAR(100),        -- Depends ONLY on StudentID (partial!)
    CourseName VARCHAR(100),  -- Depends ONLY on CourseID (partial!)
    Grade CHAR(1),            -- Depends on BOTH (OK)
    PRIMARY KEY (StudentID, CourseID)
);
```

Here:
- `StudentID, CourseID → Name` ✓ But also `StudentID → Name` (partial!)
- `StudentID, CourseID → CourseName` ✓ But also `CourseID → CourseName` (partial!)

**Solution - Decompose into 3 tables:**

```sql
CREATE TABLE Student (
    StudentID INT PRIMARY KEY,
    Name VARCHAR(100)
);

CREATE TABLE Course (
    CourseID INT PRIMARY KEY,
    CourseName VARCHAR(100)
);

CREATE TABLE Enrollment (
    StudentID INT,
    CourseID INT,
    Grade CHAR(1),
    PRIMARY KEY (StudentID, CourseID),
    FOREIGN KEY (StudentID) REFERENCES Student(StudentID),
    FOREIGN KEY (CourseID) REFERENCES Course(CourseID)
);
```

**Now:**
- Each table has a single purpose
- No partial dependencies
- Changes to Name only affect Student table

---

## **3NF (Third Normal Form) - No Transitive Dependencies**

**Rule:** No non-key attribute should depend on another non-key attribute.

**Problem - Transitive Dependency:**
```sql
CREATE TABLE Student_Bad (
    StudentID INT PRIMARY KEY,
    Name VARCHAR(100),
    DepartmentID INT,
    DepartmentName VARCHAR(100)  -- Depends on DepartmentID (non-key!)
);

-- FDs:
-- StudentID → Name ✓
-- StudentID → DepartmentID ✓
-- DepartmentID → DepartmentName ← TRANSITIVE DEPENDENCY
-- Therefore: StudentID → DepartmentName (through DepartmentID)
```

**Anomalies:**
- **Update:** Change department name? Update every student row
- **Deletion:** Delete last student in a department? Lose department info

**Solution - Decompose:**

```sql
CREATE TABLE Student (
    StudentID INT PRIMARY KEY,
    Name VARCHAR(100),
    DepartmentID INT,
    FOREIGN KEY (DepartmentID) REFERENCES Department(DepartmentID)
);

CREATE TABLE Department (
    DepartmentID INT PRIMARY KEY,
    DepartmentName VARCHAR(100)
);
```

**Now:** DepartmentName is only in the Department table. Changes are isolated.

---

## **BCNF (Boyce-Codd Normal Form) - The Strict Version**

**Rule:** Every **determinant** (left side of an FD) must be a **Candidate Key**.
This is stricter than 3NF.

**Problem case (3NF OK, BCNF not):**

```sql
CREATE TABLE Professor_Advisor_Student (
    StudentID INT,
    Professor VARCHAR(100),
    Advisor VARCHAR(100),
    PRIMARY KEY (StudentID, Professor)
);

-- FDs:
-- StudentID, Professor → Advisor
-- Advisor → Professor (each advisor teaches only one subject)
-- But Advisor is NOT a Candidate Key!
```

**Issue:** Advisor determines Professor, but Advisor isn't a key.

**Solution:**

```sql
CREATE TABLE StudentProfessor (
    StudentID INT,
    Professor VARCHAR(100),
    PRIMARY KEY (StudentID, Professor)
);

CREATE TABLE ProfessorAdvisor (
    Advisor VARCHAR(100) PRIMARY KEY,
    Professor VARCHAR(100)
);
```

**Rule of thumb:** If 3NF, check every FD's left side. If it's not a candidate key, decompose further for BCNF.

---

## **4NF (Fourth Normal Form) - Multivalued Dependencies**

**New concept: Multivalued Dependency (MVD)**

`X ⇶ Y` means: *X independently determines multiple values of Y*

**Typical scenario:** A table storing two independent many-to-many relationships.

**Problem:**

```sql
CREATE TABLE TeacherSubjectBook (
    Teacher VARCHAR(100),
    Subject VARCHAR(100),
    Book VARCHAR(100),
    PRIMARY KEY (Teacher, Subject, Book)
);

-- Data:
-- Alice | Math    | Book1
-- Alice | Math    | Book2
-- Alice | Science | Book1
-- Alice | Science | Book2

-- MVD: Teacher ⇶ Subject (Teacher determines multiple subjects)
-- MVD: Teacher ⇶ Book (Teacher determines multiple books)
-- Problem: These are INDEPENDENT, causing redundancy
```

If Alice teaches both Math and Science, and uses both Book1 and Book2, we need ALL combinations (2×2=4 rows).

**Solution - Decompose by MVD:**

```sql
CREATE TABLE TeacherSubject (
    Teacher VARCHAR(100),
    Subject VARCHAR(100),
    PRIMARY KEY (Teacher, Subject)
);

CREATE TABLE TeacherBook (
    Teacher VARCHAR(100),
    Book VARCHAR(100),
    PRIMARY KEY (Teacher, Book)
);
```

**Now:** Independent relationships are separate. No redundancy.

---

## **5NF (Projection-Join NF) - Join Dependencies**

**Concept:** Sometimes you need to split a table into 3+ pieces to reconstruct it losslessly.

**Problem case (rare in practice):**

```sql
CREATE TABLE Project (
    Supplier VARCHAR(100),
    Part VARCHAR(100),
    Project VARCHAR(100),
    PRIMARY KEY (Supplier, Part, Project)
);

-- A supplier supplies a part
-- A part is used in a project
-- A supplier works on a project
-- But these three facts are independent

-- You need the Supplier supplies the Part AND
-- Part is used in Project AND
-- Supplier works on Project
```

**Solution:**

```sql
CREATE TABLE SupplierPart (
    Supplier VARCHAR(100),
    Part VARCHAR(100),
    PRIMARY KEY (Supplier, Part)
);

CREATE TABLE PartProject (
    Part VARCHAR(100),
    Project VARCHAR(100),
    PRIMARY KEY (Part, Project)
);

CREATE TABLE SupplierProject (
    Supplier VARCHAR(100),
    Project VARCHAR(100),
    PRIMARY KEY (Supplier, Project)
);
```

**When you need to know all three facts, you JOIN these three tables.**

---

## **Part 3: Schema Decomposition Properties**

When you split a table, two critical properties matter:

### **1. Lossless Join Property**

**Definition:** When you decompose a table and rejoin the pieces, you get EXACTLY the original data back (no extra rows, no lost rows).

**Example - Lossy Decomposition (BAD):**

```
Original Table:
StudentID | Name  | Department
101       | Alice | Math
102       | Alice | CS

Split into:
Table A:
StudentID | Name
101       | Alice
102       | Alice

Table B:
StudentID | Department
101       | Math
102       | CS

JOIN A and B:
StudentID | Name  | Department
101       | Alice | Math
102       | Alice | CS
✓ Same data! LOSSLESS

But what if we did:
Table A:
Name      | Department
Alice     | Math
Alice     | CS

Table B:
StudentID | Name
101       | Alice
102       | Alice

JOIN A and B:
StudentID | Name  | Department
101       | Alice | Math
101       | Alice | CS      ← WRONG! (phantom data)
102       | Alice | Math    ← WRONG! (phantom data)
102       | Alice | CS
✗ Extra rows! LOSSY
```

**How to test: The Matrix Method (Algorithm 15.2)**

This is a formal algorithm to verify losslessness. Here's the simplified version:

1. Create a matrix with attributes as columns, decomposed tables as rows
2. Fill with symbols based on attribute presence
3. Apply FD rules to unify symbols
4. If any row becomes all identical symbols, it's lossless

**Practical check:** Use the **Lossless Decomposition Rule**:

For decomposition into R1 and R2:
- `(R1 ∩ R2) → R1` OR `(R1 ∩ R2) → R2` must be true

**Example:**

```
Original: Student(StudentID, Name, Email, DepartmentID, DepartmentName)
FD: DepartmentID → DepartmentName

Decompose into:
R1 = Student(StudentID, Name, Email, DepartmentID)
R2 = Department(DepartmentID, DepartmentName)

Check: (R1 ∩ R2) = {DepartmentID}
Does DepartmentID → R1? No
Does DepartmentID → R2? YES! (DepartmentID → DepartmentName)
Therefore: LOSSLESS ✓
```

---

### **2. Dependency Preservation**

**Definition:** All original FDs can still be enforced without joining tables.

**Example - Lost Dependency (BAD):**

```sql
Original FDs:
1. StudentID → Name
2. StudentID → Email
3. Email → Department

Decompose into:
R1 = Student(StudentID, Name, Email)
R2 = Department(Email, Department)

Can we enforce:
1. StudentID → Name? YES (in R1)
2. StudentID → Email? YES (in R1)
3. Email → Department? YES (in R2)

Result: DEPENDENCY PRESERVED ✓

But if we decomposed as:
R1 = Student(StudentID, Name)
R2 = Course(Email, Department)

Can we enforce Email → Department? YES (in R2)
But we LOST the connection between StudentID and Email!

Result: DEPENDENCY LOST ✗
```

---

## **Complete MySQL Example - From Bad to 3NF**

Let's normalize a real-world example from start to finish.

### **Step 0: Bad Table (0NF)**

```sql
CREATE TABLE StudentCourses_0NF (
    StudentID INT,
    Name VARCHAR(100),
    Courses VARCHAR(255),  -- Multiple courses as comma-separated string
    PRIMARY KEY (StudentID)
);

INSERT INTO StudentCourses_0NF VALUES
(101, 'Alice', 'Database, Algorithms, OS'),
(102, 'Bob', 'Database, Web Dev');
```

**Problems:**
- Courses attribute is not atomic
- Can't query individual courses
- Can't add course without student

### **Step 1: Fix to 1NF**

```sql
CREATE TABLE Student_1NF (
    StudentID INT PRIMARY KEY,
    Name VARCHAR(100)
);

CREATE TABLE Enrollment_1NF (
    StudentID INT,
    CourseID INT,
    CourseName VARCHAR(100),
    Instructor VARCHAR(100),
    PRIMARY KEY (StudentID, CourseID)
);

INSERT INTO Student_1NF VALUES
(101, 'Alice'),
(102, 'Bob');

INSERT INTO Enrollment_1NF VALUES
(101, 1, 'Database', 'Dr. Smith'),
(101, 2, 'Algorithms', 'Dr. Jones'),
(101, 3, 'OS', 'Dr. Lee'),
(102, 1, 'Database', 'Dr. Smith'),
(102, 4, 'Web Dev', 'Dr. Brown');
```

**Remaining problems:**
- CourseName and Instructor depend ONLY on CourseID (partial dependency)
- If Dr. Smith changes, must update multiple rows

### **Step 2: Fix to 2NF**

```sql
CREATE TABLE Student_2NF (
    StudentID INT PRIMARY KEY,
    Name VARCHAR(100)
);

CREATE TABLE Course_2NF (
    CourseID INT PRIMARY KEY,
    CourseName VARCHAR(100),
    Instructor VARCHAR(100)
);

CREATE TABLE Enrollment_2NF (
    StudentID INT,
    CourseID INT,
    PRIMARY KEY (StudentID, CourseID),
    FOREIGN KEY (StudentID) REFERENCES Student_2NF(StudentID),
    FOREIGN KEY (CourseID) REFERENCES Course_2NF(CourseID)
);
```

**Remaining problem:**
- Instructor depends on CourseID, but what if we want to track multiple instructors?

### **Step 3: Fix to 3NF**

```sql
CREATE TABLE Student_3NF (
    StudentID INT PRIMARY KEY,
    Name VARCHAR(100)
);

CREATE TABLE Instructor_3NF (
    InstructorID INT PRIMARY KEY,
    InstructorName VARCHAR(100)
);

CREATE TABLE Course_3NF (
    CourseID INT PRIMARY KEY,
    CourseName VARCHAR(100),
    InstructorID INT,
    FOREIGN KEY (InstructorID) REFERENCES Instructor_3NF(InstructorID)
);

CREATE TABLE Enrollment_3NF (
    StudentID INT,
    CourseID INT,
    PRIMARY KEY (StudentID, CourseID),
    FOREIGN KEY (StudentID) REFERENCES Student_3NF(StudentID),
    FOREIGN KEY (CourseID) REFERENCES Course_3NF(CourseID)
);

-- Queries now:
INSERT INTO Student_3NF VALUES (101, 'Alice');
INSERT INTO Instructor_3NF VALUES (1, 'Dr. Smith');
INSERT INTO Course_3NF VALUES (1, 'Database', 1);
INSERT INTO Enrollment_3NF VALUES (101, 1);

-- Update Instructor name—only one place!
UPDATE Instructor_3NF SET InstructorName = 'Dr. Smith Jr.' WHERE InstructorID = 1;

-- Query: Get all courses Alice takes
SELECT c.CourseName, i.InstructorName
FROM Enrollment_3NF e
JOIN Course_3NF c ON e.CourseID = c.CourseID
JOIN Instructor_3NF i ON c.InstructorID = i.InstructorID
WHERE e.StudentID = 101;
```

---

## **Quick Reference: Check if a Table is in 3NF**

For each FD `X → Y` in the table:

1. **Is X a Candidate Key?** → YES = OK
2. **Is Y a Prime Attribute** (part of any candidate key)? → YES = OK
3. **Otherwise** → VIOLATES 3NF, decompose


I'll teach you Transaction Management in detail with clear explanations and MySQL examples. Let me break this down systematically.

---

# Transaction Management

## 1.1 – Introduction & Definitions

**What is a Transaction?**

A transaction is a **sequence of database operations grouped together as a single logical unit of work**. Think of it as a contract: either all operations succeed and are saved permanently, or none of them happen at all.

**Real-World Analogy:**
Imagine transferring money from your bank account to a friend's account. This involves:
1. Reading your current balance
2. Deducting the transfer amount from your account
3. Reading your friend's balance
4. Adding the transfer amount to their account
5. Saving both changes

If the system crashes after step 2 but before step 4, your money would be deducted but your friend wouldn't receive it. A transaction prevents this disaster.

**MySQL Transaction Example:**
```sql
-- Start a transaction
START TRANSACTION;

-- All these operations are grouped together
SELECT balance INTO @myBalance FROM accounts WHERE account_id = 101;
UPDATE accounts SET balance = balance - 500 WHERE account_id = 101;
UPDATE accounts SET balance = balance + 500 WHERE account_id = 102;

-- Either COMMIT all changes or ROLLBACK everything
COMMIT; -- Makes changes permanent
-- OR
ROLLBACK; -- Undoes everything
```

**Why Transactions Matter:**
- Without transactions, concurrent access could corrupt data
- System failures could leave database in inconsistent state
- Multiple users could interfere with each other's operations

---

## 1.2 – ACID Properties (The Golden Rules)

These are **four fundamental guarantees** every transaction must provide:

#### **A – Atomicity (All-or-Nothing)**

**Definition:** A transaction is either **completely executed or not executed at all**. There's no middle ground.

**Who ensures it?** Transaction Management Component (through logging and rollback mechanisms)

**Example:**
```sql
START TRANSACTION;

-- Both must succeed
UPDATE accounts SET balance = balance - 500 WHERE account_id = 101;
UPDATE accounts SET balance = balance + 500 WHERE account_id = 102;

COMMIT; -- Both succeed together
```

If there's an error in the second UPDATE, the first one is automatically rolled back. You never have a situation where one account is debited but the other isn't credited.

**What if something fails?**
```sql
START TRANSACTION;

UPDATE accounts SET balance = balance - 500 WHERE account_id = 101; -- Success
UPDATE accounts SET balance = balance + 500 WHERE account_id = 999; -- Error: account doesn't exist

-- Entire transaction rolls back automatically
-- Account 101's balance change is UNDONE
```

---

#### **C – Consistency (Logical Correctness)**

**Definition:** The database moves from one **valid/correct state to another valid state**. All business rules must be satisfied.

**Who ensures it?** Application Programmer (through proper SQL logic)

**Example:**
Suppose you have a constraint: *Total balance of all accounts must remain constant during transfer*

```sql
-- Initial state: Account 101 has 1000, Account 102 has 2000, Total = 3000
START TRANSACTION;

UPDATE accounts SET balance = balance - 500 WHERE account_id = 101; -- 500
UPDATE accounts SET balance = balance + 500 WHERE account_id = 102; -- 2500

COMMIT;
-- Final state: Total = 3000 (still consistent)
```

If you accidentally wrote a transaction that violated business logic:
```sql
-- WRONG - Violates consistency
START TRANSACTION;

UPDATE accounts SET balance = balance - 500 WHERE account_id = 101;
UPDATE accounts SET balance = balance + 600 WHERE account_id = 102; -- Money created!
```

The DBMS won't stop this automatically—it's the **programmer's responsibility** to write correct SQL.

---

#### **I – Isolation (Independence)**

**Definition:** Each transaction executes **as if it's the only one in the system**. Transactions don't interfere with each other.

**Who ensures it?** Concurrency Control Component (through locking mechanisms)

**Example:**
Imagine two ATMs accessing the same account simultaneously:

**Without Isolation:**
```
Time  ATM1 (User A)              ATM2 (User B)
1     Read balance: 1000
2                                Read balance: 1000
3     Withdraw 200
4     Write balance: 800
5                                Withdraw 300
6                                Write balance: 700 (WRONG! Lost Update)
-- User B's withdrawal overwrote User A's balance change
```

**With Isolation (correct):**
```
Time  ATM1 (User A)              ATM2 (User B)
1     Read balance: 1000         (waits for ATM1 to finish)
2     Withdraw 200
3     Write balance: 800
4     Commit
5                                Now reads balance: 800
6                                Withdraw 200
7                                Write balance: 600
8                                Commit
```

**MySQL Example:**
```sql
-- Connection 1 (ATM 1)
START TRANSACTION;
SELECT balance FROM accounts WHERE account_id = 101; -- 1000
UPDATE accounts SET balance = balance - 200 WHERE account_id = 101;
COMMIT;

-- Connection 2 (ATM 2) - operates independently
START TRANSACTION;
SELECT balance FROM accounts WHERE account_id = 101; -- Either 1000 (before ATM1 commits) or 800 (after)
UPDATE accounts SET balance = balance - 300 WHERE account_id = 101;
COMMIT;
```

---

#### **D – Durability (Persistence)**

**Definition:** Once a transaction is **committed**, the changes are **permanent** even if there's a system crash, power failure, or hardware failure.

**Who ensures it?** Recovery Management Component (through transaction logs and backups)

**How it works:**
1. Changes are written to a **transaction log** on disk
2. After successful commit, the log entry is forced to stable storage
3. Even if the computer crashes immediately after, recovery can restore the committed data

**Example:**
```sql
START TRANSACTION;
UPDATE accounts SET balance = balance - 500 WHERE account_id = 101;
COMMIT; -- At this point, changes are written to disk

-- Even if power fails in the next second, the change is permanent
```

**What if crash happens before COMMIT?**
```sql
START TRANSACTION;
UPDATE accounts SET balance = balance - 500 WHERE account_id = 101;
-- CRASH happens here (no COMMIT executed)

-- After restart: The update is ROLLED BACK
-- Database is back to the state before the transaction started
```

---
	A **phantom problem** (also known as a **phantom read**) is a specific concurrency issue in database management systems where a transaction reads a set of rows that satisfy a search condition, but when it repeats the read later in the same transaction, the set of rows has changed because another transaction inserted or deleted data in the meantime.

It essentially creates a situation where data appears to pop in or out of existence like a "phantom" between two points of execution within a single transaction.

---

### Why It Happens: The "Locking Gap"

In standard **Strict Two-Phase Locking (S2PL)** with **record-level locking**, the system only places locks on records that exist at the time of the initial scan. Because the database is only locking the specific rows it finds, it doesn't "claim" the empty space where new rows might be inserted.

### A Practical Scenario

Using the example from **image_641fba.jpg**:

- **Initial Read**: Transaction $T1$ scans Page 1 to find the oldest employee in Grade 1. It sees ages $\{50, 51, 56\}$ and determines the oldest is **56**.
    
- **The Insertion**: While $T1$ is still active, Transaction $T2$ inserts a new employee record with **age 60** and **Grade 1**, then commits.
    
- **The Phantom**: $T1$ re-scans the table later in its execution. It now sees the new record $\{50, 51, 56, 60\}$. Suddenly, the "oldest age" has jumped to **60**.
    

This violates the **Isolation** property of the ACID model, as $T1$ is seeing uncoordinated changes made by $T2$ during its own lifespan.

---

### How to Solve the Phantom Problem

To prevent phantoms, you have to lock more than just the rows; you have to lock the **range** or the **container**.

1. **Page-Level Locking**: Instead of locking individual records, the database locks the entire physical page (e.g., Page 1). This prevents any other transaction from inserting _any_ new records into that block of memory until the first transaction is done.
    
2. **Index-Range / Predicate Locking**: The database locks a logical range (e.g., "all rows where Grade = 1"). If $T2$ tries to insert any record that falls into that "Grade 1" category, it will be blocked until $T1$ finishes.
### **1.3 – Types of Failures**

Understanding different failures helps us design recovery mechanisms:

#### **1. Computer Failure (System Crash)**
- Hardware error or OS crash during transaction execution
- Example: DBMS process crashes in middle of UPDATE
- **Recovery:** Use transaction logs to redo or undo operations

```sql
-- System crashes here
UPDATE accounts SET balance = 500 WHERE account_id = 101;
-- Recovery: Check logs, determine if transaction was committed, redo or undo
```

#### **2. Transaction Failure (Logical Error)**
- The transaction itself has a problem
- Example: Division by zero, constraint violation, incorrect logic

```sql
START TRANSACTION;
UPDATE accounts SET balance = balance / 0; -- ERROR!
-- Transaction automatically rolls back
-- Database unchanged
```

#### **3. Local Errors (Data Issues)**
- Required data not found or condition not met
- Example: Trying to transfer from non-existent account

```sql
START TRANSACTION;
UPDATE accounts SET balance = balance - 500 WHERE account_id = 999; -- No such account
-- Transaction canceled; no rows affected
ROLLBACK;
```

#### **4. Concurrency Control Enforcement**
- One transaction conflicts with another concurrent transaction
- The conflicting transaction is aborted

```sql
-- Two users simultaneously update the same row
-- The second one to arrive is forced to abort and retry
```

#### **5. Disk Failure**
- Physical damage to storage device
- Read/write head crash causes data loss
- **Recovery:** Use backups and archived logs

#### **6. Catastrophic Failures**
- Power outages, natural disasters, network partitions
- Beyond software control
- **Recovery:** Geographically distributed backups, redundant systems

---

### **1.4 – TCL Commands (Transaction Control Language)**

These are SQL commands you use to manage transactions:

#### **COMMIT**
- **Purpose:** Make all changes permanent
- **When used:** After all operations succeed
- Once committed, changes cannot be undone

```sql
START TRANSACTION;
INSERT INTO accounts (account_id, balance) VALUES (999, 0);
UPDATE accounts SET balance = balance + 1000 WHERE account_id = 999;
COMMIT; -- Changes are now permanent
```

#### **ROLLBACK**
- **Purpose:** Undo all changes in the current transaction
- **When used:** When you want to cancel the transaction
- Takes database back to state before transaction started

```sql
START TRANSACTION;
UPDATE accounts SET balance = balance - 500 WHERE account_id = 101;
UPDATE accounts SET balance = balance + 500 WHERE account_id = 102;

-- Oops, made a mistake!
ROLLBACK; -- Both updates are canceled, balances unchanged
```

#### **SAVEPOINT**
- **Purpose:** Create a checkpoint within a transaction
- **When used:** When you want partial rollback
- Can rollback to a specific savepoint instead of entire transaction

```sql
START TRANSACTION;

UPDATE accounts SET balance = balance - 500 WHERE account_id = 101;
SAVEPOINT sp1; -- Mark this point

UPDATE accounts SET balance = balance + 500 WHERE account_id = 102;
-- Oops, wrong account!

ROLLBACK TO sp1; -- Go back to savepoint, undo only the second update
-- First update still stands

UPDATE accounts SET balance = balance + 500 WHERE account_id = 103; -- Correct
COMMIT; -- Both the first and corrected third updates are committed
```

**MySQL Example - Multiple Savepoints:**
```sql
START TRANSACTION;

-- Step 1: Add new customer
INSERT INTO customers (id, name) VALUES (555, 'John');
SAVEPOINT sp_customer;

-- Step 2: Create account for customer
INSERT INTO accounts (customer_id, balance) VALUES (555, 1000);
SAVEPOINT sp_account;

-- Step 3: Apply signup bonus
UPDATE accounts SET balance = balance + 50 WHERE customer_id = 555;

-- Something went wrong with bonus logic
ROLLBACK TO sp_account; -- Undo bonus, but keep account
-- Now fix the bonus logic
UPDATE accounts SET balance = balance + 75 WHERE customer_id = 555; -- Correct amount

COMMIT; -- All changes saved
```

---

### **1.5 – Transaction State Diagram**

A transaction goes through different states during its lifecycle:

```
         START
          ↓
    ╔═════════════╗
    ║   ACTIVE    ║  ← Transaction executing, reading/writing data
    ╚═════════════╝
          ↓
    ╔══════════════════════╗
    ║ PARTIALLY COMMITTED  ║  ← Last statement executed, awaiting final commit
    ╚══════════════════════╝
          ↙        ↘
    ╔═════════╗  ╔══════════╗
    ║ FAILED  ║  ║ COMMITTED║  ← All changes permanent
    ╚═════════╝  ╚══════════╝
          ↓
    ╔═════════╗
    ║ ABORTED ║  ← Rolled back, database restored
    ╚═════════╝
```

**Detailed Explanation:**

1. **ACTIVE** - Initial state
   - Transaction is running
   - Reading and writing data
   - No commits yet
   ```sql
   START TRANSACTION;
   UPDATE accounts SET balance = balance - 500; -- ACTIVE state
   ```

2. **PARTIALLY COMMITTED** - After last statement completes
   - All SQL statements have executed
   - But not yet permanently saved to disk
   - Still in memory, vulnerable to failure
   ```sql
   UPDATE accounts SET balance = balance + 500; -- Just finished
   -- Currently in PARTIALLY COMMITTED state
   ```

3. **COMMITTED** - Success path
   - Changes written to disk
   - Permanent and durable
   - Transaction complete
   ```sql
   COMMIT; -- Now COMMITTED
   ```

4. **FAILED** - Error detected
   - An operation encountered an error
   - Cannot proceed normally
   - Must be rolled back
   ```sql
   UPDATE accounts SET balance = balance / 0; -- ERROR!
   -- State becomes FAILED
   ```

5. **ABORTED** - Rolled back
   - Changes undone
   - Database restored to pre-transaction state
   - Options:
     - **Restart:** Re-execute if it was a temporary issue
     - **Kill:** Terminate if it's a logical error
   ```sql
   ROLLBACK; -- Move to ABORTED state
   ```

**Complete MySQL Example:**
```sql
-- Start a transaction
START TRANSACTION;
-- State: ACTIVE

SELECT @balance := balance FROM accounts WHERE account_id = 101;
-- Still ACTIVE - reading data

UPDATE accounts SET balance = balance - 500 WHERE account_id = 101;
UPDATE accounts SET balance = balance + 500 WHERE account_id = 102;
-- Still ACTIVE - multiple writes

-- After last statement: PARTIALLY COMMITTED state

COMMIT;
-- State: COMMITTED (success)

-- If something went wrong:
-- ROLLBACK;
-- State: ABORTED
```

---

## 1.6 – Concurrent Execution

**Why run transactions concurrently?**

Imagine a bank with one customer and one teller—very inefficient. The bank uses multiple tellers to serve many customers simultaneously. Similarly, a database server handles many transactions at once.

#### **Advantages of Concurrency:**

1. **Increased Resource Utilization**
   - While one transaction waits for disk I/O, another uses CPU
   - Hardware is used continuously, not idle

2. **Better Throughput**
   - Process more transactions per second
   - More customers served in same time

3. **Reduced Average Response Time**
   - Short transactions don't wait behind long ones
   - Fair scheduling possible

**Example:**
```
Without Concurrency:
T1 (10 seconds) → T2 (2 seconds) → T3 (2 seconds)
Total time: 14 seconds, Average per transaction: 4.67 seconds

With Concurrency (interleaved):
Time 0-2:   T1, T2, T3 all running
Time 2-4:   T1, T3 running (T2 finished)
Time 4-10:  T1 running
Total time: 10 seconds, Average per transaction: 3.33 seconds
```

#### **Concurrency Control Schemes**

**Definition:** Mechanisms to manage concurrent transactions so they don't interfere with each other and maintain data consistency.

**Common Approaches:**
- **Locking:** Only one transaction can access a resource at a time
- **Timestamp Ordering:** Transactions ordered by timestamp
- **Optimistic Concurrency:** Assume conflicts rare, detect conflicts at commit time

We'll discuss these in detail in later units.

#### **Schedules**

**Definition:** A sequence showing the order in which operations of concurrent transactions are executed.

**Rules for a valid schedule:**
1. Must include all operations from all transactions
2. Must preserve the order of operations within each transaction
3. Can interleave operations from different transactions

**Example:**
```
Two transactions:
T1: Read(A), Write(A), Read(B), Write(B)
T2: Read(A), Write(A), Read(C), Write(C)

A valid schedule might be:
T1: Read(A)
T2: Read(A)
T1: Write(A)
T2: Write(A)
T1: Read(B)
T1: Write(B)
T2: Read(C)
T2: Write(C)

Note: T1 operations maintain their order: Read(A)→Write(A)→Read(B)→Write(B)
And: T2 operations maintain their order: Read(A)→Write(A)→Read(C)→Write(C)
```

---

## 1.7 – Anomalies Due to Interleaved Execution

When transactions run concurrently without proper control, bad things happen:

#### **WR Conflict – Dirty Read Problem**

**What is it?** A transaction reads data that was written by another transaction **before that transaction commits** (uncommitted data).

**Problem:** If the writing transaction fails and rolls back, the reading transaction has read incorrect ("dirty") data.

**Visual:**
```
Time  T1 (Transfer OUT)          T2 (Check Balance)
1     Read balance: 1000
2     Write balance: 500
3                                Read balance: 500 (DIRTY!)
4     ROLLBACK (transfer canceled)
5     Actual balance restored: 1000
-- T2 made decision based on balance that didn't really exist
```

**MySQL Example:**
```sql
-- Connection 1 (T1)
SET autocommit = 0; -- Disable autocommit for manual control
START TRANSACTION;
UPDATE accounts SET balance = 500 WHERE account_id = 101;
-- Not committed yet

-- Connection 2 (T2) - Set isolation level low
SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
SELECT balance FROM accounts WHERE account_id = 101; 
-- Returns 500 (DIRTY READ!)

-- Connection 1
ROLLBACK; -- Oops, transfer failed

-- Connection 2
SELECT balance FROM accounts WHERE account_id = 101; 
-- Returns 1000 (the dirty read was invalid!)
```

**Real-world impact:**
- Bank employee sees withdrawal but it never actually happened
- Stock trader sees stock price that gets reverted
- Confirmation email sent for transaction that doesn't exist

---

#### **RW Conflict – Unrepeatable Read / Incorrect Summary**

**What is it?** A transaction reads the same row multiple times and gets **different values** because another transaction updated it in between.

Also called "Inconsistent Read" or "Unrepeatable Read".

**Problem:** A transaction assumes a value stays constant, but it changes during the transaction.

**Scenario 1 - Unrepeatable Read:**
```
Time  T1 (Calculate Salary Bonus)  T2 (Salary Adjustment)
1     Read salary: 50000
2                                  Read salary: 50000
3                                  Update salary: 60000 (raise)
4                                  Commit
5     Read salary again: 60000
6     Calculate bonus: 60000 * 0.1 = 6000 (too high!)
-- Expected bonus was 5000, but got 6000
```

**Scenario 2 - Incorrect Summary:**
```
Time  T1 (Calculate Total)         T2 (Transfer Funds)
1     Read Account A: 1000, sum = 1000
2                                  Read Account A: 1000
3                                  Update Account A: 500
4                                  Read Account B: 2000
5                                  Update Account B: 2500
6                                  Commit
7     Read Account B: 2500, sum = 1000 + 2500 = 3500
-- Real sum was 3000 (500+2500), but saw 3500!
```

**MySQL Example:**
```sql
-- Connection 1 (T1)
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
START TRANSACTION;
SELECT balance FROM accounts WHERE account_id = 101; -- 1000

-- Connection 2 (T2)
UPDATE accounts SET balance = 2000 WHERE account_id = 101;
COMMIT;

-- Connection 1
SELECT balance FROM accounts WHERE account_id = 101; -- 2000 (different!)
-- Can't repeat the first read!
```

**Real-world impact:**
- Audit query shows different totals when run twice
- Salary calculation gets inconsistent data
- Reservation system double-books seats

---

#### **WW Conflict – Lost Update Problem**

**What is it?** Two transactions both write to the same data item, and **the second write overwrites the first**, losing the first transaction's update.

**Problem:** The first transaction's work is completely lost.

**Visual:**
```
Time  T1 (ATM 1: Withdraw 200)     T2 (ATM 2: Withdraw 300)
1     Read balance: 1000
2                                  Read balance: 1000
3     balance = 1000 - 200 = 800
4                                  balance = 1000 - 300 = 700
5     Write balance: 800
6                                  Write balance: 700 (OVERWRITES!)
-- LOST UPDATE! T1's withdrawal is lost
-- Should be: 1000 - 200 - 300 = 500
-- But it's: 700 (only T2's withdrawal counted)
```

**MySQL Example:**
```sql
-- Connection 1 (T1)
SET autocommit = 0;
SELECT balance FROM accounts WHERE account_id = 101; -- 1000
SET @balance = 1000;
UPDATE accounts SET balance = @balance - 200 WHERE account_id = 101;
-- Connection 1's update is: balance = 800

-- Connection 2 (T2) - interleaved
SELECT balance FROM accounts WHERE account_id = 101; -- Still reads 1000
SET @balance = 1000;
UPDATE accounts SET balance = @balance - 300 WHERE account_id = 101;
-- Connection 2's update is: balance = 700

-- Connection 1
COMMIT;

-- Connection 2
COMMIT; -- This overwrites T1's update!

-- Final result: balance = 700 (should be 500)
SELECT balance FROM accounts WHERE account_id = 101;
```

**Real-world impact:**
- Money disappears (lost withdrawal)
- Inventory count becomes wrong
- View counts on social media incorrect
- Website hit counter loses updates

---

**Summary Table of Anomalies:**

| Conflict | Read | Write | Problem |
|----------|------|-------|---------|
| **WR (Dirty Read)** | T2 reads | T1 writes uncommitted | T2 reads invalid data that T1 rolls back |
| **RW (Unrepeatable)** | T1 reads twice | T2 writes between reads | T1 gets different values for same item |
| **WW (Lost Update)** | - | Both write | T2's write overwrites T1's write |

---

### **1.8 – Scheduling & Serializability**

Now let's talk about how to fix these problems: **making concurrent execution safe**.

#### **Serial Schedule**

**Definition:** A schedule where **transactions execute one after another**, never interleaved. Each transaction completes fully before the next starts.

**Example:**
```
Serial Schedule 1:
T1: Read(A), Write(A), Read(B), Write(B) | T2: Read(A), Write(A), Read(C), Write(C)
-- T1 completes, then T2 starts

Serial Schedule 2:
T2: Read(A), Write(A), Read(C), Write(C) | T1: Read(A), Write(A), Read(B), Write(B)
-- T2 completes, then T1 starts
```

**Advantage:** No conflicts, data always consistent.
**Disadvantage:** No concurrency benefit—back to processing one transaction at a time.

---

#### **Equivalent Schedules**

**Definition:** Two schedules are **equivalent** if they produce the **same final database state** for any initial state.

**Key principle:** If both transactions maintain consistency individually, then equivalent schedules both maintain consistency.

**Example:**

Schedule S:
```
T1: Read(A)
T2: Read(A)
T1: Write(A)
T2: Write(A)
```

Schedule S':
```
T2: Read(A)
T1: Read(A)
T2: Write(A)
T1: Write(A)
```

If we apply both schedules to the same initial database state, the final result might be the same (or might differ based on the write values). The point is: if they always produce the same result, they're equivalent.

---

#### **Serializable Schedule**

**Definition:** A schedule is **serializable** if it is **equivalent to some serial schedule**.

**Why it matters:** Serializable schedules guarantee correctness even though they look interleaved. The interleaving doesn't actually cause any problems—it produces the same result as if transactions ran one-by-one.

**Example:**
```
This interleaved schedule:
T1: Read(A), Write(A) | T2: Read(A), Write(A)

is equivalent to this serial schedule:
T1: Read(A), Write(A), T2: Read(A), Write(A)

So the interleaved version is SERIALIZABLE (safe to use).
```

**The Golden Rule:**
> If each transaction preserves consistency (C in ACID), then every serializable schedule preserves consistency.

This means: **Make schedules serializable, and consistency is guaranteed**.

---

Below is a clear, comprehensive lesson on **conflict serializability** and the **precedence graph test**, with logical examples and the effects/implications of each rule.

---

## 1.9 Conflict Serializability

### 1) What is a Schedule?
A **schedule** is an interleaving of operations (read/write) from multiple transactions that preserves the **order of each transaction’s own steps**.

Example (two transactions):
- **T1**: `R1(X)  W1(X)`
- **T2**: `R2(X)  W2(X)`

A possible schedule:
```
R1(X)  R2(X)  W1(X)  W2(X)
```

---

### 2) Conflict Rules (same item Q)

Two instructions **conflict** if:
- they are from **different transactions**,  
- on the **same data item**,  
- and **at least one is a write**.

#### Conflict Table
| Operation Pair | Conflict? | Why |
|--------------|-----------|-----|
| Read–Read    | ❌ No      | Neither changes data |
| Read–Write   | ✅ Yes     | Write can change what read sees |
| Write–Read   | ✅ Yes     | Read may see old or new value |
| Write–Write  | ✅ Yes     | Final value depends on order |

#### Effect of Conflicts
Conflicts matter because they can **change the final database state** or **what a read observes**. If you swap conflicting operations, you can change results, which is unsafe.

---

### 3) Conflict Equivalence

Two schedules are **conflict equivalent** if:
- they contain the **same operations**, and  
- the **order of every conflicting pair** is the same in both.

You can transform one schedule into another by **swapping only non‑conflicting adjacent operations**.

#### Example
Transactions:
- **T1**: `R1(X) W1(X)`
- **T2**: `R2(Y) W2(Y)`

Schedule S:
```
R1(X) R2(Y) W1(X) W2(Y)
```

Here, operations on X and Y are independent → **non‑conflicting**.  
So you can swap some steps and still get the same effect.

---

### 4) Conflict Serializability

A schedule is **conflict serializable** if it is **conflict equivalent to a serial schedule** (one transaction fully before the other).

#### Example (Serializable)

Transactions:
- **T1**: `R1(X) W1(X)`
- **T2**: `R2(X) W2(X)`

Schedule:
```
R1(X) W1(X) R2(X) W2(X)
```

This is already serial (T1 → T2).  
So it is **conflict serializable**.

#### Example (Non‑Serializable)

Schedule:
```
R1(X) R2(X) W1(X) W2(X)
```

Conflicts:
- `R2(X)` and `W1(X)` → order matters
- `R1(X)` and `W2(X)` → order matters
Both directions create a **cycle** (see graph section).  
So **not conflict serializable**.

---

## 1.10 Precedence Graph (Serializability Test)

### 1) What is a Precedence Graph?
A **directed graph**:
- **Vertices** = transactions  
- **Edge Ti → Tj** if:
  - Ti and Tj conflict on some item,  
  - and Ti’s operation happens first.

#### Labeling
You can label edges with the item (e.g., `X`) to show why the edge exists.

---

### 2) Acyclic = Serializable

If the graph has **no cycle**, the schedule is **conflict serializable**.

A topological order gives the **serial equivalent order**.

#### Example (Acyclic)

Schedule:
```
R1(X) W1(X) R2(X) W2(X)
```

Conflicts:
- `W1(X)` before `R2(X)` → edge T1 → T2  
Graph: **T1 → T2** (no cycle)

Serializable as **T1 then T2**

---

### 3) Cyclic = NOT Serializable

If there is a **cycle**, no serial order can preserve all conflicts.

#### Example (Cyclic)

Schedule:
```
R1(X) R2(X) W1(X) W2(X)
```

Conflicts:
- `R2(X)` before `W1(X)` → edge T2 → T1  
- `R1(X)` before `W2(X)` → edge T1 → T2  

Graph:
```
T1 → T2
T2 → T1
```
Cycle exists → ❌ Not conflict serializable.

---

### 4) Algorithmic Effect (Time Complexity)

- **O(n²)** naive check for edges  
- **O(n + e)** cycle detection with DFS or Kahn’s algorithm  
Where:
- **n** = number of transactions  
- **e** = number of edges (conflicts)

---

## Why This Matters (Effects & Implications)

### If schedule is conflict serializable:
- You get **correctness equivalent to serial execution**
- Database remains **consistent**
- Concurrency is safe

### If not serializable:
- **Lost updates**
- **Dirty reads**
- **Unrepeatable reads**
- Final state depends on timing → unsafe

---

## Full Worked Example (Step‑by‑Step)

Transactions:
- **T1**: `R1(X) W1(X)`
- **T2**: `R2(X) W2(X)`
- **T3**: `R3(X) W3(X)`

Schedule:
```
R1(X) R2(X) W2(X) R3(X) W1(X) W3(X)
```

### Conflicts:
- `R1(X)` before `W2(X)` → T1 → T2  
- `W2(X)` before `R3(X)` → T2 → T3  
- `R3(X)` before `W1(X)` → T3 → T1  

Graph:
```
T1 → T2 → T3 → T1 (cycle)
```

Not conflict serializable.

---

## 1.11 – View Serializability

**Conflict serializability** is strict but sometimes overly restrictive. **View serializability** is more lenient.

#### **View Equivalence (Three Conditions)**

Two schedules are **view equivalent** if for every data item Q:

1. **Initial Read:** If Ti reads the **initial value** of Q in schedule S, Ti must also read the initial value in schedule S'.
2. **Intermediate Reads:** If Ti reads value **written by Tj** in schedule S, Ti must read the value written by Tj in schedule S'.
3. **Final Write:** The transaction performing the **final write** of Q in schedule S must also do so in schedule S'.

**In simpler terms:**
- Same reads of initial values
- Same reads of intermediate writes
- Same final writer

#### **Example 1: View Serializable but NOT Conflict Serializable**

```
Schedule:
T1: Write(A)          ← Final write of A (by T1)
T2: Read(A)           ← Reads T1's write
T2: Write(B)
T3: Read(B)           ← Reads T2's write

Serial equivalent: T1, T2, T3
```

**Check conflicts:**
- Precedence graph: T1 → T2 → T3 (no cycle) → CONFLICT SERIALIZABLE

Actually, this example is both conflict and view serializable. Let me give a better example.

#### **Blind Writes Concept**

A **blind write** occurs when a transaction **writes to a value without reading it first**.

```
T1: Write(A)    ← Blind write (doesn't read A first)
T2: Read(A)
T1: Read(A)
T2: Write(A)
```

**Key insight:** Every view-serializable schedule that's NOT conflict-serializable contains blind writes.

#### **Relationship Between Serializability Types**

```
Conflict Serializable ⊂ View Serializable

OR:

All conflict-serializable schedules are view-serializable.
But not all view-serializable schedules are conflict-serializable.
```

**Why care?**

Conflict serializability is easier to enforce (using locking). View serializability is harder to test in practice, so most databases use conflict serializability.

---

## 1.12 – Recoverability

Now we shift focus: instead of "are transactions safe from each other?", we ask "are transactions safe from failures?"

#### **Recoverable Schedule**

**Definition:** A schedule is **recoverable** if whenever transaction Tj **commits** after reading data **written by Ti**, Ti's commit appears **before** Tj's commit.

**In other words:** Don't commit a transaction that read uncommitted data from another transaction until that transaction itself commits.

**Visual:**
```
Recoverable:
T1: Write(A)
T1: Commit ← Must commit here (before T2 reads and commits)
T2: Read(A)
T2: Commit

NOT Recoverable:
T1: Write(A)
T2: Read(A)
T2: Commit ← Commits before T1!
T1: Rollback ← Now T2 read data that no longer exists
```

**MySQL Example:**

```sql
-- Connection 1 (T1)
START TRANSACTION;
UPDATE accounts SET balance = balance - 500 WHERE account_id = 101;
COMMIT; -- Commits before T2

-- Connection 2 (T2)
START TRANSACTION;
SELECT balance FROM accounts WHERE account_id = 101; -- Reads committed data
UPDATE accounts SET balance = balance + 500 WHERE account_id = 102;
COMMIT; -- Safe because T1 already committed
```

## Wait, but how does committing a Read affect anything?

**A Commit doesn't just mean "save data to the disk." A Commit means "I am officially finished, and any actions I took based on this data are now permanent and irreversible."**

### The Real-World Danger of "Committing a Read"

Transactions don't just read data for fun; they read data to _do something in the real world_.

Imagine a bank database:

- **$T_1$ (The Writer):** A pending wire transfer of $1,000 into your empty account.
    
- **$T_2$ (The Reader):** An ATM transaction where you are trying to withdraw $1,000 in cash.
    

Let's look at your **"NOT Recoverable"** schedule using this scenario:

1. **$T_1$: Write(A)** $\rightarrow$ The wire transfer tentatively adds $1,000 to your account. (Balance is now $1,000, but it's unconfirmed).
    
2. **$T_2$: Read(A)** $\rightarrow$ The ATM checks your balance. It sees $1,000!
    
3. **$T_2$: Commit** $\rightarrow$ The ATM transaction is complete. **It spits out $1,000 in physical cash.**
    
4. **$T_1$: Rollback** $\rightarrow$ The bank realizes the wire transfer was fraudulent and cancels it.

---

## Cascading Rollback (The Problem)

**Definition:** When one transaction fails and forces **multiple other transactions to rollback** in a chain reaction.

**Why it's bad:**
- Undoes significant amounts of work
- Time-consuming
- Can cause deadlocks
- User frustration

**Example:**

```
T10: Write(A), Commit
T11: Read(A), Write(B)
T12: Read(B), Write(C)

What if T11 fails?
→ T11 Rollback (undo B)
→ But T12 read B! (now invalid)
→ T12 must Rollback (undo C)
→ T12 cascades the rollback

Chain: T11 fails → forces T12 to rollback → could force T13, T14... to rollback
```

**Real World:**

Imagine an audit system:
```
Transaction 1: Record transaction, write to audit log
Transaction 2: Read audit log, generate audit report
Transaction 3: Read report, send to manager

If Transaction 1 fails and rolls back:
- Audit log entry deleted
- Report is invalid (read invalid audit log)
- Report must be deleted
- Manager's notification becomes invalid
- Notification must be retracted
```

---

## Cascadeless Schedule (The Solution)

**Definition:** A schedule is **cascadeless** if for each transaction Tj reading data written by Ti, the **commit of Ti appears before the read by Tj**.

**In simpler terms:** A transaction reads only **committed data**.

**Key Rule:**
```
If Tj reads data written by Ti:
    Ti must COMMIT before Tj READ

(Not just before Tj commits, but before Tj even reads!)
```

**MySQL Example:**

```sql
-- Connection 1 (T1)
START TRANSACTION;
UPDATE accounts SET balance = balance - 500 WHERE account_id = 101;
COMMIT; -- Must commit here

-- Connection 2 (T2) - can only start reading after T1 commits
START TRANSACTION;
SELECT balance FROM accounts WHERE account_id = 101; -- Now safe
UPDATE accounts SET balance = balance + 500 WHERE account_id = 102;
COMMIT;
```

**Comparison:**

```
Recoverable but NOT Cascadeless:
T1: Write(A)
T2: Read(A)      ← Read before T1 commits!
T1: Commit
T2: Commit

Cascadeless (and thus Recoverable):
T1: Write(A)
T1: Commit       ← Must commit first
T2: Read(A)      ← Now read
T2: Commit
```

**How Cascadelessness Prevents Cascading Rollbacks:**

If T1 fails and rolls back, T2 never even reads the data (because T1 hadn't committed when T2 tried to read). So T2 has nothing to roll back!

**Hierarchy of Schedules:**

```
Cascadeless Schedules ⊂ Recoverable Schedules ⊂ All Schedules

Every cascadeless schedule is recoverable.
But not all recoverable schedules are cascadeless.
```

**Best Practice (from the notes):**
> "It is desirable to restrict schedules to those that are cascadeless to avoid cascading rollbacks entirely."

---

## **Summary Table: Schedule Properties**

| Property | What it ensures | How | When to use |
|----------|-----------------|-----|------------|
| **Serializable** | Concurrency safety (no conflicts) | Precedence graph analysis | Always try for |
| **Recoverable** | Protection from failed reads | Commit Ti before Tj reads | Minimum standard |
| **Cascadeless** | No cascading rollbacks | Commit Ti before Tj reads | Best practice |

---

## **Complete MySQL Practical Example**

Let's put it all together with a real banking scenario:

```sql
-- Create accounts table
CREATE TABLE accounts (
    account_id INT PRIMARY KEY,
    name VARCHAR(50),
    balance DECIMAL(10, 2)
);

INSERT INTO accounts VALUES
(101, 'Alice', 1000),
(102, 'Bob', 2000),
(103, 'Charlie', 500);

-- SCENARIO 1: ACID Transaction
START TRANSACTION;
    -- Atomicity: All or nothing
    UPDATE accounts SET balance = balance - 300 WHERE account_id = 101;
    UPDATE accounts SET balance = balance + 300 WHERE account_id = 102;
COMMIT;
-- Consistency: Total balance maintained (3500 before and after)
-- Isolation: Other transactions don't interfere
-- Durability: Changes persistent

-- SCENARIO 2: Conflict Handling
-- Session 1
SET autocommit = 0;
START TRANSACTION;
SELECT balance FROM accounts WHERE account_id = 101; -- 700
-- At this point: ACTIVE state

UPDATE accounts SET balance = 400 WHERE account_id = 101;
-- At this point: PARTIALLY COMMITTED state

-- Session 2 (isolated, cannot see uncommitted change)
SELECT balance FROM accounts WHERE account_id = 101; -- Still sees 700

-- Session 1
COMMIT;
-- Now: COMMITTED state, Session 2 can see the change

-- SCENARIO 3: Savepoints
START TRANSACTION;
    INSERT INTO accounts VALUES (104, 'David', 1500);
    SAVEPOINT sp1;
    
    UPDATE accounts SET balance = balance + 100 WHERE account_id = 104;
    SAVEPOINT sp2;
    
    -- Oops, made a mistake
    ROLLBACK TO sp2; -- Undo the UPDATE
    
    -- Try again
    UPDATE accounts SET balance = balance + 50 WHERE account_id = 104;
COMMIT;
-- David created with balance 1550 (1500 + 50, not +100)

-- SCENARIO 4: Cascadeless Schedule Simulation
-- Connection 1
START TRANSACTION;
UPDATE accounts SET balance = 500 WHERE account_id = 101;
COMMIT; -- Must commit first

-- Connection 2 (can now safely read)
START TRANSACTION;
SELECT balance FROM accounts WHERE account_id = 101; -- 500
UPDATE accounts SET balance = balance + 100 WHERE account_id = 102;
COMMIT; 
-- If Connection 1 fails now, Connection 2 is unaffected (already committed)

-- SCENARIO 5: Preventing Dirty Reads
SET SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE;
-- Higher isolation prevents WR, RW, WW conflicts

START TRANSACTION;
SELECT SUM(balance) as total FROM accounts;
COMMIT;
-- Result is consistent, no dirty reads
```

---

## **Key Takeaways**

1. **ACID properties** guarantee database correctness and reliability
2. **Transactions** group multiple operations into atomic units
3. **Concurrency** improves performance but creates conflicts (WR, RW, WW)
4. **Serializability** makes interleaved execution safe
5. **Precedence graphs** test if a schedule is serializable
6. **Cascadeless schedules** prevent dangerous rollback chains
7. **Use MySQL's isolation levels** to enforce these properties at runtime

Below is a clean, detailed, and accurate guide to **Concurrency Control** in DBMS using **MySQL** examples. I’ll explain each topic clearly, with practical examples, and keep it direct.

---

# 2.1 Lock‑Based Protocols

## 🔴 Exclusive (X) Lock
- **Purpose:** Read *and* write.
- **Used for:** `INSERT`, `UPDATE`, `DELETE`.
- **Rule:** Only **one** X lock can exist on a data item. No other locks can coexist.

### MySQL Example
```sql
START TRANSACTION;
UPDATE accounts SET balance = balance - 500 WHERE id = 10;
```
- MySQL takes an **exclusive lock** on the matching row(s).
- Any other transaction trying to read/write those rows must wait.

---

## 🟢 Shared (S) Lock
- **Purpose:** Read only.
- **Used for:** `SELECT ... LOCK IN SHARE MODE`.
- **Rule:** Many S locks can coexist. But **cannot coexist with X lock**.

### MySQL Example
```sql
START TRANSACTION;
SELECT * FROM accounts WHERE id = 10 LOCK IN SHARE MODE;
```
- Other transactions can also read the same row.
- But no one can update/delete it until S locks are released.

---

## 📋 Lock Compatibility Matrix
- **S + S = OK**
- **S + X = NOT OK**
- **X + X = NOT OK**

If incompatible → request **waits**.

---

## Lock Table
DBMS keeps an in-memory structure:
- Item name
- Lock type (S/X)
- Who holds it
- Waiting queue

---

## Lock Manager
- Handles **lock/unlock** requests atomically.
- Grants locks or makes transactions wait/rollback.

---

## `LOCK TABLE` in MySQL
```sql
LOCK TABLES mytable READ;   -- shared
LOCK TABLES mytable WRITE;  -- exclusive
```
MySQL’s syntax differs from Oracle, but effect is similar.

Basically this is used to affect the whole tables itself to remove dependency of including locks in each query.

---

# 2.2 Two‑Phase Locking (2PL)
### What 2PL is

**2PL is a rule about when a transaction is allowed to take locks and when it’s allowed to release them.**  
If every transaction follows this rule, the resulting schedule is guaranteed to be **conflict-serializable** (equivalent to some serial order).

---

## Phase 1: Growing phase (only “take”, never “release”)

In the **growing phase**, a transaction:

- **may acquire locks** (S or X)
- **may upgrade locks** (S → X) (still counts as acquiring “stronger” lock)
- **may NOT release any lock**

Think: the transaction is “growing” its lock set.

---

## Phase 2: Shrinking phase (only “release”, never “take”)

In the **shrinking phase**, a transaction:

- **may release locks**
- **may downgrade** (X → S) (counts like releasing “strength”)
- **may NOT acquire any new locks**

The instant a transaction releases its **first** lock, it has entered shrinking phase and can’t lock anything new after that.

---

## Lock Point (key idea)

The **lock point** of a transaction = the moment it acquires its **last** lock.

2PL guarantees serializability because you can **order transactions by their lock points**, and the outcome will match a serial execution in that order.

---

# Why it matters (the 3 bullets you listed)

## 1) Guarantees conflict serializability

### Meaning of conflict-serializable

A schedule is conflict-serializable if it is equivalent to some serial order considering **conflicting operations**:

- conflicts happen when two different transactions access the same item and **at least one is a write**:
    - `R-R` does **not** conflict
    - `R-W`, `W-R`, `W-W` **do** conflict

### Why 2PL guarantees it (intuition)

Because locks prevent “illegal” interleavings:

- If T1 writes X, it holds an X-lock, so no one else can read/write X until allowed.
- The growing-then-shrinking structure prevents cycles in the precedence graph.

So: **2PL ⇒ precedence graph has no cycles ⇒ conflict-serializable.**

---

## 2) Not deadlock-free

2PL can cause deadlocks because transactions **can wait** for locks, and waiting can form cycles.

### Classic deadlock example (2 items: A and B)

- T1 locks A (X)
- T2 locks B (X)
- T1 now requests X lock on B → waits for T2
- T2 now requests X lock on A → waits for T1 Cycle formed → deadlock.

This follows 2PL perfectly (both are still “growing”), yet deadlock happens.

### MySQL note

InnoDB **detects** deadlocks and rolls back one transaction automatically:

- You’ll see `ERROR 1213: Deadlock found when trying to get lock; try restarting transaction`

---

## 3) Cascading rollbacks can still happen

### What a cascading rollback is

A **cascading rollback** happens when:

1. T1 writes something
2. T2 reads that uncommitted written value (a “dirty read”)
3. T1 aborts → now T2 must also abort because it used invalid data

That “domino effect” is cascading rollback.

### Why basic 2PL still allows it

Because **basic 2PL does not force you to keep X-locks until commit**. A transaction could:

- obtain X lock
- write
- release X lock _before commit_ (this is allowed in basic 2PL once shrinking starts) Another transaction could then read the uncommitted data **if the system allows it**.

### MySQL reality check

In MySQL with InnoDB, under normal isolation levels (READ COMMITTED / REPEATABLE READ), **dirty reads are not allowed**, so cascading rollbacks due to dirty reads are typically prevented by MVCC rules.  
But the theory point is: **2PL alone** (as a locking discipline) doesn’t guarantee recoverability/cascade-freedom. That’s why **Strict 2PL** exists.

---

# 2.3 Graph‑Based (Tree) Protocol

The **Tree protocol** is a locking protocol designed to guarantee **conflict serializability** *and* **deadlock freedom** by forcing transactions to acquire locks in a **predefined order** given by a **directed acyclic graph (DAG)** (often a tree).

Key rules (classic tree protocol):
1. **First lock can be on any node**.
2. After locking a node `X`, a transaction may lock node `Y` **only if `Y` is a child (descendant) of `X`** in the DAG.
3. A transaction **may unlock nodes at any time** (no 2-phase restriction).
4. Once a node is unlocked, that transaction **cannot relock it**.

It typically uses **X locks only** (simplifies correctness).

---

## 1) How it prevents the deadlock you mentioned (T1 has A needs B, T2 has B needs A)

### Deadlock under 2PL (no ordering)
- T1 locks **A**, then requests **B**
- T2 locks **B**, then requests **A**
→ wait cycle → deadlock.

### Under Tree protocol: this situation becomes impossible (if A and B have an ordering)

To use Tree protocol, **A and B must have a parent/child relation in the DAG** (or share an ancestor that dictates the order you must follow).

### Example DAG
Assume the database designer defines:

```
A → B
```

Meaning: **A is parent, B is child** (you must go A then B if you need both).

#### What happens now
- If T1 locks A, it is allowed to lock B next (good).
- If T2 locks B first, then T2 is **not allowed** to later lock A, because A is not a child of B.

So the “T2: lock B then request A” action is **illegal** in Tree protocol.

**Result:** the classic circular wait pattern cannot form.

#### Important: does this eliminate waiting entirely?
No. Transactions can still wait if another transaction holds a lock.  
But **deadlock (cycle of waits)** cannot occur because cycles require “out-of-order” acquisition somewhere.

---

## 2) “How is child decided?” (this is the most important conceptual piece)

The **child/parent relationship is not discovered dynamically** by the DB at runtime.

It is a **predefined partial order** over data items chosen by:
- the DB designer (in theory), or
- the DB’s internal structure (in practice), like:
  - hierarchical objects (Database → Table → Page → Row)
  - B-Tree index structure (root → internal nodes → leaves)
  - partition → page → record

### Intuition
The protocol needs a DAG so that:
- there is **no cycle** in the “lock ordering rules”
- everyone follows the same direction (top → down)

So “child” means: **a node reachable by following directed edges downward**.

---

## 3) Concrete teaching example with a real “tree”

Let’s model a table and its rows as a tree:

```
Table: accounts (T)
  ├── Row: id=10 (R10)
  └── Row: id=20 (R20)
```

Edges: `T → R10` and `T → R20`.

### Transaction that needs two rows
T1 wants to update both rows 10 and 20.

Under Tree protocol:
1. Lock T (table node) [X lock]
2. Lock R10 [X lock]
3. Unlock R10 early if done
4. Lock R20 [X lock]
5. Unlock R20
6. Unlock T

Notice: because the rule is “parent before child”, T1 is forced to lock the table node first if the graph is defined that way.

### Why this can reduce concurrency (a con)
Locking the parent `T` is broad: it may block other transactions that only wanted one row.  
That’s where “you might lock data you don’t need” comes from.

(Real DBs avoid this using intention locks / multi-granularity locking; Tree protocol is a theoretical baseline.)

---

## 4) Example showing early unlock (why it can improve concurrency)

Suppose:
- T1 updates row 10 then row 20
- T2 only needs row 10

With 2PL, T1 might hold locks until later (depending on variant), forcing T2 to wait longer.

With Tree protocol, T1 can:
- lock parent → lock row10 → update → **unlock row10 immediately**
Then T2 can proceed on row10 even while T1 later locks row20.

So **early unlock** can improve throughput.

---

## 5) Why it is conflict-serializable (simple explanation)

Because all transactions follow a consistent “top → down” locking order in an acyclic graph, you can’t get a cycle in the precedence graph that would violate serializability. The ordering constraints enforce a serializable interleaving.

---

## 6) Why cascading rollbacks can still happen (important)
Tree protocol allows **unlocking before commit**.

If the system allows another transaction to read a value written by an uncommitted transaction (dirty read), then:
- T1 writes X, unlocks X early
- T2 reads X (dirty)
- T1 aborts → T2 must abort
That’s cascading rollback.

This is why **Strict 2PL** is popular: it prevents that by holding X locks till commit.

(MySQL/InnoDB generally prevents dirty reads at normal isolation levels, but the protocol itself doesn’t guarantee it.)

---

## 7) “Some 2PL schedules impossible here and vice versa”
- Tree protocol forces an order (parent → child). If a transaction needs two unrelated nodes, you might be forced to lock some ancestor or may not be able to express a certain interleaving that 2PL could allow.
- Conversely, Tree protocol allows early unlocks, which some 2PL variants would forbid.

---

### Quick mental model
- **2PL:** “Don’t unlock until you’re done locking.” (prevents non-serializable interleavings, but can deadlock)
- **Tree protocol:** “Lock only moving downward in a predefined DAG.” (prevents deadlocks by preventing circular wait)

---

# 2.5 Timestamp‑Based Protocols

Each transaction gets a **timestamp** (older = smaller).

Each data item Q keeps:
- `R-timestamp(Q)`
- `W-timestamp(Q)`

## READ(Q) by Ti
- If `TS(Ti) < W-timestamp(Q)` → **rollback Ti**
- Else: read and update `R-timestamp(Q)`

## WRITE(Q) by Ti
- If `TS(Ti) < R-timestamp(Q)` → rollback
- If `TS(Ti) < W-timestamp(Q)` → rollback
- Else: write, set `W-timestamp(Q)=TS(Ti)`

### Properties
Serializable  
Deadlock‑free (no waiting)  
❌ Not cascade‑free  
❌ May cause many rollbacks

---
# 2.6 Thomas’ Write Rule

If a **write is too old**, instead of rollback → **ignore** the write.

### Why?
- Improves concurrency
- Allows **view‑serializable** schedules

Example:
- T1 (old) wants to write Q
- But Q already written by newer T2  
→ Ignore T1’s write

[[Detailed explanation of Thomas' Write Rule]]

---

# 2.7 Multiple Granularity Locking

Hierarchy:
**DB → Table → Page → Row → Field**

Locking higher level = implicitly locks all below.

## Lock Modes
| Mode | Meaning |
|------|---------|
| IS | Intention Shared |
| IX | Intention Exclusive |
| SIX | Shared + IX |
| S | Shared |
| X | Exclusive |

### Rules
- Lock **root first**
- Lock **top to bottom**
- Release **bottom to top**

### Example
To update a single row:
- IX on table
- X on row

---

# 2.8 Deadlock Handling

## 💀 Deadlock
Transactions waiting in a cycle:
T1 waits for T2, T2 waits for T1.

## 🚫 Starvation
A transaction waits forever (e.g., always rolled back).

---

## 2.8.1 Deadlock Prevention

### Wait-Die (non-preemptive)

When `Ti` requests a lock held by `Tj`:

- If `Ti` is **older** than `Tj` → `Ti` **waits**
- If `Ti` is **younger** than `Tj` → `Ti` **dies** (abort/rollback), and later restarts with the **same timestamp**

Why “non-preemptive”?  
Because if the lock holder is `Tj`, **you never force `Tj` to release the lock**. You only decide whether the requester waits or aborts.

**Effect:** younger transactions are more likely to be aborted.

### Wound-Wait (preemptive)

When `Ti` requests a lock held by `Tj`:

- If `Ti` is **older** than `Tj` → `Ti` **wounds** `Tj` → **abort `Tj`** (force it to release locks), then `Ti` gets the lock
- If `Ti` is **younger** than `Tj` → `Ti` **waits**

Why “preemptive”?  
Because an older requester can **preempt** (kick out) a younger lock holder.

**Effect:** younger transactions get aborted too, but _specifically when they block an older one_; older transactions tend to finish sooner.

#### The “one-line” difference

- **Wait-Die:** _young requester aborts_ (if it conflicts with older holder)
- **Wound-Wait:** _young holder aborts_ (if it conflicts with older requester)

### Timeout
Wait only fixed time → rollback.

---

## 2.8.2 Deadlock Detection & Recovery

### 🔍 Wait‑For Graph
Nodes = transactions  
Edge Ti → Tj if Ti waits for Tj  
Cycle = deadlock

### 🎯 Victim Selection
Pick cheapest rollback (least work).

### ↩️ Rollback Scope
- Full abort
- Partial rollback

### ⚡ Starvation Prevention
Count rollbacks so same victim isn’t always chosen.

---

# 3 ARIES

ARIES is the classic crash-recovery algorithm for **steal/no-force** database systems. It’s “how a serious DB can survive a power cut without losing committed work and without corrupting data,” even if dirty pages were written to disk before commit (steal) and even if pages are not forced to disk at commit (no-force).

Two important MySQL realities before we start (so we stay accurate):
- **MySQL’s InnoDB uses WAL-style recovery with redo + undo logs**, plus checkpoints, and it “replays” changes after a crash. Conceptually it’s very close to ARIES.
- InnoDB’s **exact log record format is not ARIES**, and terms differ (InnoDB has *redo log*, *undo log*, *binlog*, etc.). But the *core ideas you listed* (WAL, analysis/redo/undo phases, CLRs concept) map very well to what InnoDB does.

Below I’ll teach ARIES *properly* and then keep anchoring with **MySQL/InnoDB-style examples** so it’s not abstract.

---

## 1) The problem ARIES solves (steal/no-force)

### Buffer pool vs disk
Databases don’t update data files directly for every change. They:
1) read pages into memory (buffer pool),
2) modify memory,
3) later flush pages to disk.

### Steal / No-force
- **Steal**: the buffer manager is allowed to flush a dirty page to disk **even if the transaction hasn’t committed yet**.  
  - Good: avoids running out of memory.
  - Bad: disk might contain uncommitted data → must be able to UNDO after crash.
- **No-force**: on commit, the DB is **not required** to flush all pages touched by the transaction to disk.
  - Good: commit becomes fast.
  - Bad: committed changes might still be only in memory → must be able to REDO after crash.

So crash recovery must guarantee:
- **Atomicity**: uncommitted stuff must not remain (UNDO losers).
- **Durability**: committed stuff must not be lost (REDO winners).

ARIES is designed exactly for this.

---

## 2) WAL (Write-Ahead Logging): the non-negotiable rule

**WAL rule:** *Before a dirty page is written to disk, the log describing those changes must already be on stable storage.*

Why:
- If disk gets an uncommitted change (steal), we need log to undo it.
- If disk didn’t get a committed change (no-force), we need log to redo it.

### MySQL analogy
In InnoDB:
- Data changes generate **redo log records** (in redo log buffer → flushed to redo log files).
- Dirty pages can be flushed later.
- On crash, InnoDB uses redo logs to “roll forward” changes that weren’t in data files yet.

Commit durability in MySQL depends on settings:
- With `innodb_flush_log_at_trx_commit=1`, InnoDB flushes redo at commit (strong durability).
- With weaker settings, it may not flush each commit (faster, less durable). ARIES-style theory assumes commit record is forced to stable storage.

---

## 3) The log: LSN, pageLSN, and log tail

### LSN (Log Sequence Number)
Every log record has a monotonically increasing **LSN**.
- Think of LSN as “the byte offset / position in the log” or a unique increasing id.

### pageLSN
Each database page stores a **pageLSN**: “the LSN of the most recent update applied to this page.”
This is how redo decides whether a page already has a change.

### Log tail
The most recent part of the log sits in memory and is periodically flushed to stable storage.

---

## 4) Log record types (ARIES terminology) + MySQL mapping

### Update record
Contains enough info to redo/undo a change:
- `prevLSN`, `transID`, `pageID`, before-image, after-image, etc.

**MySQL mapping:** InnoDB redo logs are more “physical/byte-oriented” and undo is stored separately in undo segments. But conceptually, an “update happened” is logged for recovery.

### Commit record (force-written)
In ARIES, a transaction is committed when the **commit log record is on stable storage**.

**MySQL mapping:** With strict durability (`innodb_flush_log_at_trx_commit=1`), redo is flushed at commit so crash after commit won’t lose it.

### Abort / End
Abort triggers undo; End means fully cleaned up.

### CLR (Compensation Log Record)
A CLR is written **when undoing an update**.
Key properties:
- It is **redo-only** (it describes the undo action as a forward action).
- It has `undoNextLSN` so recovery can *skip work already undone* if crash happens during recovery.
- “Never undo an undo”: CLRs are not undone.

**MySQL mapping:** InnoDB has the concept of recording enough information so rollback/recovery can proceed safely. Exact CLR format isn’t exposed, but the *idea* (idempotent recovery progress markers) exists.

---

## 5) The two key in-memory tables ARIES reconstructs after a crash

### Transaction Table (TT)
One entry per active transaction:
- `tid`
- `status` (in-progress / committed / aborted)
- `lastLSN` (latest log record for that transaction)

### Dirty Page Table (DPT)
One entry per dirty page:
- `pageID`
- `recLSN`: the LSN of the **first** log record that made the page dirty.

`recLSN` is critical: it says “redo must start no later than here for this page.”

**MySQL mapping:** InnoDB has checkpoints and tracks dirty pages; during recovery it knows where to start applying redo based on checkpoint LSN and page LSN comparisons.

---

## 6) Checkpointing: “fuzzy” checkpoint (no stop-the-world)

A fuzzy checkpoint does **not** force dirty pages to disk; it just records “here’s what’s going on.”

ARIES steps:
1) write `begin_checkpoint`
2) write `end_checkpoint` containing a snapshot of TT and DPT
3) update the **master record** to point to the checkpoint LSN

Benefit: checkpoint is cheap and concurrent.

Limit: if dirty pages are very old (small `recLSN`), redo might still have to scan far back.

**MySQL mapping:** InnoDB checkpoints advance a checkpoint LSN; redo before that can be discarded. This is the same goal: bound recovery time and log growth.

---

## 7) ARIES Restart Recovery: the 3 phases

### Phase A: Analysis
Goal:
- Reconstruct TT and DPT at the time of crash
- Identify “loser” transactions (active / not committed)
- Find where redo must start

How:
- Start from the last checkpoint (via master record → begin_checkpoint → end_checkpoint).
- Scan log forward to end, updating TT and DPT.

Rules (conceptually):
- `END(T)` → remove T from TT
- any record for T → update `lastLSN`, status
- update on page P:
  - if P not in DPT: add with `recLSN = thisLSN`

Output:
- TT = who was active/committed/aborted at crash
- DPT = dirty pages and their earliest dirtier LSN

**MySQL example scenario (conceptual):**
- T1 updates account A, then commits.
- T2 updates account B, does not commit.
- Crash.
Analysis will mark:
- T1: committed (winner)
- T2: in-progress (loser)
DPT contains pages for A and B.

---

### Phase B: Redo (repeat history)
This is ARIES’s signature idea: **redo everything needed to reproduce the exact state at crash time**, including actions of loser transactions.

Start point:
- the smallest `recLSN` in the DPT (earliest thing that could matter)

Scan forward and for each redoable record, decide if it must be redone. Skip if:
1) page not in DPT, or
2) DPT says `recLSN > LSN(record)` (page got dirty later), or
3) on-disk `pageLSN >= LSN(record)` (already applied)

If redo needed:
- apply the change
- set `pageLSN = LSN(record)`

Important: **no new log record is written during redo**.

#### Why redo losers too?
Because after a crash, disk might be missing some updates that were only in memory. If a loser transaction updated a page in memory, redo must reapply it to recreate the “crash-time state,” then undo can cleanly roll it back.

**MySQL analogy:**
InnoDB crash recovery:
- applies redo to bring pages up to date as of crash
- then rolls back uncommitted transactions using undo

That is basically “repeat history, then rollback losers.”

---

### Phase C: Undo (only losers)
Undo all actions of loser transactions, in reverse order.

Mechanism:
- Initialize `ToUndo` set with `lastLSN` of each loser transaction.
- Repeatedly pick the **largest LSN** (work backwards globally).
- If record is an UPDATE:
  - write a CLR (with `undoNextLSN = prevLSN of the record being undone`)
  - apply the undo to the page
  - add `prevLSN` to `ToUndo`
- If record is a CLR:
  - follow `undoNextLSN` (this is how you skip work already undone)
- When a transaction finishes undo:
  - write `END(T)`

**Why CLRs matter (crash during recovery):**
If the system crashes halfway through undo:
- On restart, redo will reapply CLRs (so your undo progress is not lost)
- Undo will jump using `undoNextLSN` and not repeat already-undone actions

This is what makes recovery itself crash-safe.

---

## 8) A concrete teaching example (small but realistic)

Imagine one table `accounts(id, balance)` on disk. Two transactions:

- **T1**: `UPDATE accounts SET balance = balance - 100 WHERE id=1;` then COMMIT
- **T2**: `UPDATE accounts SET balance = balance + 100 WHERE id=2;` (no commit yet)
Crash happens.

Because of steal/no-force, many possibilities:
- The page for id=1 might not be on disk yet (no-force) → need REDO for T1.
- The page for id=2 might have been flushed early (steal) → need UNDO for T2.

### What ARIES does
**Analysis:** finds T1 committed, T2 loser; DPT includes both pages if dirtied.

**Redo:** reapplies updates for both T1 and T2 as needed to match crash-time state.

Now disk reflects:
- id=1 decreased
- id=2 increased  
even though T2 never committed (this is intentional after redo).

**Undo:** rolls back T2, writing CLRs. Final state:
- id=1 decreased (durable committed effect)
- id=2 unchanged (T2 undone)

That’s atomicity + durability.

### MySQL/InnoDB interpretation
- Redo log replays page modifications.
- Undo log allows rollback of uncommitted T2.
- After crash, InnoDB does crash recovery redo, then transaction rollback.

---

## 9) Media recovery (disk corruption) vs crash recovery (power loss)

Crash recovery assumes:
- data files exist but may be inconsistent w.r.t. memory
- log is intact on stable storage

Media recovery assumes:
- a disk/page/file is corrupted/lost
- you restore from a backup copy, then use logs to roll forward

ARIES media recovery:
1) restore last backup copy
2) redo from some LSN (based on checkpoint/recLSN) to bring it forward
3) undo losers as needed

**MySQL mapping:**
- “media recovery” is typically: restore backup + apply **binary logs** (point-in-time recovery).
- InnoDB redo logs are not meant to be long-term archival like binlogs; they’re for crash recovery.

So:
- **ARIES log** ≈ InnoDB **redo log** for crash recovery semantics
- **media recovery / PITR** in MySQL is more like **binlog-based** replay after restoring a backup

---

## 10) Shadow paging (why WAL/ARIES won)

Shadow paging idea:
- no log
- never overwrite a page in place
- write new pages elsewhere, update a page table
- commit = atomically switch the root/page table pointer
- abort = discard new pages

Why it’s not used for modern OLTP engines:
- fragmentation + write amplification
- complicated concurrency and fine-grained locking
- performance cliffs under high update rates

Modern systems overwhelmingly use WAL families (ARIES-like) because they:
- support high concurrency (row locks, MVCC)
- handle complex structures (B+trees) with logical/physiological logging
- keep sequential log writes (fast)

---

## 11) How this relates to what you’ll see in MySQL terms

If you keep learning MySQL/InnoDB, map terms like this:

- **Redo log (InnoDB)** ⇢ ARIES “log records used for REDO” (WAL)
- **Undo log (InnoDB)** ⇢ ARIES UNDO phase data (though ARIES describes undo via log + CLRs; InnoDB has dedicated undo structures for MVCC/rollback)
- **Checkpoint LSN** ⇢ “safe point” from which redo must start (bounded by dirty pages)
- **pageLSN** ⇢ InnoDB pages also have LSN metadata used to decide if redo is needed

If you want, I can also show you:
- how to *observe* some of this in MySQL (`SHOW ENGINE INNODB STATUS`, `performance_schema`), and
- how `innodb_flush_log_at_trx_commit` changes the durability guarantee.

---

## 12) Quick mastery checklist (if you truly understood ARIES)

You should be able to answer:
1) Why steal implies you must support undo after crash.
2) Why no-force implies you must support redo after crash.
3) Why WAL must force log before page flush.
4) What `pageLSN` is used for in redo decisions.
5) What `recLSN` means and why redo starts at min(recLSN).
6) Why ARIES redoes losers too (“repeat history”).
7) Why CLRs + `undoNextLSN` make recovery crash-safe.

---

Below is a structured, accurate “teach me from scratch” walkthrough of what you listed: **tree-structured indices (ISAM, B+ trees), B+ tree operations, bulk loading, and RAID**, plus how these ideas map to **MySQL/InnoDB** with concrete SQL examples.

I’ll be direct about what MySQL *does* vs what’s just DB theory.

---
# B+ trees and RAIDS

## 1) Tree-Structured Indices: what an index actually is

A database index is a **data structure** that lets the DBMS find rows **without scanning the whole table**.

### Key idea
- Without an index: `SELECT ... WHERE col = x` usually means scanning many pages/rows.
- With an index: the DB uses the index to jump to relevant pages quickly (logarithmic-ish).

### In MySQL (InnoDB)
- The **primary key** is stored as a **clustered B+ tree**: leaf pages contain the **entire row**.
- Secondary indexes are also **B+ trees**, but their leaf entries contain `(secondary_key, primary_key)` and then it “goes to” the clustered index to fetch the row.

---

## 2) ISAM (Index Sequential Access Method)

### What it is (theory)
- Data stored in a **sorted** sequential file.
- A **fixed** index points to ranges of the file.
- As you insert new rows, you may not be able to keep the file perfectly sorted, so inserts go into **overflow blocks** (extra pages linked off the main file pages).

### Why it becomes bad over time
- Overflow chains grow.
- Lookups and especially range scans become slower because you traverse overflow blocks.
- Eventually you need **reorganization** (rewrite/compact the whole structure).

### Where you see this idea today
Most modern DB engines avoid pure ISAM for general-purpose tables because it ages badly. Some storage engines historically used ISAM-like structures (MyISAM name hints at it), but modern InnoDB is B+ tree based.

---

## 3) B+ Trees (the workhorse index)

## 1. The Architecture: Anatomy of a B+ Tree

A B+ Tree is divided into two strictly different types of nodes. This "class system" is the most important part of its design.

### The Internal Nodes (The "Index" Layer)

- **What they are:** These are purely navigation tools. They do **not** store actual data or pointers to records.
    
- **What they hold:** They hold **Keys** and **Pointers** to other nodes.
    
- **The Math:** If an internal node has $k$ keys, it has exactly $k+1$ pointers.
    
- **Role:** Think of these as the "Address Decoding" logic in a CPU. They tell the system which memory block to jump to next.
    

### The Leaf Nodes (The "Data" Layer)

- **What they are:** This is the only place where the "real" information lives.
    
- **What they hold:** They hold the **Key** and the **Value** (either the actual data or a pointer to the row on the disk).
    
- **The Secret Sauce:** Every leaf node has a pointer to the **next** leaf node, creating a giant linked list at the bottom.
    

---

## 2. Navigation Logic: How one node handles multiple keys

You asked how a node directs a search if it has multiple keys inside it. This is where the $k+1$ pointers come in.

Inside a single internal node, the keys act as **boundary markers**. For a node with keys `[10, 20]`, there are three pointers:

1. **Left Pointer ($P_0$):** Leads to everything where $Key < 10$.
    
2. **Middle Pointer ($P_1$):** Leads to everything where $10 \le Key < 20$.
    
3. **Right Pointer ($P_2$):** Leads to everything where $Key \ge 20$.
    

When the database engine reads a node, it performs a **Binary Search** or a **Linear Search** _inside the node's memory_ to find the correct pointer. Because the node is usually small (fitting exactly in one disk page or cache line), this search is incredibly fast compared to the "cost" of fetching a new node from the disk.

---

## 3. The "Duplicate" Mystery: Why have repeats?

This is your main design doubt. Why have "5" in an internal node and "5" again in a leaf?

In a standard **B-Tree**, keys appear only once. If you find the key in an internal node, you stop. In a **B+ Tree**, every single key **must** exist in a leaf node, even if it's already in the internal nodes.

### Reason A: Separation of Concerns (Index vs. Data)

By making the internal nodes purely "road signs," we can pack them much tighter.

- Because internal nodes don't carry "Data," they can hold many more "Keys."
    
- More keys per node = Higher **Fan-out** ($m$).
    
- Higher Fan-out = Shorter Tree.
    
- A shorter tree means fewer disk I/O operations to find what you need.
    

### Reason B: The "Road Sign" Logic

Think of the keys in internal nodes as **Road Signs** on a highway.

- The sign on the highway says **"Exit 5: Downtown."**
    
- The sign is not "Downtown." You haven't arrived yet. The sign just tells you which lane to take.
    
- When you finally get to the city, you see another sign that says **"Welcome to Downtown (Population: 5)."**
    

In a B+ Tree, the internal "5" is the highway sign (navigation). The leaf "5" is the actual city (data).

---

## 4. The Real-World "Why": Range Queries

This is the ultimate reason B+ Trees won the database war.

In a standard tree, if you want to find "All students with IDs between 10 and 50," you have to constantly travel **Up** and **Down** the tree branches (In-order traversal). This is a nightmare for disk performance.

In a B+ Tree, because of the linked list at the bottom:

1. You search the tree **once** to find ID 10.
    
2. You land on the correct Leaf Node.
    
3. You simply **walk sideways** along the leaf linked list until you hit ID 50.
    

You never have to go back "Up" to an internal node again. For a student working on systems like game engines or microprocessors, you can recognize this as a **Linear Memory Access** pattern, which is significantly more cache-friendly and faster than jumping around tree pointers.

## Summary of the Design Logic

- **Internal Nodes:** Maximized for "Fan-out" to keep the tree short.
    
- **Leaf Nodes:** Maximized for "Data Storage" and "Sequential Access."
    
- **Duplicates:** The price we pay to keep the Index Layer small and the Data Layer perfectly ordered for range scans.

## Constraints
## 1. Determining the Separator

The "Separator" is the key in an internal node that acts as a fence. In a B+ Tree, we follow the **Successor Rule**: the separator is typically the smallest key in the right subtree.

### For $m=4$ (Max 3 keys, 4 pointers):

When a node overflows (you try to put a 4th key into a node that can only hold 3), we split at the **middle index** $i = \lceil m/2 \rceil$.

If we have keys $\{K_1, K_2, K_3, K_4\}$:

1. **Split Point:** $\lceil 4/2 \rceil = 2$.
    
2. **Left Node:** Gets the first 2 keys $\{K_1, K_2\}$.
    
3. **Right Node:** Gets the remaining keys $\{K_3, K_4\}$.
    
4. **The Separator:** $K_3$ is chosen because it is the "boundary" that separates the left group from the right group.
    
    - **In a Leaf Split:** $K_3$ is **copied** to the parent (it stays in the leaf because leaves must contain all data).
        
    - **In an Internal Split:** $K_3$ is **pushed** to the parent (it leaves the current level because internal nodes are just road signs).
        

---

## 2. B+ Tree Constraints & Derivations

In your studies at BITS Pilani, you'll see these rules enforced to ensure the tree never "degrades" into a slow linked list.

### Constraint Table

| **Component**      | **Constraint (Order m)**                     | **Reason**                                                 |
| ------------------ | -------------------------------------------- | ---------------------------------------------------------- |
| **Root**           | At least 2 children (if not a leaf).         | Prevents the tree from having a useless "single-path" top. |
| **Internal Nodes** | $\lceil m/2 \rceil - 1 \leq children \leq m$ | Ensures the tree is at least 50% full (space efficiency).  |
| **Leaf Nodes**     | $\lceil (m-1)/2 \rceil \leq keys \leq m-1$   | Keeps data density high and reduces total tree height.     |
| **Depth**          | All leaves must be at depth $h$.             | Guarantees that every search takes the exact same time.    |

---

### 3. Derivation of Height ($h$)

Why is a B+ Tree so fast? We can derive the maximum height to prove its efficiency.

Let $N$ be the total number of records (keys in leaves) and $m$ be the order.

To find the **maximum height** (the worst-case scenario), we assume every node is at its **minimum capacity**.

1. **Level 0 (Root):** Has at least 2 children.
    
2. **Level 1:** Each child has at least $\lceil m/2 \rceil$ children. Total = $2 \cdot \lceil m/2 \rceil$.
    
3. **Level 2:** Total = $2 \cdot \lceil m/2 \rceil^2$.
    
4. **Level $h-1$ (Leaf Level):** Total leaf nodes $L \geq 2 \cdot \lceil m/2 \rceil^{h-2}$.
    

Since each leaf node contains at least $\lceil (m-1)/2 \rceil$ keys, the total keys $N$ is:

$$N \geq 2 \cdot \lceil m/2 \rceil^{h-2} \cdot \lceil (m-1)/2 \rceil$$

Solving for $h$:

$$h \leq \log_{\lceil m/2 \rceil} \left( \frac{N}{2 \cdot \text{min\_keys\_in\_leaf}} \right) + 2$$

**The Result:** Because $m$ (the base of the log) is usually large (like 100 or 200 in real databases), the height $h$ stays incredibly small (usually 3 or 4) even for millions of records.

---

### 4. The Logic Behind the Constraints

- **Why the 50% Fill Rule?** If we allowed nodes to have only 1 key, the tree would become very "skinny" and tall, increasing disk I/O. By forcing nodes to be at least half-full, we force the tree to stay "fat" and short.
    
- **Why All Leaves at the Same Depth?** In a standard Binary Search Tree, if you insert sorted data, it becomes a line. In a B+ Tree, the only way to increase height is to split the **root**. Since every path starts from the root and goes to a split leaf, all paths stay equal.
    

As you're looking into game engine architecture, think of this as **constant-time complexity** for your asset lookups. Whether you have 1,000 or 1,000,000 items, the "cost" to find one is almost identical.

---

## 4) B+ tree queries (how a lookup works)

### Equality search: `find(k)`
1. Start at root page in buffer pool.
2. Binary search within the node to choose a pointer.
3. Repeat until leaf.
4. In leaf, binary search for `k`. If found → return row/pointer.

### Range query: `k BETWEEN a AND b`
1. Find the first leaf entry for `a`.
2. Then walk leaf chain sequentially until `b`.

That leaf chain is why B+ trees are great for ranges.

### MySQL examples
```sql
-- equality
SELECT * FROM accounts WHERE account_id = 123;

-- range
SELECT * FROM accounts
WHERE account_id BETWEEN 100 AND 200
ORDER BY account_id;
```

To see if MySQL uses an index:
```sql
EXPLAIN SELECT * FROM accounts WHERE account_id BETWEEN 100 AND 200;
```

---

## 5) Insertions in a B+ tree (what “split” really means)

### Insertion algorithm (theory)
1. Find target leaf where key belongs.
2. If leaf has space: insert key in sorted order.
3. If leaf full:
   - split the leaf into two leaves
   - redistribute keys roughly half-half
   - copy up (or push up) a separator key into parent
4. Parent may overflow → split internal node → propagate upward.
5. Root split increases height by 1.

### What “overhead” means
- Inserts may cause **page splits**, which cost extra IO and can fragment pages.
- Deletes may cause **merges** or **redistribution** (more overhead).

### MySQL mapping
InnoDB does B+ tree page splits. You’ll notice overhead when:
- inserting random keys into a primary key (like UUIDs)
- heavy write workloads
- poor fill factor due to random inserts

**Good practice:** Use monotonic increasing primary keys (`BIGINT AUTO_INCREMENT`) when possible to reduce splits.

Example:
```sql
CREATE TABLE t (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  val VARCHAR(100),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  KEY (created_at)
) ENGINE=InnoDB;
```

Insert pattern:
- Insert by auto_increment → mostly appends near the “rightmost” leaf → fewer splits.
- Insert random `id` values → causes inserts all over the tree → more splits.

---

## 6) Deletions in a B+ tree

### What can go wrong after delete
After deleting an entry, a page might become **underfull** (less than minimum occupancy).

Two approaches:

**A) Practical (“lazy”) deletion**
- Many systems tolerate some underfull pages for performance.
- Rebalancing happens less aggressively.

**B) Formal deletion**
If underfull:
1. Try to **borrow** from a sibling (redistribute keys).
2. If can’t borrow, **merge** with a sibling.
3. Update parent separator keys accordingly.
4. This may propagate upward if parent becomes underfull.

### MySQL note
InnoDB doesn’t necessarily immediately merge pages on delete; it may keep free space for future inserts. That’s why tables can become bloated and why `OPTIMIZE TABLE` (or rebuild) sometimes helps.

### B+ Tree Deletion: The Visual Mechanics

#### 1. Case: Simple Deletion

When a leaf node has enough keys to stay above the minimum ($min\_keys = \lceil (m-1)/2 \rceil$ for leaves), we just pull the key out. The index remains stable.

- **Constraint:** If the deleted key was a "separator" in a parent, we update that parent with the new **In-order Successor**.
    

#### 2. Case: Borrowing from Sibling (Redistribution)

When a node drops  below the minimum, it checks its neighbor. If the neighbor is "rich" (has more than the minimum), we perform a rotation.

- **The Move:** A key moves from the sibling to the parent, and the parent's old separator moves down to the "hungry" node.
    
- **The Result:** The tree height stays the same, and both nodes are now balanced.
    

#### 3. Case: Merging (The "Collapse" Case)

When a node underflows and its neighbor is also at the minimum, we cannot borrow. We must merge them into one.

- **The Move:** Two nodes become one.
    
- **The Consequence:** Because we have one less node, the parent **must lose a separator**.
    
- **The Chain Reaction:** If the parent now has 0 keys, it must borrow from _its_ sibling or merge again. If this happens at the root, the tree height decreases by 1.
    

---

### Step-by-Step Solution for image_65083f.png

Here is the trace of your specific problem (Order $m=4$, Min Keys = 2 for leaves, 1 for internal).

**Initial Tree Structure:**

- **Root:** `[9]`
    
- **Left Internal:** `[3, 5, 7]` $\rightarrow$ Leaves: `[1, 2]`, `[3, 4]`, `[5, 6]`, `[7, 8]`
    
- **Right Internal:** `[11]` $\rightarrow$ Leaves: `[9, 10]`, `[11, 12]`
    

#### 1. Delete 9

1. **Leaf `[9, 10]`** $\rightarrow$ `[10]` (**Underflow**).
    
2. **Merge:** Right sibling `[11, 12]` is at minimum. Combine them into `[10, 11, 12]`.
    
3. **Internal Adjustment:** The separator `11` is deleted from the parent. The parent `[11]` is now empty `[]` (**Internal Underflow**).
    
4. **Rebalance:** The empty internal node borrows from its left sibling `[3, 5, 7]`. 7 moves to the root; the old root (updated to successor 10) moves down.
    

**Structure After Deleting 9:**

Plaintext

```
          [7]
        /     \
    [3, 5]     [10]
   /  |  \    /    \
[1,2][3,4][5,6] [7,8] [10,11,12]
```

#### 2. Delete 7

1. **Leaf `[7, 8]`** $\rightarrow$ `[8]` (**Underflow**).
    
2. **Borrow:** Right sibling `[10, 11, 12]` has 3 keys. It gives **10** to the left.
    
3. **Parent Update:** The right internal separator becomes **11**.
    
4. **Root Update:** The deleted root key 7 is replaced by its successor **8**.
    

**Structure After Deleting 7:**

Plaintext

```
          [8]
        /     \
    [3, 5]     [11]
   /  |  \    /    \
[1,2][3,4][5,6] [8,10] [11,12]
```

#### 3. Delete 8

1. **Leaf `[8, 10]`** $\rightarrow$ `[10]` (**Underflow**).
    
2. **Merge:** Sibling `[11, 12]` is at minimum. Merge into `[10, 11, 12]`.
    
3. **Internal Adjustment:** Right internal node becomes empty `[]`.
    
4. **Rebalance:** Borrows from left sibling `[3, 5]`. Key **5** moves to the root; key **10** moves down.
    

**Final Tree Structure:**

Plaintext

```
          [5]
        /     \
      [3]      [10]
     /   \    /    \
  [1,2] [3,4][5,6] [10,11,12]
```

---

## 7) Bulk loading a B+ tree (why it’s faster)

### Problem
If you insert 1 billion rows one-by-one into an empty B+ tree:
- lots of random page splits
- lots of parent updates
- lots of IO

### Bulk load idea
1. Sort all entries by key.
2. Fill leaf pages sequentially to desired fill factor.
3. Build internal levels above it bottom-up.

### MySQL equivalent
- Loading data in primary key order is faster.
- Using `LOAD DATA INFILE` is faster than many single inserts.
- Disabling secondary indexes temporarily is possible in some engines; in InnoDB you can still benefit by creating indexes **after** loading.

Example pattern:
```sql
-- 1) create table without secondary indexes
CREATE TABLE events (
  id BIGINT PRIMARY KEY,
  ts DATETIME,
  user_id BIGINT,
  payload JSON
) ENGINE=InnoDB;

-- 2) bulk load
LOAD DATA INFILE '/path/events.csv'
INTO TABLE events
FIELDS TERMINATED BY ','
LINES TERMINATED BY '\n';

-- 3) then add indexes (build in bulk)
CREATE INDEX idx_events_ts ON events(ts);
CREATE INDEX idx_events_user ON events(user_id);
```

---

## 8) RAID levels: what they change for databases

RAID is about turning multiple disks into a single logical “disk array” for:
- performance (striping)
- redundancy (mirroring/parity)
- availability (survive disk failure)

### Terminology
- **Striping**: split data across disks → parallel IO
- **Mirroring**: duplicate data on another disk → redundancy
- **Parity**: extra info to reconstruct lost data if a disk fails (cheaper than mirroring)

### RAID 0 (striping only)
- Fast
- **No redundancy**
- 1 disk dies → you lose everything

**DB use:** almost never acceptable alone for real data.

### RAID 1 (mirroring)
- Writes go to both mirrors
- Reads can be balanced
- Great reliability
- Often great for random reads; writes are roughly “2x” but can be optimized by controller

**DB use:** common, especially for logs / critical data.

### RAID 5 (striping + distributed parity)
- Good capacity efficiency
- Reads are good
- Writes have a parity overhead (“read-modify-write” penalty for small random writes)
- Rebuild after failure is expensive and performance-degrading

**DB use:** good for **mostly-read / low-update** large datasets; less good for heavy OLTP random writes.

### RAID 6 (double parity)
- Survives 2 disk failures
- More write overhead than RAID 5

**DB use:** when reliability matters with many disks, but performance costs.

### RAID 0+1 vs RAID 10 (common confusion)
- **RAID 10 (1+0)**: mirror pairs, then stripe across pairs. Better fault tolerance than 0+1 typically.
- Many DBAs prefer RAID 10 for write-heavy OLTP.

### MySQL practical point (modern reality)
On cloud / SSD / NVMe with replication, people often rely more on:
- replication (MySQL replication / Group Replication)
- filesystem / cloud block storage redundancy
than classic hardware RAID. But the tradeoffs (write amplification, rebuild pain) still matter conceptually.

---

## 9) Putting it together in MySQL: why indexes + storage matter

### Example: bank accounts table
```sql
CREATE TABLE accounts (
  account_id BIGINT PRIMARY KEY,
  balance_cents BIGINT NOT NULL,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB;
```

- Primary key B+ tree leaf contains the row.
- Lookup by `account_id` is one B+ tree traversal.

### Why range scans are fast
```sql
SELECT * FROM accounts
WHERE account_id BETWEEN 100000 AND 200000;
```
This becomes: go to leaf for 100000 → walk leaf chain.

---

## 10) About your “Timestamp protocol + Thomas’ write rule” note in the sheet
You included that earlier; just to connect:

- B+ trees / RAID are storage & indexing.
- Timestamp ordering / Thomas’ write rule are concurrency control.

They solve different problems:
- B+ trees: “find data fast”
- RAID: “store data safely/fast on disks”
- concurrency control: “correct results with multiple users”

If you want, I can also teach the **timestamp protocol exercise (T19/T20/T21)** in the same “step-by-step table with R_TS/W_TS” style, but you didn’t paste the exact schedule lines—only the conclusion.

---

Below is a **complete, detailed teaching example** with **two databases**, each with **three tables**, and a **tree‑based (DAG) locking model**, followed by **6 schedules** showing where **2PL fails/works** and where **Tree (Graph‑based) protocol works/fails**.

I’ll keep it precise and illustrative.

---

# DATA MODEL (2 databases × 3 tables each)

## **DB1: SalesDB**
- **Customers** (10 rows: C1…C10)
- **Orders** (10 rows: O1…O10)
- **Payments** (10 rows: P1…P10)

## **DB2: HRDB**
- **Employees** (10 rows: E1…E10)
- **Departments** (10 rows: D1…D10)
- **Salaries** (10 rows: S1…S10)

Each table has at least 10 rows; think of “row i” as the lockable data item.

---

# ✅ TREE/DAG FOR TREE PROTOCOL

We define a DAG where **database is parent of tables**, and **tables are parent of rows**. (This is the standard hierarchical DAG.)

```
Root
├── SalesDB
│   ├── Customers
│   │   ├── C1 ... C10
│   ├── Orders
│   │   ├── O1 ... O10
│   └── Payments
│       ├── P1 ... P10
└── HRDB
    ├── Employees
    │   ├── E1 ... E10
    ├── Departments
    │   ├── D1 ... D10
    └── Salaries
        ├── S1 ... S10
```

**Tree Protocol rule:** after you lock a node, you may only lock its **children (descendants)**.

---

# ✅ 6 SCHEDULE CASES (2PL vs Tree Protocol)

I’ll use short notation:

- `Lx(Ti, X)` = exclusive lock on X by Ti  
- `Ux(Ti, X)` = unlock  
- `R(Ti, X)` / `W(Ti, X)` = read/write  

---

## ✅ CASE 1 — 2PL FAILS (deadlock), Tree Protocol PREVENTS IT
**Scenario:** classic deadlock across two rows in two tables.

### Schedule (2PL)
```
T1: Lx(Orders.O1)
T2: Lx(Payments.P1)
T1: Lx(Payments.P1)   -- waits
T2: Lx(Orders.O1)     -- waits
```
✅ 2PL allows this → **deadlock**

### Under Tree Protocol
T1 must lock parent before child.  
If it started at Orders table, it cannot jump to Payments unless it first locked SalesDB and then Payments (valid path).  
T2 locked Payments first → cannot lock Orders later because Orders is **not a child** of Payments.

So **Tree Protocol blocks this schedule** → **deadlock avoided**.

---

## ✅ CASE 2 — 2PL WORKS (serializable), Tree Protocol WORKS
Both transactions follow same top‑down order.

```
T1: Lx(SalesDB)
T1: Lx(Orders)
T1: Lx(Orders.O1) → W(O1)
T1: Ux(Orders.O1)

T2: Lx(SalesDB)
T2: Lx(Orders)
T2: Lx(Orders.O2) → W(O2)
T2: Ux(Orders.O2)

T1/T2 unlock rest
```

✅ 2PL works (serializable)  
✅ Tree protocol works (top‑down path)  
✅ No deadlock

---

## ✅ CASE 3 — 2PL WORKS, Tree Protocol FAILS (illegal path)
Transaction wants **Orders.O1 then Customers.C1**.

### Schedule
```
T1: Lx(SalesDB)
T1: Lx(Orders)
T1: Lx(Orders.O1) → W(O1)
T1: Lx(Customers.C1) → W(C1)
```

- Under 2PL this is legal (still growing).
- Under Tree protocol this is **illegal** because after locking `Orders`, you may only lock **children of Orders**, not move sideways to `Customers`.

**So Tree protocol rejects this schedule.**

---

## ✅ CASE 4 — Tree Protocol WORKS, Basic 2PL CAUSES CASCADE ROLLBACK RISK
T1 unlocks early (allowed in Tree protocol), T2 reads dirty data.

```
T1: Lx(SalesDB) → Lx(Customers) → Lx(C1)
T1: W(C1) 
T1: Ux(C1)        -- early unlock allowed in Tree protocol
T2: Lx(SalesDB) → Lx(Customers) → Lx(C1)
T2: R(C1)
T1: ABORT
```

- Tree protocol allows early unlocks → this schedule can happen.
- If T1 aborts, T2 read dirty data → **cascading rollback**.

**So Tree protocol allows schedules that are NOT recoverable.**

Under Strict 2PL this would be blocked (T1 would keep X lock).

---

## ✅ CASE 5 — 2PL FAILS (deadlock) across **databases**
T1 updates SalesDB row, then wants HRDB row.  
T2 does opposite.

```
T1: Lx(SalesDB.Customers.C1)
T2: Lx(HRDB.Employees.E1)

T1: Lx(HRDB.Employees.E1)   -- waits
T2: Lx(SalesDB.Customers.C1) -- waits
```

✅ 2PL allows → deadlock possible.

### Tree protocol?
If DAG is **Root → SalesDB → ...** and **Root → HRDB → ...**, then:
- If T1 starts in SalesDB, it **cannot later move to HRDB** (not a descendant).
- So Tree protocol blocks the schedule early → no deadlock.

---

## ✅ CASE 6 — 2PL WORKS, Tree Protocol WORKS, but Tree Protocol forces extra locks
Transaction needs **two rows in same table**.

```
T1: Lx(SalesDB) → Lx(Orders) → Lx(O1) → W(O1) → Ux(O1)
T1: Lx(O2) → W(O2) → Ux(O2)
```

✅ Tree protocol OK  
✅ 2PL OK  
But note: Tree protocol often **forces the parent locks** even if you only need rows → more overhead.

---

# ✅ SUMMARY OF WHEN EACH “WORKS” OR “FAILS”

| Case | 2PL | Tree Protocol |
|------|-----|---------------|
| 1 (deadlock classic) | ❌ possible deadlock | ✅ prevented |
| 2 (same order) | ✅ works | ✅ works |
| 3 (sideways move) | ✅ works | ❌ illegal |
| 4 (early unlock dirty read) | ✅ allowed (not strict) | ✅ allowed → can cascade |
| 5 (cross‑DB deadlock) | ❌ possible deadlock | ✅ prevented |
| 6 (same table rows) | ✅ works | ✅ works but extra locks |

---

# ✅ Key takeaways (clear & final)
- **2PL** guarantees serializability but can deadlock.
- **Tree protocol** guarantees serializability **and** deadlock freedom by forcing a fixed lock order.
- Tree protocol is **more restrictive**, so some schedules 2PL allows are **not possible**.
- Tree protocol can still allow **cascading rollbacks** because it allows early unlocks.
- In practice, DBMS use **2PL variants** + **deadlock detection** or **prevention**, not pure tree protocol.

---

If you want, I can also draw the **wait‑for graphs** for the deadlock cases or show the **exact precedence graphs** to prove serializability.

![[Introduction to Database Migrations]]