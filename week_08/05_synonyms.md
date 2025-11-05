# SQL Synonyms

## Overview

A **synonym** is an alias for a database object (table, view, sequence, etc.). Synonyms provide location transparency and simplify object references, especially across schemas or databases.

## Key Terms

**Synonym**: An alternative name for a database object.

**Private Synonym**: Synonym accessible only to the creating user.

**Public Synonym**: Synonym accessible to all database users.

**Base Object**: The actual object that a synonym references.

**Name Resolution**: Process of resolving a synonym to its underlying object.

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

**Benefits:**
- **Simplify object names** - Use short names instead of `schema.object_name`
- **Location transparency** - Hide actual location of objects
- **Ease migrations** - Point to different objects without changing code
- **Abstract complexity** - Hide schema organization from users
- **Centralize changes** - Update one synonym instead of many references

**Common use cases:**
- Accessing tables from another schema
- Simplifying long object names
- Supporting multiple environments (dev, test, prod)
- Providing backward compatibility after renaming objects

## Types of Synonyms

| Type | Scope | Created By | Accessible By |
|------|-------|------------|---------------|
| **Private** | Single user | Current user | Only creator |
| **Public** | All users | DBA (requires privilege) | All users |

**Name resolution order:**
1. Local object in current schema
2. Private synonym
3. Public synonym

## Creating Synonyms

### Private Synonym

**Syntax:**
```sql
CREATE SYNONYM synonym_name FOR object_name;
```

**Example 1: Simple shorthand**

```sql
-- Create synonym for local table
CREATE SYNONYM studs FOR students;

-- Use it
SELECT * FROM studs WHERE student_id = 1;
```

**Example 2: Access another schema's table**

```sql
-- Assuming HR schema has employees table
CREATE SYNONYM emp FOR hr.employees;

-- Use it (no need to specify HR schema)
SELECT * FROM emp;
```

### Public Synonym

**Syntax:**
```sql
CREATE PUBLIC SYNONYM synonym_name FOR object_name;
```

**Requires:** `CREATE PUBLIC SYNONYM` privilege (typically DBA)

**Example:**

```sql
-- Create public synonym accessible to all users
CREATE PUBLIC SYNONYM all_students FOR students;

-- Any user can now use
SELECT * FROM all_students;
```

### Synonyms for Other Objects

**Synonyms work for:**
- Tables
- Views
- Sequences
- Other synonyms (synonym chains)

**Example: Synonym for a view**

```sql
-- Create a view
CREATE VIEW active_students AS
SELECT * FROM students WHERE enrollment_date > DATE '2024-01-01';

-- Create synonym for the view
CREATE SYNONYM active_studs FOR active_students;

-- Use it
SELECT * FROM active_studs;
```

**Example: Synonym for a sequence**

```sql
-- Create sequence
CREATE SEQUENCE student_id_seq START WITH 100;

-- Create synonym
CREATE SYNONYM next_student_id FOR student_id_seq;

-- Use it
SELECT next_student_id.NEXTVAL FROM DUAL;
```

## Using Synonyms

Once created, use synonyms exactly like the base object:

```sql
-- SELECT
SELECT * FROM studs;

-- INSERT
INSERT INTO studs (student_id, first_name, last_name, email, major, enrollment_date, gpa)
VALUES (100, 'Test', 'User', 'test@email.com', 'CS', SYSDATE, 3.0);

-- UPDATE
UPDATE studs SET gpa = 3.5 WHERE student_id = 100;

-- DELETE
DELETE FROM studs WHERE student_id = 100;

-- JOINs
SELECT s.first_name, e.course_id
FROM studs s
JOIN enrollments e ON s.student_id = e.student_id;
```

## Viewing Synonyms

**Query your private synonyms:**

```sql
SELECT synonym_name, table_owner, table_name
FROM user_synonyms
ORDER BY synonym_name;
```

**Query all synonyms (including public):**

```sql
SELECT owner, synonym_name, table_owner, table_name
FROM all_synonyms
WHERE table_name = 'STUDENTS'
ORDER BY owner, synonym_name;
```

**Check if synonym exists:**

```sql
SELECT COUNT(*)
FROM user_synonyms
WHERE synonym_name = 'STUDS';
```

## Dropping Synonyms

**Drop private synonym:**

```sql
DROP SYNONYM synonym_name;
```

**Example:**
```sql
DROP SYNONYM studs;
```

**Drop public synonym:**

```sql
DROP PUBLIC SYNONYM synonym_name;
```

**Example:**
```sql
DROP PUBLIC SYNONYM all_students;
```

**Important notes:**
- Dropping a synonym does NOT drop the base object
- Dropping the base object does NOT drop synonyms (they become invalid)
- In Oracle 23c+, can use `DROP SYNONYM IF EXISTS`

## Practical Example: Multi-Environment Setup

**Problem:** Different table names in dev, test, and prod environments.

**Solution:** Use synonyms to abstract the environment.

```sql
-- Development environment
CREATE SYNONYM app_students FOR dev_students;

-- Test environment
CREATE SYNONYM app_students FOR test_students;

-- Production environment
CREATE SYNONYM app_students FOR prod_students;

-- Application code always uses
SELECT * FROM app_students;
```

The application code stays the same across environments.

## Common Mistakes

**Mistake 1: Dropping base object**

```sql
-- Create synonym
CREATE SYNONYM studs FOR students;

-- Drop base table
DROP TABLE students;

-- Synonym still exists but is invalid
SELECT * FROM studs;  -- ERROR: table or view does not exist
```

**Fix:** Drop synonym when dropping base object, or document the dependency.

**Mistake 2: Circular references**

```sql
-- Don't create circular chains
CREATE SYNONYM syn1 FOR syn2;
CREATE SYNONYM syn2 FOR syn1;  -- Creates infinite loop
```

**Mistake 3: Name conflicts**

```sql
-- If both table and synonym exist with same name
CREATE TABLE emp (...);
CREATE SYNONYM emp FOR hr.employees;  -- ERROR: name already used

-- Resolution: Use different names
CREATE SYNONYM emp_data FOR hr.employees;
```

## Best Practices

1. **Use descriptive names** - Make synonym purpose clear
2. **Document synonym mappings** - Maintain list of synonyms and their base objects
3. **Minimize public synonyms** - Use private synonyms when possible
4. **Clean up unused synonyms** - Remove when no longer needed
5. **Use for abstraction** - Not just for convenience; use with purpose
6. **Avoid long chains** - synonym → synonym → table is confusing

## Performance

**Good news:** Synonyms have minimal performance overhead. Oracle resolves them at parse time.

```sql
-- These perform identically
SELECT * FROM students WHERE student_id = 1;
SELECT * FROM studs WHERE student_id = 1;  -- Using synonym
```

The execution plan is the same for both queries.

## Summary

**Key Points:**

1. **Synonyms are aliases** for database objects (tables, views, sequences)
2. **Two types**: Private (one user) and Public (all users)
3. **CREATE SYNONYM** to create, **DROP SYNONYM** to remove
4. **Use like base object** - SELECT, INSERT, UPDATE, DELETE all work
5. **Benefits**: Simplify names, hide locations, ease migrations
6. **Name resolution order**: Local object → Private synonym → Public synonym
7. **Dropping synonym** does NOT drop base object
8. **Performance**: Minimal overhead (resolved at parse time)
9. **Best practice**: Use for abstraction and environment independence

Synonyms provide a simple yet powerful way to abstract and simplify database object references.
