# Introduction to SQL Study Guide

## Key Terms (Glossary)

### SQL Fundamentals

- **SQL (Structured Query Language)**: A standardized programming language for managing and manipulating relational databases
- **Query**: A request for data or information from a database
- **Statement**: A complete SQL command that performs a specific operation
- **Clause**: A component of a SQL statement (e.g., WHERE, FROM, ORDER BY)

### Database Objects

- **Table**: A collection of related data organized in rows and columns
- **Column (Field)**: A vertical entity in a table that represents a specific attribute
- **Row (Record/Tuple)**: A horizontal entity in a table that represents a single data entry
- **Schema**: The structure that defines the organization of data in a database
- **Database**: A collection of related tables and objects

### SQL Language Categories

- **DDL (Data Definition Language)**: Commands that define and modify database structure (CREATE, ALTER, DROP, TRUNCATE)
- **DML (Data Manipulation Language)**: Commands that manipulate data (SELECT, INSERT, UPDATE, DELETE)

### Data Types

- **INTEGER (INT)**: Whole numbers without decimal points
- **NUMERIC/DECIMAL**: Numbers with fixed precision and scale
- **VARCHAR(n)**: Variable-length character string with maximum length n
- **CHAR(n)**: Fixed-length character string of length n
- **DATE**: Calendar date (year, month, day)
- **TIMESTAMP**: Date and time combination
- **BOOLEAN**: True or false values

### Constraints

- **Primary Key**: Uniquely identifies each row in a table
- **Foreign Key**: References the primary key of another table
- **NOT NULL**: Ensures a column cannot have NULL values
- **UNIQUE**: Ensures all values in a column are different
- **CHECK**: Ensures values in a column satisfy a specific condition
- **DEFAULT**: Provides a default value for a column

### Additional Concepts

- **Alias**: A temporary name assigned to a table or column
- **Aggregate Function**: Functions that perform calculations on multiple rows (COUNT, SUM, AVG, MAX, MIN)
- **Index**: A database object that improves query performance by providing faster data retrieval
- **View**: A virtual table based on a SELECT query that doesn't store data itself

## Introduction to SQL

### What is SQL?

SQL (Structured Query Language) is a domain-specific language used for managing and manipulating relational databases. It was developed in the 1970s at IBM and has become the standard language for relational database management systems (RDBMS).

### Key Characteristics:

1. **Declarative Language**: You specify what you want, not how to get it
2. **Standardized**: Based on ANSI/ISO standards
3. **Portable**: Works across different database systems with minor variations
4. **Set-Based**: Operates on sets of data rather than individual records

### SQL Command Categories:

| Category | Purpose                   | Commands                       |
| -------- | ------------------------- | ------------------------------ |
| DDL      | Define database structure | CREATE, ALTER, DROP, TRUNCATE  |
| DML      | Manipulate data           | SELECT, INSERT, UPDATE, DELETE |
| DCL      | Control access            | GRANT, REVOKE                  |
| TCL      | Manage transactions       | COMMIT, ROLLBACK, SAVEPOINT    |

## Data Definition Language (DDL)

### CREATE Statement

The `CREATE` statement is used to create new database objects such as tables, indexes, and views.

#### Creating a Table

**Basic Syntax:**

```sql
CREATE TABLE table_name (
    column1 datatype constraint,
    column2 datatype constraint,
    column3 datatype constraint,
    ...
);
```

**Example 1: Simple Table**

```sql
CREATE TABLE students (
    student_id INT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE,
    enrollment_date DATE DEFAULT CURRENT_DATE
);
```

**Example 2: Table with Multiple Constraints**

```sql
CREATE TABLE courses (
    course_id VARCHAR(10) PRIMARY KEY,
    course_name VARCHAR(100) NOT NULL,
    credits INT CHECK (credits > 0 AND credits <= 6),
    department VARCHAR(50) NOT NULL,
    max_enrollment INT DEFAULT 30,
    instructor_id INT,
    FOREIGN KEY (instructor_id) REFERENCES instructors(instructor_id)
);
```

**Example 3: Composite Primary Key**

```sql
CREATE TABLE enrollments (
    student_id INT,
    course_id VARCHAR(10),
    semester VARCHAR(20),
    year INT,
    grade VARCHAR(2),
    enrollment_date DATE NOT NULL,
    PRIMARY KEY (student_id, course_id, semester, year),
    FOREIGN KEY (student_id) REFERENCES students(student_id),
    FOREIGN KEY (course_id) REFERENCES courses(course_id)
);
```

#### Common Data Types by Database System:

| Type         | Oracle      | MySQL/PostgreSQL | SQL Server   | Description          |
| ------------ | ----------- | ---------------- | ------------ | -------------------- |
| Integer      | NUMBER(p)   | INT, INTEGER     | INT, INTEGER | Whole numbers        |
| Decimal      | NUMBER(p,s) | DECIMAL(p,s)     | DECIMAL(p,s) | Fixed-point numbers  |
| String       | VARCHAR2(n) | VARCHAR(n)       | VARCHAR(n)   | Variable-length text |
| Fixed String | CHAR(n)     | CHAR(n)          | CHAR(n)      | Fixed-length text    |
| Date         | DATE        | DATE             | DATE         | Date values          |
| Timestamp    | TIMESTAMP   | TIMESTAMP        | DATETIME     | Date and time        |
| Boolean      | NUMBER(1)   | BOOLEAN          | BIT          | True/false values    |
| Large Text   | CLOB        | TEXT             | TEXT         | Large text data      |

### ALTER Statement

The `ALTER` statement modifies an existing database object.

**Adding a Column:**

```sql
ALTER TABLE students 
ADD phone_number VARCHAR(15);
```

**Modifying a Column:**

```sql
-- Change data type (Standard SQL)
ALTER TABLE students 
ALTER COLUMN email TYPE VARCHAR(150);

-- Oracle/MySQL syntax (alternative)
-- ALTER TABLE students MODIFY email VARCHAR(150);

-- Add NOT NULL constraint (Standard SQL)
ALTER TABLE students 
ALTER COLUMN phone_number SET NOT NULL;
```

**Dropping a Column:**

```sql
ALTER TABLE students 
DROP COLUMN phone_number;
```

**Adding Constraints:**

```sql
-- Add primary key
ALTER TABLE students 
ADD PRIMARY KEY (student_id);

-- Add foreign key
ALTER TABLE enrollments 
ADD FOREIGN KEY (student_id) REFERENCES students(student_id);

-- Add check constraint
ALTER TABLE courses 
ADD CONSTRAINT check_credits CHECK (credits BETWEEN 1 AND 6);

-- Add unique constraint
ALTER TABLE students 
ADD CONSTRAINT unique_email UNIQUE (email);
```

**Dropping Constraints:**

```sql
ALTER TABLE courses 
DROP CONSTRAINT check_credits;
```

### DROP Statement

The `DROP` statement deletes database objects permanently.

**Drop Table:**

```sql
DROP TABLE enrollments;
```

**Drop Table with Cascade (removes dependent objects):**

```sql
DROP TABLE students CASCADE CONSTRAINTS;
```

**Drop Multiple Tables:**

```sql
DROP TABLE enrollments, students, courses;
```

### TRUNCATE Statement

The `TRUNCATE` statement removes all rows from a table but keeps the structure.

**Syntax:**

```sql
TRUNCATE TABLE table_name;
```

**Differences: DELETE vs TRUNCATE**

| Feature        | DELETE              | TRUNCATE                                 |
| -------------- | ------------------- | ---------------------------------------- |
| Speed          | Slower (row-by-row) | Faster (all at once)                     |
| WHERE clause   | Supported           | Not supported                            |
| Rollback       | Can rollback        | Limited/No rollback (database-dependent) |
| Triggers       | Fires triggers      | Does not fire triggers                   |
| Identity reset | Doesn't reset       | Resets auto-increment                    |

**Examples:**

```sql
-- Delete specific rows
DELETE FROM students WHERE enrollment_date < '2020-01-01';

-- Remove all rows (keeps structure)
TRUNCATE TABLE students;

-- Delete all rows (slower but can rollback)
DELETE FROM students;
```

## Data Manipulation Language (DML)

### INSERT Statement

The `INSERT` statement adds new rows to a table.

#### Basic INSERT Syntax:

```sql
INSERT INTO table_name (column1, column2, column3)
VALUES (value1, value2, value3);
```

**Example 1: Insert Single Row**

```sql
INSERT INTO students (student_id, first_name, last_name, email)
VALUES (101, 'John', 'Smith', 'john.smith@university.edu');
```

**Example 2: Insert Without Specifying Columns (must provide all values in order)**

```sql
INSERT INTO students
VALUES (102, 'Jane', 'Doe', 'jane.doe@university.edu', '2024-01-15');
```

**Example 3: Insert Multiple Rows**

```sql
INSERT INTO students (student_id, first_name, last_name, email)
VALUES 
    (103, 'Bob', 'Wilson', 'bob.wilson@university.edu'),
    (104, 'Alice', 'Brown', 'alice.brown@university.edu'),
    (105, 'Charlie', 'Davis', 'charlie.davis@university.edu');
```

**Example 4: Insert with NULL and DEFAULT Values**

```sql
INSERT INTO students (student_id, first_name, last_name, email, enrollment_date)
VALUES (106, 'Emma', 'Johnson', NULL, DEFAULT);
```

**Example 5: Insert from SELECT (Copy Data)**

```sql
-- Create backup table
CREATE TABLE students_backup AS SELECT * FROM students WHERE 1=0;

-- Insert data from another table
INSERT INTO students_backup
SELECT * FROM students WHERE enrollment_date > '2024-01-01';
```

### UPDATE Statement

The `UPDATE` statement modifies existing data in a table.

#### Basic UPDATE Syntax:

```sql
UPDATE table_name
SET column1 = value1, column2 = value2
WHERE condition;
```

**Example 1: Update Single Row**

```sql
UPDATE students
SET email = 'john.smith.new@university.edu'
WHERE student_id = 101;
```

**Example 2: Update Multiple Columns**

```sql
UPDATE students
SET first_name = 'Jonathan', 
    last_name = 'Smithson',
    email = 'jonathan.smithson@university.edu'
WHERE student_id = 101;
```

**Example 3: Update Multiple Rows**

```sql
UPDATE enrollments
SET grade = 'A'
WHERE student_id = 101 AND semester = 'Fall' AND year = 2024;
```

**Example 4: Update with Calculation**

```sql
UPDATE courses
SET max_enrollment = max_enrollment * 1.1
WHERE department = 'Computer Science';
```

**Example 5: Update with Multiple Conditions**

```sql
UPDATE enrollments
SET grade = 'W'
WHERE semester = 'Fall' AND year = 2024 AND grade IS NULL;
```

**⚠️ Warning: Update Without WHERE**

```sql
-- This updates ALL rows in the table!
UPDATE students
SET enrollment_date = CURRENT_DATE;
```

### DELETE Statement

The `DELETE` statement removes rows from a table.

#### Basic DELETE Syntax:

```sql
DELETE FROM table_name
WHERE condition;
```

**Example 1: Delete Single Row**

```sql
DELETE FROM students
WHERE student_id = 101;
```

**Example 2: Delete Multiple Rows**

```sql
DELETE FROM enrollments
WHERE semester = 'Fall' AND year = 2020;
```

**Example 3: Delete Multiple Conditions**

```sql
DELETE FROM enrollments
WHERE semester = 'Fall' AND year < 2020;
```

**⚠️ Warning: Delete Without WHERE**

```sql
-- This deletes ALL rows in the table!
DELETE FROM students;
```

## SELECT Queries

### Basic SELECT Statement

The `SELECT` statement retrieves data from a database.

#### Basic Syntax:

```sql
SELECT column1, column2, ...
FROM table_name
WHERE condition
ORDER BY column1;
```

**Example 1: Select All Columns**

```sql
SELECT * FROM students;
```

**Example 2: Select Specific Columns**

```sql
SELECT first_name, last_name, email FROM students;
```

**Example 3: Select with WHERE Clause**

```sql
SELECT first_name, last_name 
FROM students
WHERE enrollment_date > '2024-01-01';
```

**Example 4: Select with Multiple Conditions**

```sql
SELECT course_id, course_name, credits
FROM courses
WHERE department = 'Computer Science' 
  AND credits >= 3;
```

**Example 5: Select with OR Condition**

```sql
SELECT * FROM students
WHERE first_name = 'John' OR first_name = 'Jane';
```

### WHERE Clause Operators

| Operator    | Description           | Example                                  |
| ----------- | --------------------- | ---------------------------------------- |
| =           | Equal                 | `WHERE age = 25`                       |
| <> or !=    | Not equal             | `WHERE status <> 'Inactive'`           |
| >           | Greater than          | `WHERE credits > 3`                    |
| <           | Less than             | `WHERE enrollment_date < '2024-01-01'` |
| >=          | Greater than or equal | `WHERE grade >= 'B'`                   |
| <=          | Less than or equal    | `WHERE credits <= 4`                   |
| BETWEEN     | Between range         | `WHERE credits BETWEEN 3 AND 6`        |
| IN          | In a list             | `WHERE grade IN ('A', 'B', 'C')`       |
| LIKE        | Pattern matching      | `WHERE email LIKE '%@university.edu'`  |
| IS NULL     | Is null value         | `WHERE phone_number IS NULL`           |
| IS NOT NULL | Is not null           | `WHERE email IS NOT NULL`              |

### LIKE Operator Patterns

**Wildcards:**

- `%` : Represents zero or more characters
- `_` : Represents exactly one character

**Examples:**

```sql
-- Starts with 'J'
SELECT * FROM students WHERE first_name LIKE 'J%';

-- Ends with 'son'
SELECT * FROM students WHERE last_name LIKE '%son';

-- Contains 'mit'
SELECT * FROM students WHERE last_name LIKE '%mit%';

-- Second letter is 'o'
SELECT * FROM students WHERE first_name LIKE '_o%';

-- Exactly 5 characters
SELECT * FROM students WHERE first_name LIKE '_____';

-- Email from specific domain
SELECT * FROM students WHERE email LIKE '%@university.edu';
```

### ORDER BY Clause

**Syntax:**

```sql
SELECT columns
FROM table
ORDER BY column1 [ASC|DESC], column2 [ASC|DESC];
```

**Examples:**

```sql
-- Sort ascending (default)
SELECT * FROM students ORDER BY last_name;

-- Sort descending
SELECT * FROM students ORDER BY enrollment_date DESC;

-- Sort by multiple columns
SELECT * FROM students 
ORDER BY last_name ASC, first_name ASC;

-- Sort by column position
SELECT first_name, last_name, email 
FROM students 
ORDER BY 2, 1;  -- Sort by 2nd column (last_name), then 1st (first_name)
```

### DISTINCT Keyword

Removes duplicate rows from result set.

**Examples:**

```sql
-- Get unique departments
SELECT DISTINCT department FROM courses;

-- Get unique combinations
SELECT DISTINCT department, credits FROM courses;

-- Count unique values
SELECT COUNT(DISTINCT department) FROM courses;
```

### Aggregate Functions

Functions that perform calculations on multiple rows.

| Function | Description              |
| -------- | ------------------------ |
| COUNT()  | Counts number of rows    |
| SUM()    | Calculates sum of values |
| AVG()    | Calculates average       |
| MAX()    | Finds maximum value      |
| MIN()    | Finds minimum value      |

**Examples:**

```sql
-- Count total students
SELECT COUNT(*) FROM students;

-- Count non-null emails
SELECT COUNT(email) FROM students;

-- Count unique departments
SELECT COUNT(DISTINCT department) FROM courses;

-- Average credits
SELECT AVG(credits) FROM courses;

-- Maximum and minimum credits
SELECT MAX(credits) AS max_credits, MIN(credits) AS min_credits 
FROM courses;

-- Sum of all credits
SELECT SUM(credits) FROM courses WHERE department = 'Computer Science';
```

### GROUP BY Clause

Groups rows with the same values into summary rows.

**Syntax:**

```sql
SELECT column1, aggregate_function(column2)
FROM table
GROUP BY column1;
```

**Examples:**

```sql
-- Count students by enrollment year
SELECT EXTRACT(YEAR FROM enrollment_date) AS year, COUNT(*) AS student_count
FROM students
GROUP BY EXTRACT(YEAR FROM enrollment_date);

-- Average credits by department
SELECT department, AVG(credits) AS avg_credits
FROM courses
GROUP BY department;

-- Count courses and average credits by department
SELECT department, 
       COUNT(*) AS course_count,
       AVG(credits) AS avg_credits,
       MAX(max_enrollment) AS largest_class
FROM courses
GROUP BY department;

-- Group by multiple columns
SELECT department, credits, COUNT(*) AS course_count
FROM courses
GROUP BY department, credits
ORDER BY department, credits;
```

### HAVING Clause

Filters groups created by GROUP BY (WHERE filters rows, HAVING filters groups).

**Examples:**

```sql
-- Departments with more than 5 courses
SELECT department, COUNT(*) AS course_count
FROM courses
GROUP BY department
HAVING COUNT(*) > 5;

-- Departments with average credits > 3
SELECT department, AVG(credits) AS avg_credits
FROM courses
GROUP BY department
HAVING AVG(credits) > 3;

-- Combined WHERE and HAVING
SELECT department, COUNT(*) AS course_count
FROM courses
WHERE credits >= 3
GROUP BY department
HAVING COUNT(*) > 2
ORDER BY course_count DESC;
```

**Order of Execution:**

```
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY
```

## Aliases

Aliases provide temporary names for tables and columns to make queries more readable.

### Column Aliases

**Syntax:**

```sql
SELECT column_name AS alias_name
FROM table_name;
```

**Examples:**

```sql
-- Simple column alias
SELECT first_name AS fname, last_name AS lname
FROM students;

-- Alias without AS keyword
SELECT first_name fname, last_name lname
FROM students;

-- Alias with spaces (requires quotes)
SELECT first_name AS "First Name", last_name AS "Last Name"
FROM students;

-- Alias with calculation (string concatenation using || - Standard SQL)
SELECT first_name || ' ' || last_name AS full_name
FROM students;
-- Note: MySQL uses CONCAT(first_name, ' ', last_name) instead

-- Alias with aggregate function
SELECT department, COUNT(*) AS total_courses
FROM courses
GROUP BY department;
```

### Table Aliases

**Syntax:**

```sql
SELECT alias.column_name
FROM table_name alias;
```

**Examples:**

```sql
-- Simple table alias
SELECT s.first_name, s.last_name, s.email
FROM students s;

-- Table alias in WHERE clause
SELECT s.first_name, s.last_name
FROM students s
WHERE s.enrollment_date > '2024-01-01';

-- Table alias with multiple conditions
SELECT e.student_id, e.course_id, e.grade
FROM enrollments e
WHERE e.semester = 'Fall' AND e.year = 2024;
```

**Why Use Table Aliases:**

1. Makes queries shorter and more readable
2. Reduces typing for long table names
3. Improves query readability in complex queries

## Indexes

Indexes are database objects that improve query performance by providing faster data retrieval.

### How Indexes Work

Think of a database index like a book index:
- **Without Index**: Database scans every row (like reading every page)
- **With Index**: Database uses index to find data quickly (like using book index)

### Types of Indexes

| Type | Description | Use Case |
|------|-------------|----------|
| B-Tree Index | Balanced tree structure (default) | General purpose, range queries |
| Unique Index | Enforces uniqueness | Primary keys, unique constraints |
| Composite Index | Index on multiple columns | Queries using multiple columns |

### Creating Indexes

**Basic Syntax:**
```sql
CREATE INDEX index_name 
ON table_name (column1, column2, ...);
```

**Example 1: Single Column Index**
```sql
CREATE INDEX idx_students_last_name 
ON students(last_name);
```

**Example 2: Unique Index**
```sql
CREATE UNIQUE INDEX idx_students_email 
ON students(email);
```

**Example 3: Composite Index**
```sql
CREATE INDEX idx_enrollments_student_course 
ON enrollments(student_id, course_id);
```

### Index Performance Impact

**Query Without Index:**
```sql
-- Full table scan (slow)
SELECT * FROM students WHERE last_name = 'Smith';
```

**Query With Index on last_name:**
```sql
-- Uses index (fast)
SELECT * FROM students WHERE last_name = 'Smith';
```

### When to Create Indexes

**✅ Create indexes for:**
- Columns frequently used in WHERE clauses
- Columns used in ORDER BY clauses
- Foreign key columns
- Columns with high selectivity (many unique values)

**❌ Avoid indexes for:**
- Small tables (< 1000 rows)
- Columns frequently updated
- Columns with low selectivity (few unique values)
- Tables with frequent INSERT/UPDATE/DELETE operations

### Dropping Indexes

**Syntax:**
```sql
DROP INDEX index_name;
```

**Example:**
```sql
DROP INDEX idx_students_last_name;
```

### Viewing Indexes

**Oracle:**
```sql
-- Indexes for a specific table
SELECT index_name, column_name, column_position
FROM user_ind_columns
WHERE table_name = 'STUDENTS'
ORDER BY index_name, column_position;

-- All indexes owned by user
SELECT index_name, table_name, uniqueness
FROM user_indexes
ORDER BY table_name, index_name;
```

**MySQL:**
```sql
SHOW INDEXES FROM students;
```

**PostgreSQL:**
```sql
SELECT * FROM pg_indexes WHERE tablename = 'students';
```

## Database Views

A view is a virtual table based on the result of a SELECT query. It doesn't store data itself but displays data from one or more tables.

### Why Use Views?

1. **Simplify Complex Queries**: Hide complexity behind simple view names
2. **Security**: Restrict access to specific columns or rows
3. **Data Abstraction**: Hide underlying table structure changes
4. **Reusability**: Define query once, use many times

### Creating Views

**Basic Syntax:**
```sql
CREATE VIEW view_name AS
SELECT columns
FROM tables
WHERE conditions;
```

**Example 1: Simple View**
```sql
CREATE VIEW active_students AS
SELECT student_id, first_name, last_name, email
FROM students
WHERE enrollment_date >= '2024-01-01';
```

**Example 2: View with Aggregate Functions**
```sql
CREATE VIEW department_summary AS
SELECT department,
       COUNT(*) AS course_count,
       AVG(credits) AS avg_credits,
       SUM(max_enrollment) AS total_capacity
FROM courses
GROUP BY department;
```

**Example 3: View with Calculated Columns**
```sql
CREATE VIEW student_summary AS
SELECT student_id,
       first_name || ' ' || last_name AS full_name,
       enrollment_date
FROM students;
```

### Using Views

Views are used exactly like tables:

```sql
-- Select from view
SELECT * FROM active_students;

-- Use view in WHERE clause
SELECT * FROM active_students WHERE last_name = 'Smith';

-- Order results from view
SELECT * FROM department_summary ORDER BY course_count DESC;
```

### Modifying Views

**Replace View:**
```sql
CREATE OR REPLACE VIEW active_students AS
SELECT student_id, first_name, last_name, email, enrollment_date
FROM students
WHERE enrollment_date >= '2023-01-01';  -- Changed date
```

### Dropping Views

**Syntax:**
```sql
DROP VIEW view_name;
```

**Example:**
```sql
DROP VIEW active_students;
```

### Updatable Views

Some views allow INSERT, UPDATE, DELETE operations (with restrictions):

**Requirements for Updatable View:**
- No aggregate functions (COUNT, SUM, AVG, etc.)
- No DISTINCT, GROUP BY, or HAVING
- Based on single table (in most cases)

**Example Updatable View:**
```sql
CREATE VIEW cs_students AS
SELECT student_id, first_name, last_name, email
FROM students
WHERE major = 'Computer Science';

-- This will work
UPDATE cs_students
SET email = 'new.email@university.edu'
WHERE student_id = 101;
```

## Query Execution Order

Understanding SQL execution order helps write better queries:

```
1. FROM       - Identify tables
2. WHERE      - Filter rows
3. GROUP BY   - Group rows
4. HAVING     - Filter groups
5. SELECT     - Select columns
6. DISTINCT   - Remove duplicates
7. ORDER BY   - Sort results
```

**Example showing all clauses:**

```sql
SELECT department, AVG(credits) AS avg_credits
FROM courses
WHERE credits >= 3
GROUP BY department
HAVING COUNT(*) > 5
ORDER BY avg_credits DESC;
```

## Best Practices

### Writing Efficient Queries

1. **Select only needed columns** (avoid SELECT *)

```sql
-- Bad
SELECT * FROM students;

-- Good
SELECT student_id, first_name, last_name FROM students;
```

2. **Use WHERE instead of HAVING when possible**

```sql
-- Less efficient
SELECT department, COUNT(*)
FROM courses
GROUP BY department
HAVING department = 'Computer Science';

-- More efficient
SELECT department, COUNT(*)
FROM courses
WHERE department = 'Computer Science'
GROUP BY department;
```

3. **Use appropriate data types** to save space and improve performance

4. **Avoid unnecessary DISTINCT** if data is already unique

5. **Create indexes on frequently queried columns**
```sql
CREATE INDEX idx_students_last_name ON students(last_name);
CREATE INDEX idx_enrollments_student ON enrollments(student_id);
```

### SQL Style Guidelines

1. **Use uppercase for SQL keywords**

```sql
SELECT first_name, last_name FROM students WHERE enrollment_date > '2024-01-01';
```

2. **Use meaningful table and column aliases**

```sql
SELECT s.first_name, s.last_name, s.email
FROM students s
WHERE s.enrollment_date > '2024-01-01';
```

3. **Format complex queries for readability**

```sql
SELECT 
    department,
    COUNT(*) AS course_count,
    AVG(credits) AS avg_credits,
    MAX(max_enrollment) AS largest_class
FROM courses
WHERE credits >= 3
GROUP BY department
HAVING COUNT(*) > 5
ORDER BY avg_credits DESC;
```

### Common Mistakes to Avoid

1. **Forgetting WHERE in UPDATE/DELETE**

```sql
-- DANGER: Updates all rows!
UPDATE students SET email = 'wrong@email.com';

-- Correct: Updates specific row
UPDATE students SET email = 'correct@email.com' WHERE student_id = 101;
```

2. **Not handling NULL values properly**

```sql
-- Wrong: This won't find NULL values
SELECT * FROM students WHERE email = NULL;

-- Correct: Use IS NULL
SELECT * FROM students WHERE email IS NULL;
```

3. **Comparing with NULL using = or <> operator**

```sql
-- Wrong
WHERE grade <> 'F'  -- This excludes NULL grades too!

-- Correct
WHERE grade <> 'F' OR grade IS NULL
```

4. **Using SELECT * in production code**

```sql
-- Bad practice
SELECT * FROM students;

-- Good practice: specify columns
SELECT student_id, first_name, last_name, email FROM students;
```

## Summary

### Key Takeaways:

1. **SQL Language Categories:**

   - **DDL**: Defines structure (CREATE, ALTER, DROP)
   - **DML**: Manipulates data (SELECT, INSERT, UPDATE, DELETE)
2. **Data Definition (DDL):**

   - CREATE TABLE with constraints (PRIMARY KEY, FOREIGN KEY, UNIQUE, CHECK, NOT NULL)
   - ALTER TABLE to modify structure
   - DROP TABLE to remove tables
   - TRUNCATE TABLE to remove all data quickly
3. **Data Manipulation (DML):**

   - INSERT to add new rows
   - UPDATE to modify existing data (always use WHERE clause!)
   - DELETE to remove rows (always use WHERE clause!)
   - SELECT to retrieve data
4. **SELECT Query Components:**

   - WHERE: Filter rows
   - GROUP BY: Group rows for aggregation
   - HAVING: Filter groups
   - ORDER BY: Sort results
   - DISTINCT: Remove duplicates
   - Aggregate functions: COUNT, SUM, AVG, MAX, MIN
5. **Aliases:**

   - Column aliases make output more readable
   - Table aliases simplify queries and reduce typing

6. **Indexes:**

   - Improve query performance dramatically
   - Create on frequently searched columns (WHERE, ORDER BY)
   - Trade-off: slower writes for faster reads
   - Avoid on small tables or frequently updated columns

7. **Views:**

   - Virtual tables based on SELECT queries
   - Simplify complex queries and improve reusability
   - Provide security by restricting column/row access
   - Can be updatable with certain restrictions

8. **Best Practices:**

   - Always use WHERE with UPDATE and DELETE
   - Select only needed columns
   - Use WHERE instead of HAVING when possible
   - Handle NULL values properly
   - Write readable, formatted queries

### SQL Command Quick Reference:

```sql
-- DDL (Data Definition Language)
CREATE TABLE, ALTER TABLE, DROP TABLE, TRUNCATE TABLE

-- DML (Data Manipulation Language)
SELECT, INSERT, UPDATE, DELETE

-- Clauses
WHERE, GROUP BY, HAVING, ORDER BY

-- Operators
=, <>, >, <, >=, <=, BETWEEN, IN, LIKE, IS NULL, IS NOT NULL

-- Aggregate Functions
COUNT(), SUM(), AVG(), MAX(), MIN()

-- Constraints
PRIMARY KEY, FOREIGN KEY, UNIQUE, NOT NULL, CHECK, DEFAULT

-- Database Objects
CREATE INDEX, DROP INDEX, CREATE VIEW, DROP VIEW
```

This foundation in SQL provides the essential skills needed for database management and application development. Practice these concepts with real databases to solidify your understanding!
