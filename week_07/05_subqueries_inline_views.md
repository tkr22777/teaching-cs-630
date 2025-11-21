# Inline Views (Derived Tables)

## Overview

An **inline view** (also called a derived table) is a subquery in the FROM clause that acts as a temporary table for the duration of the query. It allows you to treat query results as a table.

## Key Terms

**Inline View**: A subquery in the FROM clause that acts as a temporary table.

**Table Alias**: Required name for an inline view.

## Syntax

```sql
SELECT columns
FROM (
    SELECT columns
    FROM table
    WHERE conditions
) alias_name  -- Alias required!
WHERE conditions;
```

**Use inline views to:** Filter aggregated data or use calculated columns in outer queries.

## Examples

### Example 1: Aggregate then Filter

**Find courses with more than 1 enrollment:**

```sql
SELECT course_id, course_name, enrollment_count
FROM (
    SELECT 
        c.course_id,
        c.course_name,
        COUNT(e.enrollment_id) AS enrollment_count
    FROM courses c
    LEFT JOIN enrollments e ON c.course_id = e.course_id
    GROUP BY c.course_id, c.course_name
) course_stats
WHERE enrollment_count > 1
ORDER BY enrollment_count DESC;
```

**Result:**

| course_id | course_name                 | enrollment_count |
| --------- | --------------------------- | ---------------- |
| CS101     | Introduction to Programming | 3                |
| CS201     | Data Structures             | 2                |

---

### Example 2: Joining with Inline View

**Show students with their enrollment count:**

```sql
SELECT 
    s.first_name,
    s.last_name,
    s.major,
    COALESCE(e_count.enrollment_count, 0) AS enrollments
FROM students s
LEFT JOIN (
    SELECT student_id, COUNT(*) AS enrollment_count
    FROM enrollments
    GROUP BY student_id
) e_count ON s.student_id = e_count.student_id
ORDER BY enrollments DESC;
```

**Result:**

| first_name | last_name | major            | enrollments |
| ---------- | --------- | ---------------- | ----------- |
| John       | Smith     | Computer Science | 3           |
| Jane       | Doe       | Mathematics      | 2           |
| Bob        | Wilson    | Computer Science | 2           |
| Alice      | Brown     | Physics          | 1           |
| Charlie    | Davis     | NULL             | 0           |

---

## Common Mistake

**Forgetting the required alias:**

```sql
-- ERROR: Missing alias
SELECT * FROM (SELECT * FROM students);

-- Correct
SELECT * FROM (SELECT * FROM students) s;
```
