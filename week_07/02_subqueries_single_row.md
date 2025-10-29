# Single-Row Subqueries

## Overview

A **single-row subquery** returns exactly one row with one or more columns. These subqueries are used with comparison operators (=, >, <, >=, <=, !=) and must always return a single value when used in scalar contexts.

## Key Terms

**Scalar Subquery**: A subquery that returns a single value (one row, one column).

**Comparison Operators**: Operators that compare values (=, !=, >, <, >=, <=).

**Aggregate Functions**: Functions that return a single value from multiple rows (AVG, MAX, MIN, SUM, COUNT).

**Single-Row Operator**: An operator that expects exactly one value on the right side of the comparison.

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

## Characteristics of Single-Row Subqueries

### Requirements

1. **Must return exactly one row** - If zero or multiple rows are returned, an error occurs
2. **Must return one column** when used with scalar comparison operators
3. **Can use any comparison operator**: =, !=, >, <, >=, <=
4. **Execute once** (non-correlated) or once per outer row (correlated)

### Syntax

```sql
SELECT column1, column2
FROM table1
WHERE column1 operator (SELECT column_x FROM table2 WHERE condition);
```

Where `operator` is one of: =, !=, >, <, >=, <=

## Common Use Cases

### Use Case 1: Comparing to Aggregates

The most common use of single-row subqueries is comparing values to aggregate results.

**Example: Find students with GPA above average**

```sql
SELECT first_name, last_name, gpa
FROM students
WHERE gpa > (SELECT AVG(gpa) FROM students)
ORDER BY gpa DESC;
```

**Execution:**
1. Inner query calculates: `AVG(gpa)` → 3.48
2. Outer query finds: students where `gpa > 3.48`

**Result:**
| first_name | last_name | gpa |
|------------|-----------|-----|
| Jane | Doe | 3.9 |
| John | Smith | 3.8 |
| Alice | Brown | 3.7 |

**Explanation:** The subquery calculates the average GPA (3.48) once, then the outer query filters students whose GPA exceeds this value.

### Use Case 2: Finding Maximum/Minimum Values

**Example: Find the highest-paid course instructor**

```sql
SELECT 
    i.instructor_name,
    i.salary,
    i.department
FROM instructors i
WHERE i.salary = (SELECT MAX(salary) FROM instructors);
```

**Why this works:**
- `MAX(salary)` returns a single value
- The = operator compares each instructor's salary to this maximum
- All instructors with the maximum salary are returned

### Use Case 3: Dynamic Filtering

**Example: Find courses with more credits than a specific course**

```sql
SELECT course_id, course_name, credits
FROM courses
WHERE credits > (
    SELECT credits 
    FROM courses 
    WHERE course_id = 'CS101'
)
ORDER BY credits DESC;
```

**Result:**
| course_id | course_name | credits |
|-----------|-------------|---------|
| CS201 | Data Structures | 4 |
| MATH101 | Calculus I | 4 |
| PHYS101 | Physics I | 4 |

**Benefit:** The reference point (CS101's credits) is determined dynamically from the database, not hardcoded.

## Comparison Operators with Single-Row Subqueries

### Equals (=)

**Example: Find students in the same major as John Smith**

```sql
SELECT first_name, last_name, major
FROM students
WHERE major = (
    SELECT major 
    FROM students 
    WHERE first_name = 'John' AND last_name = 'Smith'
)
AND (first_name != 'John' OR last_name != 'Smith');
```

**Result:**
| first_name | last_name | major |
|------------|-----------|-------|
| Bob | Wilson | Computer Science |

### Greater Than (>)

**Example: Find enrollments with grades above average for a specific course**

```sql
SELECT e.student_id, e.course_id, e.grade, e.grade_points
FROM enrollments e
WHERE e.course_id = 'CS101'
AND e.grade_points > (
    SELECT AVG(grade_points)
    FROM enrollments
    WHERE course_id = 'CS101'
    AND grade_points IS NOT NULL
);
```

### Not Equal (!=)

**Example: Find courses taught by instructors other than Dr. Johnson**

```sql
SELECT course_name, instructor_id
FROM courses
WHERE instructor_id IS NOT NULL
AND instructor_id != (
    SELECT instructor_id
    FROM instructors
    WHERE instructor_name = 'Dr. Johnson'
);
```

### Multiple Comparisons

**Example: Find students with GPA in the upper quartile**

```sql
SELECT first_name, last_name, gpa
FROM students
WHERE gpa >= (
    SELECT PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY gpa)
    FROM students
    WHERE gpa IS NOT NULL
)
ORDER BY gpa DESC;
```

## Aggregate Functions in Single-Row Subqueries

Single-row subqueries frequently use aggregate functions because aggregates naturally return single values.

### AVG - Average Value

**Example: Find courses with above-average credit hours**

```sql
SELECT course_id, course_name, credits
FROM courses
WHERE credits > (SELECT AVG(credits) FROM courses);
```

### MAX - Maximum Value

**Example: Find the most recently hired instructor**

```sql
SELECT instructor_name, hire_date, department
FROM instructors
WHERE hire_date = (SELECT MAX(hire_date) FROM instructors);
```

### MIN - Minimum Value

**Example: Find students with the lowest enrollment date (newest students)**

```sql
SELECT first_name, last_name, enrollment_date
FROM students
WHERE enrollment_date = (SELECT MIN(enrollment_date) FROM students);
```

**Result:**
| first_name | last_name | enrollment_date |
|------------|-----------|-----------------|
| Charlie | Davis | 2024-09-01 |

### COUNT - Row Count

**Example: Check if a course has more enrollments than the average course**

```sql
SELECT c.course_id, c.course_name,
       (SELECT COUNT(*) 
        FROM enrollments e 
        WHERE e.course_id = c.course_id) AS enrollment_count
FROM courses c
WHERE (SELECT COUNT(*) 
       FROM enrollments e 
       WHERE e.course_id = c.course_id) > 
      (SELECT AVG(course_enrollment_count)
       FROM (SELECT course_id, COUNT(*) AS course_enrollment_count
             FROM enrollments
             GROUP BY course_id));
```

### SUM - Total Value

**Example: Find departments spending more than average on salaries**

```sql
SELECT department,
       (SELECT SUM(salary) 
        FROM instructors i2 
        WHERE i2.department = i1.department) AS total_salary
FROM instructors i1
GROUP BY department
HAVING (SELECT SUM(salary) 
        FROM instructors i2 
        WHERE i2.department = i1.department) >
       (SELECT AVG(dept_total) 
        FROM (SELECT department, SUM(salary) AS dept_total
              FROM instructors
              GROUP BY department));
```

## Subqueries in Different Clauses

### In WHERE Clause (Most Common)

```sql
SELECT first_name, last_name
FROM students
WHERE gpa > (SELECT AVG(gpa) FROM students);
```

### In HAVING Clause

```sql
SELECT major, AVG(gpa) AS avg_gpa
FROM students
WHERE major IS NOT NULL
GROUP BY major
HAVING AVG(gpa) > (SELECT AVG(gpa) FROM students);
```

**Result:** Shows only majors whose average GPA is above the overall student average.

### In SELECT Clause

```sql
SELECT 
    first_name,
    last_name,
    gpa,
    gpa - (SELECT AVG(gpa) FROM students) AS gpa_difference_from_avg
FROM students
ORDER BY gpa_difference_from_avg DESC;
```

**Result:**
| first_name | last_name | gpa | gpa_difference_from_avg |
|------------|-----------|-----|-------------------------|
| Jane | Doe | 3.9 | 0.42 |
| John | Smith | 3.8 | 0.32 |
| Alice | Brown | 3.7 | 0.22 |
| Bob | Wilson | 3.2 | -0.28 |
| Charlie | Davis | 2.8 | -0.68 |

## Practical Examples

### Example 1: Comparative Analysis

**Query: Find courses that are worth more credits than the average course**

```sql
SELECT 
    course_id,
    course_name,
    credits,
    credits - (SELECT AVG(credits) FROM courses) AS credits_above_avg
FROM courses
WHERE credits > (SELECT AVG(credits) FROM courses)
ORDER BY credits DESC;
```

**Business Value:** Identifies high-workload courses for curriculum planning.

### Example 2: Performance Benchmarking

**Query: Find students performing above their major's average**

```sql
SELECT 
    s.first_name,
    s.last_name,
    s.major,
    s.gpa,
    (SELECT AVG(gpa) 
     FROM students s2 
     WHERE s2.major = s.major) AS major_avg_gpa
FROM students s
WHERE s.major IS NOT NULL
AND s.gpa > (SELECT AVG(gpa) 
             FROM students s2 
             WHERE s2.major = s.major)
ORDER BY s.major, s.gpa DESC;
```

**Note:** This is a correlated subquery - it references `s.major` from the outer query.

### Example 3: Threshold-Based Filtering

**Query: Find high-enrollment courses (more than average)**

```sql
SELECT 
    c.course_id,
    c.course_name,
    COUNT(e.enrollment_id) AS enrollment_count
FROM courses c
LEFT JOIN enrollments e ON c.course_id = e.course_id
GROUP BY c.course_id, c.course_name
HAVING COUNT(e.enrollment_id) > (
    SELECT AVG(course_count)
    FROM (
        SELECT COUNT(*) AS course_count
        FROM enrollments
        GROUP BY course_id
    )
);
```

## Common Errors and Solutions

### Error 1: Multiple Rows Returned

**Problem:**
```sql
-- ERROR: single-row subquery returns more than one row
SELECT course_name
FROM courses
WHERE instructor_id = (SELECT instructor_id FROM instructors);
```

**Solution:** Use IN for multiple-row results:
```sql
SELECT course_name
FROM courses
WHERE instructor_id IN (SELECT instructor_id FROM instructors);
```

### Error 2: No Rows Returned

**Problem:**
```sql
-- Returns NULL if no matching course exists
SELECT credits
FROM courses
WHERE course_id = 'NONEXISTENT';
```

**Solution:** Use COALESCE or CASE to handle NULL:
```sql
SELECT course_name, credits
FROM courses
WHERE credits > COALESCE(
    (SELECT credits FROM courses WHERE course_id = 'NONEXISTENT'),
    0
);
```

### Error 3: NULL Comparison

**Problem:**
```sql
-- May not work as expected if subquery returns NULL
SELECT first_name
FROM students
WHERE gpa > (SELECT gpa FROM students WHERE student_id = 999);
```

**Result:** If student 999 doesn't exist, the subquery returns NULL, and all comparisons with NULL return NULL (no rows match).

**Solution:** Handle NULL explicitly:
```sql
SELECT first_name, gpa
FROM students
WHERE gpa > COALESCE(
    (SELECT gpa FROM students WHERE student_id = 999),
    0
);
```

## Performance Considerations

### Efficient Single-Row Subqueries

**Good: Aggregate subqueries run once**
```sql
SELECT first_name, gpa
FROM students
WHERE gpa > (SELECT AVG(gpa) FROM students);
-- Subquery executes once
```

**Less Efficient: Correlated subquery runs per row**
```sql
SELECT s.first_name, s.gpa
FROM students s
WHERE s.gpa > (SELECT AVG(gpa) FROM students s2 WHERE s2.major = s.major);
-- Subquery executes once per student
```

### Optimization Tips

1. **Ensure subquery has proper indexes** on columns used in WHERE clause
2. **Avoid correlated subqueries when possible** - consider joins or analytic functions
3. **Use aggregate functions efficiently** - they naturally return single rows
4. **Cache results when possible** - store subquery result in a variable (PL/SQL)

## Summary

**Key Takeaways:**

1. **Single-row subqueries return exactly one row and one column** when used with scalar comparison operators (=, >, <, etc.).

2. **Most commonly used with aggregate functions** (AVG, MAX, MIN, SUM, COUNT) which naturally return single values.

3. **Can appear in WHERE, HAVING, SELECT, and FROM clauses**, providing flexible filtering and calculation options.

4. **Comparison operators include**: = (equals), != (not equals), > (greater than), < (less than), >= (greater or equal), <= (less or equal).

5. **Common patterns**: comparing to averages, finding maximum/minimum values, dynamic threshold filtering, and performance benchmarking.

6. **Watch for errors**: Ensure subquery returns exactly one row, handle NULL results properly, and avoid using = with multi-row results.

7. **Performance**: Non-correlated single-row subqueries with aggregates are typically efficient as they execute only once.

Single-row subqueries are fundamental building blocks for creating dynamic, data-driven SQL queries that adapt to changing database contents.

