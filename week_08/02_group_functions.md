# Group Functions (Aggregate Functions)

## Overview

**Group functions** (also called **aggregate functions**) operate on sets of rows to return a single result per group. Unlike single-row functions that process each row independently, group functions combine multiple rows to produce summary statistics like totals, averages, counts, and extremes.

## Key Terms

**Group Function**: A function that operates on multiple rows and returns a single result for the group.

**Aggregate Function**: Another term for group function; emphasizes combining multiple values into one.

**GROUP BY Clause**: SQL clause that divides rows into groups for aggregate calculation.

**HAVING Clause**: Filters groups after aggregation (analogous to WHERE for rows).

**NULL Handling**: Group functions ignore NULL values (except COUNT(*)).

**DISTINCT**: Keyword to consider only unique values in aggregation.

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

## Common Group Functions

| Function | Purpose | NULL Handling |
|----------|---------|---------------|
| **COUNT** | Count number of rows/values | COUNT(*) includes NULLs, COUNT(column) excludes |
| **SUM** | Calculate total of numeric values | Ignores NULLs |
| **AVG** | Calculate average of numeric values | Ignores NULLs |
| **MAX** | Find maximum value | Ignores NULLs |
| **MIN** | Find minimum value | Ignores NULLs |
| **STDDEV** | Calculate standard deviation | Ignores NULLs |
| **VARIANCE** | Calculate variance | Ignores NULLs |

## COUNT Function

Counts the number of rows or non-NULL values.

### Syntax Variations

```sql
COUNT(*)           -- Counts all rows (including NULLs)
COUNT(column)      -- Counts non-NULL values in column
COUNT(DISTINCT column)  -- Counts unique non-NULL values
```

### Examples

**Example 1: Count All Rows**

```sql
SELECT COUNT(*) AS total_students
FROM students;
```

**Result:**
| total_students |
|----------------|
| 5 |

**Example 2: Count Non-NULL Values**

```sql
SELECT 
    COUNT(*) AS total_students,
    COUNT(major) AS students_with_major,
    COUNT(DISTINCT major) AS unique_majors
FROM students;
```

**Result:**
| total_students | students_with_major | unique_majors |
|----------------|---------------------|---------------|
| 5 | 4 | 3 |

**Explanation:** Charlie Davis has NULL major, so COUNT(major) returns 4 instead of 5.

**Example 3: Count with Conditions**

```sql
SELECT 
    COUNT(*) AS total_enrollments,
    COUNT(grade) AS graded_enrollments,
    COUNT(CASE WHEN grade = 'A' THEN 1 END) AS a_grades,
    COUNT(CASE WHEN grade_points >= 3.0 THEN 1 END) AS passing_grades
FROM enrollments;
```

**Result:**
| total_enrollments | graded_enrollments | a_grades | passing_grades |
|-------------------|--------------------| ---------|----------------|
| 8 | 7 | 3 | 6 |

**Example 4: Count by Group**

```sql
SELECT 
    major,
    COUNT(*) AS student_count
FROM students
WHERE major IS NOT NULL
GROUP BY major
ORDER BY student_count DESC;
```

**Result:**
| major | student_count |
|-------|---------------|
| Computer Science | 2 |
| Mathematics | 1 |
| Physics | 1 |

## SUM Function

Calculates the total of numeric values.

### Syntax

```sql
SUM(numeric_column)
SUM(DISTINCT numeric_column)
```

### Examples

**Example 1: Simple Sum**

```sql
SELECT 
    SUM(credits) AS total_credits,
    COUNT(*) AS number_of_courses
FROM courses;
```

**Result:**
| total_credits | number_of_courses |
|---------------|-------------------|
| 21 | 6 |

**Example 2: Sum by Group**

```sql
SELECT 
    department,
    SUM(credits) AS total_credits,
    COUNT(*) AS course_count
FROM courses
WHERE department IS NOT NULL
GROUP BY department
ORDER BY total_credits DESC;
```

**Result:**
| department | total_credits | course_count |
|------------|---------------|--------------|
| Computer Science | 10 | 3 |
| Mathematics | 4 | 1 |
| Physics | 4 | 1 |
| English | 3 | 1 |

**Example 3: Sum with DISTINCT**

```sql
SELECT 
    SUM(credits) AS total_credits,
    SUM(DISTINCT credits) AS sum_unique_credits
FROM courses;
```

**Result:**
| total_credits | sum_unique_credits |
|---------------|--------------------|
| 21 | 7 |

**Explanation:** Unique credit values are 3 and 4, so SUM(DISTINCT credits) = 7.

**Example 4: Conditional Sum**

```sql
SELECT 
    SUM(CASE WHEN grade = 'A' THEN credits ELSE 0 END) AS a_grade_credits,
    SUM(CASE WHEN grade_points >= 3.0 THEN credits ELSE 0 END) AS passing_credits
FROM enrollments e
JOIN courses c ON e.course_id = c.course_id
WHERE e.student_id = 1;
```

## AVG Function

Calculates the average of numeric values.

### Syntax

```sql
AVG(numeric_column)
AVG(DISTINCT numeric_column)
```

### Examples

**Example 1: Simple Average**

```sql
SELECT 
    AVG(gpa) AS average_gpa,
    MIN(gpa) AS lowest_gpa,
    MAX(gpa) AS highest_gpa
FROM students
WHERE gpa IS NOT NULL;
```

**Result:**
| average_gpa | lowest_gpa | highest_gpa |
|-------------|------------|-------------|
| 3.48 | 2.8 | 3.9 |

**Example 2: Average by Group**

```sql
SELECT 
    major,
    COUNT(*) AS student_count,
    AVG(gpa) AS average_gpa,
    ROUND(AVG(gpa), 2) AS rounded_avg
FROM students
WHERE major IS NOT NULL AND gpa IS NOT NULL
GROUP BY major
ORDER BY average_gpa DESC;
```

**Result:**
| major | student_count | average_gpa | rounded_avg |
|-------|---------------|-------------|-------------|
| Mathematics | 1 | 3.9 | 3.90 |
| Physics | 1 | 3.7 | 3.70 |
| Computer Science | 2 | 3.5 | 3.50 |

**Example 3: Average vs. Median**

```sql
SELECT 
    department,
    AVG(credits) AS mean_credits,
    MEDIAN(credits) AS median_credits
FROM courses
WHERE department IS NOT NULL
GROUP BY department;
```

**Example 4: Weighted Average**

```sql
-- Calculate weighted GPA based on course credits
SELECT 
    s.first_name,
    s.last_name,
    SUM(e.grade_points * c.credits) / SUM(c.credits) AS weighted_gpa
FROM students s
JOIN enrollments e ON s.student_id = e.student_id
JOIN courses c ON e.course_id = c.course_id
WHERE e.grade_points IS NOT NULL
GROUP BY s.student_id, s.first_name, s.last_name;
```

**Result:**
| first_name | last_name | weighted_gpa |
|------------|-----------|--------------|
| John | Smith | 3.60 |
| Jane | Doe | 3.87 |
| Bob | Wilson | 3.17 |
| Alice | Brown | 4.00 |

## MAX and MIN Functions

Find maximum and minimum values.

### Syntax

```sql
MAX(column)
MIN(column)
```

### Examples

**Example 1: Numeric MAX/MIN**

```sql
SELECT 
    MAX(gpa) AS highest_gpa,
    MIN(gpa) AS lowest_gpa,
    MAX(gpa) - MIN(gpa) AS gpa_range
FROM students
WHERE gpa IS NOT NULL;
```

**Result:**
| highest_gpa | lowest_gpa | gpa_range |
|-------------|------------|-----------|
| 3.9 | 2.8 | 1.1 |

**Example 2: Date MAX/MIN**

```sql
SELECT 
    MIN(enrollment_date) AS earliest_enrollment,
    MAX(enrollment_date) AS latest_enrollment,
    MAX(enrollment_date) - MIN(enrollment_date) AS days_span
FROM students;
```

**Result:**
| earliest_enrollment | latest_enrollment | days_span |
|---------------------|-------------------|-----------|
| 2023-09-01 | 2024-09-01 | 366 |

**Example 3: String MAX/MIN (Alphabetical)**

```sql
SELECT 
    MIN(last_name) AS first_alphabetically,
    MAX(last_name) AS last_alphabetically
FROM students;
```

**Result:**
| first_alphabetically | last_alphabetically |
|----------------------|---------------------|
| Brown | Wilson |

**Example 4: MAX/MIN by Group**

```sql
SELECT 
    major,
    MAX(gpa) AS top_gpa,
    MIN(gpa) AS lowest_gpa
FROM students
WHERE major IS NOT NULL AND gpa IS NOT NULL
GROUP BY major;
```

**Result:**
| major | top_gpa | lowest_gpa |
|-------|---------|------------|
| Computer Science | 3.8 | 3.2 |
| Mathematics | 3.9 | 3.9 |
| Physics | 3.7 | 3.7 |

## GROUP BY Clause

Divides rows into groups for aggregate calculations.

### Syntax

```sql
SELECT column(s), aggregate_function(column)
FROM table
WHERE condition
GROUP BY column(s)
HAVING group_condition
ORDER BY column(s);
```

### Rules

1. **All non-aggregated columns in SELECT must be in GROUP BY**
2. **WHERE filters before grouping, HAVING filters after**
3. **Can group by multiple columns**
4. **NULL values form their own group**

### Examples

**Example 1: Single Column Grouping**

```sql
SELECT 
    department,
    COUNT(*) AS course_count,
    AVG(credits) AS avg_credits
FROM courses
WHERE department IS NOT NULL
GROUP BY department
ORDER BY course_count DESC;
```

**Result:**
| department | course_count | avg_credits |
|------------|--------------|-------------|
| Computer Science | 3 | 3.33 |
| English | 1 | 3.00 |
| Mathematics | 1 | 4.00 |
| Physics | 1 | 4.00 |

**Example 2: Multiple Column Grouping**

```sql
SELECT 
    major,
    CASE 
        WHEN gpa >= 3.5 THEN 'High'
        WHEN gpa >= 3.0 THEN 'Medium'
        ELSE 'Low'
    END AS gpa_category,
    COUNT(*) AS student_count
FROM students
WHERE major IS NOT NULL AND gpa IS NOT NULL
GROUP BY major, 
         CASE 
             WHEN gpa >= 3.5 THEN 'High'
             WHEN gpa >= 3.0 THEN 'Medium'
             ELSE 'Low'
         END
ORDER BY major, gpa_category;
```

**Example 3: Grouping with JOINs**

```sql
SELECT 
    c.department,
    c.course_name,
    COUNT(e.enrollment_id) AS enrollment_count,
    AVG(e.grade_points) AS average_grade
FROM courses c
LEFT JOIN enrollments e ON c.course_id = e.course_id
GROUP BY c.department, c.course_name
HAVING COUNT(e.enrollment_id) > 0
ORDER BY enrollment_count DESC;
```

**Result:**
| department | course_name | enrollment_count | average_grade |
|------------|-------------|------------------|---------------|
| Computer Science | Introduction to Programming | 3 | 3.57 |
| Computer Science | Data Structures | 2 | 3.30 |
| Computer Science | Database Systems | 1 | NULL |
| Mathematics | Calculus I | 1 | 4.00 |
| Physics | Physics I | 1 | 4.00 |

**Example 4: Nested Grouping**

```sql
SELECT 
    semester,
    COUNT(DISTINCT student_id) AS unique_students,
    COUNT(*) AS total_enrollments,
    ROUND(AVG(grade_points), 2) AS avg_grade
FROM enrollments
WHERE grade_points IS NOT NULL
GROUP BY semester
ORDER BY semester;
```

**Result:**
| semester | unique_students | total_enrollments | avg_grade |
|----------|-----------------|-------------------|-----------|
| Fall 2023 | 2 | 3 | 3.90 |
| Spring 2024 | 3 | 4 | 3.33 |

## HAVING Clause

Filters groups after aggregation (WHERE filters before).

### Syntax

```sql
SELECT columns, aggregate_function(column)
FROM table
GROUP BY columns
HAVING aggregate_condition;
```

### Examples

**Example 1: Filter by Aggregate Value**

```sql
SELECT 
    major,
    COUNT(*) AS student_count,
    AVG(gpa) AS average_gpa
FROM students
WHERE major IS NOT NULL AND gpa IS NOT NULL
GROUP BY major
HAVING AVG(gpa) > 3.5
ORDER BY average_gpa DESC;
```

**Result:**
| major | student_count | average_gpa |
|-------|---------------|-------------|
| Mathematics | 1 | 3.9 |
| Physics | 1 | 3.7 |

**Example 2: Filter by Count**

```sql
SELECT 
    course_id,
    COUNT(*) AS enrollment_count
FROM enrollments
GROUP BY course_id
HAVING COUNT(*) >= 2
ORDER BY enrollment_count DESC;
```

**Result:**
| course_id | enrollment_count |
|-----------|------------------|
| CS101 | 3 |
| CS201 | 2 |

**Example 3: Multiple HAVING Conditions**

```sql
SELECT 
    department,
    COUNT(*) AS course_count,
    AVG(credits) AS avg_credits
FROM courses
WHERE department IS NOT NULL
GROUP BY department
HAVING COUNT(*) >= 1
   AND AVG(credits) >= 3
ORDER BY course_count DESC;
```

**Example 4: HAVING with Subquery**

```sql
SELECT 
    major,
    AVG(gpa) AS avg_gpa
FROM students
WHERE major IS NOT NULL AND gpa IS NOT NULL
GROUP BY major
HAVING AVG(gpa) > (SELECT AVG(gpa) FROM students WHERE gpa IS NOT NULL)
ORDER BY avg_gpa DESC;
```

**Result:**
| major | avg_gpa |
|-------|---------|
| Mathematics | 3.9 |
| Physics | 3.7 |
| Computer Science | 3.5 |

## WHERE vs. HAVING

Understanding the difference is crucial for correct queries.

### Comparison Table

| Aspect | WHERE | HAVING |
|--------|-------|--------|
| **Purpose** | Filters individual rows | Filters groups |
| **When Applied** | Before grouping | After grouping |
| **Can Use** | Column names, literals | Aggregate functions |
| **Cannot Use** | Aggregate functions | N/A (can use anything) |

### Example: Both WHERE and HAVING

```sql
SELECT 
    major,
    COUNT(*) AS student_count,
    AVG(gpa) AS avg_gpa
FROM students
WHERE gpa >= 3.0              -- WHERE: Filter rows before grouping
  AND major IS NOT NULL
GROUP BY major
HAVING COUNT(*) >= 1          -- HAVING: Filter groups after aggregation
ORDER BY avg_gpa DESC;
```

**Execution Order:**
1. WHERE filters individual students (gpa >= 3.0, major not null)
2. GROUP BY groups remaining students by major
3. COUNT and AVG calculated for each group
4. HAVING filters groups (count >= 1)
5. ORDER BY sorts final result

## Advanced Group Function Patterns

### Pattern 1: Statistical Summary

```sql
SELECT 
    major,
    COUNT(*) AS n,
    MIN(gpa) AS min_gpa,
    MAX(gpa) AS max_gpa,
    AVG(gpa) AS mean_gpa,
    STDDEV(gpa) AS std_dev,
    VARIANCE(gpa) AS variance
FROM students
WHERE major IS NOT NULL AND gpa IS NOT NULL
GROUP BY major;
```

### Pattern 2: Percentage Calculations

```sql
SELECT 
    major,
    COUNT(*) AS student_count,
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM students WHERE major IS NOT NULL), 2) AS percentage
FROM students
WHERE major IS NOT NULL
GROUP BY major
ORDER BY percentage DESC;
```

**Result:**
| major | student_count | percentage |
|-------|---------------|------------|
| Computer Science | 2 | 50.00 |
| Mathematics | 1 | 25.00 |
| Physics | 1 | 25.00 |

### Pattern 3: Rollup and Subtotals

```sql
SELECT 
    department,
    course_name,
    COUNT(e.enrollment_id) AS enrollments
FROM courses c
LEFT JOIN enrollments e ON c.course_id = e.course_id
WHERE c.department IS NOT NULL
GROUP BY ROLLUP(department, course_name)
ORDER BY department NULLS LAST, course_name NULLS LAST;
```

**Note:** ROLLUP creates subtotals and grand totals.

### Pattern 4: Conditional Aggregation

```sql
SELECT 
    semester,
    COUNT(*) AS total_enrollments,
    SUM(CASE WHEN grade = 'A' THEN 1 ELSE 0 END) AS a_count,
    SUM(CASE WHEN grade IN ('B', 'B+') THEN 1 ELSE 0 END) AS b_count,
    SUM(CASE WHEN grade_points < 3.0 THEN 1 ELSE 0 END) AS below_b_count,
    ROUND(AVG(grade_points), 2) AS avg_grade_points
FROM enrollments
WHERE grade IS NOT NULL
GROUP BY semester
ORDER BY semester;
```

**Result:**
| semester | total_enrollments | a_count | b_count | below_b_count | avg_grade_points |
|----------|-------------------|---------|---------|---------------|------------------|
| Fall 2023 | 3 | 2 | 0 | 0 | 3.90 |
| Spring 2024 | 4 | 1 | 3 | 0 | 3.40 |

## NULL Handling in Group Functions

### Important Rules

1. **COUNT(*) includes NULLs** - counts all rows
2. **COUNT(column) excludes NULLs** - counts non-NULL values
3. **Other aggregates ignore NULLs** - SUM, AVG, MAX, MIN skip NULL values
4. **Empty groups return NULL** - except COUNT which returns 0

### Examples

```sql
SELECT 
    COUNT(*) AS all_enrollments,
    COUNT(grade) AS graded_count,
    COUNT(grade_points) AS points_count,
    AVG(grade_points) AS avg_points,
    SUM(grade_points) AS sum_points
FROM enrollments;
```

**Result:**
| all_enrollments | graded_count | points_count | avg_points | sum_points |
|-----------------|--------------|--------------|------------|------------|
| 8 | 7 | 7 | 3.61 | 25.3 |

**Explanation:** One enrollment has NULL grade, so COUNT(grade) = 7 instead of 8.

## Practical Business Examples

### Example 1: Course Performance Report

```sql
SELECT 
    c.course_id,
    c.course_name,
    COUNT(e.enrollment_id) AS total_enrolled,
    COUNT(e.grade) AS graded,
    ROUND(AVG(e.grade_points), 2) AS avg_grade,
    MAX(e.grade) AS best_grade,
    MIN(e.grade) AS worst_grade
FROM courses c
LEFT JOIN enrollments e ON c.course_id = e.course_id
GROUP BY c.course_id, c.course_name
HAVING COUNT(e.enrollment_id) > 0
ORDER BY avg_grade DESC NULLS LAST;
```

### Example 2: Enrollment Trends

```sql
SELECT 
    semester,
    COUNT(DISTINCT student_id) AS unique_students,
    COUNT(DISTINCT course_id) AS courses_offered,
    COUNT(*) AS total_enrollments,
    ROUND(COUNT(*) * 1.0 / COUNT(DISTINCT student_id), 2) AS avg_courses_per_student
FROM enrollments
GROUP BY semester
ORDER BY semester;
```

### Example 3: Instructor Workload

```sql
SELECT 
    i.instructor_name,
    i.department,
    COUNT(c.course_id) AS courses_teaching,
    SUM(c.credits) AS total_credits,
    COUNT(e.enrollment_id) AS total_students
FROM instructors i
LEFT JOIN courses c ON i.instructor_id = c.instructor_id
LEFT JOIN enrollments e ON c.course_id = e.course_id
GROUP BY i.instructor_id, i.instructor_name, i.department
ORDER BY total_students DESC;
```

## Common Mistakes and Solutions

### Mistake 1: Missing GROUP BY Column

**Problem:**
```sql
-- ERROR: not a GROUP BY expression
SELECT department, course_name, COUNT(*)
FROM courses
GROUP BY department;
```

**Solution:**
```sql
SELECT department, course_name, COUNT(*)
FROM courses
GROUP BY department, course_name;
```

### Mistake 2: Using Aggregate in WHERE

**Problem:**
```sql
-- ERROR: cannot use aggregate function in WHERE
SELECT major, AVG(gpa)
FROM students
WHERE AVG(gpa) > 3.5
GROUP BY major;
```

**Solution:**
```sql
SELECT major, AVG(gpa)
FROM students
GROUP BY major
HAVING AVG(gpa) > 3.5;
```

### Mistake 3: Forgetting NULL Handling

**Problem:**
```sql
-- May give unexpected results with NULLs
SELECT AVG(grade_points) FROM enrollments;
```

**Better:**
```sql
-- Explicit NULL handling
SELECT 
    AVG(grade_points) AS avg_excluding_nulls,
    AVG(NVL(grade_points, 0)) AS avg_treating_null_as_zero
FROM enrollments;
```

## Summary

**Key Takeaways:**

1. **Group functions operate on sets of rows** to return single summary values per group (COUNT, SUM, AVG, MAX, MIN).

2. **COUNT has two forms**: COUNT(*) includes NULLs and counts all rows; COUNT(column) excludes NULLs and counts non-NULL values.

3. **GROUP BY divides rows into groups** where each group shares common values in the GROUP BY columns; all non-aggregated SELECT columns must appear in GROUP BY.

4. **HAVING filters groups after aggregation** while WHERE filters individual rows before grouping; HAVING can use aggregate functions, WHERE cannot.

5. **Most aggregate functions ignore NULLs** except COUNT(*) which includes all rows regardless of NULL values.

6. **DISTINCT keyword** in aggregate functions (e.g., COUNT(DISTINCT column)) considers only unique values.

7. **Common patterns**: statistical summaries, percentage calculations, conditional aggregation with CASE, multi-level grouping, and filtering with HAVING.

8. **Execution order**: WHERE → GROUP BY → Aggregation → HAVING → ORDER BY.

Group functions are essential for data analysis, reporting, and generating summary statistics from large datasets.

