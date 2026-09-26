***(Check out [[Introduction to Database Migrations]], then [[Introduction to ORMs]] after this.)***
A **database** is just an organized place in a computer where related data is stored so it can be found, changed, and used efficiently later.[^5]

## What is data?

- Data is raw facts: numbers, text, dates, etc. (e.g., a user’s name, email, order amount).[^9]
- When you organize data in a useful structure (like a table with rows and columns), it becomes easy to search, filter, and update.[^7]


## What is a database?

- A **database** is an organized collection of related data, usually stored electronically on a computer.[^5]
- Most modern databases store data in tables (rows = records, columns = fields), similar to an Excel sheet but much more powerful.[^7]


## What is a DBMS?Step 5: Mapping of Binary M:N Relationship Types.

- A **Database Management System (DBMS)** is software that lets you create, store, modify, and retrieve data from a database. Examples: **MySQL**, PostgreSQL, Oracle.
- A DBMS handles storage, security, backup, concurrent access, and performance, so applications and users don’t touch raw files directly.


## What is a database system?

- A **database system** = database (data) + DBMS (software) + applications that use it (e.g., your web app, analytics scripts).
- When people casually say “database,” they often mean this whole system together.


## What is SQL and MySQL?

- **SQL** (Structured Query Language) is a language used to talk to relational databases: create tables, insert data, query, update, delete.
- **MySQL** is a specific relational DBMS that uses SQL; it’s one of the most widely used open‑source database systems in web apps.

# ![[Hosting and Navigating the MySQL servers and Databases]]


# The Syntax
### Conventions used:
- Keywords in syntax are shown in bold letters.
- Words in syntax between `‘<’` and `‘>’` signify the name of the table or column given by the user.
- Words in syntax between `‘[’` and `‘]’` are optional.

### NOTE: 
1) Every SQL statement ends with a semicolon.
2) SQL keywords are not case sensitive, but the data in the table and the name of the table are case sensitive.
3) It is a good practice to write the query with line separator to make error detection more
convenient.

# Types of MySQL Commands

There are three types of MySQL Commands:

## Data Definition Language (DDL) Commands:

This is working, changing and managing the defination of the data in the database.

### [[CREATE DATABASE;]]

### [[Check in MySQL]]

### [[SHOW DATABASE;]]

### [[USE;]]

### [[SHOW TABLES;]]

### [[CREATE TABLE;]]

### [[DESCRIBE]]

### [[ALTER TABLE Commands]]

### [[TRUNCATE TABLE]]

### [[DROP TABLE]]

### [[DROP DATABASE]]

### [[INDEX in MySQL]]

### [[Views in MySQL]]

### [[Assertions in MySQL]]

### [[Triggers in MySQL]]

### [[User Defined Functions in MySQL]]



- ## Data Manipulation Language (DML) Commands:

This is Manipulation the data that is there in the tables in the active database.

### [[INSERT INTO;]]

### [[UPDATE Command]]

### [[DELETE Command]]

### [[WHERE Command]]

- ## Data Query Language (DQL) Commands:

### [[Backend/Database and Structures/MySQL/SELECT]]

### [[ORDER BY clause in SELECT]]

### [[LIMIT Clause]]

### [[SET Operations in MySQL]]

### [[NESTED QUERIES in MySQL]]

### [[JOIN Operations in MySQL]]

### [[GROUP BY and HAVING Clause]]

### [[CASE Statements in MySQL]]

## Views




- ## Transition Control Language:



# ![[Data types in MySQL]]

## Aggregate Functions in MySQL and how they work:

### 1. What Are They?

An aggregate function performs a calculation on a set of values (multiple rows) and returns a **single value**.

#### The Core Functions:

- `COUNT()`: Returns the number of rows.
    
- `SUM()`: Returns the total sum of a numeric column.
    
- `AVG()`: Returns the average value of a numeric column.
    
- `MAX()`: Returns the highest value (works on numbers, text, dates).
    
- `MIN()`: Returns the lowest value.
    
- `GROUP_CONCAT()`: Concatenates values from multiple rows into a single string (MySQL specific).
    

### 2. Factor 1: How `GROUP BY` Affects Them (The Bucketing System)

`GROUP BY` dictates the "scope" of your aggregate function.

- **WITHOUT `GROUP BY`**: The entire table is treated as **one giant bucket**. You get exactly one row back.
    
    ```
    -- Returns ONE total sum for the entire company
    SELECT SUM(salary) FROM employees; 
    ```
    
- **WITH `GROUP BY`**: The table is split into distinct buckets based on the column(s) you specify. The aggregate function now calculates a result **for each bucket**.
    
    ```
    -- Returns ONE total sum PER department
    SELECT department, SUM(salary) FROM employees GROUP BY department;
    ```
    
- **The Golden Rule**: If you use an aggregate function alongside standard columns in your `SELECT` clause, those standard columns **must** be listed in the `GROUP BY` clause.
    

### 3. Factor 2: `NULL` Values (The Invisible Data)

This is where most bugs happen. **Aggregate functions ignore `NULL` values** (with one exception).

- `COUNT(*)`: Counts every row, **including** those with `NULL`s.
    
- `COUNT(column_name)`: Counts only rows where `column_name` is **NOT NULL**.
    
- **The `AVG()` Trap**: If you have 3 rows with salaries `[100, 200, NULL]`, `AVG(salary)` is `150` (300 / 2), NOT `100` (300 / 3). It completely ignores the row with the `NULL` value during both summation and division. If you want to treat `NULL` as `0`, you must use `COALESCE(salary, 0)`.

```mysql
SELECT AVG(COALESCE(salary, 0)) AS true_average
FROM employees;
```


### 4. Factor 3: `WHERE` vs. `HAVING` (Order of Execution)

Both filter data, but they interact with aggregates at entirely different stages.

- **`WHERE` (Pre-Aggregation filter)**: Filters individual rows **BEFORE** the data is grouped and aggregated.
    
    - _Rule_: You **cannot** use aggregate functions in a `WHERE` clause.
        
    
    ```
    -- CORRECT: Filters out part-time workers before summing salaries
    SELECT department, SUM(salary) FROM employees WHERE status = 'Full Time' GROUP BY department;
    ```
    
- **`HAVING` (Post-Aggregation filter)**: Filters groups **AFTER** the data has been grouped and aggregated.
    
    - _Rule_: This is the ONLY place you can filter based on the result of an aggregate function.
        
    
    ```
    -- CORRECT: Only shows departments where the total sum is > 100000
    SELECT department, SUM(salary) FROM employees GROUP BY department HAVING SUM(salary) > 100000;
    ```
    

### 5. Factor 4: The `DISTINCT` Keyword (Deduplication)

You can place `DISTINCT` inside an aggregate function to force it to only consider unique values.

- `COUNT(department)`: If you have 10 employees in 'Sales' and 5 in 'IT', this returns 15.
    
- `COUNT(DISTINCT department)`: This returns 2 (because there are only 2 unique departments).
    
- `SUM(DISTINCT salary)`: If three people make 50k, 50k, and 60k, the sum is 110k (it ignores the duplicate 50k). _Rarely used, but mechanically possible._
    

### 6. Factor 5: `WITH ROLLUP` (Subtotals and Grand Totals)

This is a MySQL modifier for `GROUP BY` that affects the output of aggregate functions by adding extra rows for super-aggregates.

```
SELECT department, SUM(salary) 
FROM employees 
GROUP BY department WITH ROLLUP;
```

**Effect**: If you have 'Sales' ($100) and 'IT' ($200), the output will show three rows:

1. Sales | 100
    
2. IT | 200
    
3. NULL | 300 <-- The `WITH ROLLUP` added this row representing the Grand Total.

## SQL CLAUSE CATEGORIES & EXECUTION ORDER

### MySQL Statement Types:

| Type                                   | Purpose             | Commands                               |
| -------------------------------------- | ------------------- | -------------------------------------- |
| **DDL** (Data Definition Language)     | Define structure    | `CREATE`, `ALTER`, `DROP`, `TRUNCATE`  |
| **DML** (Data Manipulation Language)   | Manipulate data     | `SELECT`, `INSERT`, `UPDATE`, `DELETE` |
| **DCL** (Data Control Language)        | Control access      | `GRANT`, `REVOKE`                      |
| **TCL** (Transaction Control Language) | Manage transactions | `COMMIT`, `ROLLBACK`, `SAVEPOINT`      |

---

## ALL SQL CLAUSES

### Complete SELECT Query Structure:


```mySQL

SELECT column1, column2, aggregate_function(column)
FROM table1
JOIN table2 ON condition
WHERE row_filter_condition
GROUP BY column
HAVING aggregate_filter_condition
ORDER BY column ASC|DESC
LIMIT number;
```

### Execution Order (How MySQL Processes Query):

| Step | Clause             | Purpose                    | Works On                       |
| ---- | ------------------ | -------------------------- | ------------------------------ |
| 1    | `FROM`             | Specify tables             | Tables                         |
| 2    | `JOIN`             | Combine tables             | Tables                         |
| 3    | `WHERE`            | **Filter individual rows** | **Rows (before grouping)**     |
| 4    | `GROUP BY`         | Group rows                 | Rows                           |
| 5    | `HAVING`           | **Filter grouped results** | **Groups (after aggregation)** |
| 6    | `SELECT`           | Choose columns             | Result set                     |
| 7    | `DISTINCT`         | Remove duplicates          | Result set                     |
| 8    | `ORDER BY`         | Sort results               | Result set                     |
| 9    | `LIMIT` / `OFFSET` | Limit rows returned        | Result set                     |


# Operators in MySQL:

## MySQL Operators

MySQL operators are symbols used to perform operations on data in queries. They help filter, compare, calculate, and manipulate data when retrieving or modifying database records.[^1][^2]

## Types of Operators

### Comparison Operators

Used to compare values and return TRUE or FALSE:[^1]

- `=` Equal to
- `>` Greater than
- `<` Less than
- `>=` Greater than or equal to
- `<=` Less than or equal to
- `<>` or `!=` Not equal to

**Example:**

```sql
SELECT name, percentage FROM students WHERE percentage > 90.6;
SELECT id, name FROM students WHERE hostel != 10;
```


### Arithmetic Operators

Used for mathematical calculations:[^2]

- `+` Addition
- `-` Subtraction
- `*` Multiplication
- `/` Division
- `%` Modulus (remainder)

**Example:**

```sql
SELECT percentage + 10.0 FROM students WHERE hostel = 10;
```


### Logical Operators

Used to combine multiple conditions:[^2][^1]

- `AND` - TRUE if all conditions are true
- `OR` - TRUE if any condition is true
- `NOT` - Reverses the result

**Example:**

```sql
SELECT id, name FROM students WHERE percentage > 90.6 AND hostel = 10;
SELECT id, name FROM students WHERE percentage = 60.0 OR percentage = 90.6;
SELECT id, name FROM students WHERE percentage > 90.6 AND NOT hostel = 10;
```


### Special Comparison Operators

**BETWEEN** - Checks if value is within a range:[^3]

```sql
SELECT id, name FROM students WHERE percentage BETWEEN 60.0 AND 90.6;
```

This is equivalent to `percentage >= 60.0 AND percentage <= 90.6`.[^3][^4]

**IN** - Checks if value matches any in a list:[^3]

```sql
SELECT * FROM students WHERE hostel IN (10, 11, 13);
```

**LIKE** - Pattern matching with wildcards:[^4][^3]

- `%` matches any sequence of characters (including zero)
- `_` matches exactly one character

**Examples:**

```sql
SELECT * FROM students WHERE name LIKE 'b%';  -- names starting with 'b'
SELECT * FROM students WHERE name LIKE '%fy';  -- names ending with 'fy'
SELECT * FROM students WHERE name LIKE '%w%';  -- names containing 'w'
SELECT * FROM students WHERE name LIKE '_____';  -- names with exactly 5 characters
SELECT * FROM students WHERE percentage LIKE '%!%%';
```

**IS NULL / IS NOT NULL** - Checks for NULL values:

```sql
SELECT * FROM students WHERE phone IS NULL;
SELECT * FROM students WHERE phone IS NOT NULL;
```



# Aliases and DOT Operator

## 1. WHAT ARE ALIASES?

An **alias** is a **temporary name** (nickname) you give to a table or column **for the duration of a query**. It doesn't change the actual table/column name in the database.

### Types of Aliases:
1. **Table Aliases** - Shorthand names for tables
2. **Column Aliases** - Rename columns in output

---
## 2. AS Keyword (Aliases) / COLUMN ALIASES

The `AS` keyword creates temporary names (aliases) for columns or tables in query results. It makes queries more readable, especially when using calculations or functions.[^5]

### Column Aliases

**Basic syntax:**

```sql
SELECT column_name AS alias_name FROM table_name;
```

**Examples:**

```sql
-- Rename a column
SELECT name AS student_name, percentage AS marks FROM students;

-- With calculations
SELECT percentage + 10.0 AS adjusted_percentage FROM students;

-- With functions
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM employees;
```

**Without AS keyword** (optional):

```sql
SELECT percentage + 10.0 product FROM students WHERE hostel = 10;
```

**With spaces in alias** (use quotes):

```sql
SELECT bookPrice + 500 AS 'New Price' FROM books;
```


### Table Aliases

Useful when joining multiple tables:

```sql
SELECT s.name, s.percentage 
FROM students AS s 
WHERE s.hostel = 10;
```


## 3. TABLE ALIASES (The Confusing Part!)

### Syntax: `AS` is OPTIONAL for table aliases!

```sql
-- Method 1: Without AS (most common)
SELECT e.name, e.salary
FROM employees e
WHERE e.salary > 70000;

-- Method 2: With AS (also valid, but rarely used)
SELECT e.name, e.salary
FROM employees AS e
WHERE e.salary > 70000;
```

**Both are identical!** Most developers skip `AS` for table aliases because it's shorter.

### Example Table:
```sql
-- employees
+----+--------+---------+--------+
| id | name   | dept_id | salary |
+----+--------+---------+--------+
| 1  | Alice  | 10      | 75000  |
| 2  | Bob    | 10      | 65000  |
| 3  | Carol  | 20      | 80000  |
+----+--------+---------+--------+
```

### Without Alias (Verbose):
```sql
SELECT employees.name, employees.salary
FROM employees
WHERE employees.salary > 70000;
```

### With Alias (Cleaner):
```sql
SELECT e.name, e.salary
FROM employees e
WHERE e.salary > 70000;
```

**What happened?**
- `employees e` means: "Call the employees table 'e' in this query"
- Now you use `e.name` instead of `employees.name`
- **The dot operator (`.`) connects the alias to the column**

**Result:**
```
+-------+--------+
| name  | salary |
+-------+--------+
| Alice | 75000  |
| Carol | 80000  |
+-------+--------+
```

---

## 4. COLUMN ALIASES (AS is REQUIRED or use space)

For columns, you **should use** `AS` (though technically optional with space).

### Syntax:
```sql
-- Method 1: With AS (recommended, clearest)
SELECT name AS employee_name, salary AS annual_income
FROM employees;

-- Method 2: Without AS (space separation, less clear)
SELECT name employee_name, salary annual_income
FROM employees;

-- Method 3: No alias (original column names)
SELECT name, salary
FROM employees;
```

**Result (Methods 1 & 2 - same output):**
```
+---------------+---------------+
| employee_name | annual_income |
+---------------+---------------+
| Alice         | 75000         |
| Bob           | 65000         |
| Carol         | 80000         |
+---------------+---------------+
```

**Result (Method 3):**
```
+-------+--------+
| name  | salary |
+-------+--------+
| Alice | 75000  |
| Bob   | 65000  |
| Carol | 80000  |
+-------+--------+
```

---

## 5. TABLE ALIAS + DOT OPERATOR (How They Work Together)

### The Pattern:
```
alias.column_name
```

### Step-by-Step Breakdown:

```sql
SELECT e.name, e.salary
FROM employees e
WHERE e.dept_id = 10;
```

**What happens:**

1. **`FROM employees e`**
   - MySQL reads the `employees` table
   - Creates temporary alias `e` → points to `employees`
   - For this query only, `e` = `employees`

2. **`SELECT e.name, e.salary`**
   - `e.` tells MySQL: "Look in the table aliased as 'e'"
   - `.name` means: "Get the 'name' column from that table"
   - `e.name` = "Get name from employees"

3. **`WHERE e.dept_id = 10`**
   - `e.dept_id` = "Get dept_id from employees"
   - Filter rows where dept_id equals 10

**Visual Representation:**
```
employees table                  Alias 'e'
+----+-------+------+            ┌─────────────┐
| id | name  | ...  |   ───────> │ e.id        │
| 1  | Alice | ...  |            │ e.name      │
| 2  | Bob   | ...  |            │ e.dept_id   │
+----+-------+------+            │ e.salary    │
                                 └─────────────┘
                                       ↓
                               SELECT e.name, e.salary
```

---

## 6. WHY USE ALIASES? (Real Examples)

### A. Multiple Tables with Same Column Names

```sql
-- departments
+----+--------+
| id | name   |
+----+--------+
| 10 | IT     |
| 20 | Sales  |
+----+--------+

-- employees  
+----+--------+---------+
| id | name   | dept_id |
+----+--------+---------+
| 1  | Alice  | 10      |
| 2  | Bob    | 20      |
+----+--------+---------+
```

#### ❌ Without Aliases (AMBIGUOUS):
```sql
SELECT id, name
FROM employees
JOIN departments ON employees.dept_id = departments.id;
-- ERROR 1052: Column 'id' is ambiguous
-- ERROR 1052: Column 'name' is ambiguous
```

**Problem:** Both tables have `id` and `name` columns. MySQL doesn't know which one you want!

#### ✅ With Aliases (CLEAR):
```sql
SELECT 
    e.id AS employee_id,
    e.name AS employee_name,
    d.id AS department_id,
    d.name AS department_name
FROM employees e
JOIN departments d ON e.dept_id = d.id;
```

**Breakdown:**
- `employees e` → Create alias `e` for employees
- `departments d` → Create alias `d` for departments
- `e.id` → Get id from employees table (via alias e)
- `d.id` → Get id from departments table (via alias d)
- `e.name` → Get name from employees
- `d.name` → Get name from departments

**Result:**
```
+-------------+---------------+---------------+-----------------+
| employee_id | employee_name | department_id | department_name |
+-------------+---------------+---------------+-----------------+
| 1           | Alice         | 10            | IT              |
| 2           | Bob           | 20            | Sales           |
+-------------+---------------+---------------+-----------------+
```

### B. Self-Joins (Same Table Twice)

```sql
-- employees with manager_id
+----+--------+------------+--------+
| id | name   | manager_id | salary |
+----+--------+------------+--------+
| 1  | Alice  | NULL       | 90000  |
| 2  | Bob    | 1          | 70000  |
| 3  | Carol  | 1          | 75000  |
| 4  | David  | 2          | 60000  |
+----+--------+------------+--------+
```

#### Goal: Show each employee with their manager's name

```sql
SELECT 
    e.name AS employee,
    m.name AS manager,
    e.salary
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

**What's happening:**
- `employees e` → First copy of table (employees)
- `employees m` → Second copy of same table (managers)
- `e.name` → Employee's name (from first copy)
- `m.name` → Manager's name (from second copy)
- `e.manager_id = m.id` → Connect employee to their manager

**Visual:**
```
employees (as 'e')          employees (as 'm')
+----+-------+-----+        +----+-------+
| id | name  | mgr |   →    | id | name  |
+----+-------+-----+        +----+-------+
| 2  | Bob   | 1   | ────→  | 1  | Alice |
| 3  | Carol | 1   | ────→  | 1  | Alice |
| 4  | David | 2   | ────→  | 2  | Bob   |
+----+-------+-----+        +----+-------+

e.manager_id = m.id connects them
```

**Result:**
```
+----------+---------+--------+
| employee | manager | salary |
+----------+---------+--------+
| Alice    | NULL    | 90000  |
| Bob      | Alice   | 70000  |
| Carol    | Alice   | 75000  |
| David    | Bob     | 60000  |
+----------+---------+--------+
```

### C. Shorter, Cleaner Code

```sql
-- Without aliases (painful to type and read)
SELECT 
    employee_information.employee_full_name,
    employee_information.annual_salary,
    department_information.department_name
FROM employee_information
JOIN department_information 
    ON employee_information.department_id = department_information.department_id;

-- With aliases (much cleaner!)
SELECT 
    ei.employee_full_name,
    ei.annual_salary,
    di.department_name
FROM employee_information ei
JOIN department_information di 
    ON ei.department_id = di.department_id;
```

---

## 6. ALIASES IN NESTED QUERIES (Critical Understanding)

### Example: Correlated Subquery

```sql
-- Find employees earning more than their department's average

SELECT e1.name, e1.dept_id, e1.salary
FROM employees e1
WHERE e1.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.dept_id = e1.dept_id
);
```

### Let's Dissect This:

**Step 1: Create aliases**
```sql
FROM employees e1  -- Outer query: alias = e1
FROM employees e2  -- Inner query: alias = e2
```

Both reference the **same table** but need **different names** to distinguish them.

**Step 2: Understand the dot operator**
```
e1.dept_id  →  "dept_id from the OUTER query's current row"
e2.dept_id  →  "dept_id from the INNER query's rows"
```

**Step 3: Execution for Alice (dept_id=10, salary=75000)**

```sql
-- Outer query is processing this row:
e1 = {id: 1, name: 'Alice', dept_id: 10, salary: 75000}

-- Inner query executes:
SELECT AVG(e2.salary)
FROM employees e2
WHERE e2.dept_id = e1.dept_id
--    ^^^^^^^^^^^   ^^^^^^^^^^^
--    Column from    Value from outer query (10)
--    inner table
```

**Inner query becomes:**
```sql
SELECT AVG(e2.salary)
FROM employees e2
WHERE e2.dept_id = 10  -- e1.dept_id was 10
```

**Filters inner table:**
```
+----+-------+---------+--------+
| id | name  | dept_id | salary |
+----+-------+---------+--------+
| 1  | Alice | 10      | 75000  | ← Included
| 2  | Bob   | 10      | 65000  | ← Included
+----+-------+---------+--------+
```

**Calculate AVG:** (75000 + 65000) / 2 = 70,000

**Compare:** 75000 > 70000? **YES** → Alice included ✓

---

## 7. DOT OPERATOR: THE BRIDGE BETWEEN ALIAS AND COLUMN

### The Dot Operator Does 3 Things:

1. **Specifies which table** (when multiple tables exist)
2. **Resolves ambiguity** (when columns have same name)
3. **Creates correlation** (in nested queries)

### Syntax Breakdown:

```
[alias].[column_name]
   ↓         ↓
  Which    Which
  table    column
```

### Examples:

```sql
e.name
↓  ↓
│  └─ Column name
└─ Table alias (points to employees)

e1.dept_id
↓↓  ↓↓↓↓↓↓
││  └─ Column name
│└─ Instance number (first reference to employees)
└─ Base alias letter
```

---

## 8. COMPLETE EXAMPLE: Everything Together

```sql
-- employees
+----+--------+---------+--------+
| id | name   | dept_id | salary |
+----+--------+---------+--------+
| 1  | Alice  | 10      | 75000  |
| 2  | Bob    | 10      | 65000  |
| 3  | Carol  | 20      | 80000  |
| 4  | David  | 20      | 60000  |
+----+--------+---------+--------+

-- departments
+----+--------+----------+
| id | name   | budget   |
+----+--------+----------+
| 10 | IT     | 500000   |
| 20 | Sales  | 300000   |
+----+--------+----------+

-- Query: Find employees earning more than dept average, show dept info
SELECT 
    e.name AS employee_name,           -- Column alias
    e.salary AS employee_salary,       -- Column alias
    d.name AS department_name,         -- Column alias
    d.budget AS department_budget,     -- Column alias
    (SELECT AVG(e2.salary) 
     FROM employees e2 
     WHERE e2.dept_id = e.dept_id) AS dept_avg_salary  -- Column alias
FROM employees e                       -- Table alias (no AS)
INNER JOIN departments d               -- Table alias (no AS)
    ON e.dept_id = d.id
WHERE e.salary > (
    SELECT AVG(e3.salary)
    FROM employees e3
    WHERE e3.dept_id = e.dept_id
)
ORDER BY d.name, e.salary DESC;
```

### Breaking Down the Aliases:

| Alias | Type | Refers To | Used For |
|-------|------|-----------|----------|
| `e` | Table | employees (outer query) | Main employee data |
| `d` | Table | departments | Department info |
| `e2` | Table | employees (subquery 1) | Calculate dept average for SELECT |
| `e3` | Table | employees (subquery 2) | Calculate dept average for WHERE |
| `employee_name` | Column | e.name | Rename output column |
| `employee_salary` | Column | e.salary | Rename output column |
| `department_name` | Column | d.name | Rename output column |
| `dept_avg_salary` | Column | Subquery result | Rename calculated column |

### Dot Operator Usage:

```sql
e.name           → Get 'name' from employees (alias e)
e.salary         → Get 'salary' from employees (alias e)
e.dept_id        → Get 'dept_id' from employees (alias e)
d.name           → Get 'name' from departments (alias d)
d.id             → Get 'id' from departments (alias d)
e2.salary        → Get 'salary' from employees (alias e2, subquery)
e2.dept_id       → Get 'dept_id' from employees (alias e2, subquery)
e3.dept_id       → Get 'dept_id' from employees (alias e3, different subquery)
```

### Why Different Aliases (e, e2, e3)?

```sql
employees e      → Outer query, processes each employee
employees e2     → First subquery, calculates average for SELECT
employees e3     → Second subquery, calculates average for WHERE
```

**They're all the same table**, but:
- Need different names to distinguish them
- Each has its own "context" in the query
- The dot operator connects them: `e2.dept_id = e.dept_id`

**Result:**
```
+---------------+-----------------+-----------------+-------------------+-----------------+
| employee_name | employee_salary | department_name | department_budget | dept_avg_salary |
+---------------+-----------------+-----------------+-------------------+-----------------+
| Carol         | 80000           | Sales           | 300000            | 70000.00        |
| Alice         | 75000           | IT              | 500000            | 70000.00        |
+---------------+-----------------+-----------------+-------------------+-----------------+
```

---

## 9. RULES & BEST PRACTICES

### Table Aliases:

| Rule | Example | Notes |
|------|---------|-------|
| `AS` is optional | `FROM employees e` | Most common style |
| Can use `AS` | `FROM employees AS e` | Valid but verbose |
| Must be unique | `FROM emp e, emp e` ❌ | Can't reuse in same scope |
| Can't reference original | `WHERE employees.id = 1` ❌ | Must use `e.id` after aliasing |

**Important:** Once you create an alias, you **must use it**:

```sql
-- ❌ WRONG - Once aliased as 'e', can't use 'employees'
SELECT employees.name
FROM employees e
WHERE employees.salary > 70000;
-- ERROR 1054: Unknown column 'employees.name'

-- ✅ CORRECT - Use the alias
SELECT e.name
FROM employees e
WHERE e.salary > 70000;
```

### Column Aliases:

| Rule | Example | Notes |
|------|---------|-------|
| Use `AS` (recommended) | `SELECT name AS emp_name` | Clearest |
| Space works but confusing | `SELECT name emp_name` | Avoid |
| Quotes for spaces | `SELECT name AS "Employee Name"` | Use backticks in MySQL: \`Employee Name\` |
| Can't use in WHERE | `WHERE emp_name > 5` ❌ | Alias not available yet |
| Can use in ORDER BY | `ORDER BY emp_name` ✓ | Alias available here |

### WHERE vs ORDER BY with Column Aliases:

```sql
-- ❌ WRONG - Column alias not available in WHERE
SELECT salary * 12 AS annual_salary
FROM employees
WHERE annual_salary > 100000;
-- ERROR 1054: Unknown column 'annual_salary'

-- ✅ CORRECT - Use original expression in WHERE
SELECT salary * 12 AS annual_salary
FROM employees
WHERE salary * 12 > 100000;

-- ✅ ALSO CORRECT - Alias works in ORDER BY
SELECT salary * 12 AS annual_salary
FROM employees
WHERE salary * 12 > 100000
ORDER BY annual_salary DESC;  -- Alias OK here
```

**Why?** Execution order:
```
FROM → WHERE → SELECT (aliases created) → ORDER BY
```

---

## 10. VISUAL SUMMARY

### How It All Connects:

```
TABLE ALIAS (e)
     ↓
FROM employees e
     ↓
CREATE CONNECTION
     ↓
e.name  ←─ DOT OPERATOR connects alias to column
  ↓
SELECT e.name AS employee_name  ←─ COLUMN ALIAS for output
            ↓
       RESULT
    employee_name
    ─────────────
    Alice
    Bob
    Carol
```

### Nested Query Visualization:

```
OUTER: employees e1
       ↓
       e1.name, e1.salary, e1.dept_id
       ↓
WHERE e1.salary > (
       ↓
       INNER: employees e2
       ↓
       AVG(e2.salary)
       ↓
       WHERE e2.dept_id = e1.dept_id
             ↑↑↑↑↑↑↑↑↑   ↑↑↑↑↑↑↑↑↑
             Inner        Outer
             (filters)    (correlation)
       
       DOT OPERATOR creates the bridge!
)
```

---

## FINAL SUMMARY

| Concept | Syntax | Purpose |
|---------|--------|---------|
| **Table Alias** | `FROM employees e` or `FROM employees AS e` | Give table a short name |
| **Column Alias** | `SELECT name AS emp_name` | Rename output column |
| **Dot Operator** | `e.name` | Connect alias to column |
| **Why Alias?** | Clarity, brevity, necessity (self-joins, ambiguity) | Make queries readable |
| **AS keyword** | Optional for tables, recommended for columns | Clarity |

**Key Insight:** 
- **Alias** = the nickname
- **Dot operator** = how you use the nickname to access data
- Together they make complex queries manageable!

# CONSTRAINTS in MySQL:

## What are Constraints?

Constraints are **rules** you apply to columns in a table to control what kind of data can be stored. This is the logic that gets applied to the databases. 
They prevent invalid or unwanted data from entering your database and keep data accurate and consistent.

## Types of Constraints

Constraints enforce rules that columns in a table must follow, ensuring data integrity. There are three main categories:[^1]

### Domain Integrity Constraints

These set acceptable ranges and prevent operations that violate those rules.[^1]

- **NOT NULL**: Ensures a column cannot contain NULL values (by default, columns allow NULL)[^1]
- **CHECK**: Defines a specific range or condition that values must satisfy[^1]


### Entity Integrity Constraints

These ensure each record is uniquely identifiable.[^1]

- **UNIQUE**: Allows only unique values in a column or group of columns, preventing duplicate records, but permits NULL values[^1]
- **PRIMARY KEY**: Combines UNIQUE and NOT NULL—it prevents duplicates and doesn't allow NULL values. Cannot be used on LONG data type columns[^1]


### Referential Integrity Constraints

These enforce relationships between tables using foreign keys.[^1]

- **FOREIGN KEY**: Designates a column (or combination) as a foreign key that references a primary or unique key in another table. The table with the foreign key is the child table; the table with the referenced key is the parent table[^1]

## Domain Integrity Constraints:

### NOT NULL

Forces a column to **always have a value**—it cannot be empty (NULL).

```sql
CREATE TABLE students (
  id INT NOT NULL,
  name VARCHAR(30) NOT NULL
);
```

Here, every student **must** have an `id` and `name`; you can't insert a row without them.

### CHECK

Enforces a **condition** on column values—only values passing the condition are allowed.

```sql
CREATE TABLE students (
  age INT CHECK (age >= 18)
);
```

Only students aged 18 or older can be inserted.

**Using named CHECK constraint:**

```sql
CREATE TABLE students (
  age INT,
  CONSTRAINT age_check CHECK (age >= 18)
);
```

### DEFAULT
Sets a **default value** if no value is provided during INSERT.

```sql
CREATE TABLE students (
  percentage DECIMAL(5,2) DEFAULT 0.0
);
```

If you don't specify `percentage` when inserting a row, it automatically gets `0.0`.

## Entity Integrity Contraints:

### UNIQUE

Ensures **all values in a column are different**—no duplicates allowed.[^3][^1]

```sql
CREATE TABLE students (
  phone INT UNIQUE
);
```

No two students can have the same phone number.  You can have **multiple** UNIQUE constraints in a table.

Multiple `NULL` are allowed.

### PRIMARY KEY

A **combination of NOT NULL + UNIQUE**—uniquely identifies each row.  A table can have **only one** primary key.

```sql
CREATE TABLE students (
  id CHAR(12) PRIMARY KEY,
  name VARCHAR(30)
);
```

The `id` column cannot be NULL and must be unique for every student.  Other tables can reference this primary key using foreign keys.[^5][^9]

**Two ways to define PRIMARY KEY:**

```sql
-- Inline (at column level)
CREATE TABLE students (id CHAR(12) PRIMARY KEY, name VARCHAR(30));

-- At table level
CREATE TABLE students (
  id CHAR(12),
  name VARCHAR(30),
  PRIMARY KEY (id)
);
```

Both are identical.



## Using CONSTRAINT Keyword

You can **name** constraints with `CONSTRAINT` for easier management.

```sql
CREATE TABLE gradstudents (
  id CHAR(12),
  phone INT,
  CONSTRAINT ue UNIQUE (phone),
  PRIMARY KEY (id)
);
```

Here, `ue` is the name of the UNIQUE constraint on `phone`.  Named constraints make it easier to drop or modify them later.[^8]

## Complete Example
```sql
CREATE TABLE gradstudents (
  id CHARACTER(12),
  name VARCHAR(30),
  hostel INTEGER NOT NULL,
  percentage DECIMAL(5,2) DEFAULT 0.0,
  phone INT,
  bdate DATE,
  gender ENUM('F','M'),
  CONSTRAINT ue UNIQUE(phone),
  PRIMARY KEY(id)
);
```

**What each constraint does:**

- `hostel INTEGER NOT NULL` – hostel number is mandatory.[^10]
- `percentage DECIMAL(5,2) DEFAULT 0.0` – if no percentage given, defaults to 0.0.[^8]
- `CONSTRAINT ue UNIQUE(phone)` – phone numbers must be unique across all students.[^8]
- `PRIMARY KEY(id)` – `id` uniquely identifies each student, cannot be NULL or duplicate.

## Another Example:

TRY THIS (justify if you get errors):

```mysql
mysql> CREATE TABLE temp (id char(10), name varchar(30) unique, hostel int, percentage
decimal(5,2) default 0.0, phone VARCHAR(11), bdate date, gender enum('F','M'), primary
key(id), primary key(name) );
```

Correct method to create a composite primary key:

```mysql
mysql> CREATE TABLE temp (id char(10), name varchar(30) unique, hostel int, percentage
decimal(5,2) default 0.0, phone varchar(11), bdate date, gender enum('F','M'), primary
key(id,name) );
```

# Foreign Keys in MySQL

## Prerequisites Review

Since you know DQL, DML, and DDL commands, let's quickly establish the foundation:

- **DDL (Data Definition Language)**: CREATE, ALTER, DROP - defines database structure
- **DML (Data Manipulation Language)**: INSERT, UPDATE, DELETE - manipulates data
- **DQL (Data Query Language)**: SELECT - retrieves data

---

## 1. Understanding Constraints

**What are Constraints?**
Constraints are **rules** that columns in a table must follow. They enforce data integrity and prevent invalid data from entering your database.

### Types of Constraints:

#### A. **Domain Integrity Constraints** (Control what values are allowed)
1. **NOT NULL**: Column must have a value (no empty/null values)
2. **CHECK**: Column must meet specific conditions

#### B. **Entity Integrity Constraints** (Control uniqueness)
1. **UNIQUE**: No duplicate values allowed
2. **PRIMARY KEY**: No duplicates + No nulls (identifies each row uniquely)

#### C. **Referential Integrity Constraints** (Control relationships between tables)
1. **FOREIGN KEY**: Links tables together, maintains relationships

---

## What is FOREIGN KEY?

### **Definition:**
A **FOREIGN KEY** is a column (or group of columns) in one table that **references the PRIMARY KEY or UNIQUE KEY** of another table.

### **Purpose:**
- Maintains **referential integrity** (keeps relationships valid)
- Prevents orphaned records (child records without a parent)
- Enforces relationships between tables

## 2. What is INDEX? (Important Foundation)

### **What is an INDEX?**
An INDEX is like a **book's index** - it helps MySQL find data quickly without reading every row. Think of an **INDEX** in MySQL not as a "rule" (like a Key), but as a **physical data structure** created to speed up data retrieval.

"In value," an Index is a **sorted copy** of specific columns from your table, stored separately, with a "pointer" (address) that tells MySQL exactly where to find the full row.

### **Why is INDEX Required?**

**Without Index:**
```
To find student with ID=500:
MySQL scans Row 1, Row 2, Row 3... Row 500 ✓ (Slow!)
```

**With Index:**
```
MySQL directly jumps to Row 500 (Fast!)
```

### **Why INDEX is Required for Foreign Keys?**
When you use a FOREIGN KEY, MySQL needs to:
1. Check if the referenced value exists in the parent table (fast lookup needed)
2. Verify relationships during INSERT/UPDATE/DELETE operations

**InnoDB automatically creates an index on foreign key columns** to make these checks fast.

The relationship between **Foreign Keys** and **Indexes** is one of "Logical Rule" vs. "Physical Performance."

In MySQL (specifically the InnoDB engine), they are tightly connected because **MySQL requires an Index on every Foreign Key column.** If you don't create one manually, MySQL will secretly create one for you.

Here is why they are connected and how the index is actually used.

### 1. The "Integrity Check" (Why the database needs it)

The most critical use of an index on a Foreign Key is to prevent "locking" issues when you modify data.

Imagine two tables:

- **Parent:** `Users` (Primary Key: `UserID`)
    
- **Child:** `Orders` (Foreign Key: `UserID`)
    

**The Scenario:** You try to **DELETE** a user from the `Users` table.

- **Without an Index on `Orders.UserID`:** The database must verify that this user has no orders (Referential Integrity). Without an index, it has to scan the **entire** `Orders` table row-by-row to ensure `UserID 5` doesn't exist there. This locks the table and kills performance.
    
- **With an Index:** The database looks at the Index for `Orders`, jumps straight to "5", sees if any records exist, and instantly approves or rejects your DELETE command.
    

### 2. The "Join" Speed (Why you need it)

Foreign Keys are almost always used to `JOIN` tables together.

- **Query:** `SELECT * FROM Orders WHERE UserID = 5`
    
- This is the standard "Foreign Key lookup." The index acts exactly as described in the previous answer: it serves as a sorted lookup list so the database can find the connections instantly without scanning the whole table.

### **Example:**
```sql
CREATE TABLE parent (
    id INT NOT NULL,
    PRIMARY KEY (id)  -- Automatically creates an index on 'id'
) ENGINE=INNODB;

CREATE TABLE child (
    id INT,
    parent_id INT,
    INDEX par_ind (parent_id),  -- Index for fast lookups
    FOREIGN KEY (parent_id) REFERENCES parent(id)
) ENGINE=INNODB;
```

### "If I manually create an index, and then I add a Foreign Key on the same column, is MySQL smart enough to link them, or will it blindly create a second, duplicate index?"

The short answer is: **Yes, MySQL is smart enough.** It will detect your manual index and use it, provided it meets specific criteria.

Here is exactly how the "decision logic" works inside the MySQL engine (specifically InnoDB) when you add a Foreign Key.

### 1. The "Leftmost Prefix" Rule

When you try to create a Foreign Key on a column (let's say `UserID`), MySQL scans the table's existing indexes before creating a new one.

It doesn't just look for an exact match. It looks for any index where the **Foreign Key column is the first column listed (the "Leftmost Prefix").**

- **Scenario A (Exact Match):**
    
    - You create `INDEX my_custom_idx (UserID)`.
        
    - You add a Foreign Key on `UserID`.
        
    - **Result:** MySQL sees `my_custom_idx`, says "Perfect," and uses it. It creates **0** new indexes.
        
- **Scenario B (Composite Index Match):**
    
    - You create `INDEX complex_idx (UserID, Status, Date)`.
        
    - You add a Foreign Key on `UserID`.
        
    - **Result:** MySQL sees that `UserID` is the **first** column in `complex_idx`. It says "Good enough," and uses this index. It creates **0** new indexes.
        
- **Scenario C (No Match / Wrong Order):**
    
    - You create `INDEX backwards_idx (Status, UserID)`.
        
    - You add a Foreign Key on `UserID`.
        
    - **Result:** MySQL checks the index. It sees `Status` comes first, not `UserID`. This index cannot be used for efficient lookups on `UserID` alone. MySQL says "I can't use this," and **automatically creates a new, internal index** just for the Foreign Key.


---

## 4. Parent and Child Table Concept

### **What are Parent and Child Tables?**

- **Parent Table**: Contains the **original/master data** (has PRIMARY KEY)
- **Child Table**: Contains **dependent data** that references the parent (has FOREIGN KEY)

### **Relationship:**
```
Parent Table (PRIMARY KEY) ← Referenced by ← Child Table (FOREIGN KEY)
```

### **Example:**

```sql
-- PARENT TABLE (Customers)
CREATE TABLE customer (
    id INT NOT NULL,
    name VARCHAR(100),
    PRIMARY KEY (id)  -- This is the referenced key
) ENGINE=INNODB;

-- CHILD TABLE (Orders)
CREATE TABLE orders (
    order_no INT NOT NULL AUTO_INCREMENT,
    customer_id INT,  -- This is the FOREIGN KEY
    order_date DATE,
    PRIMARY KEY(order_no),
    INDEX (customer_id),
    FOREIGN KEY (customer_id) REFERENCES customer(id)
) ENGINE=INNODB;
```

**Visualization:**
```
CUSTOMER (Parent)          ORDERS (Child)
+----+---------+           +----------+-------------+
| id | name    |           | order_no | customer_id |
+----+---------+           +----------+-------------+
| 1  | Alice   | ←───────── | 101      | 1           |
| 2  | Bob     | ←───────── | 102      | 2           |
+----+---------+   │       | 103      | 1           |
                   └─────── +----------+-------------+
                  Foreign Key Relationship
```

---

## 5. Relationship Between FOREIGN KEY and PRIMARY KEY

### **The Golden Rule:**
A FOREIGN KEY in the child table **must match** a PRIMARY KEY (or UNIQUE KEY) in the parent table.

### **Key Points:**

1. **Data Types Must Match:**
   ```sql
   -- ✓ Correct
   Parent: id INT
   Child:  parent_id INT
   
   -- ✗ Wrong
   Parent: id INT
   Child:  parent_id VARCHAR(10)  -- Type mismatch!
   ```

2. **Values Must Exist in Parent:**
   ```sql
   -- If customer id=5 doesn't exist in customer table
   INSERT INTO orders (customer_id) VALUES (5);  -- ✗ REJECTED!
   ```

3. **Parent Table Must Exist First:**
   ```sql
   -- ✓ Correct Order
   CREATE TABLE parent (...);
   CREATE TABLE child (... FOREIGN KEY ...);
   
   -- ✗ Wrong Order
   CREATE TABLE child (... FOREIGN KEY ...);  -- Parent doesn't exist yet!
   ```

---

## 6. Why Don't We Add NOT NULL Constraint to Foreign Keys?

### **The Answer: It Depends on Your Business Logic**

### **Case 1: Foreign Key WITHOUT NOT NULL (Nullable)**

**When to Use:** When the relationship is **optional**

```sql
CREATE TABLE employee (
    emp_id INT PRIMARY KEY,
    name VARCHAR(100),
    manager_id INT,  -- Can be NULL (CEO has no manager)
    FOREIGN KEY (manager_id) REFERENCES employee(emp_id)
) ENGINE=INNODB;
```

**Why?**
- Not every employee has a manager (e.g., CEO)
- Allows flexibility: `manager_id = NULL` means "no manager"

```sql
INSERT INTO employee VALUES (1, 'CEO John', NULL);        -- ✓ Valid
INSERT INTO employee VALUES (2, 'Manager Sarah', 1);      -- ✓ Valid
INSERT INTO employee VALUES (3, 'Employee Tom', 2);       -- ✓ Valid
```

### **Case 2: Foreign Key WITH NOT NULL (Mandatory)**

**When to Use:** When the relationship is **mandatory**

```sql
CREATE TABLE orders (
    order_no INT PRIMARY KEY,
    customer_id INT NOT NULL,  -- Every order MUST have a customer
    FOREIGN KEY (customer_id) REFERENCES customer(id)
) ENGINE=INNODB;
```

**Why?**
- Business rule: "Every order must belong to a customer"
- Prevents incomplete data

```sql
INSERT INTO orders VALUES (101, NULL);  -- ✗ REJECTED! (NOT NULL constraint)
INSERT INTO orders VALUES (101, 1);     -- ✓ Valid (if customer id=1 exists)
```

### **Summary:**
- **NOT NULL on Foreign Key** = "This relationship is mandatory"
- **No NOT NULL** = "This relationship is optional"

---

## 7. FOREIGN KEY Syntax

### **Basic Syntax:**

```sql
CREATE TABLE child_table (
    column1 datatype,
    column2 datatype,
    foreign_key_column datatype,
    
    [CONSTRAINT constraint_name] 
    FOREIGN KEY (foreign_key_column)
    REFERENCES parent_table(parent_key_column)
    [ON DELETE action]
    [ON UPDATE action]
) ENGINE=INNODB;
```

### **Components Explained:**

1. **CONSTRAINT constraint_name** (Optional):
   - Give your foreign key a name for easy reference
   - If omitted, InnoDB generates a name like `0_38775`

2. **FOREIGN KEY (column_name)**:
   - Specify which column(s) in child table are foreign keys

3. **REFERENCES parent_table(column_name)**:
   - Specify which table and column to reference

4. **ON DELETE / ON UPDATE actions**:
   - Define what happens when parent data changes

---

## 8. Referential Actions (The Most Important Part!)

### **What Happens When Parent Data Changes?**

When you UPDATE or DELETE a row in the **parent table**, what should happen to related rows in the **child table**?

### **5 Referential Actions:**

#### **1. RESTRICT (Default)**

**What:** Prevents deletion/update if child records exist

```sql
CREATE TABLE parent (id INT PRIMARY KEY) ENGINE=INNODB;
CREATE TABLE child (
    id INT,
    parent_id INT,
    FOREIGN KEY (parent_id) REFERENCES parent(id)
    ON DELETE RESTRICT  -- Default behavior
) ENGINE=INNODB;

INSERT INTO parent VALUES (1);
INSERT INTO child VALUES (1, 1);

DELETE FROM parent WHERE id=1;  -- ✗ REJECTED! Child record exists
```

**Use Case:** Prevent accidental data loss

---

#### **2. CASCADE**

**What:** Automatically delete/update child records when parent changes

```sql
CREATE TABLE child (
    id INT,
    parent_id INT,
    FOREIGN KEY (parent_id) REFERENCES parent(id)
    ON DELETE CASCADE
    ON UPDATE CASCADE
) ENGINE=INNODB;

INSERT INTO parent VALUES (1);
INSERT INTO child VALUES (1, 1);
INSERT INTO child VALUES (2, 1);

DELETE FROM parent WHERE id=1;  -- ✓ Parent deleted
                                 -- ✓ Both child records also deleted!

-- OR

UPDATE parent SET id=10 WHERE id=1;  -- ✓ Parent updated
                                       -- ✓ Child parent_id changed to 10!
```

**Use Case:** When child records are meaningless without parent
- Example: Delete customer → Delete all their orders

---

#### **3. SET NULL**

**What:** Set foreign key to NULL when parent is deleted/updated

```sql
CREATE TABLE child (
    id INT,
    parent_id INT,  -- Must allow NULL!
    FOREIGN KEY (parent_id) REFERENCES parent(id)
    ON DELETE SET NULL
) ENGINE=INNODB;

INSERT INTO parent VALUES (1);
INSERT INTO child VALUES (1, 1);

DELETE FROM parent WHERE id=1;  -- ✓ Parent deleted
                                 -- ✓ child.parent_id = NULL
```

**Important:** You **cannot** use SET NULL if the foreign key column has NOT NULL constraint!

```sql
-- ✗ WRONG:
parent_id INT NOT NULL,
FOREIGN KEY (parent_id) ... ON DELETE SET NULL  -- Contradiction!
```

**Use Case:** Keep child records but break the relationship
- Example: Delete category → Set product category to NULL (product remains)

---

#### **4. NO ACTION**

**What:** Same as RESTRICT in MySQL (immediate check)

```sql
FOREIGN KEY (parent_id) REFERENCES parent(id)
ON DELETE NO ACTION
```

---

#### **5. SET DEFAULT**

**What:** Set foreign key to default value (NOT supported by InnoDB)

```sql
-- ✗ Parser accepts but InnoDB rejects:
FOREIGN KEY (parent_id) ... ON DELETE SET DEFAULT
```

---

### **Comparison Table:**

| Action | Parent Deleted | Child Records | Use Case |
|--------|---------------|---------------|----------|
| **RESTRICT** | ✗ Rejected | Unchanged | Protect data |
| **CASCADE** | ✓ Deleted | Also deleted | Dependent data |
| **SET NULL** | ✓ Deleted | FK = NULL | Optional relationship |
| **NO ACTION** | ✗ Rejected | Unchanged | Same as RESTRICT |

---

## 9. Complete Examples

### **Example 1: Simple Parent-Child**

```sql
-- Parent: Departments
CREATE TABLE department (
    dept_id INT NOT NULL,
    dept_name VARCHAR(100),
    PRIMARY KEY (dept_id)
) ENGINE=INNODB;

-- Child: Employees
CREATE TABLE employee (
    emp_id INT NOT NULL,
    emp_name VARCHAR(100),
    dept_id INT NOT NULL,  -- NOT NULL: Every employee must have department
    PRIMARY KEY (emp_id),
    INDEX (dept_id),
    FOREIGN KEY (dept_id) REFERENCES department(dept_id)
        ON DELETE RESTRICT    -- Can't delete department if employees exist
        ON UPDATE CASCADE     -- Update dept_id in both tables
) ENGINE=INNODB;

-- Test Data
INSERT INTO department VALUES (10, 'IT'), (20, 'HR');
INSERT INTO employee VALUES (1, 'Alice', 10), (2, 'Bob', 10);

-- Try to delete department
DELETE FROM department WHERE dept_id=10;  
-- ✗ ERROR: Cannot delete - employees exist!

-- Update department
UPDATE department SET dept_id=15 WHERE dept_id=10;
-- ✓ Success! Employee dept_id also updated to 15 (CASCADE)
```

---

### **Example 2: Composite Foreign Key**

```sql
-- Parent: Products (Composite Primary Key)
CREATE TABLE product (
    category INT NOT NULL,
    id INT NOT NULL,
    price DECIMAL,
    PRIMARY KEY(category, id)  -- Composite key
) ENGINE=INNODB;

-- Child: Product Orders
CREATE TABLE product_order (
    no INT NOT NULL AUTO_INCREMENT,
    product_category INT NOT NULL,
    product_id INT NOT NULL,
    PRIMARY KEY(no),
    INDEX (product_category, product_id),  -- Index on both columns
    FOREIGN KEY (product_category, product_id)  -- Both columns together
        REFERENCES product(category, id)
        ON UPDATE CASCADE 
        ON DELETE RESTRICT
) ENGINE=INNODB;

-- Test Data
INSERT INTO product VALUES (1, 100, 50.00);  -- Category 1, Product 100
INSERT INTO product_order VALUES (NULL, 1, 100);  -- References (1,100)

-- Try invalid reference
INSERT INTO product_order VALUES (NULL, 1, 999);  
-- ✗ ERROR: Product (1,999) doesn't exist!
```

---

### **Example 3: Self-Referential Foreign Key**

```sql
-- Employee table referencing itself
CREATE TABLE employee (
    emp_id INT PRIMARY KEY,
    name VARCHAR(100),
    manager_id INT,  -- References another employee in same table
    FOREIGN KEY (manager_id) REFERENCES employee(emp_id)
        ON DELETE SET NULL  -- If manager deleted, set to NULL
) ENGINE=INNODB;

-- Test Data
INSERT INTO employee VALUES (1, 'CEO', NULL);           -- No manager
INSERT INTO employee VALUES (2, 'Manager', 1);          -- Reports to CEO
INSERT INTO employee VALUES (3, 'Employee', 2);         -- Reports to Manager

-- Delete manager
DELETE FROM employee WHERE emp_id=2;
-- ✓ Success! Employee 3's manager_id becomes NULL
```

---

## 10. Adding/Dropping Foreign Keys with ALTER TABLE

### **Adding Foreign Key:**

```sql
-- Create tables without foreign key
CREATE TABLE parent (id INT PRIMARY KEY) ENGINE=INNODB;
CREATE TABLE child (id INT, parent_id INT, INDEX(parent_id)) ENGINE=INNODB;

-- Add foreign key later
ALTER TABLE child
ADD CONSTRAINT fk_child_parent
FOREIGN KEY (parent_id) REFERENCES parent(id)
ON DELETE CASCADE;
```

**Important:** Create the index FIRST before adding foreign key!

---

### **Dropping Foreign Key:**

#### **Method 1: If you know the constraint name**

```sql
ALTER TABLE child DROP FOREIGN KEY fk_child_parent;
```

#### **Method 2: Find the auto-generated name**

```sql
-- Show table structure
SHOW CREATE TABLE child\G

-- Output:
-- CONSTRAINT `0_38775` FOREIGN KEY (`parent_id`) 
-- REFERENCES `parent` (`id`)

-- Drop using that name
ALTER TABLE child DROP FOREIGN KEY `0_38775`;
```

---

### **Important Rules:**

1. **Cannot add and drop foreign key in same statement:**
   ```sql
   -- ✗ WRONG:
   ALTER TABLE child 
       DROP FOREIGN KEY fk1,
       ADD FOREIGN KEY (col) REFERENCES parent(id);
   
   -- ✓ CORRECT: Use separate statements
   ALTER TABLE child DROP FOREIGN KEY fk1;
   ALTER TABLE child ADD FOREIGN KEY (col) REFERENCES parent(id);
   ```

2. **Remember to create index first:**
   ```sql
   -- Add index if not exists
   ALTER TABLE child ADD INDEX (parent_id);
   
   -- Then add foreign key
   ALTER TABLE child ADD FOREIGN KEY (parent_id) REFERENCES parent(id);
   ```

---

## 11. Common Mistakes and Troubleshooting

### **Mistake 1: Type Mismatch**
```sql
-- ✗ Parent: INT, Child: VARCHAR
CREATE TABLE parent (id INT PRIMARY KEY);
CREATE TABLE child (
    parent_id VARCHAR(10),  -- Wrong type!
    FOREIGN KEY (parent_id) REFERENCES parent(id)  -- ERROR
);
```

**Fix:** Match data types exactly

---

### **Mistake 2: Missing Index**
```sql
-- ✗ No index on foreign key column
CREATE TABLE child (
    parent_id INT,
    FOREIGN KEY (parent_id) REFERENCES parent(id)  -- InnoDB creates it automatically
);
```

**Note:** InnoDB creates index automatically, but it's good practice to create it explicitly

---

### **Mistake 3: SET NULL with NOT NULL**
```sql
-- ✗ Contradiction
CREATE TABLE child (
    parent_id INT NOT NULL,
    FOREIGN KEY (parent_id) REFERENCES parent(id)
        ON DELETE SET NULL  -- Can't set NULL on NOT NULL column!
);
```

**Fix:** Remove NOT NULL or use different action

---

### **Mistake 4: Wrong Order of Table Creation**
```sql
-- ✗ Child created before parent
CREATE TABLE child (
    FOREIGN KEY (parent_id) REFERENCES parent(id)  -- parent doesn't exist!
);
CREATE TABLE parent (id INT PRIMARY KEY);
```

**Fix:** Always create parent table first

---

## 12. Why Use InnoDB Engine?

### **What is InnoDB?**
InnoDB is MySQL's default storage engine (since MySQL 5.5)

### **Why InnoDB for Foreign Keys?**

1. **Only InnoDB supports FOREIGN KEY constraints**
   - MyISAM engine accepts syntax but doesn't enforce it!

2. **ACID Compliance:**
   - **A**tomicity: All or nothing transactions
   - **C**onsistency: Data integrity maintained
   - **I**solation: Transactions don't interfere
   - **D**urability: Data persists after commit

3. **Row-level locking:**
   - Better performance for concurrent users

4. **Crash recovery:**
   - Automatic recovery after crashes

### **Checking Engine:**
```sql
SHOW ENGINES;

-- Output shows InnoDB as DEFAULT
```

---

## 13. Best Practices

1. **Always Use InnoDB for Tables with Foreign Keys**
   ```sql
   CREATE TABLE mytable (...) ENGINE=INNODB;
   ```

2. **Name Your Constraints**
   ```sql
   CONSTRAINT fk_orders_customer  -- Clear, descriptive name
   FOREIGN KEY (customer_id) REFERENCES customer(id)
   ```

3. **Create Indexes Explicitly**
   ```sql
   INDEX (foreign_key_column),
   FOREIGN KEY (foreign_key_column) ...
   ```

4. **Choose Appropriate Referential Actions**
   - Use CASCADE carefully (can delete lots of data!)
   - RESTRICT is safest default

5. **Document Relationships**
   ```sql
   -- Orders depend on customers
   -- CASCADE: Deleting customer deletes their orders
   FOREIGN KEY (customer_id) REFERENCES customer(id)
       ON DELETE CASCADE
   ```

---

## 14. Quick Reference Summary

| Concept            | Key Points                                  |
| ------------------ | ------------------------------------------- |
| **Foreign Key**    | Links child table to parent table           |
| **Parent Table**   | Has PRIMARY KEY being referenced            |
| **Child Table**    | Has FOREIGN KEY referencing parent          |
| **INDEX**          | Required for fast FK lookups (auto-created) |
| **NOT NULL on FK** | Use only when relationship is mandatory     |
| **RESTRICT**       | Prevents parent deletion if children exist  |
| **CASCADE**        | Automatically deletes/updates children      |
| **SET NULL**       | Sets FK to NULL (requires nullable column)  |
| **InnoDB**         | Only engine that enforces FK constraints    |


# Implemtations:

## EER Diagrams
This file covers the exact next step you need for Lab 5 and your mid-sem: **Mapping your EER diagrams into actual Relational Tables.** Drawing the diagram is only half the battle. Now you have to convert those Superclasses, Subclasses, and Union Types into a database schema. Here is the no-bullshit breakdown of the rules from Navathe Chapter 9.

### Mapping Superclasses and Subclasses (4 Options)

When you have a superclass/subclass relationship, you have four distinct ways to convert it into tables. The constraint (Disjoint/Overlapping, Total/Partial) dictates which option you should use.

**Option 8A: The "Safe Bet" (Multiple Tables)**

- **How it works:** Create one table for the superclass and one table for _each_ subclass. The primary key of the superclass becomes the primary key _and_ foreign key in all subclass tables.
    
- **When to use it:** This works for **any** specialization (Total, Partial, Disjoint, or Overlapping). It's the most flexible but requires SQL `JOIN`s to get complete data.
    

**Option 8B: The "Subclass Only" Approach (Multiple Tables)**

- **How it works:** You completely delete the superclass table. You only create tables for the subclasses. You copy all the superclass attributes directly into each subclass table.
    
- **When to use it:** Only use this if the constraint is **Total**. If it were Partial, entities that don't belong to a subclass would have nowhere to be stored because the superclass table is gone!
    

**Option 8C: The "Single Giant Table" (One Type Attribute)**

- **How it works:** You smash the superclass and all subclasses into one massive table. You add a single new column called a "Type Attribute" (or discriminator) to tell you which subclass the row belongs to (e.g., `Job_Type = 'Secretary'`). Attributes for other subclasses just get filled with `NULL`.
    
- **When to use it:** Only use this if the constraint is **Disjoint**. Since there's only one "Type" column, a row can only belong to one subclass.
    

**Option 8D: The "Single Giant Table" (Multiple Boolean Attributes)**

- **How it works:** Again, one massive table for everything. But instead of one "Type" column, you add a Boolean (True/False) flag for _every_ subclass (e.g., `Is_Manufactured = Yes`, `Is_Purchased = Yes`).
    
- **When to use it:** This is perfect for **Overlapping** constraints because an entity can have "True" for multiple subclasses at the same time.
    

---

### Mapping Categories (Union Types)

Remember the `OWNER` category that could be a `PERSON`, a `BANK`, or a `COMPANY`? Mapping these is tricky because the superclasses usually have completely different Primary Keys (e.g., SSN for Person, BName for Bank).

- **The Fix (Surrogate Key):** You create a brand new table for the Category (e.g., `OWNER`). Since you can't reuse the primary keys from the superclasses, you invent a new one—a **Surrogate Key** (like `Owner_ID`). You then put this Surrogate Key into the superclass tables as a Foreign Key to link them up.
    

---

# Practice Questions

[[Practice for Views, Assertions and Triggers]]
[[Practice for User Defined Functions]]

# AUTO_INCREMENT in MySQL for Primary Keys

### **What is AUTO_INCREMENT?**

`AUTO_INCREMENT` automatically assigns the **next available number** to the primary key.

You don't have to manually specify IDs—MySQL does it for you.

**AUTO_INCREMENT is NOT a constraint.**

It's a **column property/attribute** that tells MySQL to automatically generate sequential numbers for that column.

**Constraints** (like PRIMARY KEY, FOREIGN KEY, CHECK, NOT NULL) enforce **rules about what values are allowed**.

**AUTO_INCREMENT** is a **generation mechanism** that **produces values automatically**.

**Simple analogy:**
- **Constraint** = "No negative numbers allowed" (rule)
- **AUTO_INCREMENT** = "Make the next number for me" (automation)

---

**Classification:**
- ✓ Column property/attribute
- ✓ Works with PRIMARY KEY/UNIQUE KEY
- ✗ NOT a constraint
- ✗ NOT a data type (INT, VARCHAR etc. are data types)

**It's best described as a "Column Modifier" or "Column Attribute".**

### **Basic Syntax:**

SQL

```
CREATE TABLE movies (
    movie_id INT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(100),
    rental_price DECIMAL(5,2)
);
```

**What this means:**

- `movie_id` is the primary key
- `AUTO_INCREMENT` = MySQL automatically assigns 1, 2, 3, 4...

![[ALTER TABLE Commands#1. MODIFY – Change Column Data Type or Attributes]]

---

### **How AUTO_INCREMENT Works:**

SQL

```
-- Insert without specifying movie_id
INSERT INTO movies (title, rental_price) 
VALUES ('Inception', 5.99);
-- MySQL assigns: movie_id = 1

INSERT INTO movies (title, rental_price) 
VALUES ('Avatar', 6.99);
-- MySQL assigns: movie_id = 2

INSERT INTO movies (title, rental_price) 
VALUES ('Titanic', 4.50);
-- MySQL assigns: movie_id = 3

-- View the table
SELECT * FROM movies;

-- Result:
-- movie_id | title      | rental_price
-- ---------|------------|---------------
-- 1        | Inception  | 5.99
-- 2        | Avatar     | 6.99
-- 3        | Titanic    | 4.50
```

**You never had to type the IDs!** MySQL handled it automatically. ✓

---

### **Inserting Through a View with AUTO_INCREMENT:**

SQL

```
CREATE VIEW EXPENSIVE_MOVIES AS
SELECT movie_id, title, rental_price
FROM movies
WHERE rental_price > 5.00
WITH CHECK OPTION;

-- Insert through view (without specifying movie_id)
INSERT INTO EXPENSIVE_MOVIES (title, rental_price) 
VALUES ('New Movie', 5.99);

-- ✓ SUCCESS
-- MySQL auto-generates movie_id = 4
-- Inserted into base table (movies)
-- View shows it because 5.99 > 5.00

SELECT * FROM EXPENSIVE_MOVIES;
-- Shows the new movie with auto-generated ID
```

---

### **How AUTO_INCREMENT Tracks the Next ID:**

MySQL keeps track of the **last used ID**:

SQL

```
-- After inserting movies with IDs 1, 2, 3
-- MySQL's counter = 3

INSERT INTO movies (title, rental_price) 
VALUES ('Movie 4', 5.50);
-- MySQL sees counter = 3, so assigns 4
-- Updates counter to 4

INSERT INTO movies (title, rental_price) 
VALUES ('Movie 5', 6.00);
-- MySQL sees counter = 4, so assigns 5
-- Updates counter to 5
```

**The counter is NEVER reset** (until you manually reset it).

---

### **What if You Manually Specify an ID?**

SQL

```
-- Manually specify ID = 10
INSERT INTO movies (movie_id, title, rental_price) 
VALUES (10, 'Manual Movie', 5.99);

-- ✓ Succeeds

-- Next insertion:
INSERT INTO movies (title, rental_price) 
VALUES ('Auto Movie', 6.50);
-- MySQL assigns movie_id = 11
-- (not 4, because it saw ID 10 was used)
```

**MySQL updates its counter to the highest ID it sees.**


Next, more to [[Introduction to PostgreSQL]]

