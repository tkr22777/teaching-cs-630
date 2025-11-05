# SQL Set Operators

## Overview

**Set operators** combine results from two or more SELECT statements. They treat query results as mathematical sets, allowing operations like union, intersection, and difference.

## Key Terms

**Set Operator**: Combines results from multiple queries (UNION, UNION ALL, INTERSECT, MINUS).

**UNION**: Combines results and removes duplicates.

**UNION ALL**: Combines results and keeps all duplicates.

**INTERSECT**: Returns only rows common to both queries.

**MINUS**: Returns rows from first query that are not in second query.

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

## Set Operators Overview

| Operator | Description | Duplicates | Oracle SQL |
|----------|-------------|------------|------------|
| **UNION** | Combines results, removes duplicates | Removed | ✓ |
| **UNION ALL** | Combines results, keeps duplicates | Kept | ✓ |
| **INTERSECT** | Only common rows | Removed | ✓ |
| **MINUS** | Rows in first but not second | Removed | ✓ (EXCEPT in other databases) |

**Visual representation:**

```
Query A: {1, 2, 3, 4}
Query B: {3, 4, 5, 6}

UNION:      {1, 2, 3, 4, 5, 6}
INTERSECT:  {3, 4}
MINUS:      {1, 2}  (A - B)
```

## Universal Rules for Set Operators

1. **Same number of columns** in both queries
2. **Compatible data types** in corresponding columns
3. **Column names** come from first query
4. **ORDER BY** only at the end (applies to final result)

**Example of compatible types:**
```sql
-- Valid: NUMBER and NUMBER
SELECT student_id FROM students
UNION
SELECT instructor_id FROM instructors;

-- Valid: VARCHAR2 and VARCHAR2
SELECT first_name FROM students
UNION
SELECT instructor_name FROM instructors;

-- Invalid: NUMBER and VARCHAR2
SELECT student_id FROM students
UNION
SELECT instructor_name FROM instructors;  -- ERROR!
```

## UNION Operator

**Purpose:** Combine results from multiple queries and remove duplicates.

**Syntax:**
```sql
SELECT columns FROM table1
UNION
SELECT columns FROM table2;
```

**Example: Get all departments from courses and instructors**

```sql
SELECT department FROM courses
UNION
SELECT department FROM instructors
ORDER BY department;
```

**Result:**
| department |
|------------|
| Chemistry |
| Computer Science |
| English |
| Mathematics |
| Physics |

**Note:** Duplicates like "Computer Science" appear only once.

## UNION ALL Operator

**Purpose:** Combine results and keep all duplicates (faster than UNION).

**Syntax:**
```sql
SELECT columns FROM table1
UNION ALL
SELECT columns FROM table2;
```

**UNION vs. UNION ALL:**

| Aspect | UNION | UNION ALL |
|--------|-------|-----------|
| **Duplicates** | Removed | Kept |
| **Performance** | Slower (sorts to remove dups) | Faster |
| **Use when** | Need unique results | All rows needed or no duplicates exist |

**Example: Count all courses and instructors by department (including duplicates)**

```sql
SELECT department, 'Course' AS type FROM courses
UNION ALL
SELECT department, 'Instructor' AS type FROM instructors
ORDER BY department, type;
```

**Result:**
| department | type |
|------------|------|
| Chemistry | Instructor |
| Computer Science | Course |
| Computer Science | Course |
| Computer Science | Course |
| Computer Science | Instructor |
| English | Course |
| Mathematics | Course |
| Mathematics | Instructor |
| Physics | Course |
| Physics | Instructor |

## INTERSECT Operator

**Purpose:** Return only rows that appear in both queries.

**Syntax:**
```sql
SELECT columns FROM table1
INTERSECT
SELECT columns FROM table2;
```

**Example: Find departments that have both courses and instructors**

```sql
SELECT department FROM courses
INTERSECT
SELECT department FROM instructors
ORDER BY department;
```

**Result:**
| department |
|------------|
| Computer Science |
| Mathematics |
| Physics |

**Explanation:** Chemistry (instructor only) and English (course only) are excluded.

## MINUS Operator

**Purpose:** Return rows from first query that are NOT in second query.

**Note:** Called `EXCEPT` in other databases (PostgreSQL, SQL Server), but Oracle uses `MINUS`.

**Syntax:**
```sql
SELECT columns FROM table1
MINUS
SELECT columns FROM table2;
```

**Example: Find departments with courses but no instructors**

```sql
SELECT department FROM courses
MINUS
SELECT department FROM instructors
ORDER BY department;
```

**Result:**
| department |
|------------|
| English |

**Order matters!** Reversing the queries gives different results:

```sql
-- Departments with instructors but no courses
SELECT department FROM instructors
MINUS
SELECT department FROM courses;
```

**Result:**
| department |
|------------|
| Chemistry |

## ORDER BY with Set Operators

**Rule:** ORDER BY can only appear at the very end and applies to the entire result.

**Example:**

```sql
SELECT first_name AS name, 'Student' AS type
FROM students
WHERE student_id <= 2
UNION
SELECT instructor_name AS name, 'Instructor' AS type
FROM instructors
WHERE instructor_id = 10
ORDER BY name;  -- Orders the combined result
```

**Result:**
| name | type |
|------|------|
| Dr. Johnson | Instructor |
| Jane | Student |
| John | Student |

**Invalid:**
```sql
-- ERROR: ORDER BY in middle
SELECT first_name FROM students ORDER BY first_name
UNION
SELECT instructor_name FROM instructors;
```

## Practical Example: Master Contact List

**Create a unified contact list from students and instructors:**

```sql
SELECT 
    first_name || ' ' || last_name AS full_name,
    email,
    'Student' AS role,
    major AS department
FROM students
UNION ALL
SELECT 
    instructor_name AS full_name,
    instructor_name || '@university.edu' AS email,
    'Instructor' AS role,
    department
FROM instructors
ORDER BY role, full_name;
```

## Performance Considerations

**Tips:**
1. **Use UNION ALL when possible** - Faster (no duplicate removal)
2. **Filter early** - Add WHERE clauses to reduce data before combining
3. **Index appropriately** - Indexes on columns used in WHERE and ORDER BY
4. **Consider alternatives** - Sometimes JOINs or CASE statements are faster

**Performance comparison:**
```sql
-- Slower: UNION removes duplicates
SELECT department FROM courses UNION SELECT department FROM instructors;

-- Faster: UNION ALL keeps all rows
SELECT department FROM courses UNION ALL SELECT department FROM instructors;
```

## Common Mistakes

**Mistake 1: Column count mismatch**
```sql
-- ERROR: Different number of columns
SELECT first_name, last_name FROM students
UNION
SELECT instructor_name FROM instructors;  -- Only one column!

-- Correct: Match column count
SELECT first_name, last_name FROM students
UNION
SELECT instructor_name, department FROM instructors;
```

**Mistake 2: Data type incompatibility**
```sql
-- ERROR: NUMBER vs VARCHAR2
SELECT student_id FROM students
UNION
SELECT instructor_name FROM instructors;

-- Correct: Same data types
SELECT CAST(student_id AS VARCHAR2(50)) FROM students
UNION
SELECT instructor_name FROM instructors;
```

**Mistake 3: ORDER BY in wrong place**
```sql
-- ERROR
SELECT first_name FROM students ORDER BY first_name
UNION
SELECT instructor_name FROM instructors;

-- Correct: ORDER BY at end
SELECT first_name FROM students
UNION
SELECT instructor_name FROM instructors
ORDER BY first_name;
```

## Summary

**Key Points:**

1. **Set operators combine multiple queries**: UNION, UNION ALL, INTERSECT, MINUS
2. **UNION** removes duplicates; **UNION ALL** keeps all rows (faster)
3. **INTERSECT** returns common rows; **MINUS** returns difference (first - second)
4. **Rules**: Same column count, compatible data types, ORDER BY only at end
5. **Performance**: UNION ALL is faster than UNION
6. **Use UNION ALL** when duplicates don't matter or don't exist
7. **MINUS is Oracle-specific** - other databases use EXCEPT
8. **Column names** come from first query

Set operators provide powerful ways to combine and compare result sets from multiple queries.
