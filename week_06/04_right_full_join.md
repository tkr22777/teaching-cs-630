# RIGHT JOIN and FULL OUTER JOIN

## Overview

This guide covers two less-commonly-used but important join types:
- **RIGHT JOIN**: Returns all rows from the right table with matches from the left
- **FULL OUTER JOIN**: Returns all rows from both tables with matches where they exist

## RIGHT JOIN (RIGHT OUTER JOIN)

### What is a RIGHT JOIN?

A **RIGHT JOIN** returns all rows from the right (second) table and matching rows from the left (first) table. If there's no match, NULL values are returned for columns from the left table.

**Note:** RIGHT JOIN is essentially the mirror image of LEFT JOIN.

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

### How RIGHT JOIN Works

**Visual Representation:**

```
Table A          Table B          Result (A RIGHT JOIN B)
┌────┬────┐      ┌────┬────┐      ┌────┬──────┬────┐
│ ID │Val │      │ ID │Val │      │ ID │ ValA │ValB│
├────┼────┤      ├────┼────┤      ├────┼──────┼────┤
│  1 │ A1 │◄────►│  1 │ B1 │─────►│  1 │  A1  │ B1 │
│  2 │ A2 │◄────►│  2 │ B2 │─────►│  2 │  A2  │ B2 │
│  4 │ A4 │      │  3 │ B3 │   ┌─►│  3 │ NULL │ B3 │
└────┴────┘      └────┴────┘   │  └────┴──────┴────┘
                    ↑           │  All right rows kept,
                 Row 3 kept ────┘  NULL when no match
```

**Key Point:** ALL rows from the right table appear in the result, with NULLs for unmatched left-side columns.

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
| course_id | course_name | student_id | semester | grade |
|-----------|-------------|------------|----------|-------|
| CS101 | Introduction to Programming | 1 | Fall 2023 | A |
| CS101 | Introduction to Programming | 2 | Fall 2023 | A- |
| CS101 | Introduction to Programming | 3 | Spring 2024 | B |
| CS201 | Data Structures | 1 | Spring 2024 | B+ |
| CS201 | Data Structures | 3 | Spring 2024 | B+ |
| CS301 | Database Systems | 1 | Fall 2024 | NULL |
| ENG101 | English Composition | NULL | NULL | NULL |
| MATH101 | Calculus I | 2 | Fall 2023 | A |
| PHYS101 | Physics I | 4 | Spring 2024 | A |

**Explanation:**
- All 6 courses appear (right table)
- ENG101 has NULL for enrollment columns (no enrollments)
- Similar to courses LEFT JOIN enrollments

> Note: RIGHT JOINs are uncommon in modern codebases because any RIGHT JOIN can be rewritten as a LEFT JOIN by swapping table order.

### Example 3: RIGHT JOIN Rewritten as LEFT JOIN

**RIGHT JOIN version:**
```sql
SELECT c.course_name, e.grade
FROM enrollments e
RIGHT JOIN courses c ON e.course_id = c.course_id;
```

**LEFT JOIN version (preferred):**
```sql
SELECT c.course_name, e.grade
FROM courses c
LEFT JOIN enrollments e ON c.course_id = e.course_id;
```

**Result (identical):**
| course_name | grade |
|-------------|-------|
| Introduction to Programming | A |
| Introduction to Programming | A- |
| Introduction to Programming | B |
| Data Structures | B+ |
| Data Structures | B+ |
| Database Systems | NULL |
| English Composition | NULL |
| Calculus I | A |
| Physics I | A |

## FULL OUTER JOIN

### What is a FULL OUTER JOIN?

A **FULL OUTER JOIN** returns all rows from both tables. Where there's a match, it combines the rows. Where there's no match, it includes the row with NULLs for the missing side.

**Think of it as:** LEFT JOIN + RIGHT JOIN combined.

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

### How FULL OUTER JOIN Works

**Visual Representation:**

```
Table A          Table B          Result (A FULL OUTER JOIN B)
┌────┬────┐      ┌────┬────┐      ┌────┬──────┬──────┐
│ ID │Val │      │ ID │Val │      │ ID │ ValA │ ValB │
├────┼────┤      ├────┼────┤      ├────┼──────┼──────┤
│  1 │ A1 │◄────►│  1 │ B1 │─────►│  1 │  A1  │  B1  │
│  2 │ A2 │◄────►│  2 │ B2 │─────►│  2 │  A2  │  B2  │
│  3 │ A3 │      │  4 │ B4 │   ┌─►│  3 │  A3  │ NULL │
└────┴────┘      └────┴────┘   ├─►│  4 │ NULL │  B4  │
   ↑                 ↑          │  └────┴──────┴──────┘
Row 3 kept ────────────────────┤  All rows from both tables,
                                │  NULL when no match
Row 4 kept ─────────────────────┘
```

**Key Point:** ALL rows from BOTH tables appear, with NULLs where there's no match on either side.

<details>
<summary>Database Support for FULL OUTER JOIN</summary>

- **PostgreSQL:** ✅ Fully supported  
- **Oracle:** ✅ Fully supported  
- **SQL Server:** ✅ Fully supported  
- **MySQL:** ❌ Not supported (use UNION of LEFT and RIGHT JOINs instead)

</details>

### Example 1: All Students and All Courses

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
| student_id | first_name | last_name | course_id | course_name | semester | grade |
|------------|------------|-----------|-----------|-------------|----------|-------|
| 1 | John | Smith | CS101 | Introduction to Programming | Fall 2023 | A |
| 1 | John | Smith | CS201 | Data Structures | Spring 2024 | B+ |
| 1 | John | Smith | CS301 | Database Systems | Fall 2024 | NULL |
| 2 | Jane | Doe | CS101 | Introduction to Programming | Fall 2023 | A- |
| 2 | Jane | Doe | MATH101 | Calculus I | Fall 2023 | A |
| 3 | Bob | Wilson | CS101 | Introduction to Programming | Spring 2024 | B |
| 3 | Bob | Wilson | CS201 | Data Structures | Spring 2024 | B+ |
| 4 | Alice | Brown | PHYS101 | Physics I | Spring 2024 | A |
| 5 | Charlie | Davis | NULL | NULL | NULL | NULL |
| NULL | NULL | NULL | ENG101 | English Composition | NULL | NULL |

**Explanation:**
- Charlie Davis appears (student with no enrollments)
- ENG101 appears (course with no enrollments)
- All enrollments shown with both student and course info

###

### Simulating FULL OUTER JOIN (MySQL)

Since MySQL doesn't support FULL OUTER JOIN, use UNION of LEFT and RIGHT results (with NULL filters) to emulate it:

```sql
-- Left side (all students)
SELECT s.student_id, s.first_name, s.last_name, 
       e.course_id, e.semester
FROM students s
LEFT JOIN enrollments e ON s.student_id = e.student_id

UNION

-- Right side (all enrollments not already covered)
SELECT s.student_id, s.first_name, s.last_name,
       e.course_id, e.semester
FROM students s
RIGHT JOIN enrollments e ON s.student_id = e.student_id
WHERE s.student_id IS NULL;
```

##

