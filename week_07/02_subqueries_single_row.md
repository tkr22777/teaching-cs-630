# Single-Row Subqueries

## Overview

A **single-row subquery** returns exactly one row with one or more columns. These subqueries are used with comparison operators (=, >, <, >=, <=, !=) and must always return a single value when used in scalar contexts.

## Key Terms

**Single-Row Subquery**: A subquery that returns exactly one value.

## Syntax

```sql
SELECT column1, column2
FROM table1
WHERE column1 operator (SELECT column_x FROM table2 WHERE condition);
```

**Operators:** =, !=, >, <, >=, <=

**Important:** Subquery must return exactly one row, or an error occurs.

## Examples

### Example 1: Comparing to Aggregates

**Find students with GPA above average:**

```sql
SELECT first_name, last_name, gpa
FROM students
WHERE gpa > (SELECT AVG(gpa) FROM students)
ORDER BY gpa DESC;
```

**Execution:**

1. Inner query calculates: `AVG(gpa)` → 3.48
2. Outer query finds: students where `gpa > 3.48`

**Result:**

| first_name | last_name | gpa |
| ---------- | --------- | --- |
| Jane       | Doe       | 3.9 |
| John       | Smith     | 3.8 |
| Alice      | Brown     | 3.7 |

**Explanation:** The subquery calculates the average GPA (3.48) once, then the outer query filters students whose GPA exceeds this value.

### Use Case 2: Finding Maximum/Minimum Values

**Example: Find the highest-paid course instructor**

```sql
SELECT 
    i.instructor_name,
    i.salary,
    i.department
FROM instructors i
WHERE i.salary = (SELECT MAX(salary) FROM instructors);
```

---

### Example 3: Dynamic Filtering

**Find courses with more credits than CS101:**

```sql
SELECT course_id, course_name, credits
FROM courses
WHERE credits > (
    SELECT credits 
    FROM courses 
    WHERE course_id = 'CS101'
)
ORDER BY credits DESC;
```

**Result:**

| course_id | course_name     | credits |
| --------- | --------------- | ------- |
| CS201     | Data Structures | 4       |
| MATH101   | Calculus I      | 4       |
| PHYS101   | Physics I       | 4       |

---

## Comparison Operators with Single-Row Subqueries

You can use any comparison operator (=, !=, >, <, >=, <=) with single-row subqueries.

**Example: Find students in the same major as John Smith**

```sql
SELECT first_name, last_name, major
FROM students
WHERE major = (
    SELECT major 
    FROM students 
    WHERE first_name = 'John' AND last_name = 'Smith'
)
AND (first_name != 'John' OR last_name != 'Smith');
```

**Result:**

| first_name | last_name | major            |
| ---------- | --------- | ---------------- |
| Bob        | Wilson    | Computer Science |

---

## Aggregate Functions in Single-Row Subqueries

Single-row subqueries frequently use aggregate functions because aggregates naturally return single values.

**Example: Find courses with above-average credit hours (AVG)**

```sql
SELECT course_id, course_name, credits
FROM courses
WHERE credits > (SELECT AVG(credits) FROM courses);
```

**Example: Find the most recently hired instructor (MAX)**

```sql
SELECT instructor_name, hire_date, department
FROM instructors
WHERE hire_date = (SELECT MAX(hire_date) FROM instructors);
```

**Example: Find newest students (MAX)**

```sql
SELECT first_name, last_name, enrollment_date
FROM students
WHERE enrollment_date = (SELECT MAX(enrollment_date) FROM students);
```

**Result:**

| first_name | last_name | enrollment_date |
| ---------- | --------- | --------------- |
| Charlie    | Davis     | 2024-09-01      |

---

## Subqueries in Different Clauses

Subqueries can be used in multiple parts of a SQL statement for different purposes.

**In WHERE Clause (most common):**

```sql
SELECT first_name, last_name
FROM students
WHERE gpa > (SELECT AVG(gpa) FROM students);
```

**In HAVING Clause (filter grouped results):**

```sql
SELECT major, AVG(gpa) AS avg_gpa
FROM students
WHERE major IS NOT NULL
GROUP BY major
HAVING AVG(gpa) > (SELECT AVG(gpa) FROM students);
```

**In SELECT Clause (calculated columns):**

```sql
SELECT 
    first_name,
    last_name,
    gpa,
    gpa - (SELECT AVG(gpa) FROM students) AS gpa_diff_from_avg
FROM students
ORDER BY gpa_diff_from_avg DESC;
```

## Practical Examples

### Comparative Analysis

**Query: Find courses that are worth more credits than the average course**

```sql
SELECT 
    course_id,
    course_name,
    credits,
    credits - (SELECT AVG(credits) FROM courses) AS credits_above_avg
FROM courses
WHERE credits > (SELECT AVG(credits) FROM courses)
ORDER BY credits DESC;
```

**Business Value:** Identifies high-workload courses for curriculum planning.

### Common Errors

**Multiple Rows Returned**

```sql
-- ERROR: single-row subquery returns more than one row
SELECT course_name
FROM courses
WHERE instructor_id = (SELECT instructor_id FROM instructors);
```

**Solution:** Use IN for multiple-row results:

```sql
SELECT course_name
FROM courses
WHERE instructor_id IN (SELECT instructor_id FROM instructors);
```
