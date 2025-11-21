# RIGHT JOIN and FULL OUTER JOIN

## Overview

This guide covers two less-commonly-used but important join types:

- **RIGHT JOIN**: Returns all rows from the right table with matches from the left
- **FULL OUTER JOIN**: Returns all rows from both tables with matches where they exist

## RIGHT JOIN (RIGHT OUTER JOIN)

### What is a RIGHT JOIN?

A **RIGHT JOIN** returns all rows from the right (second) table and matching rows from the left (first) table. If there's no match, NULL values are returned for columns from the left table.

**Note:** RIGHT JOIN is essentially the mirror image of LEFT JOIN.

### Visual: How RIGHT JOIN Works

**Note:** This diagram shows simplified data for clarity. Each enrollment represents one student's enrollment in a course.

```text
LEFT Table (Enrollments)        RIGHT Table (Courses)
┌─────────────┬───────────┐    ┌───────────┬──────────────┐
│ student_id  │ course_id │    │ course_id │ course_name  │
├─────────────┼───────────┤    ├───────────┼──────────────┤
│ 1           │ CS101     │ ◄──│ CS101     │ Programming  │  ✓ MATCH
│ 2           │ CS201     │ ◄──│ CS201     │ Data Struct  │  ✓ MATCH
│ 3           │ MATH101   │ ◄──│ MATH101   │ Calculus     │  ✓ MATCH
│ 4           │ PHYS101   │ ◄──│ PHYS101   │ Physics      │  ✓ MATCH
(no enrollment for ENG101)  ┌─│ ENG101    │ English      │  ✗ NO MATCH
                            │  └───────────┴──────────────┘
                            └─ Still included with NULL

                      ↓ RIGHT JOIN ↓

              Result (All from RIGHT + Matches)
         ┌─────────────┬───────────┬──────────────┐
         │ student_id  │ course_id │ course_name  │
         ├─────────────┼───────────┼──────────────┤
         │ 1           │ CS101     │ Programming  │  ✓
         │ 2           │ CS201     │ Data Struct  │  ✓
         │ 3           │ MATH101   │ Calculus     │  ✓
         │ 4           │ PHYS101   │ Physics      │  ✓
         │ NULL        │ ENG101    │ English      │  ← Preserved with NULL
         └─────────────┴───────────┴──────────────┘
```

**How RIGHT JOIN matches rows:** The database takes every row from the RIGHT table (Courses) and looks for matches in the LEFT table (Enrollments). When a match is found, rows are combined. When no match exists (ENG101 has no enrollments), the row from the right table is still included with NULL values for the left table's columns. ALL rows from the right table appear in the result.

### Syntax

```sql
SELECT columns
FROM table1
RIGHT JOIN table2 ON table1.column = table2.column;
```

**Alternative (equivalent):**

```sql
SELECT columns
FROM table1
RIGHT OUTER JOIN table2 ON table1.column = table2.column;
```

**How it works:** ALL rows from the right table appear in the result, with NULLs for unmatched left-side columns.

---

### Example 1: All Courses with Enrollments

**Query:** List all courses and their enrollments (including courses with no enrollments).

```sql
SELECT c.course_id,
       c.course_name,
       e.student_id,
       e.semester,
       e.grade
FROM enrollments e
RIGHT JOIN courses c ON e.course_id = c.course_id
ORDER BY c.course_id, e.semester;
```

**Result:**

| course_id | course_name                 | student_id | semester    | grade |
| --------- | --------------------------- | ---------- | ----------- | ----- |
| CS101     | Introduction to Programming | 1          | Fall 2023   | A     |
| CS101     | Introduction to Programming | 2          | Fall 2023   | A-    |
| CS101     | Introduction to Programming | 3          | Spring 2024 | B     |
| CS201     | Data Structures             | 1          | Spring 2024 | B+    |
| CS201     | Data Structures             | 3          | Spring 2024 | B+    |
| CS301     | Database Systems            | 1          | Fall 2024   | NULL  |
| ENG101    | English Composition         | NULL       | NULL        | NULL  |
| MATH101   | Calculus I                  | 2          | Fall 2023   | A     |
| PHYS101   | Physics I                   | 4          | Spring 2024 | A     |

**Explanation:** All 6 courses appear (right table). ENG101 has NULL for enrollment columns (no enrollments).

**Note:** RIGHT JOINs are uncommon because any RIGHT JOIN can be rewritten as a LEFT JOIN by swapping table order.

---

## FULL OUTER JOIN

### What is a FULL OUTER JOIN?

A **FULL OUTER JOIN** returns all rows from both tables. Where there's a match, it combines the rows. Where there's no match, it includes the row with NULLs for the missing side.

**Think of it as:** LEFT JOIN + RIGHT JOIN combined.

### Visual: How FULL OUTER JOIN Works

**Note:** This conceptual diagram shows students and courses through enrollments. In practice, FULL OUTER JOIN combines all rows from both tables with matches where they exist.

```text
LEFT Table (Students)           RIGHT Table (Courses)
┌─────────────┬─────────┐      ┌───────────┬──────────────┐
│ student_id  │ name    │      │ course_id │ course_name  │
├─────────────┼─────────┤      ├───────────┼──────────────┤
│ 1           │ John    │ ───► │ CS101     │ Programming  │  ✓ MATCH
│ 2           │ Jane    │ ───► │ CS201     │ Data Struct  │  ✓ MATCH
│ 3           │ Bob     │ ───► │ MATH101   │ Calculus     │  ✓ MATCH
│ 4           │ Alice   │ ───► │ PHYS101   │ Physics      │  ✓ MATCH
│ 5           │ Charlie │      │ ENG101    │ English      │  (no connections)
└─────────────┴─────────┘      └───────────┴──────────────┘
           │                              │
           └──────── Both included ───────┘

                   ↓ FULL OUTER JOIN ↓

           Result (Everything from BOTH tables)
     ┌─────────────┬─────────┬───────────┬──────────────┐
     │ student_id  │ name    │ course_id │ course_name  │
     ├─────────────┼─────────┼───────────┼──────────────┤
     │ 1           │ John    │ CS101     │ Programming  │  ✓ Matched
     │ 2           │ Jane    │ CS201     │ Data Struct  │  ✓ Matched
     │ 3           │ Bob     │ MATH101   │ Calculus     │  ✓ Matched
     │ 4           │ Alice   │ PHYS101   │ Physics      │  ✓ Matched
     │ 5           │ Charlie │ NULL      │ NULL         │  ← Left only
     │ NULL        │ NULL    │ ENG101    │ English      │  ← Right only
     └─────────────┴─────────┴───────────┴──────────────┘
```

**How FULL OUTER JOIN matches rows:** The database combines ALL rows from BOTH tables. When a match is found, rows are combined. When a row in the left table has no match, it's included with NULLs for right table columns (Charlie). When a row in the right table has no match, it's included with NULLs for left table columns (ENG101). Nothing is lost from either table.

### Syntax

```sql
SELECT columns
FROM table1
FULL OUTER JOIN table2 ON table1.column = table2.column;
```

**Alternative:**

```sql
SELECT columns
FROM table1
FULL JOIN table2 ON table1.column = table2.column;
```

**How it works:** ALL rows from BOTH tables appear, with NULLs where there's no match on either side. Think of it as LEFT JOIN + RIGHT JOIN combined.

---

### Example: All Students and All Courses

**Query:** Show relationship between all students and all courses (with enrollments where they exist).

```sql
SELECT s.student_id,
       s.first_name,
       s.last_name,
       c.course_id,
       c.course_name,
       e.semester,
       e.grade
FROM students s
FULL OUTER JOIN enrollments e ON s.student_id = e.student_id
FULL OUTER JOIN courses c ON e.course_id = c.course_id
ORDER BY s.student_id NULLS LAST, c.course_id NULLS LAST;
```

**Result (key rows):**

| student_id | first_name | last_name | course_id | course_name                 | semester    | grade |
| ---------- | ---------- | --------- | --------- | --------------------------- | ----------- | ----- |
| 1          | John       | Smith     | CS101     | Introduction to Programming | Fall 2023   | A     |
| 1          | John       | Smith     | CS201     | Data Structures             | Spring 2024 | B+    |
| 1          | John       | Smith     | CS301     | Database Systems            | Fall 2024   | NULL  |
| 2          | Jane       | Doe       | CS101     | Introduction to Programming | Fall 2023   | A-    |
| 2          | Jane       | Doe       | MATH101   | Calculus I                  | Fall 2023   | A     |
| 3          | Bob        | Wilson    | CS101     | Introduction to Programming | Spring 2024 | B     |
| 3          | Bob        | Wilson    | CS201     | Data Structures             | Spring 2024 | B+    |
| 4          | Alice      | Brown     | PHYS101   | Physics I                   | Spring 2024 | A     |
| 5          | Charlie    | Davis     | NULL      | NULL                        | NULL        | NULL  |
| NULL       | NULL       | NULL      | ENG101    | English Composition         | NULL        | NULL  |

**Explanation:** Charlie Davis appears (student with no enrollments), ENG101 appears (course with no enrollments), and all enrollments are shown with both student and course info.
