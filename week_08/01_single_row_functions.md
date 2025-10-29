# Single-Row Functions

## Overview

**Single-row functions** are SQL functions that operate on individual rows and return one result per row. They can manipulate data types, perform calculations, format output, and transform values. These functions are essential for data processing and presentation in SQL queries.

## Key Terms

**Single-Row Function**: A function that processes one row at a time and returns one result per input row.

**String Function**: Functions that manipulate character data (UPPER, LOWER, SUBSTR, etc.).

**Numeric Function**: Functions that perform mathematical operations (ROUND, TRUNC, MOD, etc.).

**Date Function**: Functions that work with date and time values (SYSDATE, ADD_MONTHS, MONTHS_BETWEEN, etc.).

**Conversion Function**: Functions that convert data from one type to another (TO_CHAR, TO_DATE, TO_NUMBER).

**Null Function**: Functions that handle NULL values (NVL, NVL2, COALESCE, NULLIF).

**Nesting**: Using the output of one function as input to another function.

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

## Categories of Single-Row Functions

| Category | Purpose | Examples |
|----------|---------|----------|
| **Character** | String manipulation | UPPER, LOWER, INITCAP, SUBSTR, LENGTH, CONCAT |
| **Numeric** | Mathematical operations | ROUND, TRUNC, MOD, ABS, CEIL, FLOOR |
| **Date** | Date/time operations | SYSDATE, ADD_MONTHS, MONTHS_BETWEEN, TRUNC |
| **Conversion** | Data type conversion | TO_CHAR, TO_DATE, TO_NUMBER |
| **NULL Handling** | Work with NULL values | NVL, NVL2, COALESCE, NULLIF |
| **Conditional** | Conditional logic | CASE, DECODE |

## Character (String) Functions

### UPPER, LOWER, INITCAP

Transform case of text strings.

**Syntax:**
```sql
UPPER(string)    -- Converts to uppercase
LOWER(string)    -- Converts to lowercase
INITCAP(string)  -- Capitalizes first letter of each word
```

**Examples:**

```sql
SELECT 
    first_name,
    UPPER(first_name) AS uppercase,
    LOWER(first_name) AS lowercase,
    INITCAP(LOWER(email)) AS formatted_email
FROM students
WHERE student_id = 1;
```

**Result:**
| first_name | uppercase | lowercase | formatted_email |
|------------|-----------|-----------|-----------------|
| John | JOHN | john | John.Smith@University.Edu |

**Use Cases:**
- Case-insensitive searches: `WHERE UPPER(last_name) = 'SMITH'`
- Standardizing data entry
- Formatting output for reports

### LENGTH and LENGTHB

Calculate string length.

**Syntax:**
```sql
LENGTH(string)   -- Returns character length
LENGTHB(string)  -- Returns byte length
```

**Examples:**

```sql
SELECT 
    course_name,
    LENGTH(course_name) AS char_length,
    course_id,
    LENGTH(course_id) AS id_length
FROM courses;
```

**Result:**
| course_name | char_length | course_id | id_length |
|-------------|-------------|-----------|-----------|
| Introduction to Programming | 27 | CS101 | 5 |
| Data Structures | 15 | CS201 | 5 |

**Use Cases:**
- Data validation: `WHERE LENGTH(phone) = 10`
- Finding long/short entries
- Column width determination

### SUBSTR

Extract substring from a string.

**Syntax:**
```sql
SUBSTR(string, start_position, length)
-- start_position: 1-based index (positive from start, negative from end)
-- length: optional, number of characters to extract
```

**Examples:**

```sql
SELECT 
    course_id,
    course_name,
    SUBSTR(course_id, 1, 2) AS department_code,
    SUBSTR(course_id, 3) AS course_number,
    SUBSTR(course_name, 1, 20) AS short_name
FROM courses;
```

**Result:**
| course_id | course_name | department_code | course_number | short_name |
|-----------|-------------|-----------------|---------------|------------|
| CS101 | Introduction to Programming | CS | 101 | Introduction to Prog |
| MATH101 | Calculus I | MA | TH101 | Calculus I |

**Additional Examples:**

```sql
-- Extract from end of string
SELECT SUBSTR('Hello World', -5) FROM DUAL;  -- Returns: World

-- Extract middle portion
SELECT SUBSTR('Database Systems', 6, 4) FROM DUAL;  -- Returns: base

-- Extract email domain
SELECT 
    email,
    SUBSTR(email, INSTR(email, '@') + 1) AS domain
FROM students;
```

### CONCAT and || Operator

Concatenate strings.

**Syntax:**
```sql
CONCAT(string1, string2)  -- Joins two strings
string1 || string2        -- Concatenation operator (can join multiple)
```

**Examples:**

```sql
SELECT 
    first_name,
    last_name,
    CONCAT(first_name, last_name) AS name_concat,
    first_name || ' ' || last_name AS full_name,
    last_name || ', ' || first_name AS last_first
FROM students;
```

**Result:**
| first_name | last_name | name_concat | full_name | last_first |
|------------|-----------|-------------|-----------|------------|
| John | Smith | JohnSmith | John Smith | Smith, John |
| Jane | Doe | JaneDoe | Jane Doe | Doe, Jane |

### TRIM, LTRIM, RTRIM

Remove spaces or characters from strings.

**Syntax:**
```sql
TRIM(string)           -- Removes spaces from both ends
LTRIM(string)          -- Removes leading spaces
RTRIM(string)          -- Removes trailing spaces
TRIM(char FROM string) -- Removes specific character
```

**Examples:**

```sql
SELECT 
    '|' || TRIM('  Hello  ') || '|' AS trimmed,
    '|' || LTRIM('  Hello  ') || '|' AS left_trimmed,
    '|' || RTRIM('  Hello  ') || '|' AS right_trimmed,
    TRIM('.' FROM '...Hello...') AS trim_dots
FROM DUAL;
```

**Result:**
| trimmed | left_trimmed | right_trimmed | trim_dots |
|---------|--------------|---------------|-----------|
| \|Hello\| | \|Hello  \| | \|  Hello\| | Hello |

### INSTR

Find position of substring within a string.

**Syntax:**
```sql
INSTR(string, substring, start_position, occurrence)
-- Returns position of substring (0 if not found)
```

**Examples:**

```sql
SELECT 
    email,
    INSTR(email, '@') AS at_position,
    INSTR(email, '.', INSTR(email, '@')) AS dot_position
FROM students
WHERE student_id = 1;
```

**Result:**
| email | at_position | dot_position |
|-------|-------------|--------------|
| john.smith@university.edu | 11 | 22 |

**Practical Use - Extract Email Parts:**

```sql
SELECT 
    email,
    SUBSTR(email, 1, INSTR(email, '@') - 1) AS username,
    SUBSTR(email, INSTR(email, '@') + 1) AS domain
FROM students;
```

### REPLACE

Replace occurrences of a substring.

**Syntax:**
```sql
REPLACE(string, search_string, replacement_string)
```

**Examples:**

```sql
SELECT 
    course_name,
    REPLACE(course_name, 'Programming', 'Coding') AS updated_name,
    REPLACE(course_id, 'CS', 'CSCI') AS new_id
FROM courses
WHERE course_id LIKE 'CS%';
```

**Result:**
| course_name | updated_name | new_id |
|-------------|--------------|--------|
| Introduction to Programming | Introduction to Coding | CSCI101 |
| Data Structures | Data Structures | CSCI201 |

## Numeric Functions

### ROUND

Round numbers to specified decimal places.

**Syntax:**
```sql
ROUND(number, decimal_places)
-- decimal_places: positive for decimals, negative for whole numbers
```

**Examples:**

```sql
SELECT 
    gpa,
    ROUND(gpa, 0) AS rounded_gpa,
    ROUND(gpa, 1) AS one_decimal,
    ROUND(gpa * 100, -1) AS rounded_tens
FROM students
WHERE gpa IS NOT NULL;
```

**Result:**
| gpa | rounded_gpa | one_decimal | rounded_tens |
|-----|-------------|-------------|--------------|
| 3.87 | 4 | 3.9 | 390 |
| 3.23 | 3 | 3.2 | 320 |

**More Examples:**

```sql
SELECT 
    ROUND(3.14159, 2) AS two_decimals,      -- 3.14
    ROUND(3.14159, 0) AS no_decimals,       -- 3
    ROUND(3.14159, -1) AS round_tens,       -- 0
    ROUND(156.789, -2) AS round_hundreds    -- 200
FROM DUAL;
```

### TRUNC

Truncate numbers to specified decimal places (no rounding).

**Syntax:**
```sql
TRUNC(number, decimal_places)
```

**Examples:**

```sql
SELECT 
    gpa,
    TRUNC(gpa, 0) AS truncated_gpa,
    TRUNC(gpa, 1) AS one_decimal,
    ROUND(gpa, 1) AS rounded_comparison
FROM students
WHERE gpa IS NOT NULL;
```

**Result:**
| gpa | truncated_gpa | one_decimal | rounded_comparison |
|-----|---------------|-------------|--------------------|
| 3.87 | 3 | 3.8 | 3.9 |
| 3.23 | 3 | 3.2 | 3.2 |

**Difference between ROUND and TRUNC:**

```sql
SELECT 
    3.7 AS original,
    ROUND(3.7) AS rounded,  -- 4
    TRUNC(3.7) AS truncated -- 3
FROM DUAL;
```

### MOD

Calculate remainder of division.

**Syntax:**
```sql
MOD(dividend, divisor)
```

**Examples:**

```sql
SELECT 
    student_id,
    first_name,
    MOD(student_id, 2) AS odd_or_even,
    CASE 
        WHEN MOD(student_id, 2) = 0 THEN 'Even'
        ELSE 'Odd'
    END AS parity
FROM students;
```

**Result:**
| student_id | first_name | odd_or_even | parity |
|------------|------------|-------------|--------|
| 1 | John | 1 | Odd |
| 2 | Jane | 0 | Even |
| 3 | Bob | 1 | Odd |

**Use Cases:**
- Identifying even/odd numbers
- Creating groups: `MOD(student_id, 5)` creates 5 groups
- Scheduling rotations

### ABS, CEIL, FLOOR

Absolute value and rounding functions.

**Syntax:**
```sql
ABS(number)    -- Absolute value (remove negative sign)
CEIL(number)   -- Round up to nearest integer
FLOOR(number)  -- Round down to nearest integer
```

**Examples:**

```sql
SELECT 
    3.2 AS original,
    ABS(-3.2) AS absolute,
    CEIL(3.2) AS ceiling,
    FLOOR(3.2) AS floor,
    CEIL(-3.2) AS ceil_negative,
    FLOOR(-3.2) AS floor_negative
FROM DUAL;
```

**Result:**
| original | absolute | ceiling | floor | ceil_negative | floor_negative |
|----------|----------|---------|-------|---------------|----------------|
| 3.2 | 3.2 | 4 | 3 | -3 | -4 |

### POWER and SQRT

Exponents and square roots.

**Syntax:**
```sql
POWER(base, exponent)
SQRT(number)
```

**Examples:**

```sql
SELECT 
    credits,
    POWER(credits, 2) AS credits_squared,
    SQRT(credits) AS credits_sqrt
FROM courses
WHERE course_id = 'CS201';
```

**Result:**
| credits | credits_squared | credits_sqrt |
|---------|-----------------|--------------|
| 4 | 16 | 2 |

## Date Functions

### SYSDATE and CURRENT_DATE

Get current date and time.

**Syntax:**
```sql
SYSDATE         -- Current date and time (database server)
CURRENT_DATE    -- Current date (session time zone)
```

**Examples:**

```sql
SELECT 
    SYSDATE AS current_datetime,
    TRUNC(SYSDATE) AS current_date,
    TO_CHAR(SYSDATE, 'YYYY-MM-DD HH24:MI:SS') AS formatted
FROM DUAL;
```

### ADD_MONTHS

Add or subtract months from a date.

**Syntax:**
```sql
ADD_MONTHS(date, number_of_months)
```

**Examples:**

```sql
SELECT 
    first_name,
    enrollment_date,
    ADD_MONTHS(enrollment_date, 12) AS one_year_later,
    ADD_MONTHS(enrollment_date, -6) AS six_months_before
FROM students
WHERE student_id = 1;
```

**Result:**
| first_name | enrollment_date | one_year_later | six_months_before |
|------------|-----------------|----------------|-------------------|
| John | 2023-09-01 | 2024-09-01 | 2023-03-01 |

### MONTHS_BETWEEN

Calculate months between two dates.

**Syntax:**
```sql
MONTHS_BETWEEN(date1, date2)
-- Returns fractional months (can be positive or negative)
```

**Examples:**

```sql
SELECT 
    first_name,
    enrollment_date,
    ROUND(MONTHS_BETWEEN(SYSDATE, enrollment_date)) AS months_enrolled,
    ROUND(MONTHS_BETWEEN(SYSDATE, enrollment_date) / 12, 1) AS years_enrolled
FROM students;
```

**Result:**
| first_name | enrollment_date | months_enrolled | years_enrolled |
|------------|-----------------|-----------------|----------------|
| John | 2023-09-01 | 14 | 1.2 |
| Charlie | 2024-09-01 | 2 | 0.2 |

### NEXT_DAY and LAST_DAY

Find next occurrence of a day or last day of month.

**Syntax:**
```sql
NEXT_DAY(date, day_of_week)
LAST_DAY(date)
```

**Examples:**

```sql
SELECT 
    SYSDATE AS today,
    NEXT_DAY(SYSDATE, 'MONDAY') AS next_monday,
    LAST_DAY(SYSDATE) AS last_day_of_month,
    TRUNC(LAST_DAY(SYSDATE)) - TRUNC(SYSDATE) AS days_left_in_month
FROM DUAL;
```

### TRUNC (for dates)

Truncate time portion from date.

**Syntax:**
```sql
TRUNC(date, format)
```

**Examples:**

```sql
SELECT 
    SYSDATE AS full_datetime,
    TRUNC(SYSDATE) AS date_only,
    TRUNC(SYSDATE, 'MONTH') AS first_day_of_month,
    TRUNC(SYSDATE, 'YEAR') AS first_day_of_year
FROM DUAL;
```

### Date Arithmetic

Perform calculations with dates.

**Operations:**
```sql
date + number      -- Add days
date - number      -- Subtract days
date1 - date2      -- Difference in days
date + number/24   -- Add hours
```

**Examples:**

```sql
SELECT 
    enrollment_date,
    enrollment_date + 7 AS one_week_later,
    enrollment_date - 30 AS thirty_days_before,
    SYSDATE - enrollment_date AS days_since_enrollment,
    enrollment_date + 1/24 AS one_hour_later
FROM students
WHERE student_id = 1;
```

## Conversion Functions

### TO_CHAR (Date to String)

Convert dates to formatted strings.

**Syntax:**
```sql
TO_CHAR(date, format_mask)
```

**Common Format Masks:**
| Mask | Meaning | Example |
|------|---------|---------|
| YYYY | 4-digit year | 2024 |
| MM | Month (01-12) | 10 |
| MON | Month abbreviation | OCT |
| MONTH | Month full name | OCTOBER |
| DD | Day of month | 29 |
| DY | Day abbreviation | WED |
| DAY | Day full name | WEDNESDAY |
| HH24 | Hour (00-23) | 14 |
| MI | Minute | 30 |
| SS | Second | 45 |

**Examples:**

```sql
SELECT 
    enrollment_date,
    TO_CHAR(enrollment_date, 'YYYY-MM-DD') AS iso_format,
    TO_CHAR(enrollment_date, 'Month DD, YYYY') AS long_format,
    TO_CHAR(enrollment_date, 'MM/DD/YY') AS short_format,
    TO_CHAR(enrollment_date, 'Day, Month DD, YYYY') AS full_format
FROM students
WHERE student_id = 1;
```

**Result:**
| enrollment_date | iso_format | long_format | short_format | full_format |
|-----------------|------------|-------------|--------------|-------------|
| 2023-09-01 | 2023-09-01 | September 01, 2023 | 09/01/23 | Friday, September 01, 2023 |

### TO_CHAR (Number to String)

Convert numbers to formatted strings.

**Syntax:**
```sql
TO_CHAR(number, format_mask)
```

**Common Format Masks:**
| Mask | Meaning | Example |
|------|---------|---------|
| 9 | Display digit (suppress leading zeros) | 9999 |
| 0 | Display digit (show leading zeros) | 0999 |
| $ | Dollar sign | $9999 |
| . | Decimal point | 9999.99 |
| , | Comma separator | 9,999 |
| FM | Fill mode (removes padding) | FM9999 |

**Examples:**

```sql
SELECT 
    gpa,
    TO_CHAR(gpa, '9.99') AS formatted_gpa,
    TO_CHAR(gpa * 1000, '$9,999.99') AS as_currency,
    TO_CHAR(student_id, '0000') AS padded_id
FROM students
WHERE student_id <= 3;
```

**Result:**
| gpa | formatted_gpa | as_currency | padded_id |
|-----|---------------|-------------|-----------|
| 3.8 | 3.80 | $3,800.00 | 0001 |
| 3.9 | 3.90 | $3,900.00 | 0002 |
| 3.2 | 3.20 | $3,200.00 | 0003 |

### TO_DATE

Convert strings to dates.

**Syntax:**
```sql
TO_DATE(string, format_mask)
```

**Examples:**

```sql
SELECT 
    TO_DATE('2024-10-29', 'YYYY-MM-DD') AS date1,
    TO_DATE('10/29/2024', 'MM/DD/YYYY') AS date2,
    TO_DATE('October 29, 2024', 'Month DD, YYYY') AS date3
FROM DUAL;
```

**Practical Use:**

```sql
-- Find students enrolled after a specific date
SELECT first_name, last_name, enrollment_date
FROM students
WHERE enrollment_date > TO_DATE('2024-01-01', 'YYYY-MM-DD');
```

### TO_NUMBER

Convert strings to numbers.

**Syntax:**
```sql
TO_NUMBER(string, format_mask)
```

**Examples:**

```sql
SELECT 
    TO_NUMBER('12345') AS simple_number,
    TO_NUMBER('$1,234.56', '$9,999.99') AS from_currency,
    TO_NUMBER('  123  ') AS with_spaces
FROM DUAL;
```

## NULL Handling Functions

### NVL

Replace NULL with a value.

**Syntax:**
```sql
NVL(expression, replacement_value)
```

**Examples:**

```sql
SELECT 
    first_name,
    major,
    NVL(major, 'Undeclared') AS major_display,
    gpa,
    NVL(gpa, 0.0) AS gpa_display
FROM students;
```

**Result:**
| first_name | major | major_display | gpa | gpa_display |
|------------|-------|---------------|-----|-------------|
| John | Computer Science | Computer Science | 3.8 | 3.8 |
| Charlie | NULL | Undeclared | 2.8 | 2.8 |

### NVL2

Return different values based on whether expression is NULL.

**Syntax:**
```sql
NVL2(expression, value_if_not_null, value_if_null)
```

**Examples:**

```sql
SELECT 
    course_name,
    instructor_id,
    NVL2(instructor_id, 'Assigned', 'Unassigned') AS instructor_status,
    NVL2(instructor_id, 'Instructor: ' || instructor_id, 'TBA') AS instructor_info
FROM courses;
```

**Result:**
| course_name | instructor_id | instructor_status | instructor_info |
|-------------|---------------|-------------------|-----------------|
| Introduction to Programming | 10 | Assigned | Instructor: 10 |
| English Composition | NULL | Unassigned | TBA |

### COALESCE

Return first non-NULL value from a list.

**Syntax:**
```sql
COALESCE(expr1, expr2, expr3, ...)
```

**Examples:**

```sql
SELECT 
    first_name,
    major,
    email,
    COALESCE(major, 'Undeclared', 'Unknown') AS major_display,
    COALESCE(NULL, NULL, gpa, 0) AS first_non_null
FROM students;
```

**Practical Use - Fallback Values:**

```sql
SELECT 
    student_id,
    COALESCE(
        email,
        phone,
        mailing_address,
        'No contact info'
    ) AS primary_contact
FROM students;
```

### NULLIF

Return NULL if two expressions are equal.

**Syntax:**
```sql
NULLIF(expr1, expr2)
-- Returns NULL if expr1 = expr2, otherwise returns expr1
```

**Examples:**

```sql
SELECT 
    course_id,
    credits,
    NULLIF(credits, 3) AS non_standard_credits
FROM courses;
```

**Result:**
| course_id | credits | non_standard_credits |
|-----------|---------|----------------------|
| CS101 | 3 | NULL |
| CS201 | 4 | 4 |
| PHYS101 | 4 | 4 |

**Use Case - Avoid Division by Zero:**

```sql
SELECT 
    total_points / NULLIF(attempts, 0) AS average
FROM game_scores;
```

## Function Nesting

Functions can be nested (output of one function used as input to another).

**Examples:**

```sql
SELECT 
    email,
    UPPER(SUBSTR(email, 1, INSTR(email, '@') - 1)) AS username_upper,
    REPLACE(LOWER(email), '@university.edu', '@alumni.edu') AS alumni_email
FROM students
WHERE student_id = 1;
```

**Result:**
| email | username_upper | alumni_email |
|-------|----------------|--------------|
| john.smith@university.edu | JOHN.SMITH | john.smith@alumni.edu |

**Complex Example:**

```sql
SELECT 
    first_name,
    last_name,
    enrollment_date,
    TO_CHAR(
        ADD_MONTHS(enrollment_date, 48),
        'Month DD, YYYY'
    ) AS expected_graduation
FROM students;
```

## Summary

**Key Takeaways:**

1. **Single-row functions process one row at a time** and return one result per row, enabling row-level data transformation.

2. **Character functions** (UPPER, LOWER, SUBSTR, CONCAT, TRIM, etc.) manipulate text strings for formatting, searching, and standardization.

3. **Numeric functions** (ROUND, TRUNC, MOD, ABS, CEIL, FLOOR) perform mathematical operations and rounding on numbers.

4. **Date functions** (SYSDATE, ADD_MONTHS, MONTHS_BETWEEN, TRUNC) enable date arithmetic and formatting for temporal data.

5. **Conversion functions** (TO_CHAR, TO_DATE, TO_NUMBER) convert between data types with format masks for precise control.

6. **NULL handling functions** (NVL, NVL2, COALESCE, NULLIF) manage NULL values to prevent errors and provide default values.

7. **Functions can be nested** to perform complex transformations by using the output of one function as input to another.

8. **Common patterns**: Case-insensitive searches with UPPER/LOWER, string parsing with SUBSTR and INSTR, date calculations with arithmetic, NULL replacement with NVL/COALESCE.

Single-row functions are fundamental tools for data transformation, validation, formatting, and presentation in SQL queries.

