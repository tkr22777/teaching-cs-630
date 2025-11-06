# Correlated Subqueries

## Overview

A **correlated subquery** references columns from the outer query and executes once for each row processed by the outer query.

## Key Terms

**Correlated Subquery**: A subquery that references columns from the outer query and executes repeatedly for each outer row.

**Row-by-Row Processing**: The execution pattern where the subquery runs once for each row in the outer query.

**Table Alias**: Required shorthand name for tables to distinguish between inner and outer references.

## Sample Database Schema

University enrollment system. Setup: `00_initialization.md`

## Key Characteristics

**Correlated subqueries:**
1. Reference outer query columns
2. Execute once per outer row
3. Require table aliases
4. Enable row-specific comparisons
5. Generally slower than non-correlated subqueries

**Execution Flow:**
```sql
SELECT outer.column
FROM table1 outer
WHERE outer.value = (
    SELECT inner.value
    FROM table2 inner
    WHERE inner.id = outer.id  -- Correlation
);
```

## Correlated vs. Non-Correlated

| Aspect | Non-Correlated | Correlated |
|--------|----------------|------------|
| **Execution** | Once | Once per outer row |
| **References** | No outer columns | References outer columns |
| **Performance** | Faster | Slower |
| **Use Case** | Global comparisons | Row-specific comparisons |

**Non-Correlated Example:**
```sql
-- Executes once
SELECT first_name, last_name, gpa
FROM students
WHERE gpa > (SELECT AVG(gpa) FROM students);
```

**Correlated Example:**
```sql
-- Executes once per student
SELECT s.first_name, s.last_name, s.major, s.gpa
FROM students s
WHERE s.gpa > (
    SELECT AVG(s2.gpa)
    FROM students s2
    WHERE s2.major = s.major  -- Correlation
)
AND s.major IS NOT NULL;
```

## Common Patterns

### Pattern 1: Above-Average Within Group

**Find students with GPA above their major's average:**

```sql
SELECT s.first_name, s.last_name, s.major, s.gpa,
       (SELECT AVG(s2.gpa) 
        FROM students s2 
        WHERE s2.major = s.major) AS major_avg
FROM students s
WHERE s.gpa > (
    SELECT AVG(s2.gpa)
    FROM students s2
    WHERE s2.major = s.major
)
AND s.major IS NOT NULL
ORDER BY s.major, s.gpa DESC;
```

**Result:**
| first_name | last_name | major | gpa | major_avg |
|------------|-----------|-------|-----|-----------|
| John | Smith | Computer Science | 3.8 | 3.5 |

### Pattern 2: Count Related Records

**Show each course with its enrollment count:**

```sql
SELECT 
    c.course_id,
    c.course_name,
    (SELECT COUNT(*)
     FROM enrollments e
     WHERE e.course_id = c.course_id) AS enrollment_count
FROM courses c
ORDER BY enrollment_count DESC;
```

**Result:**
| course_id | course_name | enrollment_count |
|-----------|-------------|------------------|
| CS101 | Introduction to Programming | 3 |
| CS201 | Data Structures | 2 |
| CS301 | Database Systems | 1 |
| MATH101 | Calculus I | 1 |
| PHYS101 | Physics I | 1 |
| ENG101 | English Composition | 0 |

### Pattern 3: Latest/Most Recent Record

**Find each student's most recent enrollment:**

```sql
SELECT s.first_name, s.last_name, e.course_id, e.semester
FROM students s
JOIN enrollments e ON s.student_id = e.student_id
WHERE e.enrollment_id = (
    SELECT MAX(e2.enrollment_id)
    FROM enrollments e2
    WHERE e2.student_id = s.student_id
);
```

**Result:**
| first_name | last_name | course_id | semester |
|------------|-----------|-----------|----------|
| John | Smith | CS301 | Fall 2024 |
| Jane | Doe | CS101 | Fall 2023 |
| Bob | Wilson | CS201 | Spring 2024 |
| Alice | Brown | PHYS101 | Spring 2024 |

## Using in Different Clauses

**In SELECT Clause:**

```sql
SELECT 
    s.first_name,
    s.last_name,
    (SELECT COUNT(*)
     FROM enrollments e
     WHERE e.student_id = s.student_id) AS course_count,
    (SELECT AVG(e.grade_points)
     FROM enrollments e
     WHERE e.student_id = s.student_id) AS avg_grade
FROM students s;
```

**In WHERE Clause:**

```sql
SELECT s.first_name, s.last_name, s.major, s.gpa
FROM students s
WHERE s.gpa > (
    SELECT AVG(s2.gpa)
    FROM students s2
    WHERE s2.major = s.major AND s2.gpa IS NOT NULL
)
AND s.major IS NOT NULL;
```

**In HAVING Clause:**

```sql
SELECT s.major, AVG(s.gpa) AS major_avg_gpa
FROM students s
WHERE s.major IS NOT NULL
GROUP BY s.major
HAVING AVG(s.gpa) > (
    SELECT AVG(gpa) 
    FROM students
    WHERE gpa IS NOT NULL
);
```

**Result:**
| major | major_avg_gpa |
|-------|---------------|
| Mathematics | 3.9 |
| Physics | 3.7 |
| Computer Science | 3.5 |

## Performance Considerations

Correlated subqueries execute once per outer row with no result caching, making them slower than alternatives.

**Optimization strategies:**

**1. Add indexes on correlated columns**
```sql
CREATE INDEX idx_enrollments_student ON enrollments(student_id);
```

**2. Use JOINs when appropriate**
```sql
-- Correlated subquery
SELECT s.first_name, 
       (SELECT COUNT(*) FROM enrollments e WHERE e.student_id = s.student_id)
FROM students s;

-- JOIN alternative (often faster)
SELECT s.first_name, COUNT(e.enrollment_id)
FROM students s
LEFT JOIN enrollments e ON s.student_id = e.student_id
GROUP BY s.student_id, s.first_name;
```

**3. Consider analytic functions**
```sql
-- Instead of correlated subquery for group averages
SELECT first_name, last_name, gpa,
       AVG(gpa) OVER (PARTITION BY major) AS major_avg
FROM students;
```

## Common Mistakes

**Mistake 1: Missing table aliases**
```sql
-- ERROR: Which table's major?
WHERE gpa > (SELECT AVG(gpa) FROM students WHERE major = major)

-- Correct
WHERE s1.gpa > (SELECT AVG(s2.gpa) FROM students s2 WHERE s2.major = s1.major)
```

**Mistake 2: Forgetting NULL handling**
```sql
-- May fail with NULLs
WHERE s.major IS NOT NULL
AND gpa > (SELECT AVG(gpa) FROM students s2 WHERE s2.major = s.major AND gpa IS NOT NULL)
```

**Mistake 3: Repeating identical subqueries**
```sql
-- Inefficient: same subquery twice
SELECT (SELECT COUNT(*) FROM enrollments e WHERE e.student_id = s.student_id) AS cnt1,
       (SELECT COUNT(*) FROM enrollments e WHERE e.student_id = s.student_id) AS cnt2
FROM students s;

-- Better: use WITH clause
WITH counts AS (
    SELECT student_id, COUNT(*) AS cnt FROM enrollments GROUP BY student_id
)
SELECT c.cnt AS cnt1, c.cnt AS cnt2
FROM students s LEFT JOIN counts c ON s.student_id = c.student_id;
```

## When to Use Correlated Subqueries

**Use for:** Row-specific comparisons, calculated columns, existence checks, complex filtering

**Avoid when:** Performance critical, simple global comparisons, or need multiple columns (use JOINs instead)

