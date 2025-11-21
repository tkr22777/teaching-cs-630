# LEFT JOIN (LEFT OUTER JOIN)

## Overview

A **LEFT JOIN** returns all rows from the left (first) table and the matching rows from the right (second) table. When there's no match, NULL values are returned for columns from the right table.

LEFT JOIN is essential when you need to preserve all records from your primary table, even if they don't have related records.

### Visual: How LEFT JOIN Works

**Note:** This diagram shows simplified data for clarity. In reality, students can have multiple enrollments.

```text
LEFT Table (Students)           RIGHT Table (Enrollments)
┌─────────────┬─────────┐      ┌─────────────┬───────────┐
│ student_id  │ name    │      │ student_id  │ course_id │
├─────────────┼─────────┤      ├─────────────┼───────────┤
│ 1           │ John    │ ───► │ 1           │ CS101     │  ✓ MATCH
│ 2           │ Jane    │ ───► │ 2           │ MATH101   │  ✓ MATCH
│ 3           │ Bob     │ ───► │ 3           │ CS201     │  ✓ MATCH
│ 4           │ Alice   │ ───► │ 4           │ PHYS101   │  ✓ MATCH
│ 5           │ Charlie │ ─┐   (no matching enrollment)    ✗ NO MATCH
└─────────────┴─────────┘  │
                           └─► Still included with NULL

                      ↓ LEFT JOIN ↓

              Result (All from LEFT + Matches)
         ┌─────────────┬─────────┬───────────┐
         │ student_id  │ name    │ course_id │
         ├─────────────┼─────────┼───────────┤
         │ 1           │ John    │ CS101     │  ✓
         │ 2           │ Jane    │ MATH101   │  ✓
         │ 3           │ Bob     │ CS201     │  ✓
         │ 4           │ Alice   │ PHYS101   │  ✓
         │ 5           │ Charlie │ NULL      │  ← Preserved with NULL
         └─────────────┴─────────┴───────────┘
```

**How LEFT JOIN matches rows:** The database takes every row from the LEFT table (Students) and looks for matches in the RIGHT table (Enrollments). When a match is found, the rows are combined. When no match exists (Charlie has no enrollment), the row from the left table is still included with NULL values for the right table's columns. ALL rows from the left table appear in the result.

## Syntax

```sql
SELECT columns
FROM table1
LEFT JOIN table2 ON table1.column = table2.column;
```

**Alternative (equivalent):**

```sql
SELECT columns
FROM table1
LEFT OUTER JOIN table2 ON table1.column = table2.column;
```

**Note:** `LEFT JOIN` and `LEFT OUTER JOIN` are identical.

**How it works:** ALL rows from the left table appear in the result, with NULLs for unmatched right-side columns.

---

## Basic LEFT JOIN Examples

### Example 1: All Students with Their Enrollments

**Query:** List all students and their enrollments (including students with no enrollments).

```sql
SELECT s.student_id,
       s.first_name,
       s.last_name,
       e.course_id,
       e.semester,
       e.grade
FROM students s
LEFT JOIN enrollments e ON s.student_id = e.student_id
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
| 5          | Charlie    | Davis     | NULL      | NULL        | NULL  |

**Explanation:** All 5 students appear. Charlie Davis (ID 5) has NULL for enrollment columns because he has no enrollments. This is the key difference from INNER JOIN, which would exclude Charlie.

---

### Example 2: All Courses with Enrollment Counts

**Query:** Show all courses with their enrollment counts.

```sql
SELECT c.course_id,
       c.course_name,
       c.department,
       COUNT(e.enrollment_id) AS enrollment_count
FROM courses c
LEFT JOIN enrollments e ON c.course_id = e.course_id
GROUP BY c.course_id, c.course_name, c.department
ORDER BY enrollment_count DESC, c.course_id;
```

**Result:**

| course_id | course_name                 | department       | enrollment_count |
| --------- | --------------------------- | ---------------- | ---------------- |
| CS101     | Introduction to Programming | Computer Science | 3                |
| CS201     | Data Structures             | Computer Science | 2                |
| CS301     | Database Systems            | Computer Science | 1                |
| MATH101   | Calculus I                  | Mathematics      | 1                |
| PHYS101   | Physics I                   | Physics          | 1                |
| ENG101    | English Composition         | English          | 0                |

**Explanation:** All 6 courses appear, including ENG101 with 0 enrollments. COUNT(e.enrollment_id) counts non-NULL values, so it correctly returns 0 for ENG101.

---

## Finding Missing Relationships

One of the most powerful uses of LEFT JOIN is finding "orphan" records.

### Example 3: Find Students with No Enrollments

**Query:** Identify students who haven't enrolled in any courses.

```sql
SELECT s.student_id,
       s.first_name,
       s.last_name,
       s.email,
       s.enrollment_date
FROM students s
LEFT JOIN enrollments e ON s.student_id = e.student_id
WHERE e.enrollment_id IS NULL;
```

**Result:**

| student_id | first_name | last_name | email                        | enrollment_date |
| ---------- | ---------- | --------- | ---------------------------- | --------------- |
| 5          | Charlie    | Davis     | charlie.davis@university.edu | 2024-09-01      |

**Explanation:** LEFT JOIN ensures all students appear. WHERE e.enrollment_id IS NULL filters for students with no enrollments. This pattern is extremely useful for finding missing relationships.

**Important:** Use `IS NULL`, not `= NULL` (which doesn't work in SQL).

---

## LEFT JOIN vs INNER JOIN Comparison

### Example 9: Side-by-Side Comparison

**INNER JOIN Query:**

```sql
SELECT s.first_name, s.last_name, COUNT(e.enrollment_id) AS courses
FROM students s
INNER JOIN enrollments e ON s.student_id = e.student_id
GROUP BY s.student_id, s.first_name, s.last_name;
```

**INNER JOIN Result:**

| first_name | last_name | courses |
| ---------- | --------- | ------- |
| John       | Smith     | 3       |
| Jane       | Doe       | 2       |
| Bob        | Wilson    | 2       |
| Alice      | Brown     | 1       |

**LEFT JOIN Query:**

```sql
SELECT s.first_name, s.last_name, COUNT(e.enrollment_id) AS courses
FROM students s
LEFT JOIN enrollments e ON s.student_id = e.student_id
GROUP BY s.student_id, s.first_name, s.last_name;
```

**LEFT JOIN Result:**

| first_name | last_name | courses |
| ---------- | --------- | ------- |
| John       | Smith     | 3       |
| Jane       | Doe       | 2       |
| Bob        | Wilson    | 2       |
| Alice      | Brown     | 1       |
| Charlie    | Davis     | 0       |

**Key Difference:** Charlie Davis appears with LEFT JOIN but not with INNER JOIN.
