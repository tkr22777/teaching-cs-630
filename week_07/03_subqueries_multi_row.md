# Multiple-Row Subqueries

## Overview

A **multiple-row subquery** returns zero or more rows. These require special operators: IN, NOT IN, ANY, or ALL (EXISTS covered separately).

## Key Terms

**Multiple-Row Subquery**: A subquery returning zero, one, or more rows.

**IN Operator**: Tests if a value matches any value in a list.

**ANY Operator**: Compares a value to each subquery value; TRUE if any comparison is true.

**ALL Operator**: Compares a value to each subquery value; TRUE only if all comparisons are true.

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

## Multiple-Row Operators

| Operator | Description | Example |
|----------|-------------|---------|
| **IN** | Matches any value in list | `WHERE id IN (1, 2, 3)` |
| **NOT IN** | Does not match any value | `WHERE id NOT IN (4, 5)` |
| **ANY** | Compare to any value (>, <, =, etc.) | `WHERE salary > ANY (...)` |
| **ALL** | Compare to all values (all must match) | `WHERE salary > ALL (...)` |

**Quick Reference:**
- `= ANY` is same as `IN`
- `!= ALL` is same as `NOT IN`
- `> ANY` means "greater than minimum"
- `> ALL` means "greater than maximum"

## IN Operator

Tests if a value matches any value in the subquery result.

**Syntax:**
```sql
WHERE column IN (subquery)
```

**Example: Find students enrolled in Computer Science courses**

```sql
SELECT first_name, last_name, major
FROM students
WHERE student_id IN (
    SELECT DISTINCT e.student_id
    FROM enrollments e
    JOIN courses c ON e.course_id = c.course_id
    WHERE c.department = 'Computer Science'
);
```

**Result:**
| first_name | last_name | major |
|------------|-----------|-------|
| John | Smith | Computer Science |
| Jane | Doe | Mathematics |
| Bob | Wilson | Computer Science |

**How it works:**
1. Inner query returns student IDs: `(1, 2, 3)`
2. Outer query finds students where `student_id` matches any of these values

## NOT IN Operator

Tests if a value does NOT match any value in the list.

**Syntax:**
```sql
WHERE column NOT IN (subquery)
```

**Example: Find students who have NOT enrolled in any courses**

```sql
SELECT first_name, last_name
FROM students
WHERE student_id NOT IN (
    SELECT student_id 
    FROM enrollments
    WHERE student_id IS NOT NULL  -- Important!
);
```

**Result:**
| first_name | last_name |
|------------|-----------|
| Charlie | Davis |

### Critical: NOT IN and NULL Values

**Problem:** If the subquery returns any NULL, `NOT IN` may return no rows!

```sql
-- This can fail if subquery has NULL
WHERE id NOT IN (SELECT id FROM table)
```

**Why:** `NOT IN (1, 2, NULL)` becomes `!= 1 AND != 2 AND != NULL`. Comparing to NULL returns NULL (not TRUE/FALSE), making the entire condition NULL, which is treated as FALSE.

**Solutions:**

```sql
-- Solution 1: Filter out NULLs
WHERE id NOT IN (SELECT id FROM table WHERE id IS NOT NULL)

-- Solution 2: Use NOT EXISTS (preferred)
WHERE NOT EXISTS (
    SELECT 1 FROM table t WHERE t.id = outer.id
)
```

## ANY Operator

Compares a value to each subquery result. Returns TRUE if ANY comparison is true.

**Syntax:**
```sql
WHERE column operator ANY (subquery)
-- operator can be: =, !=, >, <, >=, <=
```

**Meaning:**
| Expression | Equivalent |
|------------|------------|
| `> ANY (100, 200, 300)` | `> 100` (greater than minimum) |
| `< ANY (100, 200, 300)` | `< 300` (less than maximum) |
| `= ANY (100, 200, 300)` | Same as `IN` |

**Example: Find students with GPA higher than any CS major**

```sql
SELECT first_name, last_name, major, gpa
FROM students
WHERE gpa > ANY (
    SELECT gpa
    FROM students
    WHERE major = 'Computer Science' AND gpa IS NOT NULL
)
AND major != 'Computer Science';
```

**Logic:**
- Subquery returns: `(3.8, 3.2)`
- `> ANY (3.8, 3.2)` means `> 3.2`
- Returns non-CS students with GPA > 3.2

**Result:**
| first_name | last_name | major | gpa |
|------------|-----------|-------|-----|
| Jane | Doe | Mathematics | 3.9 |
| Alice | Brown | Physics | 3.7 |

## ALL Operator

Compares a value to each subquery result. Returns TRUE only if ALL comparisons are true.

**Syntax:**
```sql
WHERE column operator ALL (subquery)
```

**Meaning:**
| Expression | Equivalent |
|------------|------------|
| `> ALL (100, 200, 300)` | `> 300` (greater than maximum) |
| `< ALL (100, 200, 300)` | `< 100` (less than minimum) |
| `!= ALL (100, 200, 300)` | Same as `NOT IN` |

**Example: Find student with highest GPA**

```sql
SELECT first_name, last_name, gpa
FROM students s1
WHERE gpa > ALL (
    SELECT gpa
    FROM students s2
    WHERE s2.student_id != s1.student_id AND gpa IS NOT NULL
)
AND gpa IS NOT NULL;
```

**Alternative (simpler):**
```sql
SELECT first_name, last_name, gpa
FROM students
WHERE gpa = (SELECT MAX(gpa) FROM students);
```

## Best Practices

**1. Use IN for simple membership tests**
```sql
-- Good
WHERE student_id IN (SELECT student_id FROM enrollments)
```

**2. Always handle NULLs with NOT IN**
```sql
-- Add NULL filter or use NOT EXISTS
WHERE id NOT IN (SELECT id FROM table WHERE id IS NOT NULL)
```

**3. Prefer EXISTS for complex queries**
```sql
-- Often faster than IN
WHERE EXISTS (
    SELECT 1 FROM enrollments e 
    WHERE e.student_id = students.student_id
)
```

**4. Use DISTINCT to avoid duplicate processing**
```sql
WHERE student_id IN (SELECT DISTINCT student_id FROM enrollments)
```

## Common Errors

**Error 1: Using = with multiple rows**
```sql
-- ERROR: subquery returns more than one row
WHERE gpa > (SELECT gpa FROM students WHERE major = 'Computer Science')

-- Fix: Use ANY or ALL
WHERE gpa > ALL (SELECT gpa FROM students WHERE major = 'Computer Science')
```

**Error 2: Forgetting NULL handling**
```sql
-- May return no rows
WHERE id NOT IN (SELECT id FROM table)

-- Fix
WHERE id NOT IN (SELECT id FROM table WHERE id IS NOT NULL)
```

## Summary

**Key Points:**

1. **Multiple-row subqueries** require special operators: IN, NOT IN, ANY, ALL
2. **IN** checks for membership in a list (= ANY)
3. **NOT IN** requires NULL handling - filter NULLs or use NOT EXISTS
4. **ANY** returns TRUE if at least one comparison matches
   - `> ANY` means "greater than minimum"
   - `< ANY` means "less than maximum"
5. **ALL** returns TRUE only if every comparison matches
   - `> ALL` means "greater than maximum"
   - `< ALL` means "less than minimum"
6. **Best practice**: Use IN for simple cases, EXISTS for complex ones, always handle NULLs

Multiple-row subqueries enable powerful filtering based on dynamic data sets.
