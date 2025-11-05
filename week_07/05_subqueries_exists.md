# EXISTS and NOT EXISTS

## Overview

**EXISTS** tests for the existence of rows in a subquery. It returns TRUE if the subquery returns any rows, FALSE otherwise. It's efficient and handles NULLs better than IN.

## Key Terms

**EXISTS Operator**: Returns TRUE if subquery returns one or more rows.

**NOT EXISTS Operator**: Returns TRUE if subquery returns zero rows.

**Short-Circuit Evaluation**: EXISTS stops as soon as it finds the first matching row.

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

## How EXISTS Works

**Syntax:**
```sql
SELECT columns
FROM table1 outer
WHERE EXISTS (
    SELECT 1
    FROM table2 inner
    WHERE inner.column = outer.column
);
```

**Execution:**
1. For each outer row, execute the subquery
2. If subquery returns any rows → EXISTS is TRUE → include row
3. If subquery returns no rows → EXISTS is FALSE → exclude row

**Key Points:**
- EXISTS doesn't care what columns are selected or how many rows match
- Stops at first match (short-circuit evaluation)
- Use `SELECT 1` by convention (efficient)

## EXISTS vs. IN

| Aspect | IN | EXISTS |
|--------|-----|--------|
| **Performance** | Processes all results | Stops at first match |
| **NULL Handling** | Fails with NOT IN | Not affected by NULLs |
| **Use Case** | Simple value lists | Complex existence checks |
| **Correlation** | Usually non-correlated | Always correlated |

**Example - Both return students with enrollments:**

```sql
-- Using IN
SELECT first_name FROM students
WHERE student_id IN (SELECT student_id FROM enrollments);

-- Using EXISTS (often preferred)
SELECT first_name FROM students s
WHERE EXISTS (SELECT 1 FROM enrollments e WHERE e.student_id = s.student_id);
```

## EXISTS Examples

### Example 1: Find Related Records

**Find students enrolled in at least one course:**

```sql
SELECT s.first_name, s.last_name, s.major
FROM students s
WHERE EXISTS (
    SELECT 1
    FROM enrollments e
    WHERE e.student_id = s.student_id
);
```

**Result:**
| first_name | last_name | major |
|------------|-----------|-------|
| John | Smith | Computer Science |
| Jane | Doe | Mathematics |
| Bob | Wilson | Computer Science |
| Alice | Brown | Physics |

Charlie is excluded (no enrollments).

### Example 2: Complex Conditions

**Find students enrolled in Computer Science courses:**

```sql
SELECT s.first_name, s.last_name
FROM students s
WHERE EXISTS (
    SELECT 1
    FROM enrollments e
    JOIN courses c ON e.course_id = c.course_id
    WHERE e.student_id = s.student_id
    AND c.department = 'Computer Science'
);
```

**Result:**
| first_name | last_name |
|------------|-----------|
| John | Smith |
| Jane | Doe |
| Bob | Wilson |

## NOT EXISTS Examples

### Example 1: Find Unrelated Records

**Find students with NO enrollments:**

```sql
SELECT s.first_name, s.last_name, s.enrollment_date
FROM students s
WHERE NOT EXISTS (
    SELECT 1
    FROM enrollments e
    WHERE e.student_id = s.student_id
);
```

**Result:**
| first_name | last_name | enrollment_date |
|------------|-----------|-----------------|
| Charlie | Davis | 2024-09-01 |

### Example 2: Find Missing Prerequisites

**Find courses with no enrollments:**

```sql
SELECT c.course_id, c.course_name
FROM courses c
WHERE NOT EXISTS (
    SELECT 1
    FROM enrollments e
    WHERE e.course_id = c.course_id
);
```

**Result:**
| course_id | course_name |
|-----------|-------------|
| ENG101 | English Composition |

## Combining EXISTS with Other Conditions

**Find CS majors who haven't taken CS301:**

```sql
SELECT s.first_name, s.last_name
FROM students s
WHERE s.major = 'Computer Science'
AND NOT EXISTS (
    SELECT 1
    FROM enrollments e
    WHERE e.student_id = s.student_id
    AND e.course_id = 'CS301'
);
```

**Result:**
| first_name | last_name |
|------------|-----------|
| Bob | Wilson |

John has enrolled in CS301 (even though grade is NULL).

## Performance Optimization

**Best practices:**

**1. EXISTS is often faster than IN**
```sql
-- Good: Uses EXISTS
WHERE EXISTS (SELECT 1 FROM enrollments e WHERE e.student_id = s.student_id)

-- Less efficient: Uses IN
WHERE student_id IN (SELECT student_id FROM enrollments)
```

**2. Use appropriate indexes**
```sql
CREATE INDEX idx_enrollments_student ON enrollments(student_id);
```

**3. Use SELECT 1 (not SELECT *)**
```sql
-- Efficient
WHERE EXISTS (SELECT 1 FROM enrollments e WHERE...)

-- Wasteful
WHERE EXISTS (SELECT * FROM enrollments e WHERE...)
```

**4. Prefer NOT EXISTS over NOT IN**
```sql
-- Good: Safe with NULLs
WHERE NOT EXISTS (SELECT 1 FROM table t WHERE t.id = outer.id)

-- Risky: Fails if subquery has NULL
WHERE id NOT IN (SELECT id FROM table)
```

## Common Mistakes

**Mistake 1: Forgetting correlation**
```sql
-- Wrong: Not correlated
WHERE EXISTS (SELECT 1 FROM enrollments)

-- Correct: Correlated
WHERE EXISTS (SELECT 1 FROM enrollments e WHERE e.student_id = s.student_id)
```

**Mistake 2: Using COUNT instead of EXISTS**
```sql
-- Inefficient: Counts all rows
WHERE (SELECT COUNT(*) FROM enrollments e WHERE e.student_id = s.student_id) > 0

-- Better: Stops at first match
WHERE EXISTS (SELECT 1 FROM enrollments e WHERE e.student_id = s.student_id)
```

**Mistake 3: Using NOT IN with potential NULLs**
```sql
-- Dangerous: Returns no rows if subquery has NULL
WHERE id NOT IN (SELECT id FROM table)

-- Safe: Use NOT EXISTS
WHERE NOT EXISTS (SELECT 1 FROM table t WHERE t.id = outer.id)
```

## When to Use EXISTS

**Use EXISTS when:**
- Testing if related records exist
- Working with correlated subqueries
- Using NOT EXISTS (safer than NOT IN)
- Performance is important (short-circuit evaluation)

**Use IN when:**
- Comparing to a small, hardcoded list: `WHERE id IN (1, 2, 3)`
- Subquery is simple and non-correlated
- Values definitely don't contain NULLs

**Use JOINs when:**
- Need columns from related table
- Performing complex multi-table queries
- Want more explicit relationship representation

