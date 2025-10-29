# Multiple-Row Subqueries

## Overview

A **multiple-row subquery** returns zero or more rows (typically with one column). These subqueries cannot be used with simple comparison operators (=, >, <) and instead require special operators: IN, ANY, ALL, or EXISTS.

## Key Terms

**Multiple-Row Subquery**: A subquery that returns zero, one, or more rows.

**IN Operator**: Tests whether a value matches any value in a list.

**ANY Operator**: Compares a value to each value returned by the subquery; returns TRUE if any comparison is true.

**ALL Operator**: Compares a value to each value returned by the subquery; returns TRUE only if all comparisons are true.

**List Subquery**: A subquery that returns a single column with multiple rows, creating a list of values.

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

### Comparison Table

| Operator | Description | Equivalent | Example |
|----------|-------------|------------|---------|
| **IN** | Equal to any value in list | = ANY | `WHERE id IN (1, 2, 3)` |
| **NOT IN** | Not equal to any value in list | != ALL | `WHERE id NOT IN (4, 5)` |
| **ANY** | Compare to any value (with >, <, =, etc.) | At least one match | `WHERE salary > ANY (...)` |
| **ALL** | Compare to all values (with >, <, =, etc.) | All must match | `WHERE salary > ALL (...)` |
| **EXISTS** | TRUE if subquery returns rows | N/A | `WHERE EXISTS (SELECT ...)` |
| **NOT EXISTS** | TRUE if subquery returns no rows | N/A | `WHERE NOT EXISTS (...)` |

## IN Operator

The **IN** operator checks if a value matches any value in a list of values returned by the subquery.

### Syntax

```sql
WHERE column IN (subquery)
```

### Example 1: Basic IN Subquery

**Query: Find all students enrolled in Computer Science courses**

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

**Execution:**
1. Inner query returns: `(1, 2, 3)` - student IDs enrolled in CS courses
2. Outer query finds: students where student_id is 1, 2, or 3

**Result:**
| first_name | last_name | major |
|------------|-----------|-------|
| John | Smith | Computer Science |
| Jane | Doe | Mathematics |
| Bob | Wilson | Computer Science |

### Example 2: IN with Aggregate Filter

**Query: Find courses taught by instructors in the Computer Science department**

```sql
SELECT course_id, course_name, instructor_id
FROM courses
WHERE instructor_id IN (
    SELECT instructor_id
    FROM instructors
    WHERE department = 'Computer Science'
);
```

### Example 3: Hardcoded List vs. Subquery

**Hardcoded:**
```sql
SELECT first_name, last_name
FROM students
WHERE student_id IN (1, 2, 3);
```

**Dynamic (with subquery):**
```sql
SELECT first_name, last_name
FROM students
WHERE student_id IN (
    SELECT student_id 
    FROM enrollments 
    WHERE grade = 'A'
);
```

**Advantage:** The subquery version adapts automatically as data changes.

## NOT IN Operator

The **NOT IN** operator checks if a value does NOT match any value in the list.

### Syntax

```sql
WHERE column NOT IN (subquery)
```

### Example 1: Finding Unmatched Records

**Query: Find students who have NOT enrolled in any courses**

```sql
SELECT first_name, last_name, enrollment_date
FROM students
WHERE student_id NOT IN (
    SELECT DISTINCT student_id 
    FROM enrollments
);
```

**Result:**
| first_name | last_name | enrollment_date |
|------------|-----------|-----------------|
| Charlie | Davis | 2024-09-01 |

### Example 2: Exclusion Filter

**Query: Find courses not taught by Dr. Johnson**

```sql
SELECT course_id, course_name, instructor_id
FROM courses
WHERE instructor_id NOT IN (
    SELECT instructor_id
    FROM instructors
    WHERE instructor_name = 'Dr. Johnson'
)
OR instructor_id IS NULL;
```

**Important:** Note the `OR instructor_id IS NULL` - this is critical! (See NULL handling section below)

### NOT IN and NULL - A Critical Issue

**Problem:**
```sql
-- May return NO rows if subquery contains NULL
SELECT first_name, last_name
FROM students
WHERE student_id NOT IN (1, 2, NULL);
```

**Why:** 
- `NOT IN (1, 2, NULL)` becomes: `!= 1 AND != 2 AND != NULL`
- Anything compared to NULL returns NULL (not TRUE or FALSE)
- The entire WHERE condition evaluates to NULL, which is treated as FALSE

**Solution 1: Filter NULLs in subquery**
```sql
SELECT first_name, last_name
FROM students
WHERE student_id NOT IN (
    SELECT student_id 
    FROM enrollments
    WHERE student_id IS NOT NULL
);
```

**Solution 2: Use NOT EXISTS (preferred)**
```sql
SELECT first_name, last_name
FROM students s
WHERE NOT EXISTS (
    SELECT 1
    FROM enrollments e
    WHERE e.student_id = s.student_id
);
```

## ANY Operator

The **ANY** operator compares a value to each value returned by the subquery using a comparison operator. Returns TRUE if ANY comparison is true.

### Syntax

```sql
WHERE column operator ANY (subquery)
```

Where operator is: =, !=, >, <, >=, <=

### Operator Meanings

| Expression | Meaning |
|------------|---------|
| `= ANY` | Same as IN |
| `!= ANY` | Not equal to at least one value |
| `> ANY` | Greater than the minimum value |
| `< ANY` | Less than the maximum value |
| `>= ANY` | Greater than or equal to the minimum |
| `<= ANY` | Less than or equal to the maximum |

### Example 1: Greater Than ANY (> ANY)

**Query: Find students with GPA higher than any Computer Science major**

```sql
SELECT first_name, last_name, major, gpa
FROM students
WHERE gpa > ANY (
    SELECT gpa
    FROM students
    WHERE major = 'Computer Science'
    AND gpa IS NOT NULL
)
AND major != 'Computer Science';
```

**Logic:**
- Subquery returns: `(3.8, 3.2)` - GPAs of CS majors
- `> ANY (3.8, 3.2)` means `> 3.2` (greater than minimum)
- Returns students with GPA > 3.2 who are not CS majors

**Result:**
| first_name | last_name | major | gpa |
|------------|-----------|-------|-----|
| Jane | Doe | Mathematics | 3.9 |
| Alice | Brown | Physics | 3.7 |

### Example 2: Equals ANY (= ANY)

**Query: Find students in the same department as course instructors**

```sql
SELECT first_name, last_name, major
FROM students
WHERE major = ANY (
    SELECT department
    FROM instructors
);
```

**Note:** `= ANY` is functionally identical to `IN`.

### Example 3: Less Than ANY (< ANY)

**Query: Find courses with fewer credits than any 4-credit course**

```sql
SELECT course_id, course_name, credits
FROM courses
WHERE credits < ANY (
    SELECT credits
    FROM courses
    WHERE credits = 4
);
```

**Logic:** 
- `< ANY (4, 4, 4, 4)` means `< 4` (less than maximum)
- Returns courses with less than 4 credits

## ALL Operator

The **ALL** operator compares a value to each value in the subquery. Returns TRUE only if ALL comparisons are true.

### Syntax

```sql
WHERE column operator ALL (subquery)
```

### Operator Meanings

| Expression | Meaning |
|------------|---------|
| `> ALL` | Greater than the maximum value |
| `< ALL` | Less than the minimum value |
| `>= ALL` | Greater than or equal to the maximum |
| `<= ALL` | Less than or equal to the minimum |
| `!= ALL` | Same as NOT IN |
| `= ALL` | Equals to all (only true if subquery returns one distinct value) |

### Example 1: Greater Than ALL (> ALL)

**Query: Find students with the highest GPA (higher than all others)**

```sql
SELECT first_name, last_name, gpa
FROM students s1
WHERE gpa > ALL (
    SELECT gpa
    FROM students s2
    WHERE s2.student_id != s1.student_id
    AND gpa IS NOT NULL
)
AND gpa IS NOT NULL;
```

**Alternative (clearer):**
```sql
SELECT first_name, last_name, gpa
FROM students
WHERE gpa = (SELECT MAX(gpa) FROM students);
```

### Example 2: Less Than ALL (< ALL)

**Query: Find courses with fewer credits than any other course**

```sql
SELECT course_id, course_name, credits
FROM courses
WHERE credits < ALL (
    SELECT credits
    FROM courses c2
    WHERE c2.course_id != courses.course_id
);
```

**Logic:** Credits must be less than ALL other courses (i.e., the minimum).

### Example 3: Not Equal to ALL (!= ALL)

**Query: Find students not enrolled in any Computer Science courses**

```sql
SELECT first_name, last_name
FROM students
WHERE student_id != ALL (
    SELECT DISTINCT student_id
    FROM enrollments e
    JOIN courses c ON e.course_id = c.course_id
    WHERE c.department = 'Computer Science'
);
```

**Note:** `!= ALL` is equivalent to `NOT IN`.

## Comparing ANY vs. ALL

### Visual Comparison

**Subquery returns:** `(100, 200, 300)`

| Condition | Equivalent To | Example Values That Match |
|-----------|---------------|---------------------------|
| `> ANY (100, 200, 300)` | `> 100` (min) | 101, 150, 200, 500 |
| `> ALL (100, 200, 300)` | `> 300` (max) | 301, 400, 500 |
| `< ANY (100, 200, 300)` | `< 300` (max) | 50, 100, 250 |
| `< ALL (100, 200, 300)` | `< 100` (min) | 50, 75, 99 |

### Example: Salary Comparison

**Subquery returns salaries:** `(50000, 60000, 70000)`

```sql
-- Earns more than the lowest-paid person
WHERE salary > ANY (50000, 60000, 70000)  
-- TRUE for: 51000, 60000, 75000

-- Earns more than the highest-paid person
WHERE salary > ALL (50000, 60000, 70000)  
-- TRUE for: 71000, 80000, 100000
```

## Practical Examples

### Example 1: Course Enrollment Analysis

**Query: Find courses with more enrollments than any Math course**

```sql
SELECT c.course_id, c.course_name, COUNT(e.enrollment_id) AS enrollment_count
FROM courses c
LEFT JOIN enrollments e ON c.course_id = e.course_id
WHERE c.department != 'Mathematics'
GROUP BY c.course_id, c.course_name
HAVING COUNT(e.enrollment_id) > ANY (
    SELECT COUNT(*)
    FROM enrollments e2
    JOIN courses c2 ON e2.course_id = c2.course_id
    WHERE c2.department = 'Mathematics'
    GROUP BY c2.course_id
);
```

### Example 2: Student Performance Tracking

**Query: Find students who scored higher than all students in a different major**

```sql
SELECT s1.first_name, s1.last_name, s1.major, s1.gpa
FROM students s1
WHERE s1.gpa > ALL (
    SELECT s2.gpa
    FROM students s2
    WHERE s2.major = 'Physics'
    AND s2.gpa IS NOT NULL
)
AND s1.major != 'Physics'
AND s1.gpa IS NOT NULL;
```

### Example 3: Course Prerequisite Checking

**Query: Find students eligible for advanced courses (completed all prerequisites)**

```sql
SELECT s.first_name, s.last_name
FROM students s
WHERE s.student_id NOT IN (
    SELECT DISTINCT student_id
    FROM enrollments
    WHERE course_id IN ('CS101', 'CS201')
    AND (grade IS NULL OR grade NOT IN ('A', 'A-', 'B+', 'B'))
)
AND s.student_id IN (
    SELECT student_id
    FROM enrollments
    WHERE course_id = 'CS101'
);
```

## Multiple-Row Subquery Best Practices

### 1. Use IN for Simple Membership Tests

**Good:**
```sql
WHERE student_id IN (SELECT student_id FROM enrollments)
```

**Avoid:**
```sql
WHERE student_id = ANY (SELECT student_id FROM enrollments)
```

### 2. Handle NULLs with NOT IN

**Problematic:**
```sql
WHERE id NOT IN (SELECT id FROM table WHERE condition)
-- Fails if subquery returns NULL
```

**Better:**
```sql
WHERE id NOT IN (SELECT id FROM table WHERE condition AND id IS NOT NULL)
```

**Best:**
```sql
WHERE NOT EXISTS (SELECT 1 FROM table WHERE table.id = outer.id AND condition)
```

### 3. Prefer EXISTS over IN for Complex Subqueries

**IN approach:**
```sql
WHERE student_id IN (
    SELECT student_id FROM enrollments WHERE ...
)
```

**EXISTS approach (often faster):**
```sql
WHERE EXISTS (
    SELECT 1 FROM enrollments e WHERE e.student_id = students.student_id AND ...
)
```

### 4. Use DISTINCT to Avoid Duplicates

```sql
WHERE student_id IN (
    SELECT DISTINCT student_id FROM enrollments
)
```

## Common Errors and Solutions

### Error 1: Subquery Returns NULL

**Problem:**
```sql
-- Returns no rows if any enrollment has NULL student_id
SELECT * FROM students
WHERE student_id NOT IN (SELECT student_id FROM enrollments);
```

**Solution:**
```sql
SELECT * FROM students
WHERE student_id NOT IN (
    SELECT student_id FROM enrollments WHERE student_id IS NOT NULL
);
```

### Error 2: Using Single-Row Operator

**Problem:**
```sql
-- ERROR: subquery returns more than one row
SELECT * FROM students
WHERE gpa > (SELECT gpa FROM students WHERE major = 'Computer Science');
```

**Solution:**
```sql
SELECT * FROM students
WHERE gpa > ALL (SELECT gpa FROM students WHERE major = 'Computer Science');
```

### Error 3: Ambiguous ANY/ALL Logic

**Problem:** Confusion about what ANY and ALL actually test.

**Clarification:**
- `> ANY` means "greater than at least one" (greater than minimum)
- `> ALL` means "greater than every value" (greater than maximum)
- `< ANY` means "less than at least one" (less than maximum)
- `< ALL` means "less than every value" (less than minimum)

## Summary

**Key Takeaways:**

1. **Multiple-row subqueries return zero or more rows** and require special operators: IN, NOT IN, ANY, ALL, or EXISTS.

2. **IN operator checks for membership** in a list of values; functionally equivalent to = ANY.

3. **NOT IN requires careful NULL handling** - always filter NULLs in the subquery or use NOT EXISTS instead.

4. **ANY operator returns TRUE if at least one comparison matches** - use with >, <, =, etc.

5. **ALL operator returns TRUE only if every comparison matches** - use to compare against the entire set.

6. **Operator meanings**: > ANY (greater than min), > ALL (greater than max), < ANY (less than max), < ALL (less than min).

7. **Best practices**: Use IN for simple membership, filter NULLs with NOT IN, prefer EXISTS for complex queries, use DISTINCT to avoid duplicates.

8. **Common patterns**: finding related records, excluding subsets, comparing to ranges, and filtering based on aggregate conditions.

Multiple-row subqueries provide powerful filtering capabilities for working with dynamic sets of data, enabling sophisticated queries that adapt to database contents.

