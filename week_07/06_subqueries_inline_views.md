# Inline Views and Derived Tables

## Overview

An **inline view** (also called a **derived table** or **subquery in FROM clause**) is a subquery placed in the FROM clause that acts as a temporary table for the duration of the query. Unlike regular subqueries that return values for comparison, inline views return entire result sets that can be queried like tables.

## Key Terms

**Inline View**: A subquery in the FROM clause that creates a temporary result set.

**Derived Table**: Another term for inline view; emphasizes that it's a table derived from a query.

**Virtual Table**: A temporary, query-scoped table that exists only for the current query execution.

**Table Alias**: A required name given to an inline view, used to reference its columns.

**Nested Query**: When inline views contain other subqueries or are themselves nested within larger queries.

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

## Characteristics of Inline Views

### Key Features

1. **Appears in FROM clause** - Treated as a table in the query
2. **Must have an alias** - Required to reference the derived table
3. **Temporary existence** - Only exists during query execution
4. **Can be joined** - Can participate in joins with other tables or inline views
5. **Can have WHERE, GROUP BY, ORDER BY** - Full query capabilities within the subquery
6. **Column aliases propagate** - Columns named in the subquery can be referenced in outer query

### Basic Syntax

```sql
SELECT columns
FROM (
    SELECT columns
    FROM table
    WHERE condition
) alias_name
WHERE condition;
```

## Why Use Inline Views?

### Benefits

1. **Simplify Complex Queries** - Break down multi-step logic into manageable pieces
2. **Pre-aggregate Data** - Perform aggregations before joining
3. **Avoid Repetition** - Calculate values once and reuse
4. **Improve Readability** - Make query intent clearer
5. **Enable Advanced Analytics** - Perform operations on aggregated results

### When to Use

| Use Case | Example |
|----------|---------|
| **Aggregate then filter** | Find departments with avg salary > $50k |
| **Pre-calculate values** | Compute totals before joining |
| **Avoid repeated subqueries** | Calculate once, use multiple times |
| **Multi-level grouping** | Group by aggregate results |
| **Complex joins** | Join on calculated values |

## Basic Inline View Examples

### Example 1: Simple Derived Table

**Query: Calculate and display enrollment statistics**

```sql
SELECT 
    enrollment_stats.course_id,
    enrollment_stats.enrollment_count,
    c.course_name
FROM (
    SELECT course_id, COUNT(*) AS enrollment_count
    FROM enrollments
    GROUP BY course_id
) enrollment_stats
JOIN courses c ON enrollment_stats.course_id = c.course_id
ORDER BY enrollment_stats.enrollment_count DESC;
```

**Result:**
| course_id | enrollment_count | course_name |
|-----------|------------------|-------------|
| CS101 | 3 | Introduction to Programming |
| CS201 | 2 | Data Structures |
| CS301 | 1 | Database Systems |
| MATH101 | 1 | Calculus I |
| PHYS101 | 1 | Physics I |

**Explanation:** The inline view calculates enrollment counts, then the outer query joins with courses to get names.

### Example 2: Filtering Aggregated Results

**Query: Find courses with more than 1 enrollment**

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

**Note:** This accomplishes the same as HAVING but demonstrates inline view filtering.

### Example 3: Calculating Percentages

**Query: Show each major's enrollment percentage**

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

## Advanced Inline View Patterns

### Pattern 1: Multi-Level Aggregation

**Query: Find departments with above-average instructor count**

```sql
SELECT dept_name, instructor_count
FROM (
    SELECT 
        department AS dept_name,
        COUNT(*) AS instructor_count
    FROM instructors
    GROUP BY department
) dept_counts
WHERE instructor_count > (
    SELECT AVG(instructor_count)
    FROM (
        SELECT COUNT(*) AS instructor_count
        FROM instructors
        GROUP BY department
    )
)
ORDER BY instructor_count DESC;
```

### Pattern 2: Ranking and Top-N Queries

**Query: Get top 3 most enrolled courses**

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

### Pattern 3: Self-Join Using Inline View

**Query: Compare each student's GPA to their major average**

```sql
SELECT 
    s.first_name,
    s.last_name,
    s.major,
    s.gpa,
    major_avg.avg_gpa AS major_average,
    ROUND(s.gpa - major_avg.avg_gpa, 2) AS difference
FROM students s
JOIN (
    SELECT major, AVG(gpa) AS avg_gpa
    FROM students
    WHERE major IS NOT NULL AND gpa IS NOT NULL
    GROUP BY major
) major_avg ON s.major = major_avg.major
WHERE s.gpa IS NOT NULL
ORDER BY s.major, s.gpa DESC;
```

**Result:**
| first_name | last_name | major | gpa | major_average | difference |
|------------|-----------|-------|-----|---------------|------------|
| John | Smith | Computer Science | 3.8 | 3.5 | 0.30 |
| Bob | Wilson | Computer Science | 3.2 | 3.5 | -0.30 |
| Jane | Doe | Mathematics | 3.9 | 3.9 | 0.00 |
| Alice | Brown | Physics | 3.7 | 3.7 | 0.00 |

### Pattern 4: Creating Running Totals

**Query: Show cumulative enrollments by semester**

```sql
SELECT 
    semester,
    enrollments_this_semester,
    SUM(enrollments_this_semester) OVER (ORDER BY semester) AS cumulative_enrollments
FROM (
    SELECT 
        semester,
        COUNT(*) AS enrollments_this_semester
    FROM enrollments
    GROUP BY semester
) semester_counts
ORDER BY semester;
```

**Result:**
| semester | enrollments_this_semester | cumulative_enrollments |
|----------|---------------------------|------------------------|
| Fall 2023 | 3 | 3 |
| Spring 2024 | 4 | 7 |
| Fall 2024 | 1 | 8 |

## Inline Views with Joins

### Example 1: Join Inline View with Regular Table

**Query: Show instructors with their course counts**

```sql
SELECT 
    i.instructor_name,
    i.department,
    COALESCE(course_counts.num_courses, 0) AS courses_taught
FROM instructors i
LEFT JOIN (
    SELECT instructor_id, COUNT(*) AS num_courses
    FROM courses
    WHERE instructor_id IS NOT NULL
    GROUP BY instructor_id
) course_counts ON i.instructor_id = course_counts.instructor_id
ORDER BY courses_taught DESC;
```

**Result:**
| instructor_name | department | courses_taught |
|-----------------|------------|----------------|
| Dr. Johnson | Computer Science | 3 |
| Dr. Lee | Mathematics | 1 |
| Dr. Martinez | Physics | 1 |
| Dr. Taylor | Chemistry | 0 |

### Example 2: Multiple Inline Views Joined

**Query: Compare student enrollment counts to major averages**

```sql
SELECT 
    student_info.first_name,
    student_info.last_name,
    student_info.major,
    student_info.enrollment_count,
    major_avg.avg_enrollments
FROM (
    SELECT 
        s.student_id,
        s.first_name,
        s.last_name,
        s.major,
        COUNT(e.enrollment_id) AS enrollment_count
    FROM students s
    LEFT JOIN enrollments e ON s.student_id = e.student_id
    GROUP BY s.student_id, s.first_name, s.last_name, s.major
) student_info
LEFT JOIN (
    SELECT 
        s.major,
        AVG(enrollment_count) AS avg_enrollments
    FROM (
        SELECT s.student_id, s.major, COUNT(e.enrollment_id) AS enrollment_count
        FROM students s
        LEFT JOIN enrollments e ON s.student_id = e.student_id
        WHERE s.major IS NOT NULL
        GROUP BY s.student_id, s.major
    )
    GROUP BY s.major
) major_avg ON student_info.major = major_avg.major
WHERE student_info.major IS NOT NULL
ORDER BY student_info.major, student_info.enrollment_count DESC;
```

## Inline Views vs. WITH Clause (CTEs)

The WITH clause (Common Table Expressions) provides an alternative syntax for inline views that's often more readable.

### Inline View Syntax

```sql
SELECT s.first_name, ec.enrollment_count
FROM students s
JOIN (
    SELECT student_id, COUNT(*) AS enrollment_count
    FROM enrollments
    GROUP BY student_id
) ec ON s.student_id = ec.student_id;
```

### WITH Clause Syntax (CTE)

```sql
WITH enrollment_counts AS (
    SELECT student_id, COUNT(*) AS enrollment_count
    FROM enrollments
    GROUP BY student_id
)
SELECT s.first_name, ec.enrollment_count
FROM students s
JOIN enrollment_counts ec ON s.student_id = ec.student_id;
```

### Comparison

| Aspect | Inline View | WITH Clause |
|--------|-------------|-------------|
| **Readability** | Can be nested and complex | Clearer, separate definitions |
| **Reusability** | Must repeat if used multiple times | Can reference multiple times |
| **Scope** | Local to FROM clause | Available throughout query |
| **Debugging** | Harder to test parts separately | Easier to test incrementally |
| **Nesting** | Can get deeply nested | Flatter structure |

**Recommendation:** Use WITH clause for complex queries with multiple derived tables; use inline views for simple, one-off transformations.

## Practical Business Examples

### Example 1: Sales Performance Dashboard

**Query: Monthly sales summary with year-over-year comparison**

```sql
SELECT 
    current_month.month,
    current_month.total_sales AS sales_2024,
    prior_month.total_sales AS sales_2023,
    ROUND(
        (current_month.total_sales - prior_month.total_sales) / prior_month.total_sales * 100, 
        2
    ) AS yoy_growth_pct
FROM (
    SELECT 
        TO_CHAR(order_date, 'MM') AS month,
        SUM(order_total) AS total_sales
    FROM orders
    WHERE EXTRACT(YEAR FROM order_date) = 2024
    GROUP BY TO_CHAR(order_date, 'MM')
) current_month
LEFT JOIN (
    SELECT 
        TO_CHAR(order_date, 'MM') AS month,
        SUM(order_total) AS total_sales
    FROM orders
    WHERE EXTRACT(YEAR FROM order_date) = 2023
    GROUP BY TO_CHAR(order_date, 'MM')
) prior_month ON current_month.month = prior_month.month
ORDER BY current_month.month;
```

### Example 2: Student Academic Standing Report

**Query: Classify students by performance tier**

```sql
SELECT 
    first_name,
    last_name,
    gpa,
    enrollment_count,
    CASE 
        WHEN gpa >= 3.7 THEN 'Dean''s List'
        WHEN gpa >= 3.3 THEN 'Honor Roll'
        WHEN gpa >= 2.0 THEN 'Good Standing'
        ELSE 'Academic Probation'
    END AS academic_standing
FROM (
    SELECT 
        s.first_name,
        s.last_name,
        s.gpa,
        COUNT(e.enrollment_id) AS enrollment_count
    FROM students s
    LEFT JOIN enrollments e ON s.student_id = e.student_id
    WHERE s.gpa IS NOT NULL
    GROUP BY s.student_id, s.first_name, s.last_name, s.gpa
) student_summary
ORDER BY gpa DESC;
```

### Example 3: Course Capacity Analysis

**Query: Identify under-enrolled and over-enrolled courses**

```sql
SELECT 
    course_id,
    course_name,
    current_enrollment,
    max_capacity,
    max_capacity - current_enrollment AS available_seats,
    ROUND(current_enrollment * 100.0 / max_capacity, 1) AS fill_rate,
    CASE 
        WHEN current_enrollment >= max_capacity THEN 'Full'
        WHEN current_enrollment >= max_capacity * 0.8 THEN 'Nearly Full'
        WHEN current_enrollment >= max_capacity * 0.5 THEN 'Moderate'
        ELSE 'Under-enrolled'
    END AS enrollment_status
FROM (
    SELECT 
        c.course_id,
        c.course_name,
        COUNT(e.enrollment_id) AS current_enrollment,
        30 AS max_capacity  -- Assume 30 seat capacity
    FROM courses c
    LEFT JOIN enrollments e ON c.course_id = e.course_id
    GROUP BY c.course_id, c.course_name
) course_stats
ORDER BY fill_rate DESC;
```

## Common Mistakes and Solutions

### Mistake 1: Forgetting Table Alias

**Problem:**
```sql
-- ERROR: derived table must have alias
SELECT course_id, enrollment_count
FROM (
    SELECT course_id, COUNT(*) AS enrollment_count
    FROM enrollments
    GROUP BY course_id
);
```

**Solution:**
```sql
SELECT course_id, enrollment_count
FROM (
    SELECT course_id, COUNT(*) AS enrollment_count
    FROM enrollments
    GROUP BY course_id
) enrollment_stats;  -- Alias required
```

### Mistake 2: Column Name Conflicts

**Problem:**
```sql
-- Ambiguous column reference
SELECT course_id
FROM courses c
JOIN (
    SELECT course_id, COUNT(*) AS cnt
    FROM enrollments
    GROUP BY course_id
) e ON c.course_id = e.course_id;
```

**Solution:**
```sql
-- Qualify column names
SELECT c.course_id, c.course_name, e.cnt
FROM courses c
JOIN (
    SELECT course_id, COUNT(*) AS cnt
    FROM enrollments
    GROUP BY course_id
) e ON c.course_id = e.course_id;
```

### Mistake 3: Overly Complex Nesting

**Problem:** Deep nesting makes queries hard to read and maintain.

```sql
SELECT * FROM (
    SELECT * FROM (
        SELECT * FROM (
            SELECT * FROM table1
        ) t1
    ) t2
) t3;
```

**Solution:** Use WITH clause for better readability:

```sql
WITH step1 AS (
    SELECT * FROM table1
),
step2 AS (
    SELECT * FROM step1
)
SELECT * FROM step2;
```

## Performance Considerations

### Optimization Tips

1. **Filter Early**
```sql
-- Good: Filter in inline view
SELECT * FROM (
    SELECT * FROM large_table WHERE date >= '2024-01-01'
) filtered;

-- Less efficient: Filter after
SELECT * FROM (
    SELECT * FROM large_table
) all_rows
WHERE date >= '2024-01-01';
```

2. **Avoid Unnecessary Columns**
```sql
-- Good: Select only needed columns
SELECT student_id, gpa FROM (
    SELECT student_id, gpa FROM students
) s;

-- Wasteful: Select all then project
SELECT student_id, gpa FROM (
    SELECT * FROM students
) s;
```

3. **Use Indexes on Join Columns**
```sql
-- Ensure indexed columns in inline view joins
CREATE INDEX idx_student_id ON enrollments(student_id);

SELECT * FROM students s
JOIN (SELECT student_id, COUNT(*) AS cnt FROM enrollments GROUP BY student_id) e
ON s.student_id = e.student_id;
```

4. **Consider Materialized Views**

For frequently used inline views, consider creating materialized views:
```sql
CREATE MATERIALIZED VIEW enrollment_counts AS
SELECT course_id, COUNT(*) AS enrollment_count
FROM enrollments
GROUP BY course_id;
```

## Summary

**Key Takeaways:**

1. **Inline views are subqueries in the FROM clause** that create temporary result sets treated as tables.

2. **Must have an alias** - every inline view requires a table alias to reference it in the outer query.

3. **Enable multi-step logic** - break complex queries into pre-aggregation, calculation, and filtering stages.

4. **Can be joined** with regular tables or other inline views using standard join syntax.

5. **Useful for**: aggregating before joining, avoiding repeated calculations, filtering on aggregate results, and creating running totals or rankings.

6. **WITH clause alternative** - CTEs provide cleaner syntax for complex queries with multiple derived tables.

7. **Performance tips**: filter early in the inline view, select only needed columns, ensure proper indexes on join columns, and consider materialized views for frequently used patterns.

8. **Common patterns**: multi-level aggregation, top-N queries, comparing to group averages, and creating analytical reports.

Inline views are powerful tools for creating modular, readable queries that break down complex analytical logic into manageable steps.

