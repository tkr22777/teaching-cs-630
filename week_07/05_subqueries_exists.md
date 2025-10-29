# EXISTS and NOT EXISTS

## Overview

**EXISTS** is a special operator used with correlated subqueries to test for the existence of rows. Unlike IN, ANY, or ALL which compare values, EXISTS simply checks whether a subquery returns any rows at all. It's one of the most efficient ways to test relationships between tables.

## Key Terms

**EXISTS Operator**: Returns TRUE if the subquery returns one or more rows, FALSE otherwise.

**NOT EXISTS Operator**: Returns TRUE if the subquery returns zero rows, FALSE otherwise.

**Existence Test**: A logical test that checks for the presence or absence of related records.

**Short-Circuit Evaluation**: EXISTS stops processing as soon as it finds the first matching row.

**Correlation**: EXISTS is typically used with correlated subqueries that reference the outer query.

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

### Basic Syntax

```sql
SELECT columns
FROM table1 outer
WHERE EXISTS (
    SELECT 1
    FROM table2 inner
    WHERE inner.column = outer.column
);
```

### Execution Logic

1. For each row in the outer query:
   - Execute the subquery
   - If subquery returns **any rows** → EXISTS is TRUE → include outer row
   - If subquery returns **no rows** → EXISTS is FALSE → exclude outer row

2. **Important:** EXISTS doesn't care about:
   - What columns the subquery selects
   - How many rows match
   - The actual values returned

3. **Performance:** EXISTS stops as soon as it finds the first match (short-circuit evaluation)

### EXISTS vs. IN Comparison

**Using IN:**
```sql
SELECT first_name, last_name
FROM students
WHERE student_id IN (
    SELECT student_id
    FROM enrollments
);
```

**Using EXISTS:**
```sql
SELECT first_name, last_name
FROM students s
WHERE EXISTS (
    SELECT 1
    FROM enrollments e
    WHERE e.student_id = s.student_id
);
```

**Result:** Both return students who have enrollments.

### Why Use EXISTS?

| Advantage | Explanation |
|-----------|-------------|
| **Performance** | Stops at first match; doesn't process entire subquery |
| **NULL Handling** | Not affected by NULL values (unlike NOT IN) |
| **Readability** | Clearly expresses "check if related records exist" |
| **Optimization** | Database can use efficient existence checks |

## EXISTS Examples

### Example 1: Finding Related Records

**Query: Find students who have enrolled in at least one course**

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

**Explanation:** Returns all students who have at least one enrollment record.

### Example 2: Complex Existence Conditions

**Query: Find students enrolled in Computer Science courses**

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

### Example 3: Multiple Conditions

**Query: Find students with at least one 'A' grade**

```sql
SELECT s.first_name, s.last_name, s.gpa
FROM students s
WHERE EXISTS (
    SELECT 1
    FROM enrollments e
    WHERE e.student_id = s.student_id
    AND e.grade = 'A'
);
```

**Result:**
| first_name | last_name | gpa |
|------------|-----------|-----|
| John | Smith | 3.8 |
| Jane | Doe | 3.9 |
| Alice | Brown | 3.7 |

### Example 4: Checking Multiple Related Tables

**Query: Find instructors who teach courses with enrollments**

```sql
SELECT i.instructor_name, i.department
FROM instructors i
WHERE EXISTS (
    SELECT 1
    FROM courses c
    WHERE c.instructor_id = i.instructor_id
    AND EXISTS (
        SELECT 1
        FROM enrollments e
        WHERE e.course_id = c.course_id
    )
);
```

## NOT EXISTS Examples

### Example 1: Finding Unrelated Records

**Query: Find students who have NOT enrolled in any courses**

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

**Explanation:** Returns students with no enrollment records.

### Example 2: Finding Gaps

**Query: Find courses with no enrollments**

```sql
SELECT c.course_id, c.course_name, c.department
FROM courses c
WHERE NOT EXISTS (
    SELECT 1
    FROM enrollments e
    WHERE e.course_id = c.course_id
);
```

**Result:**
| course_id | course_name | department |
|-----------|-------------|------------|
| ENG101 | English Composition | English |

### Example 3: Exclusion with Conditions

**Query: Find students who have never received a failing grade (C or below)**

```sql
SELECT s.first_name, s.last_name
FROM students s
WHERE NOT EXISTS (
    SELECT 1
    FROM enrollments e
    WHERE e.student_id = s.student_id
    AND e.grade IN ('C', 'C+', 'C-', 'D', 'F')
);
```

### Example 4: Finding Prerequisite Gaps

**Query: Find students enrolled in CS201 who haven't taken CS101**

```sql
SELECT s.first_name, s.last_name
FROM students s
WHERE EXISTS (
    SELECT 1
    FROM enrollments e
    WHERE e.student_id = s.student_id
    AND e.course_id = 'CS201'
)
AND NOT EXISTS (
    SELECT 1
    FROM enrollments e
    WHERE e.student_id = s.student_id
    AND e.course_id = 'CS101'
);
```

## EXISTS vs. IN: Deep Comparison

### Scenario 1: Simple Membership Test

**Using IN:**
```sql
SELECT first_name, last_name
FROM students
WHERE student_id IN (1, 2, 3);
```

**Using EXISTS:**
```sql
-- Not natural for static lists; IN is better here
```

**Winner:** IN for static value lists

### Scenario 2: Subquery with Multiple Columns

**Using IN (doesn't work well):**
```sql
-- Can't easily check multiple conditions
SELECT first_name FROM students
WHERE student_id IN (SELECT student_id FROM enrollments WHERE grade = 'A');
```

**Using EXISTS (more flexible):**
```sql
SELECT s.first_name
FROM students s
WHERE EXISTS (
    SELECT 1
    FROM enrollments e
    WHERE e.student_id = s.student_id
    AND e.grade = 'A'
    AND e.semester = 'Fall 2023'
);
```

**Winner:** EXISTS for complex conditions

### Scenario 3: NULL Values

**Problem with NOT IN:**
```sql
-- Returns NO rows if subquery contains NULL
SELECT first_name
FROM students
WHERE student_id NOT IN (SELECT student_id FROM enrollments);
-- Fails if enrollments.student_id has NULL
```

**NOT EXISTS (handles NULLs correctly):**
```sql
SELECT first_name
FROM students s
WHERE NOT EXISTS (
    SELECT 1
    FROM enrollments e
    WHERE e.student_id = s.student_id
);
-- Works correctly regardless of NULLs
```

**Winner:** EXISTS/NOT EXISTS for NULL safety

### Performance Comparison Table

| Aspect | IN | EXISTS |
|--------|-------|--------|
| **Simple lists** | Fast | Overkill |
| **Large subquery results** | Can be slow | Faster (short-circuits) |
| **NULL handling** | Problematic with NOT IN | Safe |
| **Multiple conditions** | Limited | Flexible |
| **Readability** | Clear for simple cases | Better for existence checks |

## Advanced EXISTS Patterns

### Pattern 1: Set Difference (Anti-Join)

**Query: Find students in Computer Science major who haven't taken any Physics courses**

```sql
SELECT s.first_name, s.last_name, s.major
FROM students s
WHERE s.major = 'Computer Science'
AND NOT EXISTS (
    SELECT 1
    FROM enrollments e
    JOIN courses c ON e.course_id = c.course_id
    WHERE e.student_id = s.student_id
    AND c.department = 'Physics'
);
```

### Pattern 2: Mutual Existence

**Query: Find students enrolled in both CS101 AND CS201**

```sql
SELECT s.first_name, s.last_name
FROM students s
WHERE EXISTS (
    SELECT 1
    FROM enrollments e
    WHERE e.student_id = s.student_id
    AND e.course_id = 'CS101'
)
AND EXISTS (
    SELECT 1
    FROM enrollments e
    WHERE e.student_id = s.student_id
    AND e.course_id = 'CS201'
);
```

**Result:**
| first_name | last_name |
|------------|-----------|
| John | Smith |
| Bob | Wilson |

### Pattern 3: Hierarchical Existence

**Query: Find departments with instructors who teach courses that have enrollments**

```sql
SELECT DISTINCT i.department
FROM instructors i
WHERE EXISTS (
    SELECT 1
    FROM courses c
    WHERE c.instructor_id = i.instructor_id
    AND EXISTS (
        SELECT 1
        FROM enrollments e
        WHERE e.course_id = c.course_id
    )
);
```

### Pattern 4: Exclusive Relationship

**Query: Find students enrolled ONLY in Computer Science courses**

```sql
SELECT s.first_name, s.last_name
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
    JOIN courses c ON e.course_id = c.course_id
    WHERE e.student_id = s.student_id
    AND c.department != 'Computer Science'
);
```

## Double Negation (NOT NOT EXISTS)

Sometimes logical complexity requires double negation, though it can be confusing.

**Query: Find students for whom there is no course they haven't taken**
(i.e., students who have taken ALL courses)

```sql
SELECT s.first_name, s.last_name
FROM students s
WHERE NOT EXISTS (
    SELECT 1
    FROM courses c
    WHERE NOT EXISTS (
        SELECT 1
        FROM enrollments e
        WHERE e.student_id = s.student_id
        AND e.course_id = c.course_id
    )
);
```

**Translation:** "There does not exist a course for which there does not exist an enrollment for this student."

**Simpler Alternative (using COUNT):**
```sql
SELECT s.first_name, s.last_name
FROM students s
WHERE (SELECT COUNT(DISTINCT course_id) FROM enrollments WHERE student_id = s.student_id)
    = (SELECT COUNT(*) FROM courses);
```

## Performance Optimization with EXISTS

### Best Practices

**1. Keep subquery simple**
```sql
-- Good: Simple existence check
WHERE EXISTS (SELECT 1 FROM table WHERE condition)

-- Avoid: Complex calculations in EXISTS
WHERE EXISTS (SELECT SUM(complex_calc) FROM table WHERE ...)
```

**2. Use appropriate indexes**
```sql
-- Index the correlated column
CREATE INDEX idx_enrollments_student ON enrollments(student_id);

-- Then this runs efficiently
WHERE EXISTS (SELECT 1 FROM enrollments WHERE student_id = s.student_id);
```

**3. Use SELECT 1 or SELECT NULL**
```sql
-- Both are equivalent and optimized
WHERE EXISTS (SELECT 1 FROM enrollments WHERE ...)
WHERE EXISTS (SELECT NULL FROM enrollments WHERE ...)
WHERE EXISTS (SELECT * FROM enrollments WHERE ...)

-- All perform the same; SELECT 1 is convention
```

**4. Filter in the subquery**
```sql
-- Good: Filter early in subquery
WHERE EXISTS (
    SELECT 1 FROM enrollments
    WHERE student_id = s.student_id
    AND grade = 'A'
)

-- Less efficient: Filter in outer query
WHERE grade_value = 'A'
AND EXISTS (SELECT 1 FROM enrollments WHERE student_id = s.student_id)
```

## Common Mistakes

### Mistake 1: Forgetting Correlation

**Problem:**
```sql
-- Returns all students if ANY enrollment exists
SELECT first_name FROM students
WHERE EXISTS (SELECT 1 FROM enrollments);
```

**Solution:**
```sql
-- Correlate with outer query
SELECT first_name FROM students s
WHERE EXISTS (
    SELECT 1 FROM enrollments e WHERE e.student_id = s.student_id
);
```

### Mistake 2: Using SELECT * Unnecessarily

**Not wrong, but less clear:**
```sql
WHERE EXISTS (SELECT * FROM enrollments WHERE ...)
```

**Better (shows intent):**
```sql
WHERE EXISTS (SELECT 1 FROM enrollments WHERE ...)
```

### Mistake 3: Using COUNT Instead of EXISTS

**Inefficient:**
```sql
WHERE (SELECT COUNT(*) FROM enrollments WHERE student_id = s.student_id) > 0
```

**Better:**
```sql
WHERE EXISTS (SELECT 1 FROM enrollments WHERE student_id = s.student_id)
```

**Why:** EXISTS stops at first match; COUNT processes all rows.

## Practical Business Scenarios

### Scenario 1: Active Customer Identification

**Query: Find customers who have placed orders in the last 90 days**

```sql
SELECT c.customer_name, c.email
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
    AND o.order_date >= SYSDATE - 90
);
```

### Scenario 2: Inventory Management

**Query: Find products that have never been ordered**

```sql
SELECT p.product_id, p.product_name, p.stock_quantity
FROM products p
WHERE NOT EXISTS (
    SELECT 1
    FROM order_items oi
    WHERE oi.product_id = p.product_id
);
```

### Scenario 3: Employee Performance

**Query: Find employees who have managed at least one successful project**

```sql
SELECT e.employee_name, e.department
FROM employees e
WHERE EXISTS (
    SELECT 1
    FROM projects p
    WHERE p.manager_id = e.employee_id
    AND p.status = 'Completed'
    AND p.budget_variance >= 0
);
```

## Summary

**Key Takeaways:**

1. **EXISTS tests for the existence of rows** in a subquery, returning TRUE if any rows match, FALSE otherwise.

2. **NOT EXISTS returns TRUE when no rows match**, making it perfect for finding gaps or missing relationships.

3. **Short-circuit evaluation** means EXISTS stops as soon as it finds the first matching row, providing better performance than IN for large result sets.

4. **NULL-safe** unlike NOT IN, EXISTS and NOT EXISTS handle NULL values correctly without unexpected behavior.

5. **Always use correlation** - EXISTS typically references columns from the outer query to test row-specific relationships.

6. **Use SELECT 1 by convention** - the actual column selected doesn't matter; SELECT 1 clearly shows you're testing existence.

7. **Prefer EXISTS over IN when**: checking complex conditions, dealing with potentially NULL columns, or when performance matters with large datasets.

8. **Common patterns**: finding related records, identifying gaps, implementing anti-joins, checking mutual existence, and filtering based on related table conditions.

9. **Performance optimization**: Keep subqueries simple, index correlated columns, filter early in the subquery, and use EXISTS instead of COUNT(*) > 0.

EXISTS and NOT EXISTS are powerful, efficient tools for testing relationships between tables and should be your go-to choice for existence checks in SQL.

