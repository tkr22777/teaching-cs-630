# SQL Group Functions (Aggregate Functions)

## Overview

**Group functions** (also called aggregate functions) operate on multiple rows and return a single result. They're essential for statistical analysis, reporting, and data summarization.

## Key Terms

**Group Function**: Function that operates on multiple rows and returns a single value.

**Aggregate Function**: Another name for group function.

**GROUP BY**: Clause that divides rows into groups for aggregation.

**HAVING**: Clause that filters groups (used with GROUP BY).

**NULL Handling**: Group functions ignore NULL values (except COUNT(*)).

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

| Function | Description | NULL Handling |
|----------|-------------|---------------|
| **COUNT(*)** | Count all rows | Includes NULLs |
| **COUNT(column)** | Count non-NULL values | Ignores NULLs |
| **SUM(column)** | Total of values | Ignores NULLs |
| **AVG(column)** | Average of values | Ignores NULLs |
| **MAX(column)** | Maximum value | Ignores NULLs |
| **MIN(column)** | Minimum value | Ignores NULLs |

## COUNT Function

**Purpose:** Count rows or non-NULL values.

**Syntax:**
- `COUNT(*)` - Count all rows (including NULLs)
- `COUNT(column)` - Count non-NULL values in column
- `COUNT(DISTINCT column)` - Count unique non-NULL values

**Example:**

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

Charlie has NULL major, so `COUNT(major)` excludes him.

## SUM Function

**Purpose:** Calculate total of numeric values.

**Syntax:** `SUM(column)`

**Example:**

```sql
SELECT 
    SUM(credits) AS total_credits_offered,
    SUM(CASE WHEN department = 'Computer Science' THEN credits ELSE 0 END) AS cs_credits
FROM courses;
```

**Result:**
| total_credits_offered | cs_credits |
|----------------------|------------|
| 21 | 10 |

## AVG Function

**Purpose:** Calculate average of numeric values.

**Syntax:** `AVG(column)`

**Example:**

```sql
SELECT 
    AVG(gpa) AS overall_avg_gpa,
    ROUND(AVG(gpa), 2) AS rounded_avg,
    AVG(CASE WHEN major = 'Computer Science' THEN gpa END) AS cs_avg_gpa
FROM students
WHERE gpa IS NOT NULL;
```

**Result:**
| overall_avg_gpa | rounded_avg | cs_avg_gpa |
|-----------------|-------------|------------|
| 3.48 | 3.48 | 3.5 |

**Note:** AVG ignores NULL values automatically.

## MAX and MIN Functions

**Purpose:** Find maximum or minimum value.

**Syntax:** `MAX(column)`, `MIN(column)`

**Works with:** Numbers, dates, strings (alphabetical order)

**Example:**

```sql
SELECT 
    MAX(gpa) AS highest_gpa,
    MIN(gpa) AS lowest_gpa,
    MAX(enrollment_date) AS most_recent_enrollment,
    MIN(last_name) AS first_alphabetically
FROM students
WHERE gpa IS NOT NULL;
```

**Result:**
| highest_gpa | lowest_gpa | most_recent_enrollment | first_alphabetically |
|-------------|------------|------------------------|----------------------|
| 3.9 | 2.8 | 2024-09-01 | Brown |

## GROUP BY Clause

**Purpose:** Divide rows into groups and apply aggregate functions to each group.

**Syntax:**
```sql
SELECT column, aggregate_function(column)
FROM table
WHERE conditions
GROUP BY column
ORDER BY column;
```

**Rules:**
1. Every column in SELECT (except aggregate functions) must be in GROUP BY
2. GROUP BY executes after WHERE
3. Can group by multiple columns

**Example 1: Count students by major**

```sql
SELECT 
    major,
    COUNT(*) AS student_count,
    AVG(gpa) AS avg_gpa
FROM students
WHERE major IS NOT NULL
GROUP BY major
ORDER BY student_count DESC;
```

**Result:**
| major | student_count | avg_gpa |
|-------|---------------|---------|
| Computer Science | 2 | 3.5 |
| Mathematics | 1 | 3.9 |
| Physics | 1 | 3.7 |

**Example 2: Enrollment statistics by semester**

```sql
SELECT 
    semester,
    COUNT(*) AS total_enrollments,
    COUNT(DISTINCT student_id) AS unique_students,
    AVG(grade_points) AS avg_grade_points
FROM enrollments
WHERE grade IS NOT NULL
GROUP BY semester
ORDER BY semester;
```

**Result:**
| semester | total_enrollments | unique_students | avg_grade_points |
|----------|-------------------|-----------------|------------------|
| Fall 2023 | 3 | 2 | 3.90 |
| Spring 2024 | 4 | 3 | 3.40 |

## HAVING Clause

**Purpose:** Filter groups after aggregation (WHERE filters rows before aggregation).

**Syntax:**
```sql
SELECT column, aggregate_function(column)
FROM table
WHERE row_conditions
GROUP BY column
HAVING group_conditions
ORDER BY column;
```

**Example: Find majors with average GPA above 3.5**

```sql
SELECT 
    major,
    COUNT(*) AS student_count,
    AVG(gpa) AS avg_gpa
FROM students
WHERE major IS NOT NULL
GROUP BY major
HAVING AVG(gpa) > 3.5
ORDER BY avg_gpa DESC;
```

**Result:**
| major | student_count | avg_gpa |
|-------|---------------|---------|
| Mathematics | 1 | 3.9 |
| Physics | 1 | 3.7 |

Computer Science (3.5) is excluded because it's not > 3.5.

## WHERE vs. HAVING

| Aspect | WHERE | HAVING |
|--------|-------|--------|
| **Filters** | Individual rows | Groups |
| **Executes** | Before GROUP BY | After GROUP BY |
| **Can use aggregates** | No | Yes |
| **Use for** | Row-level conditions | Group-level conditions |

**Example using both:**

```sql
SELECT 
    c.department,
    COUNT(*) AS course_count,
    AVG(c.credits) AS avg_credits
FROM courses c
WHERE c.instructor_id IS NOT NULL  -- WHERE: filter rows
GROUP BY c.department
HAVING COUNT(*) >= 2               -- HAVING: filter groups
ORDER BY course_count DESC;
```

**Result:**
| department | course_count | avg_credits |
|------------|--------------|-------------|
| Computer Science | 3 | 3.33 |

**Explanation:**
- WHERE removes ENG101 (NULL instructor)
- GROUP BY creates groups
- HAVING keeps only departments with 2+ courses

## Advanced Patterns

### Pattern 1: Conditional Aggregation

**Count specific conditions within groups:**

```sql
SELECT 
    major,
    COUNT(*) AS total_students,
    COUNT(CASE WHEN gpa >= 3.5 THEN 1 END) AS high_performers,
    COUNT(CASE WHEN gpa < 3.0 THEN 1 END) AS at_risk
FROM students
WHERE major IS NOT NULL
GROUP BY major;
```

**Result:**
| major | total_students | high_performers | at_risk |
|-------|----------------|-----------------|---------|
| Computer Science | 2 | 1 | 0 |
| Mathematics | 1 | 1 | 0 |
| Physics | 1 | 0 | 0 |

### Pattern 2: Multiple Grouping Columns

**Group by semester and course:**

```sql
SELECT 
    semester,
    c.department,
    COUNT(*) AS enrollments,
    AVG(e.grade_points) AS avg_grade
FROM enrollments e
JOIN courses c ON e.course_id = c.course_id
WHERE e.grade IS NOT NULL
GROUP BY semester, c.department
ORDER BY semester, c.department;
```

**Result:**
| semester | department | enrollments | avg_grade |
|----------|------------|-------------|-----------|
| Fall 2023 | Computer Science | 2 | 3.85 |
| Fall 2023 | Mathematics | 1 | 4.0 |
| Spring 2024 | Computer Science | 2 | 3.15 |
| Spring 2024 | Physics | 1 | 4.0 |

## NULL Handling in Group Functions

**Important rules:**
1. `COUNT(*)` includes all rows (even with NULLs)
2. `COUNT(column)` ignores NULLs
3. All other functions (SUM, AVG, MAX, MIN) ignore NULLs
4. If all values are NULL, result is NULL (not 0)

**Example:**

```sql
SELECT 
    COUNT(*) AS total_enrollments,
    COUNT(grade) AS graded_enrollments,
    COUNT(grade_points) AS with_points,
    AVG(grade_points) AS avg_points
FROM enrollments;
```

**Result:**
| total_enrollments | graded_enrollments | with_points | avg_points |
|-------------------|--------------------| ------------|------------|
| 8 | 7 | 7 | 3.53 |

**Explanation:** Enrollment 108 has NULL grade and grade_points, so it's excluded from counts of those columns and the average.

## Common Mistakes

**Mistake 1: Forgetting GROUP BY for non-aggregate columns**

```sql
-- ERROR: major not in GROUP BY
SELECT major, COUNT(*)
FROM students;

-- Correct
SELECT major, COUNT(*)
FROM students
GROUP BY major;
```

**Mistake 2: Using WHERE with aggregate functions**

```sql
-- ERROR: Can't use aggregate in WHERE
SELECT major, AVG(gpa)
FROM students
WHERE AVG(gpa) > 3.5
GROUP BY major;

-- Correct: Use HAVING
SELECT major, AVG(gpa)
FROM students
GROUP BY major
HAVING AVG(gpa) > 3.5;
```

**Mistake 3: Confusing COUNT(*) with COUNT(column)**

```sql
-- COUNT(*) includes NULLs
SELECT COUNT(*) FROM students;  -- Returns: 5

-- COUNT(column) excludes NULLs
SELECT COUNT(major) FROM students;  -- Returns: 4
```

## Summary

**Key Points:**

1. **Group functions** aggregate multiple rows into a single result
2. **Common functions**: COUNT, SUM, AVG, MAX, MIN
3. **COUNT(*)** includes all rows; **COUNT(column)** excludes NULLs
4. **GROUP BY** divides rows into groups for aggregation
5. **HAVING** filters groups (use after GROUP BY)
6. **WHERE vs. HAVING**: WHERE filters rows before grouping, HAVING filters groups after
7. **All columns** in SELECT must be in GROUP BY or be aggregate functions
8. **NULL handling**: Most functions ignore NULLs (except COUNT(*))
9. **Common patterns**: Conditional aggregation with CASE, multiple grouping columns

Group functions are essential for data analysis and reporting in SQL.
