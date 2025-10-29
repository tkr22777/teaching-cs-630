# Correlated Subqueries

## Overview

A **correlated subquery** is a subquery that references columns from the outer query. Unlike non-correlated subqueries that execute once, correlated subqueries execute once for each row processed by the outer query, making them more computationally intensive but enabling powerful row-by-row comparisons.

## Key Terms

**Correlated Subquery**: A subquery that references one or more columns from the outer query and executes repeatedly for each outer row.

**Non-Correlated Subquery**: An independent subquery that executes once and returns results to the outer query.

**Correlation**: The relationship between the inner and outer queries through shared column references.

**Row-by-Row Processing**: The execution pattern where the subquery runs once for each row in the outer query.

**Table Alias**: A required shorthand name for tables in correlated subqueries to distinguish between inner and outer references.

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

## Characteristics of Correlated Subqueries

### Key Features

1. **References outer query columns** - Uses columns from the parent query in the WHERE clause
2. **Executes repeatedly** - Runs once for each row in the outer query
3. **Requires table aliases** - Must distinguish between inner and outer table references
4. **Row-dependent results** - Each execution can return different results based on the current outer row
5. **More processing** - Generally slower than non-correlated subqueries

### Execution Flow

```sql
SELECT outer.column1, outer.column2
FROM table1 outer
WHERE outer.column3 = (
    SELECT inner.column_x
    FROM table2 inner
    WHERE inner.column_y = outer.column_z  -- Correlation point
);
```

**Execution Steps:**
1. Outer query reads first row
2. Inner query executes with values from that row
3. Result used to evaluate WHERE condition
4. Process repeats for each outer row

## Correlated vs. Non-Correlated Subqueries

### Non-Correlated Example

```sql
-- Executes subquery ONCE
SELECT first_name, last_name, gpa
FROM students
WHERE gpa > (SELECT AVG(gpa) FROM students);
```

**Execution:** 
1. Calculate average GPA once: 3.48
2. Find all students where gpa > 3.48

### Correlated Example

```sql
-- Executes subquery ONCE PER STUDENT
SELECT s.first_name, s.last_name, s.major, s.gpa
FROM students s
WHERE s.gpa > (
    SELECT AVG(s2.gpa)
    FROM students s2
    WHERE s2.major = s.major  -- Correlation: references outer query
);
```

**Execution:**
1. For John (CS major): Calculate AVG(gpa) for CS majors → Compare John's GPA
2. For Jane (Math major): Calculate AVG(gpa) for Math majors → Compare Jane's GPA
3. Repeat for each student

**Result:** Students performing above their own major's average.

### Comparison Table

| Aspect | Non-Correlated | Correlated |
|--------|----------------|------------|
| **Independence** | Independent of outer query | Depends on outer query |
| **Execution** | Once | Once per outer row |
| **References** | No outer table references | References outer columns |
| **Performance** | Generally faster | Generally slower |
| **Use Case** | Global comparisons | Row-specific comparisons |

## Common Patterns with Correlated Subqueries

### Pattern 1: Above-Average Within Group

**Query: Find students with GPA above their major's average**

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

**Explanation:** Only John's GPA (3.8) is above his major's average (3.5). Jane's GPA equals the Mathematics average (3.9 = 3.9), and Alice's equals the Physics average (3.7 = 3.7), so they are not included since the query uses greater than (>), not greater than or equal to (>=).

### Pattern 2: Ranking Within Categories

**Query: Find the highest-enrolled course in each department**

```sql
SELECT c.course_id, c.course_name, c.department,
       (SELECT COUNT(*)
        FROM enrollments e
        WHERE e.course_id = c.course_id) AS enrollment_count
FROM courses c
WHERE (SELECT COUNT(*)
       FROM enrollments e
       WHERE e.course_id = c.course_id) = (
           SELECT MAX(course_enrollment)
           FROM (
               SELECT COUNT(*) AS course_enrollment
               FROM enrollments e2
               JOIN courses c2 ON e2.course_id = c2.course_id
               WHERE c2.department = c.department
               GROUP BY e2.course_id
           )
       );
```

### Pattern 3: Latest/Most Recent Record

**Query: Find each student's most recent enrollment**

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

## Correlated Subqueries in SELECT Clause

Correlated subqueries in the SELECT clause create calculated columns based on related data.

### Example 1: Count Related Records

**Query: Show each course with its enrollment count**

```sql
SELECT 
    c.course_id,
    c.course_name,
    c.credits,
    (SELECT COUNT(*)
     FROM enrollments e
     WHERE e.course_id = c.course_id) AS enrollment_count
FROM courses c
ORDER BY enrollment_count DESC;
```

**Result:**
| course_id | course_name | credits | enrollment_count |
|-----------|-------------|---------|------------------|
| CS101 | Introduction to Programming | 3 | 3 |
| CS201 | Data Structures | 4 | 2 |
| CS301 | Database Systems | 3 | 1 |
| MATH101 | Calculus I | 4 | 1 |
| PHYS101 | Physics I | 4 | 1 |
| ENG101 | English Composition | 3 | 0 |

### Example 2: Calculate Running Totals

**Query: Show cumulative enrollments for each student**

```sql
SELECT 
    s.first_name,
    s.last_name,
    (SELECT COUNT(*)
     FROM enrollments e
     WHERE e.student_id = s.student_id) AS total_enrollments,
    (SELECT COUNT(*)
     FROM enrollments e
     WHERE e.student_id = s.student_id
     AND e.grade IS NOT NULL) AS completed_courses
FROM students s
ORDER BY total_enrollments DESC;
```

### Example 3: Retrieve Related Single Value

**Query: Show each student with their best grade**

```sql
SELECT 
    s.first_name,
    s.last_name,
    (SELECT MAX(e.grade)
     FROM enrollments e
     WHERE e.student_id = s.student_id) AS best_grade,
    (SELECT AVG(e.grade_points)
     FROM enrollments e
     WHERE e.student_id = s.student_id
     AND e.grade_points IS NOT NULL) AS avg_grade_points
FROM students s;
```

## Correlated Subqueries in WHERE Clause

### Example 1: Comparison Within Same Table

**Query: Find students with above-average GPA in their major**

```sql
SELECT s1.first_name, s1.last_name, s1.major, s1.gpa
FROM students s1
WHERE s1.gpa > (
    SELECT AVG(s2.gpa)
    FROM students s2
    WHERE s2.major = s1.major
    AND s2.gpa IS NOT NULL
)
AND s1.major IS NOT NULL;
```

### Example 2: Comparing Across Tables

**Query: Find courses with more enrollments than their department's average**

```sql
SELECT c.course_id, c.course_name, c.department
FROM courses c
WHERE (SELECT COUNT(*)
       FROM enrollments e
       WHERE e.course_id = c.course_id) > (
           SELECT AVG(dept_avg)
           FROM (
               SELECT c2.course_id, COUNT(*) AS dept_avg
               FROM courses c2
               LEFT JOIN enrollments e2 ON c2.course_id = e2.course_id
               WHERE c2.department = c.department
               GROUP BY c2.course_id
           )
       );
```

### Example 3: Conditional Filtering

**Query: Find students who have taken more courses than Jane Doe**

```sql
SELECT s.first_name, s.last_name,
       (SELECT COUNT(*) 
        FROM enrollments e 
        WHERE e.student_id = s.student_id) AS course_count
FROM students s
WHERE (SELECT COUNT(*) 
       FROM enrollments e 
       WHERE e.student_id = s.student_id) > (
           SELECT COUNT(*)
           FROM enrollments e2
           JOIN students s2 ON e2.student_id = s2.student_id
           WHERE s2.first_name = 'Jane' AND s2.last_name = 'Doe'
       )
ORDER BY course_count DESC;
```

## Correlated Subqueries in HAVING Clause

**Query: Find majors where the average GPA is higher than the overall average**

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

## Advanced Correlated Subquery Patterns

### Pattern 1: Finding Gaps or Missing Values

**Query: Find students who haven't taken CS101 but have taken other CS courses**

```sql
SELECT DISTINCT s.first_name, s.last_name
FROM students s
WHERE EXISTS (
    SELECT 1
    FROM enrollments e
    JOIN courses c ON e.course_id = c.course_id
    WHERE e.student_id = s.student_id
    AND c.department = 'Computer Science'
)
AND NOT EXISTS (
    SELECT 1
    FROM enrollments e
    WHERE e.student_id = s.student_id
    AND e.course_id = 'CS101'
);
```

### Pattern 2: Self-Comparison

**Query: Find students with identical GPAs**

```sql
SELECT DISTINCT s1.first_name, s1.last_name, s1.gpa
FROM students s1
WHERE EXISTS (
    SELECT 1
    FROM students s2
    WHERE s2.gpa = s1.gpa
    AND s2.student_id != s1.student_id
)
ORDER BY s1.gpa;
```

### Pattern 3: Hierarchical Data

**Query: Find instructors who teach more courses than their department average**

```sql
SELECT i.instructor_name, i.department,
       (SELECT COUNT(*)
        FROM courses c
        WHERE c.instructor_id = i.instructor_id) AS courses_taught
FROM instructors i
WHERE (SELECT COUNT(*)
       FROM courses c
       WHERE c.instructor_id = i.instructor_id) > (
           SELECT AVG(course_count)
           FROM (
               SELECT i2.instructor_id, COUNT(*) AS course_count
               FROM instructors i2
               LEFT JOIN courses c2 ON i2.instructor_id = c2.instructor_id
               WHERE i2.department = i.department
               GROUP BY i2.instructor_id
           )
       );
```

## Performance Considerations

### Understanding Performance Impact

Correlated subqueries can be expensive because:
1. **Multiple executions** - Run once per outer row
2. **No result caching** - Each execution is independent
3. **Index dependency** - Performance heavily depends on indexes on correlated columns

### Example Performance Analysis

```sql
-- Correlated subquery (potentially slower)
SELECT s.first_name, s.last_name
FROM students s
WHERE (SELECT COUNT(*) 
       FROM enrollments e 
       WHERE e.student_id = s.student_id) > 2;

-- Equivalent JOIN (potentially faster)
SELECT s.first_name, s.last_name
FROM students s
JOIN enrollments e ON s.student_id = e.student_id
GROUP BY s.student_id, s.first_name, s.last_name
HAVING COUNT(*) > 2;
```

### Optimization Strategies

**1. Add Indexes on Correlated Columns**
```sql
CREATE INDEX idx_enrollments_student ON enrollments(student_id);
```

**2. Consider Analytic Functions**
```sql
-- Instead of correlated subquery
SELECT first_name, last_name, gpa,
       AVG(gpa) OVER (PARTITION BY major) AS major_avg
FROM students;
```

**3. Use JOINs When Possible**
```sql
-- Instead of subquery in SELECT
SELECT s.first_name, COUNT(e.enrollment_id) AS enrollment_count
FROM students s
LEFT JOIN enrollments e ON s.student_id = e.student_id
GROUP BY s.student_id, s.first_name;
```

**4. Materialize Intermediate Results**
```sql
-- Create temporary table for repeated calculations
CREATE TEMPORARY TABLE major_averages AS
SELECT major, AVG(gpa) AS avg_gpa
FROM students
GROUP BY major;

-- Use the materialized results
SELECT s.first_name, s.last_name
FROM students s
JOIN major_averages ma ON s.major = ma.major
WHERE s.gpa > ma.avg_gpa;
```

## Common Mistakes and Solutions

### Mistake 1: Missing Table Aliases

**Problem:**
```sql
-- ERROR: ambiguous column reference
SELECT first_name, last_name
FROM students
WHERE gpa > (SELECT AVG(gpa) FROM students WHERE major = major);
```

**Solution:**
```sql
SELECT s1.first_name, s1.last_name
FROM students s1
WHERE s1.gpa > (
    SELECT AVG(s2.gpa) 
    FROM students s2 
    WHERE s2.major = s1.major
);
```

### Mistake 2: Forgetting NULL Handling

**Problem:**
```sql
-- May not work correctly with NULL majors
SELECT first_name FROM students s1
WHERE gpa > (SELECT AVG(gpa) FROM students s2 WHERE s2.major = s1.major);
```

**Solution:**
```sql
SELECT first_name FROM students s1
WHERE s1.major IS NOT NULL
AND gpa > (
    SELECT AVG(gpa) 
    FROM students s2 
    WHERE s2.major = s1.major 
    AND s2.gpa IS NOT NULL
);
```

### Mistake 3: Inefficient Repeated Subqueries

**Problem:**
```sql
-- Same subquery executed twice per row
SELECT first_name,
       (SELECT COUNT(*) FROM enrollments e WHERE e.student_id = s.student_id),
       (SELECT COUNT(*) FROM enrollments e WHERE e.student_id = s.student_id) * 2
FROM students s;
```

**Solution:**
```sql
-- Use JOIN or WITH clause to calculate once
WITH enrollment_counts AS (
    SELECT student_id, COUNT(*) AS cnt
    FROM enrollments
    GROUP BY student_id
)
SELECT s.first_name, ec.cnt, ec.cnt * 2
FROM students s
LEFT JOIN enrollment_counts ec ON s.student_id = ec.student_id;
```

## When to Use Correlated Subqueries

### Good Use Cases

1. **Row-specific comparisons** - Comparing each row to its own group average
2. **Existence checks** - Testing if related records exist (use EXISTS)
3. **Calculated columns** - Adding computed values to result sets
4. **Complex filtering** - Conditions based on related table aggregates

### When to Consider Alternatives

1. **Simple aggregates** - Use non-correlated subquery if comparison is global
2. **Multiple columns from related table** - Use JOIN instead
3. **Performance critical queries** - Consider JOINs or analytic functions
4. **Large datasets** - Optimize with indexes or rewrite using JOINs

## Summary

**Key Takeaways:**

1. **Correlated subqueries reference outer query columns** and execute once for each row processed by the outer query.

2. **Require table aliases** to distinguish between inner and outer table references (e.g., students s1 vs. students s2).

3. **Execute repeatedly** making them potentially slower than non-correlated subqueries, but enabling row-specific comparisons.

4. **Common patterns include**: comparing to group averages, finding latest/most recent records, calculating per-row aggregates, and ranking within categories.

5. **Can appear in any clause**: SELECT (calculated columns), WHERE (row filtering), HAVING (group filtering), FROM (inline views).

6. **Performance optimization**: Add indexes on correlated columns, consider JOINs or analytic functions, materialize intermediate results when possible.

7. **Best practices**: Always use table aliases, handle NULLs explicitly, avoid repeating identical subqueries, and choose based on readability vs. performance needs.

8. **Use when**: Row-by-row comparison is needed, testing existence with EXISTS, or when logic is clearer with nested structure.

Correlated subqueries provide powerful capabilities for row-specific analysis but should be used judiciously with attention to performance implications.

