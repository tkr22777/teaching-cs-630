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

University enrollment system. Setup: `00_initialization.md`

## Set Operators Overview

| Operator            | Description                          | Duplicates | Oracle SQL                     |
| ------------------- | ------------------------------------ | ---------- | ------------------------------ |
| **UNION**     | Combines results, removes duplicates | Removed    | ✓                             |
| **UNION ALL** | Combines results, keeps duplicates   | Kept       | ✓                             |
| **INTERSECT** | Only common rows                     | Removed    | ✓                             |
| **MINUS**     | Rows in first but not second         | Removed    | ✓ (EXCEPT in other databases) |

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

```sql
-- Valid: compatible types
SELECT first_name FROM students
UNION
SELECT instructor_name FROM instructors;

-- Invalid: incompatible types (NUMBER vs VARCHAR2)
SELECT student_id FROM students
UNION
SELECT instructor_name FROM instructors;  -- ERROR!
```

## UNION Operator

Combines results from multiple queries and removes duplicates.

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

| department       |
| ---------------- |
| Chemistry        |
| Computer Science |
| English          |
| Mathematics      |
| Physics          |

## UNION ALL Operator

Combines results and keeps all duplicates (faster than UNION).

**Syntax:**

```sql
SELECT columns FROM table1
UNION ALL
SELECT columns FROM table2;
```

**UNION vs. UNION ALL:**

| Aspect                | UNION                         | UNION ALL                              |
| --------------------- | ----------------------------- | -------------------------------------- |
| **Duplicates**  | Removed                       | Kept                                   |
| **Performance** | Slower (sorts to remove dups) | Faster                                 |
| **Use when**    | Need unique results           | All rows needed or no duplicates exist |

**Example: Count all courses and instructors by department (including duplicates)**

```sql
SELECT department, 'Course' AS type FROM courses
UNION ALL
SELECT department, 'Instructor' AS type FROM instructors
ORDER BY department, type;
```

**Result:**

| department       | type       |
| ---------------- | ---------- |
| Chemistry        | Instructor |
| Computer Science | Course     |
| Computer Science | Course     |
| Computer Science | Course     |
| Computer Science | Instructor |
| English          | Course     |
| Mathematics      | Course     |
| Mathematics      | Instructor |
| Physics          | Course     |
| Physics          | Instructor |

## INTERSECT Operator

Returns only rows that appear in both queries.

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

| department       |
| ---------------- |
| Computer Science |
| Mathematics      |
| Physics          |

## MINUS Operator

Returns rows from first query that are NOT in second query. Called `EXCEPT` in other databases.

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
| ---------- |
| English    |

**Note:** Order matters with MINUS. Reversing gives `Chemistry` (instructors without courses).
