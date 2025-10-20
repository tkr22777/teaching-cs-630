# Introduction to SQL Joins

## Overview

Joins are fundamental SQL operations that combine rows from two or more tables based on related columns. Understanding joins is essential for working with normalized relational databases.

## What Are Joins?

In relational databases, data is organized into multiple related tables to:
- Reduce data redundancy
- Improve data integrity
- Follow normalization principles
- Make updates and maintenance easier

**Joins** allow us to query data across these related tables and present results as if the data were in a single table.

## Why Use Joins?

### Benefits of Using Joins:

1. **Retrieve Related Data** - Access information spread across multiple tables in a single query
2. **Maintain Normalized Structure** - Keep tables organized without duplicating data
3. **Data Integrity** - Ensure relationships between tables remain consistent
4. **Flexible Queries** - Combine data in various ways depending on business needs

### Example Scenario:

Imagine an e-commerce database:
- **Customers Table** stores customer information
- **Orders Table** stores order details
- **Products Table** stores product information

Without joins, you'd need:
- 3 separate queries to get customer name, order date, and product name
- Application-level code to combine results

With joins:
- 1 query to get all related information at once

## Join Syntax

### ANSI Standard Syntax (Preferred)

```sql
SELECT columns
FROM table1
JOIN table2 ON table1.column = table2.column;
```

**Advantages:**
- Explicit and clear
- Separates join conditions from filter conditions
- Better readability
- Industry standard

## Types of Joins

### 1. INNER JOIN
Returns only rows with matches in both tables.

**Use when:** You need data that exists in both tables.

```sql
SELECT *
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id;
```

### 2. LEFT JOIN (LEFT OUTER JOIN)
Returns all rows from the left table and matching rows from the right table.

**Use when:** You need all records from the primary table, even if there are no matches.

```sql
SELECT *
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id;
```

### 3. RIGHT JOIN (RIGHT OUTER JOIN)
Returns all rows from the right table and matching rows from the left table.

**Use when:** Similar to LEFT JOIN but with reversed tables (less common).

```sql
SELECT *
FROM orders o
RIGHT JOIN customers c ON o.customer_id = c.customer_id;
```

### 4. FULL OUTER JOIN
Returns all rows from both tables, with NULLs where there are no matches.

**Use when:** You need everything from both tables regardless of matches.

```sql
SELECT *
FROM customers c
FULL OUTER JOIN orders o ON c.customer_id = o.customer_id;
```

### 5. CROSS JOIN
Returns the Cartesian product (all possible combinations).

**Use when:** You need all possible pairings between two tables.

```sql
SELECT *
FROM products
CROSS JOIN colors;
```

### 6. SELF JOIN
A table joined with itself.

**Use when:** Comparing rows within the same table or handling hierarchical data.

```sql
SELECT e1.name AS employee, e2.name AS manager
FROM employees e1
JOIN employees e2 ON e1.manager_id = e2.employee_id;
```

## Sample Database Schema

For all examples in this course module, we'll use a university enrollment system:

<details>
<summary>Click to expand: Database setup script</summary>

```sql
-- Create Students table
CREATE TABLE students (
    student_id INTEGER PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    major VARCHAR(50),
    enrollment_date DATE DEFAULT CURRENT_DATE
);

-- Create Instructors table
CREATE TABLE instructors (
    instructor_id INTEGER PRIMARY KEY,
    instructor_name VARCHAR(100) NOT NULL,
    department VARCHAR(50),
    hire_date DATE
);

-- Create Courses table
CREATE TABLE courses (
    course_id VARCHAR(10) PRIMARY KEY,
    course_name VARCHAR(100) NOT NULL,
    department VARCHAR(50),
    credits INTEGER,
    instructor_id INTEGER REFERENCES instructors(instructor_id)
);

-- Create Enrollments table (junction table)
CREATE TABLE enrollments (
    enrollment_id INTEGER PRIMARY KEY,
    student_id INTEGER REFERENCES students(student_id),
    course_id VARCHAR(10) REFERENCES courses(course_id),
    semester VARCHAR(20),
    grade VARCHAR(5)
);

-- Insert Students
INSERT INTO students (student_id, first_name, last_name, email, major, enrollment_date) VALUES
(1, 'John', 'Smith', 'john.smith@university.edu', 'Computer Science', '2023-09-01'),
(2, 'Jane', 'Doe', 'jane.doe@university.edu', 'Mathematics', '2023-09-01'),
(3, 'Bob', 'Wilson', 'bob.wilson@university.edu', 'Computer Science', '2024-01-15'),
(4, 'Alice', 'Brown', 'alice.brown@university.edu', 'Physics', '2024-01-15'),
(5, 'Charlie', 'Davis', 'charlie.davis@university.edu', NULL, '2024-09-01');

-- Insert Instructors
INSERT INTO instructors (instructor_id, instructor_name, department, hire_date) VALUES
(10, 'Dr. Johnson', 'Computer Science', '2018-08-15'),
(11, 'Dr. Lee', 'Mathematics', '2019-01-10'),
(12, 'Dr. Martinez', 'Physics', '2020-09-01'),
(13, 'Dr. Taylor', 'Chemistry', '2021-06-15');

-- Insert Courses
INSERT INTO courses (course_id, course_name, department, credits, instructor_id) VALUES
('CS101', 'Introduction to Programming', 'Computer Science', 3, 10),
('CS201', 'Data Structures', 'Computer Science', 4, 10),
('MATH101', 'Calculus I', 'Mathematics', 4, 11),
('PHYS101', 'Physics I', 'Physics', 4, 12),
('CS301', 'Database Systems', 'Computer Science', 3, 10),
('ENG101', 'English Composition', 'English', 3, NULL);

-- Insert Enrollments
INSERT INTO enrollments (enrollment_id, student_id, course_id, semester, grade) VALUES
(101, 1, 'CS101', 'Fall 2023', 'A'),
(102, 1, 'CS201', 'Spring 2024', 'B+'),
(103, 2, 'MATH101', 'Fall 2023', 'A'),
(104, 2, 'CS101', 'Fall 2023', 'A-'),
(105, 3, 'CS101', 'Spring 2024', 'B'),
(106, 3, 'CS201', 'Spring 2024', 'B+'),
(107, 4, 'PHYS101', 'Spring 2024', 'A'),
(108, 1, 'CS301', 'Fall 2024', NULL);
```

</details>

### Students Table
| student_id | first_name | last_name | email | major | enrollment_date |
|------------|------------|-----------|-------|-------|-----------------|
| 1 | John | Smith | john.smith@university.edu | Computer Science | 2023-09-01 |
| 2 | Jane | Doe | jane.doe@university.edu | Mathematics | 2023-09-01 |
| 3 | Bob | Wilson | bob.wilson@university.edu | Computer Science | 2024-01-15 |
| 4 | Alice | Brown | alice.brown@university.edu | Physics | 2024-01-15 |
| 5 | Charlie | Davis | charlie.davis@university.edu | NULL | 2024-09-01 |

### Instructors Table
| instructor_id | instructor_name | department | hire_date |
|---------------|-----------------|------------|-----------|
| 10 | Dr. Johnson | Computer Science | 2018-08-15 |
| 11 | Dr. Lee | Mathematics | 2019-01-10 |
| 12 | Dr. Martinez | Physics | 2020-09-01 |
| 13 | Dr. Taylor | Chemistry | 2021-06-15 |

### Courses Table
| course_id | course_name | department | credits | instructor_id |
|-----------|-------------|------------|---------|---------------|
| CS101 | Introduction to Programming | Computer Science | 3 | 10 |
| CS201 | Data Structures | Computer Science | 4 | 10 |
| MATH101 | Calculus I | Mathematics | 4 | 11 |
| PHYS101 | Physics I | Physics | 4 | 12 |
| CS301 | Database Systems | Computer Science | 3 | 10 |
| ENG101 | English Composition | English | 3 | NULL |

### Enrollments Table
| enrollment_id | student_id | course_id | semester | grade |
|---------------|------------|-----------|----------|-------|
| 101 | 1 | CS101 | Fall 2023 | A |
| 102 | 1 | CS201 | Spring 2024 | B+ |
| 103 | 2 | MATH101 | Fall 2023 | A |
| 104 | 2 | CS101 | Fall 2023 | A- |
| 105 | 3 | CS101 | Spring 2024 | B |
| 106 | 3 | CS201 | Spring 2024 | B+ |
| 107 | 4 | PHYS101 | Spring 2024 | A |
| 108 | 1 | CS301 | Fall 2024 | NULL |

## Database Relationships

### One-to-Many Relationships

**Instructors → Courses**
- One instructor can teach multiple courses
- Each course has one instructor (or none)

**Students → Enrollments**
- One student can have multiple enrollments
- Each enrollment belongs to one student

**Courses → Enrollments**
- One course can have multiple enrollments
- Each enrollment is for one course

### Many-to-Many Relationships

**Students ↔ Courses** (through Enrollments)
- One student can enroll in multiple courses
- One course can have multiple students
- The `enrollments` table is a **junction table** that implements this relationship

## Key Concepts

### Primary Key
A column (or combination of columns) that uniquely identifies each row in a table.

Examples:
- `students.student_id`
- `courses.course_id`
- `enrollments.enrollment_id`

### Foreign Key
A column that references the primary key of another table, establishing a relationship.

Examples:
- `enrollments.student_id` references `students.student_id`
- `enrollments.course_id` references `courses.course_id`
- `courses.instructor_id` references `instructors.instructor_id`

### Join Condition
The criteria used to match rows from different tables, typically using the ON clause.

Example:
```sql
ON students.student_id = enrollments.student_id
```

### Table Alias
A temporary shorthand name for a table used in queries.

Example:
```sql
FROM students s  -- 's' is the alias for 'students'
JOIN enrollments e ON s.student_id = e.student_id
```

**Why use aliases:**
- Makes queries shorter and more readable
- Required for self-joins
- Makes column references clearer

## Simple Join Example

Let's see a basic join in action:

**Query:** Get student names with their enrolled course names.

```sql
SELECT s.first_name,
       s.last_name,
       c.course_name,
       e.grade
FROM students s
JOIN enrollments e ON s.student_id = e.student_id
JOIN courses c ON e.course_id = c.course_id
ORDER BY s.last_name, s.first_name;
```

**Result:**
| first_name | last_name | course_name | grade |
|------------|-----------|-------------|-------|
| Alice | Brown | Physics I | A |
| Jane | Doe | Calculus I | A |
| Jane | Doe | Introduction to Programming | A- |
| John | Smith | Introduction to Programming | A |
| John | Smith | Data Structures | B+ |
| John | Smith | Database Systems | NULL |
| Bob | Wilson | Introduction to Programming | B |
| Bob | Wilson | Data Structures | B+ |

**What happened:**
1. Started with students table (s)
2. Joined enrollments (e) where student IDs match
3. Joined courses (c) where course IDs match
4. Selected specific columns from each table
5. Only students with enrollments appear (Charlie Davis excluded)

## Common Join Mistakes

### Mistake 1: Missing Join Condition (Cartesian Product)

**Problem:**
```sql
SELECT s.first_name, c.course_name
FROM students s, courses c;  -- No join condition!
```

**Result:** 5 students × 6 courses = 30 rows (every student paired with every course)

**Solution:**
```sql
SELECT s.first_name, c.course_name
FROM students s
JOIN enrollments e ON s.student_id = e.student_id
JOIN courses c ON e.course_id = c.course_id;
```

### Mistake 2: Ambiguous Column Names

**Problem:**
```sql
SELECT student_id, course_id
FROM students s
JOIN enrollments e ON s.student_id = e.student_id;
-- ERROR: column "student_id" is ambiguous
```

**Solution:** Qualify column names with table aliases:
```sql
SELECT s.student_id, e.course_id
FROM students s
JOIN enrollments e ON s.student_id = e.student_id;
```

