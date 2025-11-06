# Inline Views (Derived Tables)

## Overview

An **inline view** (also called a derived table) is a subquery in the FROM clause that acts as a temporary table for the duration of the query. It allows you to treat query results as a table.

## Key Terms

**Inline View**: A subquery in the FROM clause that creates a temporary result set.

**Derived Table**: Another name for an inline view.

**Table Alias**: Required name for an inline view (mandatory in Oracle SQL).

## Sample Database Schema

This module uses the university enrollment system.

**Note:** The complete database setup script is available in `00_initialization.md` in this directory.

## Basic Syntax

```sql
SELECT columns
FROM (
    SELECT columns
    FROM table
    WHERE conditions
) alias_name  -- Alias is required!
WHERE conditions;
```

**Key requirements:**
- Inline view must have an alias
- Can reference inline view columns in outer query
- Inline view executes first, then outer query uses results

## Why Use Inline Views?

**Benefits:**
- Break complex queries into logical steps
- Apply filtering after aggregation
- Calculate intermediate results
- Improve query readability

**Use when:**
- Need to filter aggregated data
- Want to reuse calculated columns
- Simplifying complex multi-step logic
- Creating rankings or top-N queries

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
| course_id | course_name | enrollment_count |
|-----------|-------------|------------------|
| CS101 | Introduction to Programming | 3 |
| CS201 | Data Structures | 2 |

**Why inline view:** Allows filtering on the aggregated `enrollment_count`.

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
| first_name | last_name | major | enrollments |
|------------|-----------|-------|-------------|
| John | Smith | Computer Science | 3 |
| Jane | Doe | Mathematics | 2 |
| Bob | Wilson | Computer Science | 2 |
| Alice | Brown | Physics | 1 |
| Charlie | Davis | NULL | 0 |

## Inline Views vs. WITH Clause (CTEs)

Both achieve similar results. WITH clause is often more readable.

**Inline View:**
```sql
SELECT course_id, enrollment_count
FROM (
    SELECT course_id, COUNT(*) AS enrollment_count
    FROM enrollments
    GROUP BY course_id
) enrollment_stats
WHERE enrollment_count > 1;
```

**WITH Clause (CTE):**
```sql
WITH enrollment_stats AS (
    SELECT course_id, COUNT(*) AS enrollment_count
    FROM enrollments
    GROUP BY course_id
)
SELECT course_id, enrollment_count
FROM enrollment_stats
WHERE enrollment_count > 1;
```

**Comparison:**

| Aspect | Inline View | WITH Clause (CTE) |
|--------|-------------|-------------------|
| **Readability** | Less readable (nested) | More readable (sequential) |
| **Reusability** | Use once | Can reference multiple times |
| **Debugging** | Harder to debug | Easier to test separately |
| **Oracle Support** | All versions | Oracle 9i+ |

**Recommendation:** Use WITH clause for complex queries or when reusing subqueries.

## Common Mistakes

**Mistake 1: Forgetting table alias**
```sql
-- ERROR: Missing alias
SELECT * FROM (SELECT * FROM students);

-- Correct
SELECT * FROM (SELECT * FROM students) s;
```

**Mistake 2: Column name conflicts**
```sql
-- Ambiguous: Which student_id?
SELECT student_id
FROM (SELECT student_id FROM students) s
JOIN enrollments e ON student_id = e.student_id;

-- Correct: Use aliases
SELECT s.student_id
FROM (SELECT student_id FROM students) s
JOIN enrollments e ON s.student_id = e.student_id;
```

## Performance Tips

1. **Add appropriate indexes** on columns used in WHERE/JOIN
2. **Minimize inline view size** - filter early
3. **Use WITH clause** for better optimization by Oracle

