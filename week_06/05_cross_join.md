# CROSS JOIN

## Overview

A **CROSS JOIN** produces the Cartesian product of two tables: every row from the first table combined with every row from the second. If table A has M rows and table B has N rows, the result has M × N rows. No join condition required.

### Visual: How CROSS JOIN Works

```text
Table A (Sizes)       Table B (Colors)    
┌──────┐             ┌───────┐           
│ size │             │ color │           
├──────┤             ├───────┤           
│ S    │ ─┬─────────► RED   │ ── S, RED
│ M    │  │     ┌───► BLUE  │ ── S, BLUE
│ L    │  │     │   └───────┘
└──────┘  │     │
          │     │    Result: Every possible combination
          ├─────┼──► (3 sizes × 2 colors = 6 rows)
          │     │
          │     └────────────────────────┐
          └──────────────────────┐       │
                                 ↓       ↓
                        ┌──────┬───────┐
                        │ size │ color │
                        ├──────┼───────┤
                        │ S    │ RED   │  ← S paired with RED
                        │ S    │ BLUE  │  ← S paired with BLUE
                        │ M    │ RED   │  ← M paired with RED
                        │ M    │ BLUE  │  ← M paired with BLUE
                        │ L    │ RED   │  ← L paired with RED
                        │ L    │ BLUE  │  ← L paired with BLUE
                        └──────┴───────┘
```

**How CROSS JOIN matches rows:** The database creates every possible pairing between tables. Each row from Table A is combined with each row from Table B. If Table A has 3 rows and Table B has 2 rows, the result has 3 × 2 = 6 rows. No join condition is needed - every combination is created automatically.

## Syntax

**Explicit syntax (preferred):**

```sql
SELECT columns
FROM table1
CROSS JOIN table2;
```

**How it works:** Every row from table A is paired with every row from table B. If A has M rows and B has N rows, result has M × N rows.

---

## Basic CROSS JOIN Example

### Example 1: Simple Cartesian Product

**Query:** Generate all possible student-course combinations.

```sql
SELECT s.student_id,
       s.first_name,
       s.last_name,
       c.course_id,
       c.course_name
FROM students s
CROSS JOIN courses c
ORDER BY s.student_id, c.course_id
FETCH FIRST 10 ROWS ONLY;
```

**Result (First 10 rows of 30 total):**

| student_id | first_name | last_name | course_id | course_name                 |
| ---------- | ---------- | --------- | --------- | --------------------------- |
| 1          | John       | Smith     | CS101     | Introduction to Programming |
| 1          | John       | Smith     | CS201     | Data Structures             |
| 1          | John       | Smith     | CS301     | Database Systems            |
| 1          | John       | Smith     | ENG101    | English Composition         |
| 1          | John       | Smith     | MATH101   | Calculus I                  |
| 1          | John       | Smith     | PHYS101   | Physics I                   |
| 2          | Jane       | Doe       | CS101     | Introduction to Programming |
| 2          | Jane       | Doe       | CS201     | Data Structures             |
| 2          | Jane       | Doe       | CS301     | Database Systems            |
| 2          | Jane       | Doe       | ENG101    | English Composition         |

**Explanation:** 5 students × 6 courses = 30 total combinations.
