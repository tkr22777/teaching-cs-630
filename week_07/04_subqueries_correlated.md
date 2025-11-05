# Correlated Subqueries

## Overview

A **correlated subquery** references columns from the outer query and executes once for each row processed by the outer query. This enables powerful row-by-row comparisons but is more computationally intensive than non-correlated subqueries.

## Key Terms

**Correlated Subquery**: A subquery that references columns from the outer query and executes repeatedly for each outer row.

**Row-by-Row Processing**: The execution pattern where the subquery runs once for each row in the outer query.

**Table Alias**: Required shorthand name for tables to distinguish between inner and outer references.

## Sample Database Schema

This module uses the university enrollment system. If you haven't set it up yet:

<details>
<parameter name="summary">Click to expand: Database setup script</summary>

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

## Key Characteristics

**Correlated subqueries:**
1. Reference outer query columns
2. Execute once per outer row
3. Require table aliases
4. Enable row-specific comparisons
5. Generally slower than non-correlated subqueries

**Execution Flow:**
```sql
SELECT outer.column
FROM table1 outer
WHERE outer.value = (
    SELECT inner.value
    FROM table2 inner
    WHERE inner.id = outer.id  -- Correlation
);
```

**Steps:**
1. Outer query reads first row
2. Inner query executes with that row's values
3. Result evaluates WHERE condition
4. Repeat for each outer row

## Correlated vs. Non-Correlated

| Aspect | Non-Correlated | Correlated |
|--------|----------------|------------|
| **Execution** | Once | Once per outer row |
| **References** | No outer columns | References outer columns |
| **Performance** | Faster | Slower |
| **Use Case** | Global comparisons | Row-specific comparisons |

**Non-Correlated Example:**
```sql
-- Executes once
SELECT first_name, last_name, gpa
FROM students
WHERE gpa > (SELECT AVG(gpa) FROM students);
```

**Correlated Example:**
```sql
-- Executes once per student
SELECT s.first_name, s.last_name, s.major, s.gpa
FROM students s
WHERE s.gpa > (
    SELECT AVG(s2.gpa)
    FROM students s2
    WHERE s2.major = s.major  -- Correlation
)
AND s.major IS NOT NULL;
```

## Common Patterns

### Pattern 1: Above-Average Within Group

**Find students with GPA above their major's average:**

```sql
SELECT s.first_name, s.last_name, s.major, s.gpa,
       (SELECT AVG(s2.gpa) 
        FROM students s2 
        WHERE s2.major = s.major) AS major_avg
FROM students s
WHERE s.gpa > (
    SELECT AVG(s2.gpa)
    FROM students s2
    WHERE s2.major = s.major
)
AND s.major IS NOT NULL
ORDER BY s.major, s.gpa DESC;
```

**Result:**
| first_name | last_name | major | gpa | major_avg |
|------------|-----------|-------|-----|-----------|
| John | Smith | Computer Science | 3.8 | 3.5 |

Only John's GPA (3.8) exceeds his major's average (3.5).

### Pattern 2: Count Related Records

**Show each course with its enrollment count:**

```sql
SELECT 
    c.course_id,
    c.course_name,
    (SELECT COUNT(*)
     FROM enrollments e
     WHERE e.course_id = c.course_id) AS enrollment_count
FROM courses c
ORDER BY enrollment_count DESC;
```

**Result:**
| course_id | course_name | enrollment_count |
|-----------|-------------|------------------|
| CS101 | Introduction to Programming | 3 |
| CS201 | Data Structures | 2 |
| CS301 | Database Systems | 1 |
| MATH101 | Calculus I | 1 |
| PHYS101 | Physics I | 1 |
| ENG101 | English Composition | 0 |

### Pattern 3: Latest/Most Recent Record

**Find each student's most recent enrollment:**

```sql
SELECT s.first_name, s.last_name, e.course_id, e.semester
FROM students s
JOIN enrollments e ON s.student_id = e.student_id
WHERE e.enrollment_id = (
    SELECT MAX(e2.enrollment_id)
    FROM enrollments e2
    WHERE e2.student_id = s.student_id
);
```

**Result:**
| first_name | last_name | course_id | semester |
|------------|-----------|-----------|----------|
| John | Smith | CS301 | Fall 2024 |
| Jane | Doe | CS101 | Fall 2023 |
| Bob | Wilson | CS201 | Spring 2024 |
| Alice | Brown | PHYS101 | Spring 2024 |

## Using in Different Clauses

### In SELECT Clause (Calculated Columns)

```sql
SELECT 
    s.first_name,
    s.last_name,
    (SELECT COUNT(*)
     FROM enrollments e
     WHERE e.student_id = s.student_id) AS course_count,
    (SELECT AVG(e.grade_points)
     FROM enrollments e
     WHERE e.student_id = s.student_id) AS avg_grade
FROM students s;
```

### In WHERE Clause (Row Filtering)

```sql
SELECT s.first_name, s.last_name, s.major, s.gpa
FROM students s
WHERE s.gpa > (
    SELECT AVG(s2.gpa)
    FROM students s2
    WHERE s2.major = s.major AND s2.gpa IS NOT NULL
)
AND s.major IS NOT NULL;
```

### In HAVING Clause (Group Filtering)

```sql
SELECT s.major, AVG(s.gpa) AS major_avg_gpa
FROM students s
WHERE s.major IS NOT NULL
GROUP BY s.major
HAVING AVG(s.gpa) > (
    SELECT AVG(gpa) 
    FROM students
    WHERE gpa IS NOT NULL
);
```

**Result:**
| major | major_avg_gpa |
|-------|---------------|
| Mathematics | 3.9 |
| Physics | 3.7 |
| Computer Science | 3.5 |

## Performance Considerations

**Why correlated subqueries can be slow:**
- Execute once per outer row
- No result caching between executions
- Performance depends heavily on indexes

**Optimization strategies:**

**1. Add indexes on correlated columns**
```sql
CREATE INDEX idx_enrollments_student ON enrollments(student_id);
```

**2. Use JOINs when appropriate**
```sql
-- Correlated subquery
SELECT s.first_name, 
       (SELECT COUNT(*) FROM enrollments e WHERE e.student_id = s.student_id)
FROM students s;

-- JOIN alternative (often faster)
SELECT s.first_name, COUNT(e.enrollment_id)
FROM students s
LEFT JOIN enrollments e ON s.student_id = e.student_id
GROUP BY s.student_id, s.first_name;
```

**3. Consider analytic functions**
```sql
-- Instead of correlated subquery for group averages
SELECT first_name, last_name, gpa,
       AVG(gpa) OVER (PARTITION BY major) AS major_avg
FROM students;
```

## Common Mistakes

**Mistake 1: Missing table aliases**
```sql
-- ERROR: Which table's major?
WHERE gpa > (SELECT AVG(gpa) FROM students WHERE major = major)

-- Correct
WHERE s1.gpa > (SELECT AVG(s2.gpa) FROM students s2 WHERE s2.major = s1.major)
```

**Mistake 2: Forgetting NULL handling**
```sql
-- May fail with NULLs
WHERE s.major IS NOT NULL
AND gpa > (SELECT AVG(gpa) FROM students s2 WHERE s2.major = s.major AND gpa IS NOT NULL)
```

**Mistake 3: Repeating identical subqueries**
```sql
-- Inefficient: same subquery twice
SELECT (SELECT COUNT(*) FROM enrollments e WHERE e.student_id = s.student_id) AS cnt1,
       (SELECT COUNT(*) FROM enrollments e WHERE e.student_id = s.student_id) AS cnt2
FROM students s;

-- Better: use WITH clause
WITH counts AS (
    SELECT student_id, COUNT(*) AS cnt FROM enrollments GROUP BY student_id
)
SELECT c.cnt AS cnt1, c.cnt AS cnt2
FROM students s LEFT JOIN counts c ON s.student_id = c.student_id;
```

## When to Use Correlated Subqueries

**Good use cases:**
- Row-specific comparisons (e.g., above own group average)
- Calculated columns in SELECT
- Existence checks with EXISTS
- Complex row-level filtering

**Consider alternatives when:**
- Query performance is critical
- Simple global comparisons (use non-correlated)
- Need multiple columns from related table (use JOIN)
- Working with large datasets (optimize with indexes or JOINs)

## Summary

**Key Points:**

1. **Correlated subqueries reference outer query columns** and execute once per outer row
2. **Require table aliases** to distinguish inner/outer references (s1, s2)
3. **Enable row-specific comparisons** like "above own group average"
4. **Can appear in any clause**: SELECT, WHERE, HAVING
5. **Performance**: Slower than non-correlated; add indexes on correlated columns
6. **Common patterns**: Group comparisons, counting related records, finding latest records
7. **Alternatives**: JOINs, analytic functions, materialized intermediate results
8. **Best practices**: Use aliases, handle NULLs, avoid repeating identical subqueries

Correlated subqueries provide powerful row-level analysis but should be used with attention to performance.
