# Inline Views (Derived Tables)

## Overview

An **inline view** (also called a derived table) is a subquery in the FROM clause that acts as a temporary table for the duration of the query. It allows you to treat query results as a table.

## Key Terms

**Inline View**: A subquery in the FROM clause that creates a temporary result set.

**Derived Table**: Another name for an inline view.

**Table Alias**: Required name for an inline view (mandatory in Oracle SQL).

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

## Basic Syntax

```sql
SELECT columns
FROM (
    SELECT columns
    FROM table
    WHERE conditions
) alias_name  -- Alias is required!
WHERE conditions;
```

**Key requirements:**
- Inline view must have an alias
- Can reference inline view columns in outer query
- Inline view executes first, then outer query uses results

## Why Use Inline Views?

**Benefits:**
- Break complex queries into logical steps
- Apply filtering after aggregation
- Calculate intermediate results
- Improve query readability

**Use when:**
- Need to filter aggregated data
- Want to reuse calculated columns
- Simplifying complex multi-step logic
- Creating rankings or top-N queries

## Examples

### Example 1: Aggregate then Filter

**Find courses with more than 1 enrollment:**

```sql
SELECT course_id, course_name, enrollment_count
FROM (
    SELECT 
        c.course_id,
        c.course_name,
        COUNT(e.enrollment_id) AS enrollment_count
    FROM courses c
    LEFT JOIN enrollments e ON c.course_id = e.course_id
    GROUP BY c.course_id, c.course_name
) course_stats
WHERE enrollment_count > 1
ORDER BY enrollment_count DESC;
```

**Result:**
| course_id | course_name | enrollment_count |
|-----------|-------------|------------------|
| CS101 | Introduction to Programming | 3 |
| CS201 | Data Structures | 2 |

**Why inline view:** Allows filtering on the aggregated `enrollment_count`.

### Example 2: Calculate Percentages

**Show each major's percentage of total students:**

```sql
SELECT 
    major,
    student_count,
    ROUND(student_count * 100.0 / total_students, 2) AS percentage
FROM (
    SELECT major, COUNT(*) AS student_count
    FROM students
    WHERE major IS NOT NULL
    GROUP BY major
) major_counts
CROSS JOIN (
    SELECT COUNT(*) AS total_students
    FROM students
    WHERE major IS NOT NULL
) totals
ORDER BY student_count DESC;
```

**Result:**
| major | student_count | percentage |
|-------|---------------|------------|
| Computer Science | 2 | 50.00 |
| Mathematics | 1 | 25.00 |
| Physics | 1 | 25.00 |

### Example 3: Ranking with ROW_NUMBER

**Get top 3 most enrolled courses:**

```sql
SELECT course_id, course_name, enrollment_count
FROM (
    SELECT 
        c.course_id,
        c.course_name,
        COUNT(e.enrollment_id) AS enrollment_count,
        ROW_NUMBER() OVER (ORDER BY COUNT(e.enrollment_id) DESC) AS rank
    FROM courses c
    LEFT JOIN enrollments e ON c.course_id = e.course_id
    GROUP BY c.course_id, c.course_name
) ranked_courses
WHERE rank <= 3;
```

**Result:**
| course_id | course_name | enrollment_count |
|-----------|-------------|------------------|
| CS101 | Introduction to Programming | 3 |
| CS201 | Data Structures | 2 |
| CS301 | Database Systems | 1 |

### Example 4: Joining with Inline View

**Show students with their enrollment count:**

```sql
SELECT 
    s.first_name,
    s.last_name,
    s.major,
    COALESCE(e_count.enrollment_count, 0) AS enrollments
FROM students s
LEFT JOIN (
    SELECT student_id, COUNT(*) AS enrollment_count
    FROM enrollments
    GROUP BY student_id
) e_count ON s.student_id = e_count.student_id
ORDER BY enrollments DESC;
```

**Result:**
| first_name | last_name | major | enrollments |
|------------|-----------|-------|-------------|
| John | Smith | Computer Science | 3 |
| Jane | Doe | Mathematics | 2 |
| Bob | Wilson | Computer Science | 2 |
| Alice | Brown | Physics | 1 |
| Charlie | Davis | NULL | 0 |

## Inline Views vs. WITH Clause (CTEs)

Both achieve similar results. WITH clause is often more readable.

**Inline View:**
```sql
SELECT course_id, enrollment_count
FROM (
    SELECT course_id, COUNT(*) AS enrollment_count
    FROM enrollments
    GROUP BY course_id
) enrollment_stats
WHERE enrollment_count > 1;
```

**WITH Clause (CTE):**
```sql
WITH enrollment_stats AS (
    SELECT course_id, COUNT(*) AS enrollment_count
    FROM enrollments
    GROUP BY course_id
)
SELECT course_id, enrollment_count
FROM enrollment_stats
WHERE enrollment_count > 1;
```

**Comparison:**

| Aspect | Inline View | WITH Clause (CTE) |
|--------|-------------|-------------------|
| **Readability** | Less readable (nested) | More readable (sequential) |
| **Reusability** | Use once | Can reference multiple times |
| **Debugging** | Harder to debug | Easier to test separately |
| **Oracle Support** | All versions | Oracle 9i+ |

**Recommendation:** Use WITH clause for complex queries or when reusing subqueries.

## Common Mistakes

**Mistake 1: Forgetting table alias**
```sql
-- ERROR: Missing alias
SELECT * FROM (SELECT * FROM students);

-- Correct
SELECT * FROM (SELECT * FROM students) s;
```

**Mistake 2: Column name conflicts**
```sql
-- Ambiguous: Which student_id?
SELECT student_id
FROM (SELECT student_id FROM students) s
JOIN enrollments e ON student_id = e.student_id;

-- Correct: Use aliases
SELECT s.student_id
FROM (SELECT student_id FROM students) s
JOIN enrollments e ON s.student_id = e.student_id;
```

## Performance Tips

1. **Add appropriate indexes** on columns used in WHERE/JOIN
2. **Minimize inline view size** - filter early
3. **Use WITH clause** for better optimization by Oracle
4. **Avoid unnecessary columns** - select only what you need
5. **Consider materialized views** for frequently-used complex queries

## Summary

**Key Points:**

1. **Inline views are subqueries in FROM clause** that act as temporary tables
2. **Must have an alias** (required in Oracle SQL)
3. **Use for multi-step logic**: aggregate then filter, calculate then use
4. **Common patterns**: filtering aggregates, percentages, ranking, top-N queries
5. **WITH clause alternative**: More readable for complex queries
6. **Best practices**: Use aliases, avoid deep nesting, filter early
7. **Performance**: Add indexes, minimize data, consider WITH clause

Inline views simplify complex queries by breaking logic into manageable steps.
