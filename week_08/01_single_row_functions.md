# SQL Single-Row Functions

## Overview

**Single-row functions** operate on one row at a time and return one result per row. They can manipulate data types, perform calculations, and modify output format.

## Key Terms

**Single-Row Function**: Operates on one row, returns one result per row.

## Function Categories

| Category | Purpose | Examples |
|----------|---------|----------|
| **Character** | String manipulation | UPPER, LOWER, SUBSTR, CONCAT |
| **Numeric** | Math operations | ROUND, TRUNC, MOD |
| **Date** | Date/time operations | SYSDATE, ADD_MONTHS |
| **NULL Handling** | Handle NULL values | NVL, COALESCE |

## Character Functions

### UPPER, LOWER, INITCAP

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

### SUBSTR

Extracts part of a string.

**Syntax:** `SUBSTR(string, start_position, length)`

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

**Syntax:** `string1 || string2`

```sql
SELECT first_name || ' ' || last_name AS full_name,
       first_name || ' (' || major || ')' AS name_with_major
FROM students
WHERE student_id <= 2;
```

**Result:**
| full_name | name_with_major |
|-----------|-----------------|
| John Smith | John (Computer Science) |
| Jane Doe | Jane (Mathematics) |

---

### TRIM

Removes spaces.

```sql
SELECT TRIM('  Computer Science  ') AS trimmed
FROM DUAL;
```

**Result:** `Computer Science`

## Numeric Functions

### ROUND

Rounds numbers to specified decimal places.

**Syntax:** `ROUND(number, decimal_places)`

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

---

### TRUNC

Truncates numbers without rounding.

```sql
SELECT gpa, TRUNC(gpa) AS truncated
FROM students
WHERE gpa IS NOT NULL;
```

### MOD

Calculates remainder after division.

**Syntax:** `MOD(dividend, divisor)`

**Example:**

```sql
SELECT student_id,
       first_name,
       MOD(student_id, 2) AS is_odd
FROM students
ORDER BY student_id;
```

**Result:**
| student_id | first_name | is_odd |
|------------|------------|--------|
| 1 | John | 1 |
| 2 | Jane | 0 |
| 3 | Bob | 1 |

---

## Date Functions

### SYSDATE

Returns current date and time.

```sql
SELECT SYSDATE AS current_datetime FROM DUAL;
```

---

### ADD_MONTHS

Adds or subtracts months from a date.

```sql
SELECT enrollment_date,
       ADD_MONTHS(enrollment_date, 12) AS one_year_later
FROM students
WHERE student_id = 1;
```

---

### MONTHS_BETWEEN

Calculates months between two dates.

```sql
SELECT first_name,
       ROUND(MONTHS_BETWEEN(SYSDATE, enrollment_date)) AS months_enrolled
FROM students
WHERE student_id <= 2;
```

---

## Conversion Functions

### TO_CHAR

Converts dates to formatted strings.

**Common formats:** `YYYY` (year), `MM` (month), `DD` (day), `MON` (Sep)

```sql
SELECT enrollment_date,
       TO_CHAR(enrollment_date, 'MM/DD/YYYY') AS us_format,
       TO_CHAR(enrollment_date, 'Month DD, YYYY') AS long_format
FROM students
WHERE student_id = 1;
```

---

### TO_DATE

Converts strings to dates.

```sql
SELECT TO_DATE('2024-12-25', 'YYYY-MM-DD') AS christmas
FROM DUAL;
```

---

## NULL Handling Functions

### NVL

Replaces NULL with a value.

```sql
SELECT first_name,
       major,
       NVL(major, 'Undeclared') AS major_status
FROM students;
```

**Result:**
| first_name | major | major_status |
|------------|-------|--------------|
| Charlie | NULL | Undeclared |

---

### COALESCE

Returns first non-NULL value from a list.

```sql
SELECT grade,
       COALESCE(grade, 'IP') AS grade_status
FROM enrollments
WHERE student_id = 1;
```

---

## Function Nesting

Functions can be nested (inner executes first).

```sql
SELECT UPPER(first_name || ' ' || last_name) AS formatted_name,
       ROUND(NVL(gpa, 0), 1) AS safe_gpa
FROM students
WHERE student_id = 1;
```

