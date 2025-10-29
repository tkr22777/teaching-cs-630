# Introduction to Subqueries

## Overview

A **subquery** (also called an inner query or nested query) is a query embedded within another SQL statement. Subqueries allow you to break complex queries into logical steps and perform operations that would otherwise require multiple separate queries or complex joins.

## Key Terms

**Subquery**: A SELECT statement nested inside another SQL statement (SELECT, INSERT, UPDATE, DELETE, or within a WHERE/HAVING clause).

**Outer Query**: The main query that contains the subquery.

**Inner Query**: The subquery that executes first and provides results to the outer query.

**Scalar Subquery**: A subquery that returns exactly one row and one column (a single value).

**Row Subquery**: A subquery that returns a single row with multiple columns.

**Table Subquery**: A subquery that returns multiple rows and columns (a result set).

**Correlated Subquery**: A subquery that references columns from the outer query and executes once for each row processed by the outer query.

**Non-Correlated Subquery**: A subquery that is independent of the outer query and executes only once.

## What Are Subqueries?

Subqueries are queries nested within other queries. They provide a way to:
- Use the result of one query as input to another
- Break complex problems into manageable parts
- Perform comparisons against calculated or filtered values
- Create dynamic conditions based on data

### Basic Structure

```sql
SELECT column1, column2
FROM table1
WHERE column1 = (SELECT column_x FROM table2 WHERE condition);
                 └─────── Subquery ───────────┘
```

The subquery executes first, and its result is used by the outer query.

## Why Use Subqueries?

### Benefits

1. **Logical Decomposition** - Break complex queries into understandable steps
2. **Dynamic Filtering** - Create conditions based on calculated values
3. **Readability** - Make query logic clearer and more maintainable
4. **Flexibility** - Solve problems that are difficult with joins alone
5. **Modularity** - Build reusable query components

### Real-World Scenarios

**Scenario 1: Finding Above-Average Values**
```sql
-- Find all students with GPAs above the average
SELECT student_name, gpa
FROM students
WHERE gpa > (SELECT AVG(gpa) FROM students);
```

**Scenario 2: Comparing Against Maximum/Minimum**
```sql
-- Find the employee(s) with the highest salary
SELECT employee_name, salary
FROM employees
WHERE salary = (SELECT MAX(salary) FROM employees);
```

**Scenario 3: Checking for Existence**
```sql
-- Find customers who have placed at least one order
SELECT customer_name
FROM customers
WHERE customer_id IN (SELECT DISTINCT customer_id FROM orders);
```

## Types of Subqueries

### By Result Type

| Type | Returns | Used With | Example |
|------|---------|-----------|---------|
| **Single-Row** | One row, one column | =, >, <, >=, <=, != | `WHERE price = (SELECT AVG(price) FROM products)` |
| **Multiple-Row** | Multiple rows, one column | IN, ANY, ALL | `WHERE dept_id IN (SELECT dept_id FROM departments WHERE location = 'NY')` |
| **Multiple-Column** | One or more rows, multiple columns | IN, comparison operators | `WHERE (dept_id, salary) IN (SELECT dept_id, MAX(salary) FROM employees GROUP BY dept_id)` |

### By Location

| Location | Description | Example |
|----------|-------------|---------|
| **WHERE clause** | Most common, filters rows | `WHERE salary > (SELECT AVG(salary) FROM employees)` |
| **HAVING clause** | Filters grouped results | `HAVING COUNT(*) > (SELECT AVG(order_count) FROM ...)` |
| **FROM clause** | Creates inline view/derived table | `FROM (SELECT dept_id, AVG(salary) AS avg_sal FROM employees GROUP BY dept_id) dept_avg` |
| **SELECT clause** | Returns calculated values | `SELECT name, (SELECT COUNT(*) FROM orders WHERE orders.cust_id = customers.id) AS order_count` |

### By Dependency

| Type | Description | Execution | Performance |
|------|-------------|-----------|-------------|
| **Non-Correlated** | Independent of outer query | Once | Usually faster |
| **Correlated** | References outer query columns | Once per outer row | Can be slower |

## Sample Database Schema

For all examples in this module, we'll continue using the university enrollment system:

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

### Students Table
| student_id | first_name | last_name | email | major | enrollment_date | gpa |
|------------|------------|-----------|-------|-------|-----------------|-----|
| 1 | John | Smith | john.smith@university.edu | Computer Science | 2023-09-01 | 3.8 |
| 2 | Jane | Doe | jane.doe@university.edu | Mathematics | 2023-09-01 | 3.9 |
| 3 | Bob | Wilson | bob.wilson@university.edu | Computer Science | 2024-01-15 | 3.2 |
| 4 | Alice | Brown | alice.brown@university.edu | Physics | 2024-01-15 | 3.7 |
| 5 | Charlie | Davis | charlie.davis@university.edu | NULL | 2024-09-01 | 2.8 |

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
| enrollment_id | student_id | course_id | semester | grade | grade_points |
|---------------|------------|-----------|----------|-------|--------------|
| 101 | 1 | CS101 | Fall 2023 | A | 4.0 |
| 102 | 1 | CS201 | Spring 2024 | B+ | 3.3 |
| 103 | 2 | MATH101 | Fall 2023 | A | 4.0 |
| 104 | 2 | CS101 | Fall 2023 | A- | 3.7 |
| 105 | 3 | CS101 | Spring 2024 | B | 3.0 |
| 106 | 3 | CS201 | Spring 2024 | B+ | 3.3 |
| 107 | 4 | PHYS101 | Spring 2024 | A | 4.0 |
| 108 | 1 | CS301 | Fall 2024 | NULL | NULL |

## Basic Subquery Examples

### Example 1: Single-Row Subquery

**Query:** Find all courses with more credits than CS101.

```sql
SELECT course_name, credits
FROM courses
WHERE credits > (SELECT credits FROM courses WHERE course_id = 'CS101');
```

**Execution Steps:**
1. Inner query executes: `SELECT credits FROM courses WHERE course_id = 'CS101'` → returns 3
2. Outer query executes: `SELECT course_name, credits FROM courses WHERE credits > 3`

**Result:**
| course_name | credits |
|-------------|---------|
| Data Structures | 4 |
| Calculus I | 4 |
| Physics I | 4 |

**What happened:** The subquery found that CS101 has 3 credits, then the outer query found all courses with more than 3 credits.

### Example 2: Multiple-Row Subquery with IN

**Query:** Find all students enrolled in Computer Science courses.

```sql
SELECT first_name, last_name, major
FROM students
WHERE student_id IN (
    SELECT DISTINCT student_id
    FROM enrollments e
    JOIN courses c ON e.course_id = c.course_id
    WHERE c.department = 'Computer Science'
);
```

**Execution Steps:**
1. Inner query finds student_ids enrolled in CS courses → returns (1, 2, 3)
2. Outer query retrieves student details for those IDs

**Result:**
| first_name | last_name | major |
|------------|-----------|-------|
| John | Smith | Computer Science |
| Jane | Doe | Mathematics |
| Bob | Wilson | Computer Science |

### Example 3: Subquery in SELECT Clause

**Query:** Show each student with their enrollment count.

```sql
SELECT 
    first_name,
    last_name,
    (SELECT COUNT(*) 
     FROM enrollments e 
     WHERE e.student_id = s.student_id) AS enrollment_count
FROM students s
ORDER BY last_name;
```

**Result:**
| first_name | last_name | enrollment_count |
|------------|-----------|------------------|
| Alice | Brown | 1 |
| Charlie | Davis | 0 |
| Jane | Doe | 2 |
| John | Smith | 3 |
| Bob | Wilson | 2 |

**Note:** This is a correlated subquery - it references `s.student_id` from the outer query and executes once for each student.

## Subquery vs. JOIN

Many queries can be written using either subqueries or joins. Here's a comparison:

### Using Subquery
```sql
SELECT first_name, last_name
FROM students
WHERE student_id IN (
    SELECT student_id 
    FROM enrollments 
    WHERE course_id = 'CS101'
);
```

### Using JOIN
```sql
SELECT DISTINCT s.first_name, s.last_name
FROM students s
JOIN enrollments e ON s.student_id = e.student_id
WHERE e.course_id = 'CS101';
```

### When to Use Which?

| Use Subquery When | Use JOIN When |
|-------------------|---------------|
| Logic is clearer as separate steps | You need columns from multiple tables |
| Testing for existence/non-existence | You need to combine data horizontally |
| Result is naturally hierarchical | Performance is better with JOIN |
| Query reads more like English | Multiple tables interact equally |

**Performance Note:** Modern database optimizers often convert between subqueries and joins automatically, so choose based on readability and maintainability.

## Common Subquery Patterns

### Pattern 1: Comparing to Aggregates
```sql
-- Find students with GPA above average
SELECT first_name, last_name, gpa
FROM students
WHERE gpa > (SELECT AVG(gpa) FROM students);
```

### Pattern 2: Finding Unmatched Records
```sql
-- Find students with no enrollments
SELECT first_name, last_name
FROM students
WHERE student_id NOT IN (
    SELECT DISTINCT student_id 
    FROM enrollments
);
```

### Pattern 3: Finding Duplicates
```sql
-- Find courses with multiple enrollments
SELECT course_id, course_name
FROM courses
WHERE course_id IN (
    SELECT course_id
    FROM enrollments
    GROUP BY course_id
    HAVING COUNT(*) > 1
);
```

## Subquery Rules and Restrictions

### Important Rules

1. **Subqueries must return appropriate result size**
   - Single-row subqueries (with =, >, <) must return exactly one row
   - Multiple-row subqueries require IN, ANY, or ALL

2. **Column count must match**
   - Multi-column comparisons require matching column counts
   - Example: `WHERE (col1, col2) IN (SELECT col_a, col_b FROM ...)`

3. **Parentheses are required**
   - Every subquery must be enclosed in parentheses
   - Example: `WHERE id = (SELECT ...)`

4. **ORDER BY in subqueries**
   - Usually not needed (outer query determines final order)
   - Only useful with ROWNUM or FETCH FIRST

### Common Errors

**Error 1: Too Many Rows**
```sql
-- ERROR: subquery returns more than one row
SELECT name
FROM students
WHERE student_id = (SELECT student_id FROM enrollments);
```
**Fix:** Use IN instead of =
```sql
SELECT name
FROM students
WHERE student_id IN (SELECT student_id FROM enrollments);
```

**Error 2: NULL Handling**
```sql
-- May not return expected results if subquery returns NULL
SELECT name
FROM students
WHERE student_id NOT IN (SELECT student_id FROM enrollments WHERE grade IS NULL);
```
**Fix:** Use NOT EXISTS or filter out NULLs
```sql
SELECT name
FROM students s
WHERE NOT EXISTS (
    SELECT 1 
    FROM enrollments e 
    WHERE e.student_id = s.student_id
);
```

## Summary

**Key Takeaways:**

1. **Subqueries are queries nested within other queries** that execute first and provide results to the outer query.

2. **Three main types by result**: Single-row (one value), multiple-row (list of values), and multiple-column (table-like results).

3. **Two types by dependency**: Non-correlated (independent, runs once) and correlated (dependent, runs per outer row).

4. **Common uses include**: comparing to aggregates, filtering against dynamic lists, testing for existence, and breaking complex logic into steps.

5. **Subqueries can appear in**: WHERE clauses (filtering), FROM clauses (derived tables), SELECT clauses (calculated columns), and HAVING clauses (group filtering).

6. **Choose between subqueries and joins** based on readability, maintainability, and which best expresses your logical intent.

7. **Watch for errors**: Match operator to result size (= for single-row, IN for multiple-row), handle NULLs carefully, and always use parentheses.

In the following sections, we'll explore each type of subquery in detail with practical examples and real-world applications.

