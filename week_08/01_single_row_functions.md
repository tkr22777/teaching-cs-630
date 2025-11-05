# SQL Single-Row Functions

## Overview

**Single-row functions** operate on one row at a time and return one result per row. They can manipulate data types, perform calculations, and modify output format.

## Key Terms

**Single-Row Function**: Function that processes one row and returns one value per row.

**Character Function**: Manipulates text/string data.

**Numeric Function**: Performs mathematical operations.

**Date Function**: Works with date and time values.

**Conversion Function**: Converts between data types.

**NULL Handling Function**: Manages NULL values in expressions.

**Function Nesting**: Using one function's output as another function's input.

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

## Function Categories

Single-row functions are organized into categories:

| Category | Purpose | Examples |
|----------|---------|----------|
| **Character** | String manipulation | UPPER, LOWER, SUBSTR, CONCAT, TRIM |
| **Numeric** | Mathematical operations | ROUND, TRUNC, MOD |
| **Date** | Date/time operations | SYSDATE, ADD_MONTHS, MONTHS_BETWEEN |
| **Conversion** | Data type conversion | TO_CHAR, TO_DATE |
| **NULL Handling** | NULL value management | NVL, COALESCE |

## Character Functions

### UPPER, LOWER, INITCAP

**Purpose:** Change text case.

**Syntax:**
- `UPPER(string)` - Converts to uppercase
- `LOWER(string)` - Converts to lowercase
- `INITCAP(string)` - Capitalizes first letter of each word

**Example:**

```sql
SELECT 
    first_name,
    UPPER(first_name) AS upper_name,
    LOWER(first_name) AS lower_name,
    INITCAP(email) AS formatted_email
FROM students
WHERE student_id = 1;
```

**Result:**
| first_name | upper_name | lower_name | formatted_email |
|------------|------------|------------|-----------------|
| John | JOHN | john | John.Smith@University.Edu |

**Common use:** Case-insensitive searches:
```sql
WHERE UPPER(major) = 'COMPUTER SCIENCE'
```

### SUBSTR

**Purpose:** Extract part of a string.

**Syntax:** `SUBSTR(string, start_position, length)`
- `start_position`: Starting position (1-based)
- `length`: Optional - number of characters

**Example:**

```sql
SELECT 
    course_id,
    course_name,
    SUBSTR(course_id, 1, 2) AS dept_code,
    SUBSTR(course_id, 3) AS course_number
FROM courses
WHERE course_id LIKE 'CS%';
```

**Result:**
| course_id | course_name | dept_code | course_number |
|-----------|-------------|-----------|---------------|
| CS101 | Introduction to Programming | CS | 101 |
| CS201 | Data Structures | CS | 201 |
| CS301 | Database Systems | CS | 301 |

### CONCAT and || Operator

**Purpose:** Join strings together.

**Syntax:**
- `CONCAT(string1, string2)` - Joins two strings
- `string1 || string2` - Concatenation operator (preferred, can chain)

**Example:**

```sql
SELECT 
    first_name || ' ' || last_name AS full_name,
    CONCAT(first_name, last_name) AS no_space_name,
    email || ' (' || major || ')' AS contact_info
FROM students
WHERE student_id <= 2;
```

**Result:**
| full_name | no_space_name | contact_info |
|-----------|---------------|--------------|
| John Smith | JohnSmith | john.smith@university.edu (Computer Science) |
| Jane Doe | JaneDoe | jane.doe@university.edu (Mathematics) |

### TRIM, LTRIM, RTRIM

**Purpose:** Remove spaces or characters.

**Syntax:**
- `TRIM(string)` - Remove spaces from both sides
- `LTRIM(string)` - Remove from left
- `RTRIM(string)` - Remove from right

**Example:**

```sql
SELECT 
    TRIM('  Computer Science  ') AS trimmed,
    LTRIM('  Left spaces') AS left_trimmed,
    RTRIM('Right spaces  ') AS right_trimmed
FROM DUAL;
```

**Result:**
| trimmed | left_trimmed | right_trimmed |
|---------|--------------|---------------|
| Computer Science | Left spaces | Right spaces |

## Numeric Functions

### ROUND

**Purpose:** Round numbers to specified decimal places.

**Syntax:** `ROUND(number, decimal_places)`
- Positive: decimal places
- Negative: digits before decimal
- Omitted: rounds to integer

**Example:**

```sql
SELECT 
    first_name,
    gpa,
    ROUND(gpa) AS rounded_gpa,
    ROUND(gpa, 1) AS one_decimal,
    ROUND(gpa * 100, -1) AS nearest_ten
FROM students
WHERE gpa IS NOT NULL
ORDER BY student_id;
```

**Result:**
| first_name | gpa | rounded_gpa | one_decimal | nearest_ten |
|------------|-----|-------------|-------------|-------------|
| John | 3.8 | 4 | 3.8 | 380 |
| Jane | 3.9 | 4 | 3.9 | 390 |
| Bob | 3.2 | 3 | 3.2 | 320 |

### TRUNC

**Purpose:** Truncate (chop off) numbers without rounding.

**Syntax:** `TRUNC(number, decimal_places)`

**Example:**

```sql
SELECT 
    first_name,
    gpa,
    TRUNC(gpa) AS truncated,
    TRUNC(gpa, 1) AS one_decimal,
    gpa - TRUNC(gpa) AS decimal_part
FROM students
WHERE gpa IS NOT NULL
AND student_id <= 3;
```

**Result:**
| first_name | gpa | truncated | one_decimal | decimal_part |
|------------|-----|-----------|-------------|--------------|
| John | 3.8 | 3 | 3.8 | 0.8 |
| Jane | 3.9 | 3 | 3.9 | 0.9 |
| Bob | 3.2 | 3 | 3.2 | 0.2 |

### MOD

**Purpose:** Calculate remainder after division.

**Syntax:** `MOD(dividend, divisor)`

**Example:**

```sql
SELECT 
    student_id,
    first_name,
    MOD(student_id, 2) AS is_odd,
    CASE 
        WHEN MOD(student_id, 2) = 0 THEN 'Even'
        ELSE 'Odd'
    END AS parity
FROM students
ORDER BY student_id;
```

**Result:**
| student_id | first_name | is_odd | parity |
|------------|------------|--------|--------|
| 1 | John | 1 | Odd |
| 2 | Jane | 0 | Even |
| 3 | Bob | 1 | Odd |
| 4 | Alice | 0 | Even |
| 5 | Charlie | 1 | Odd |

## Date Functions

### SYSDATE

**Purpose:** Get current date and time.

**Syntax:** `SYSDATE` (no parentheses)

**Example:**

```sql
SELECT 
    SYSDATE AS current_datetime,
    TRUNC(SYSDATE) AS current_date,
    SYSDATE - TRUNC(SYSDATE) AS time_fraction
FROM DUAL;
```

### ADD_MONTHS

**Purpose:** Add or subtract months from a date.

**Syntax:** `ADD_MONTHS(date, number_of_months)`

**Example:**

```sql
SELECT 
    first_name,
    enrollment_date,
    ADD_MONTHS(enrollment_date, 12) AS one_year_later,
    ADD_MONTHS(enrollment_date, -6) AS six_months_before
FROM students
WHERE student_id <= 2;
```

**Result:**
| first_name | enrollment_date | one_year_later | six_months_before |
|------------|-----------------|----------------|-------------------|
| John | 2023-09-01 | 2024-09-01 | 2023-03-01 |
| Jane | 2023-09-01 | 2024-09-01 | 2023-03-01 |

### MONTHS_BETWEEN

**Purpose:** Calculate number of months between two dates.

**Syntax:** `MONTHS_BETWEEN(date1, date2)`

**Example:**

```sql
SELECT 
    first_name,
    enrollment_date,
    ROUND(MONTHS_BETWEEN(SYSDATE, enrollment_date)) AS months_enrolled,
    TRUNC(MONTHS_BETWEEN(SYSDATE, enrollment_date) / 12) AS years_enrolled
FROM students
WHERE enrollment_date IS NOT NULL
ORDER BY enrollment_date;
```

## Conversion Functions

### TO_CHAR (Date to String)

**Purpose:** Convert dates to formatted strings.

**Syntax:** `TO_CHAR(date, format_mask)`

**Common Format Masks:**
| Mask | Description | Example |
|------|-------------|---------|
| `YYYY` | 4-digit year | 2024 |
| `MM` | 2-digit month | 09 |
| `DD` | 2-digit day | 15 |
| `MON` | Abbreviated month | SEP |
| `MONTH` | Full month name | SEPTEMBER |
| `DY` | Abbreviated day | MON |
| `DAY` | Full day name | MONDAY |

**Example:**

```sql
SELECT 
    first_name,
    enrollment_date,
    TO_CHAR(enrollment_date, 'MM/DD/YYYY') AS us_format,
    TO_CHAR(enrollment_date, 'Month DD, YYYY') AS long_format,
    TO_CHAR(enrollment_date, 'DY, MON DD') AS short_format
FROM students
WHERE student_id <= 2;
```

**Result:**
| first_name | enrollment_date | us_format | long_format | short_format |
|------------|-----------------|-----------|-------------|--------------|
| John | 2023-09-01 | 09/01/2023 | September 01, 2023 | FRI, SEP 01 |
| Jane | 2023-09-01 | 09/01/2023 | September 01, 2023 | FRI, SEP 01 |

### TO_DATE

**Purpose:** Convert strings to dates.

**Syntax:** `TO_DATE(string, format_mask)`

**Example:**

```sql
SELECT 
    TO_DATE('2024-12-25', 'YYYY-MM-DD') AS christmas,
    TO_DATE('12/31/2024', 'MM/DD/YYYY') AS new_years_eve,
    TO_DATE('January 1, 2025', 'Month DD, YYYY') AS new_year
FROM DUAL;
```

**Practical use:**

```sql
-- Find students enrolled after a specific date
SELECT first_name, last_name, enrollment_date
FROM students
WHERE enrollment_date > TO_DATE('2024-01-01', 'YYYY-MM-DD');
```

## NULL Handling Functions

### NVL

**Purpose:** Replace NULL with a specified value.

**Syntax:** `NVL(expression, replacement_value)`

**Example:**

```sql
SELECT 
    first_name,
    last_name,
    major,
    NVL(major, 'Undeclared') AS major_status
FROM students
ORDER BY student_id;
```

**Result:**
| first_name | last_name | major | major_status |
|------------|-----------|-------|--------------|
| John | Smith | Computer Science | Computer Science |
| Jane | Doe | Mathematics | Mathematics |
| Bob | Wilson | Computer Science | Computer Science |
| Alice | Brown | Physics | Physics |
| Charlie | Davis | NULL | Undeclared |

### COALESCE

**Purpose:** Return first non-NULL value from a list.

**Syntax:** `COALESCE(value1, value2, value3, ...)`

**Example:**

```sql
SELECT 
    e.enrollment_id,
    e.course_id,
    e.grade,
    e.grade_points,
    COALESCE(e.grade, 'IP') AS grade_status,
    COALESCE(e.grade_points, 0) AS points_earned
FROM enrollments e
WHERE e.student_id = 1
ORDER BY e.enrollment_id;
```

**Result:**
| enrollment_id | course_id | grade | grade_points | grade_status | points_earned |
|---------------|-----------|-------|--------------|--------------|---------------|
| 101 | CS101 | A | 4.0 | A | 4.0 |
| 102 | CS201 | B+ | 3.3 | B+ | 3.3 |
| 108 | CS301 | NULL | NULL | IP | 0 |

**Multiple values:**
```sql
SELECT COALESCE(NULL, NULL, 'First non-NULL', 'Another') AS result FROM DUAL;
-- Returns: 'First non-NULL'
```

## Function Nesting

Functions can be nested - the inner function executes first, and its result becomes input to the outer function.

**Example: Formatted full name in uppercase**

```sql
SELECT 
    UPPER(first_name || ' ' || last_name) AS formatted_name,
    TO_CHAR(enrollment_date, 'Month YYYY') AS enrollment_month,
    ROUND(NVL(gpa, 0), 1) AS safe_gpa
FROM students
WHERE student_id <= 3;
```

**Result:**
| formatted_name | enrollment_month | safe_gpa |
|----------------|------------------|----------|
| JOHN SMITH | September 2023 | 3.8 |
| JANE DOE | September 2023 | 3.9 |
| BOB WILSON | January 2024 | 3.2 |

**Execution order:** Inner to outer
1. `NVL(gpa, 0)` replaces NULL with 0
2. `ROUND(..., 1)` rounds the result
3. `TO_CHAR(enrollment_date, 'Month YYYY')` formats date

## Summary

**Key Points:**

1. **Single-row functions** operate on one row at a time, returning one result per row
2. **Character functions**: UPPER, LOWER, SUBSTR, CONCAT (||), TRIM
3. **Numeric functions**: ROUND, TRUNC, MOD
4. **Date functions**: SYSDATE, ADD_MONTHS, MONTHS_BETWEEN
5. **Conversion functions**: TO_CHAR, TO_DATE
6. **NULL handling**: NVL (2 values), COALESCE (multiple values)
7. **Nesting**: Functions can be nested (inner executes first)
8. **Common uses**: Formatting output, data cleaning, calculations, NULL handling

Single-row functions are essential tools for data manipulation and presentation in SQL queries.
