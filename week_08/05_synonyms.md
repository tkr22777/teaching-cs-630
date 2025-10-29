# Synonyms

## Overview

A **synonym** is an alias or alternative name for a database object such as a table, view, sequence, or another synonym. Synonyms provide location transparency and simplify SQL statements by allowing users to reference objects without knowing their complete qualified names or locations. They are particularly useful in distributed databases and for abstracting schema details from applications.

## Key Terms

**Synonym**: An alternative name for a database object that provides a layer of abstraction.

**Public Synonym**: A synonym accessible to all database users.

**Private Synonym**: A synonym accessible only to the user who created it (default).

**Schema**: A collection of database objects owned by a database user.

**Qualified Name**: Full name of an object including schema (e.g., SCHEMA.TABLE_NAME).

**Location Transparency**: Ability to access objects without knowing their physical location or schema.

**Database Link**: Connection to a remote database, can be referenced through synonyms.

## Sample Database Schema

This module uses the university enrollment system. If you haven't set it up yet:

<details>
<summary>Click to expand: Database setup script</summary>

```sql
-- Create Students table
CREATE TABLE students (
    student_id INTEGER PRIMARY KEY,
    first_name VARCHAR2(50) NOT NULL,
    last_name VARCHAR2(50) NOT NULL,
    email VARCHAR2(100) UNIQUE NOT NULL,
    major VARCHAR2(50),
    enrollment_date DATE DEFAULT SYSDATE,
    gpa NUMBER(3, 2)
);

-- Create Instructors table
CREATE TABLE instructors (
    instructor_id INTEGER PRIMARY KEY,
    instructor_name VARCHAR2(100) NOT NULL,
    department VARCHAR2(50),
    hire_date DATE
);

-- Create Courses table
CREATE TABLE courses (
    course_id VARCHAR2(10) PRIMARY KEY,
    course_name VARCHAR2(100) NOT NULL,
    department VARCHAR2(50),
    credits INTEGER,
    instructor_id INTEGER REFERENCES instructors(instructor_id)
);

-- Create Enrollments table (junction table)
CREATE TABLE enrollments (
    enrollment_id INTEGER PRIMARY KEY,
    student_id INTEGER REFERENCES students(student_id),
    course_id VARCHAR2(10) REFERENCES courses(course_id),
    semester VARCHAR2(20),
    grade VARCHAR2(5),
    grade_points NUMBER(3, 2)
);

-- Insert Students
INSERT INTO students (student_id, first_name, last_name, email, major, enrollment_date, gpa) VALUES
(1, 'John', 'Smith', 'john.smith@university.edu', 'Computer Science', DATE '2023-09-01', 3.8);
INSERT INTO students (student_id, first_name, last_name, email, major, enrollment_date, gpa) VALUES
(2, 'Jane', 'Doe', 'jane.doe@university.edu', 'Mathematics', DATE '2023-09-01', 3.9);
INSERT INTO students (student_id, first_name, last_name, email, major, enrollment_date, gpa) VALUES
(3, 'Bob', 'Wilson', 'bob.wilson@university.edu', 'Computer Science', DATE '2024-01-15', 3.2);
INSERT INTO students (student_id, first_name, last_name, email, major, enrollment_date, gpa) VALUES
(4, 'Alice', 'Brown', 'alice.brown@university.edu', 'Physics', DATE '2024-01-15', 3.7);
INSERT INTO students (student_id, first_name, last_name, email, major, enrollment_date, gpa) VALUES
(5, 'Charlie', 'Davis', 'charlie.davis@university.edu', NULL, DATE '2024-09-01', 2.8);

-- Insert Instructors
INSERT INTO instructors (instructor_id, instructor_name, department, hire_date) VALUES
(10, 'Dr. Johnson', 'Computer Science', DATE '2018-08-15');
INSERT INTO instructors (instructor_id, instructor_name, department, hire_date) VALUES
(11, 'Dr. Lee', 'Mathematics', DATE '2019-01-10');
INSERT INTO instructors (instructor_id, instructor_name, department, hire_date) VALUES
(12, 'Dr. Martinez', 'Physics', DATE '2020-09-01');
INSERT INTO instructors (instructor_id, instructor_name, department, hire_date) VALUES
(13, 'Dr. Taylor', 'Chemistry', DATE '2021-06-15');

-- Insert Courses
INSERT INTO courses (course_id, course_name, department, credits, instructor_id) VALUES
('CS101', 'Introduction to Programming', 'Computer Science', 3, 10);
INSERT INTO courses (course_id, course_name, department, credits, instructor_id) VALUES
('CS201', 'Data Structures', 'Computer Science', 4, 10);
INSERT INTO courses (course_id, course_name, department, credits, instructor_id) VALUES
('MATH101', 'Calculus I', 'Mathematics', 4, 11);
INSERT INTO courses (course_id, course_name, department, credits, instructor_id) VALUES
('PHYS101', 'Physics I', 'Physics', 4, 12);
INSERT INTO courses (course_id, course_name, department, credits, instructor_id) VALUES
('CS301', 'Database Systems', 'Computer Science', 3, 10);
INSERT INTO courses (course_id, course_name, department, credits, instructor_id) VALUES
('ENG101', 'English Composition', 'English', 3, NULL);

-- Insert Enrollments
INSERT INTO enrollments (enrollment_id, student_id, course_id, semester, grade, grade_points) VALUES
(101, 1, 'CS101', 'Fall 2023', 'A', 4.0);
INSERT INTO enrollments (enrollment_id, student_id, course_id, semester, grade, grade_points) VALUES
(102, 1, 'CS201', 'Spring 2024', 'B+', 3.3);
INSERT INTO enrollments (enrollment_id, student_id, course_id, semester, grade, grade_points) VALUES
(103, 2, 'MATH101', 'Fall 2023', 'A', 4.0);
INSERT INTO enrollments (enrollment_id, student_id, course_id, semester, grade, grade_points) VALUES
(104, 2, 'CS101', 'Fall 2023', 'A-', 3.7);
INSERT INTO enrollments (enrollment_id, student_id, course_id, semester, grade, grade_points) VALUES
(105, 3, 'CS101', 'Spring 2024', 'B', 3.0);
INSERT INTO enrollments (enrollment_id, student_id, course_id, semester, grade, grade_points) VALUES
(106, 3, 'CS201', 'Spring 2024', 'B+', 3.3);
INSERT INTO enrollments (enrollment_id, student_id, course_id, semester, grade, grade_points) VALUES
(107, 4, 'PHYS101', 'Spring 2024', 'A', 4.0);
INSERT INTO enrollments (enrollment_id, student_id, course_id, semester, grade, grade_points) VALUES
(108, 1, 'CS301', 'Fall 2024', NULL, NULL);

COMMIT;
```

</details>

## Why Use Synonyms?

### Benefits

1. **Simplified References** - Use short names instead of schema.object_name
2. **Location Transparency** - Hide object location from users
3. **Schema Independence** - Applications don't need to know which schema owns objects
4. **Easier Migration** - Change underlying objects without changing application code
5. **Security** - Control access through synonyms instead of direct grants
6. **Convenience** - Shorter names for frequently used objects
7. **Remote Access** - Simplify references to objects in remote databases

### Use Cases

| Scenario | Benefit |
|----------|---------|
| **Multi-schema environment** | Hide schema names from users |
| **Application deployment** | Same code works across dev/test/prod |
| **Remote databases** | Simplify database link references |
| **Long object names** | Create shorter aliases |
| **Schema refactoring** | Change locations without breaking code |
| **Security** | Control access layer |

## Types of Synonyms

### Private Synonyms

- Created in your own schema
- Visible only to you (unless granted)
- Default type when creating synonyms
- No special privileges required

### Public Synonyms

- Created in public schema
- Visible to all database users
- Requires CREATE PUBLIC SYNONYM privilege
- Use for widely-used objects

### Comparison

| Aspect | Private Synonym | Public Synonym |
|--------|-----------------|----------------|
| **Visibility** | Owner only | All users |
| **Privilege Required** | CREATE SYNONYM | CREATE PUBLIC SYNONYM |
| **Namespace** | User's schema | Public |
| **Use Case** | Personal shortcuts | System-wide access |
| **Dropped By** | Owner | Any user with DROP PUBLIC SYNONYM |

## Creating Synonyms

### Private Synonym Syntax

```sql
CREATE SYNONYM synonym_name
FOR object_name;
```

### Public Synonym Syntax

```sql
CREATE PUBLIC SYNONYM synonym_name
FOR object_name;
```

### Example 1: Simple Private Synonym

```sql
-- Create synonym for a table
CREATE SYNONYM stud
FOR students;

-- Now can use either name
SELECT * FROM students;   -- Original
SELECT * FROM stud;       -- Synonym (same result)
```

### Example 2: Synonym for Another Schema's Object

```sql
-- Full qualified name
SELECT * FROM hr.employees;

-- Create synonym
CREATE SYNONYM emp
FOR hr.employees;

-- Simplified reference
SELECT * FROM emp;
```

### Example 3: Public Synonym

```sql
-- Create public synonym (requires privilege)
CREATE PUBLIC SYNONYM courses
FOR university.course_catalog;

-- All users can now access
SELECT * FROM courses;  -- Works for all users
```

### Example 4: Synonym for a View

```sql
-- Create view
CREATE VIEW active_students AS
SELECT student_id, first_name, last_name, major
FROM students
WHERE enrollment_date >= ADD_MONTHS(SYSDATE, -12);

-- Create synonym for view
CREATE SYNONYM active_stud
FOR active_students;

-- Use synonym
SELECT * FROM active_stud;
```

### Example 5: Synonym for a Sequence

```sql
-- Create synonym for sequence
CREATE SYNONYM stud_seq
FOR student_id_sequence;

-- Use synonym to generate values
INSERT INTO students (student_id, first_name, last_name)
VALUES (stud_seq.NEXTVAL, 'John', 'Doe');
```

### Example 6: Synonym for Remote Database Object

```sql
-- Remote table via database link
SELECT * FROM employees@remote_db;

-- Create synonym
CREATE SYNONYM remote_emp
FOR employees@remote_db;

-- Simplified access
SELECT * FROM remote_emp;
```

## Using Synonyms

### In SELECT Statements

```sql
-- Original table name
SELECT * FROM university.students WHERE major = 'Computer Science';

-- With synonym
CREATE SYNONYM stud FOR university.students;
SELECT * FROM stud WHERE major = 'Computer Science';
```

### In INSERT Statements

```sql
-- Using synonym
INSERT INTO stud (student_id, first_name, last_name, email)
VALUES (1001, 'Alice', 'Johnson', 'alice.j@university.edu');
```

### In UPDATE Statements

```sql
-- Using synonym
UPDATE stud
SET major = 'Computer Engineering'
WHERE student_id = 1001;
```

### In DELETE Statements

```sql
-- Using synonym
DELETE FROM stud
WHERE student_id = 1001;
```

### In JOINs

```sql
-- Original
SELECT s.first_name, e.course_id
FROM university.students s
JOIN university.enrollments e ON s.student_id = e.student_id;

-- With synonyms
CREATE SYNONYM stud FOR university.students;
CREATE SYNONYM enroll FOR university.enrollments;

SELECT s.first_name, e.course_id
FROM stud s
JOIN enroll e ON s.student_id = e.student_id;
```

## Viewing Synonyms

### Query User Synonyms

```sql
-- View your private synonyms
SELECT 
    synonym_name,
    table_owner,
    table_name,
    db_link
FROM user_synonyms
ORDER BY synonym_name;
```

**Example Result:**
| synonym_name | table_owner | table_name | db_link |
|--------------|-------------|------------|---------|
| STUD | UNIVERSITY | STUDENTS | NULL |
| EMP | HR | EMPLOYEES | NULL |
| REMOTE_EMP | HR | EMPLOYEES | REMOTE_DB |

### Query All Synonyms

```sql
-- View all synonyms you have access to
SELECT 
    owner,
    synonym_name,
    table_owner,
    table_name
FROM all_synonyms
WHERE table_owner = 'UNIVERSITY'
ORDER BY synonym_name;
```

### Query Public Synonyms

```sql
-- View public synonyms
SELECT 
    synonym_name,
    table_owner,
    table_name
FROM dba_synonyms
WHERE owner = 'PUBLIC'
ORDER BY synonym_name;
```

### Check if Synonym Exists

```sql
-- Check before creating
SELECT COUNT(*) 
FROM user_synonyms 
WHERE synonym_name = 'STUD';
```

## Dropping Synonyms

### Drop Private Synonym

```sql
DROP SYNONYM synonym_name;
```

**Example:**
```sql
DROP SYNONYM stud;
```

### Drop Public Synonym

```sql
DROP PUBLIC SYNONYM synonym_name;
```

**Example:**
```sql
DROP PUBLIC SYNONYM courses;
```

### Drop if Exists (Oracle 23c+)

```sql
DROP SYNONYM IF EXISTS stud;
DROP PUBLIC SYNONYM IF EXISTS courses;
```

### Important Notes

- Dropping a synonym does **not** delete the underlying object
- Only the alias is removed
- Users referencing the synonym will get errors after drop
- Must recreate synonym if needed again

## Synonym Resolution

### Name Resolution Order

When you reference an object name, Oracle searches in this order:

1. **Local objects** - Tables, views in your schema
2. **Private synonyms** - Synonyms you created
3. **Public synonyms** - Public synonyms

### Example: Resolution Priority

```sql
-- Assume three objects named "employees"
-- 1. Your table: employees
-- 2. Your private synonym: employees → hr.employees
-- 3. Public synonym: employees → company.employees

SELECT * FROM employees;
-- Uses: Your table (highest priority)

-- To use synonym, drop or rename your table
DROP TABLE employees;
SELECT * FROM employees;
-- Now uses: Your private synonym

-- To use public synonym, drop both
DROP SYNONYM employees;
SELECT * FROM employees;
-- Now uses: Public synonym
```

## Practical Examples

### Example 1: Multi-Environment Deployment

**Problem:** Application needs to work across dev, test, and production with different schemas.

**Solution:**

```sql
-- Development environment
CREATE SYNONYM app_users FOR dev_schema.users;
CREATE SYNONYM app_orders FOR dev_schema.orders;

-- Production environment (same synonym names, different targets)
CREATE SYNONYM app_users FOR prod_schema.users;
CREATE SYNONYM app_orders FOR prod_schema.orders;

-- Application code (works in all environments)
SELECT * FROM app_users;
SELECT * FROM app_orders;
```

### Example 2: Schema Migration

**Scenario:** Moving tables from old_schema to new_schema without breaking applications.

```sql
-- Step 1: Create synonyms pointing to old location
CREATE PUBLIC SYNONYM customers FOR old_schema.customers;
CREATE PUBLIC SYNONYM orders FOR old_schema.orders;

-- Applications continue working
SELECT * FROM customers;  -- Uses old_schema.customers

-- Step 2: Migrate data to new schema
-- (copy data, test thoroughly)

-- Step 3: Update synonyms
DROP PUBLIC SYNONYM customers;
CREATE PUBLIC SYNONYM customers FOR new_schema.customers;

DROP PUBLIC SYNONYM orders;
CREATE PUBLIC SYNONYM orders FOR new_schema.orders;

-- Applications still work, now using new_schema
SELECT * FROM customers;  -- Now uses new_schema.customers
```

### Example 3: Simplified Remote Access

```sql
-- Create database link
CREATE DATABASE LINK remote_warehouse
CONNECT TO warehouse_user IDENTIFIED BY password
USING 'warehouse_db';

-- Without synonym (complex)
SELECT * FROM inventory@remote_warehouse;
UPDATE product_stock@remote_warehouse SET quantity = 100;

-- With synonym (simple)
CREATE SYNONYM inventory FOR inventory@remote_warehouse;

SELECT * FROM inventory;
UPDATE inventory SET quantity = 100;
```

### Example 4: Long Object Names

```sql
-- Original long name
CREATE TABLE employee_performance_review_history (
    review_id NUMBER PRIMARY KEY,
    employee_id NUMBER,
    review_date DATE,
    rating NUMBER
);

-- Create synonym with shorter name
CREATE SYNONYM emp_reviews FOR employee_performance_review_history;

-- Much easier to use
SELECT * FROM emp_reviews WHERE rating >= 4;
```

### Example 5: Backward Compatibility

```sql
-- Old table name
CREATE TABLE old_customer_table (
    customer_id NUMBER,
    customer_name VARCHAR2(100)
);

-- Rename table
RENAME old_customer_table TO customers;

-- Create synonym for backward compatibility
CREATE SYNONYM old_customer_table FOR customers;

-- Old code still works
SELECT * FROM old_customer_table;  -- Still works via synonym

-- New code uses better name
SELECT * FROM customers;           -- Direct access
```

## Synonyms and Security

### Controlling Access Through Synonyms

```sql
-- Schema owner creates view with filtered data
CREATE VIEW public_employee_data AS
SELECT employee_id, first_name, last_name, department
FROM employees;  -- Excludes salary, SSN, etc.

-- Create public synonym
CREATE PUBLIC SYNONYM employees FOR public_employee_data;

-- Grant access to synonym
GRANT SELECT ON public_employee_data TO PUBLIC;

-- Users see filtered data
SELECT * FROM employees;  -- Can only see safe columns
```

### Hiding Schema Complexity

```sql
-- Complex schema structure
CREATE TABLE dept_emp_v2_prod (
    emp_id NUMBER,
    dept_code VARCHAR2(10)
);

-- Simple public synonym
CREATE PUBLIC SYNONYM employee_departments FOR dept_emp_v2_prod;

-- Users don't need to know complex naming
SELECT * FROM employee_departments;
```

## Synonyms vs. Other Alternatives

### Comparison Table

| Feature | Synonym | View | Database Link |
|---------|---------|------|---------------|
| **Purpose** | Alias for objects | Virtual table | Remote connection |
| **Performance** | No overhead | Can have overhead | Network latency |
| **Flexibility** | Points to single object | Can join multiple | Accesses remote DB |
| **Security** | Pass-through | Row-level filtering | Separate credentials |
| **Use Case** | Naming simplification | Data abstraction | Distributed data |

### When to Use Each

**Use Synonym when:**
- You need a simpler name
- You want location transparency
- You're abstracting schema names

**Use View when:**
- You need to filter or transform data
- You want to hide columns
- You need to join multiple tables

**Use Database Link when:**
- Accessing remote databases
- Data is physically distributed
- Different database instances

## Common Mistakes and Solutions

### Mistake 1: Circular References

**Problem:**
```sql
CREATE SYNONYM syn1 FOR syn2;
CREATE SYNONYM syn2 FOR syn1;
-- Creates circular reference
```

**Solution:** Synonyms should point to actual objects, not other synonyms.

### Mistake 2: Synonym Name Conflicts

**Problem:**
```sql
-- You have a table named "employees"
CREATE SYNONYM employees FOR hr.employees;
-- ERROR: Name already used
```

**Solution:** Use different name or drop/rename existing object.

### Mistake 3: Dropping Underlying Object

**Problem:**
```sql
CREATE SYNONYM stud FOR students;
DROP TABLE students;
-- Synonym still exists but points to nothing
SELECT * FROM stud;  -- ERROR: table or view does not exist
```

**Solution:** Drop synonym when dropping underlying object, or recreate base object.

### Mistake 4: Insufficient Privileges

**Problem:**
```sql
CREATE SYNONYM emp FOR hr.employees;
SELECT * FROM emp;
-- ERROR: table or view does not exist
```

**Explanation:** Synonym exists, but you lack SELECT privilege on hr.employees.

**Solution:**
```sql
-- HR must grant you access
GRANT SELECT ON hr.employees TO your_username;
```

### Mistake 5: Public vs. Private Confusion

**Problem:**
```sql
-- Create private synonym
CREATE SYNONYM courses FOR university.courses;

-- Other users try to use it
-- ERROR: table or view does not exist
```

**Solution:** Use CREATE PUBLIC SYNONYM for shared access.

## Best Practices

### 1. Naming Conventions

```sql
-- Good: Descriptive, clear
CREATE SYNONYM emp FOR employees;
CREATE SYNONYM stud FOR students;

-- Avoid: Ambiguous or cryptic
CREATE SYNONYM x FOR employees;
CREATE SYNONYM s1 FOR students;
```

### 2. Documentation

```sql
-- Document synonyms in comments
COMMENT ON TABLE stud IS 'Synonym for students table in university schema';
```

### 3. Consistent Usage

```sql
-- Choose one approach per application
-- Option 1: Always use synonyms
SELECT * FROM stud;
SELECT * FROM courses;

-- Option 2: Always use qualified names
SELECT * FROM university.students;
SELECT * FROM university.courses;

-- Don't mix both in same application
```

### 4. Minimize Public Synonyms

- Use private synonyms when possible
- Reserve public synonyms for truly shared objects
- Reduces namespace pollution

### 5. Clean Up Unused Synonyms

```sql
-- Periodically review and drop unused synonyms
SELECT synonym_name, table_name
FROM user_synonyms
WHERE table_name NOT IN (SELECT table_name FROM user_tables);
```

## Performance Considerations

### Synonyms Have Minimal Overhead

- Synonyms are resolved at parse time
- No runtime performance impact
- Essentially zero overhead compared to direct access

### Example: Performance Equivalence

```sql
-- These have identical performance
SELECT * FROM university.students WHERE student_id = 1;
SELECT * FROM stud WHERE student_id = 1;  -- Using synonym
```

### When Synonyms May Matter

- **Remote synonyms**: Network latency from database link
- **Synonym chains**: Synonym → Synonym → Object (avoid this)
- **Name resolution**: Very slight overhead during parse phase (negligible)

## Summary

**Key Takeaways:**

1. **Synonyms are aliases for database objects** (tables, views, sequences, etc.) that provide simpler names and location transparency.

2. **Two types**: Private synonyms (visible to creator only) and public synonyms (visible to all users, requires special privilege).

3. **Main benefits**: Simplified references, schema independence, easier migrations, location transparency, and security abstraction.

4. **Common pattern**: Create synonyms to hide schema names, allowing `SELECT * FROM emp` instead of `SELECT * FROM hr.employees`.

5. **Resolution order**: Local objects → Private synonyms → Public synonyms (Oracle searches in this order).

6. **Synonyms don't copy data** - they're just alternative names pointing to the actual object; dropping a synonym doesn't affect the base object.

7. **Use cases**: Multi-environment deployment (dev/test/prod), schema migrations, remote database access, shortening long names, backward compatibility.

8. **Performance**: Synonyms have essentially zero runtime overhead; they're resolved during query parsing.

9. **Best practices**: Use descriptive names, prefer private over public, document usage, clean up unused synonyms, avoid synonym chains.

Synonyms are powerful tools for creating abstraction layers in database applications, enabling cleaner code and easier maintenance by hiding physical schema details from users and applications.

