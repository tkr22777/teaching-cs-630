# INNER JOIN

## Overview

An **INNER JOIN** returns only the rows where there is a match in both tables based on the join condition. Rows without matches in either table are excluded from the result.

INNER JOIN is the most commonly used join type because it returns only the data that has complete relationships.

### Visual: How INNER JOIN Works

**Note:** This diagram shows simplified data for clarity. In reality, students can have multiple enrollments.

```text
Table A (Students)              Table B (Enrollments)
┌─────────────┬─────────┐      ┌─────────────┬───────────┐
│ student_id  │ name    │      │ student_id  │ course_id │
├─────────────┼─────────┤      ├─────────────┼───────────┤
│ 1           │ John    │ ───► │ 1           │ CS101     │  ✓ MATCH
│ 2           │ Jane    │ ───► │ 2           │ MATH101   │  ✓ MATCH
│ 3           │ Bob     │ ───► │ 3           │ CS201     │  ✓ MATCH
│ 4           │ Alice   │ ───► │ 4           │ PHYS101   │  ✓ MATCH
│ 5           │ Charlie │  ✗   (no matching enrollment)    ✗ NO MATCH
└─────────────┴─────────┘      └─────────────┴───────────┘

                        ↓ INNER JOIN ↓

                    Result (Only Matches)
            ┌─────────────┬─────────┬───────────┐
            │ student_id  │ name    │ course_id │
            ├─────────────┼─────────┼───────────┤
            │ 1           │ John    │ CS101     │  ✓
            │ 2           │ Jane    │ MATH101   │  ✓
            │ 3           │ Bob     │ CS201     │  ✓
            │ 4           │ Alice   │ PHYS101   │  ✓
            └─────────────┴─────────┴───────────┘
            (Charlie excluded - no enrollment found)
```

**How INNER JOIN matches rows:** The database looks at each row in Table A and checks if there's a matching `student_id` in Table B. Only rows with matches in BOTH tables appear in the result. Charlie (student_id 5) has no enrollment, so he's excluded from the final result.

## Syntax

```sql
SELECT columns
FROM table1
INNER JOIN table2 ON table1.column = table2.column;
```

**Alternative (INNER is optional):**

```sql
SELECT columns
FROM table1
JOIN table2 ON table1.column = table2.column;
```

**Note:** `JOIN` without any modifier defaults to `INNER JOIN`.

**How it works:** Only rows with matching join keys appear in the result. Non-matching rows are excluded from both tables.

---

## Basic INNER JOIN Examples

### Example 1: Simple Two-Table Join

**Query:** Get student names with their enrollments.

```sql
SELECT s.student_id,
       s.first_name,
       s.last_name,
       e.course_id,
       e.semester,
       e.grade
FROM students s
INNER JOIN enrollments e ON s.student_id = e.student_id
ORDER BY s.student_id, e.semester;
```

**Result:**

| student_id | first_name | last_name | course_id | semester    | grade |
| ---------- | ---------- | --------- | --------- | ----------- | ----- |
| 1          | John       | Smith     | CS101     | Fall 2023   | A     |
| 1          | John       | Smith     | CS301     | Fall 2024   | NULL  |
| 1          | John       | Smith     | CS201     | Spring 2024 | B+    |
| 2          | Jane       | Doe       | CS101     | Fall 2023   | A-    |
| 2          | Jane       | Doe       | MATH101   | Fall 2023   | A     |
| 3          | Bob        | Wilson    | CS101     | Spring 2024 | B     |
| 3          | Bob        | Wilson    | CS201     | Spring 2024 | B+    |
| 4          | Alice      | Brown     | PHYS101   | Spring 2024 | A     |

**Explanation:**

- Returns 8 rows (8 enrollments with matching students)
- Student Charlie Davis (ID 5) is excluded because he has no enrollments
- Each enrollment is paired with its corresponding student information

### Example 2: Three-Table Join

**Query:** Get complete enrollment information with student and course names.

```sql
SELECT s.first_name,
       s.last_name,
       c.course_name,
       c.department,
       e.semester,
       e.grade
FROM students s
INNER JOIN enrollments e ON s.student_id = e.student_id
INNER JOIN courses c ON e.course_id = c.course_id
ORDER BY s.last_name, s.first_name, e.semester;
```

**Result:**

| first_name | last_name | course_name                 | department       | semester    | grade |
| ---------- | --------- | --------------------------- | ---------------- | ----------- | ----- |
| Alice      | Brown     | Physics I                   | Physics          | Spring 2024 | A     |
| Jane       | Doe       | Calculus I                  | Mathematics      | Fall 2023   | A     |
| Jane       | Doe       | Introduction to Programming | Computer Science | Fall 2023   | A-    |
| John       | Smith     | Introduction to Programming | Computer Science | Fall 2023   | A     |
| John       | Smith     | Database Systems            | Computer Science | Fall 2024   | NULL  |
| John       | Smith     | Data Structures             | Computer Science | Spring 2024 | B+    |
| Bob        | Wilson    | Introduction to Programming | Computer Science | Spring 2024 | B     |
| Bob        | Wilson    | Data Structures             | Computer Science | Spring 2024 | B+    |

**Explanation:** Joins three tables: students → enrollments → courses. Only returns enrollments that have both a valid student AND a valid course.

---

## INNER JOIN with Aggregate Functions

### Example 3: Count Enrollments Per Student

**Query:** How many courses is each student taking?

```sql
SELECT s.student_id,
       s.first_name,
       s.last_name,
       COUNT(e.enrollment_id) AS course_count
FROM students s
INNER JOIN enrollments e ON s.student_id = e.student_id
GROUP BY s.student_id, s.first_name, s.last_name
ORDER BY course_count DESC;
```

**Result:**

| student_id | first_name | last_name | course_count |
| ---------- | ---------- | --------- | ------------ |
| 1          | John       | Smith     | 3            |
| 2          | Jane       | Doe       | 2            |
| 3          | Bob        | Wilson    | 2            |
| 4          | Alice      | Brown     | 1            |

**Explanation:** Charlie Davis (ID 5) doesn't appear because INNER JOIN excludes students with no enrollments.
