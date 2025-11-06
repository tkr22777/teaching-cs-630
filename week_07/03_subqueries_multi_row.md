# Multiple-Row Subqueries

## Overview

A **multiple-row subquery** returns zero or more rows. These require special operators: IN, NOT IN, ANY, or ALL (EXISTS covered separately).

## Key Terms

**Multiple-Row Subquery**: A subquery returning zero, one, or more rows.

**IN Operator**: Tests if a value matches any value in a list.

**ANY Operator**: Compares a value to each subquery value; TRUE if any comparison is true.

**ALL Operator**: Compares a value to each subquery value; TRUE only if all comparisons are true.

## Sample Database Schema

University enrollment system. Setup: `00_initialization.md`

## Multiple-Row Operators

| Operator | Description | Example |
|----------|-------------|---------|
| **IN** | Matches any value in list | `WHERE id IN (1, 2, 3)` |
| **NOT IN** | Does not match any value | `WHERE id NOT IN (4, 5)` |
| **ANY** | Compare to any value (>, <, =, etc.) | `WHERE salary > ANY (...)` |
| **ALL** | Compare to all values (all must match) | `WHERE salary > ALL (...)` |

**Quick Reference:**
- `= ANY` is same as `IN`
- `!= ALL` is same as `NOT IN`
- `> ANY` means "greater than minimum"
- `> ALL` means "greater than maximum"

## IN Operator

**Syntax:**
```sql
WHERE column IN (subquery)
```

**Example: Find students enrolled in Computer Science courses**

```sql
SELECT first_name, last_name, major
FROM students
WHERE student_id IN (
    SELECT DISTINCT e.student_id
    FROM enrollments e
    JOIN courses c ON e.course_id = c.course_id
    WHERE c.department = 'Computer Science'
);
```

**Result:**
| first_name | last_name | major |
|------------|-----------|-------|
| John | Smith | Computer Science |
| Jane | Doe | Mathematics |
| Bob | Wilson | Computer Science |

## NOT IN Operator

**Syntax:**
```sql
WHERE column NOT IN (subquery)
```

**Example: Find students who have NOT enrolled in any courses**

```sql
SELECT first_name, last_name
FROM students
WHERE student_id NOT IN (
    SELECT student_id 
    FROM enrollments
    WHERE student_id IS NOT NULL  -- Important!
);
```

**Result:**
| first_name | last_name |
|------------|-----------|
| Charlie | Davis |

### Critical: NOT IN and NULL Values

**Problem:** If the subquery returns any NULL, `NOT IN` may return no rows!

```sql
-- This can fail if subquery has NULL
WHERE id NOT IN (SELECT id FROM table)
```

**Why:** `NOT IN (1, 2, NULL)` becomes `!= 1 AND != 2 AND != NULL`. Comparing to NULL returns NULL (not TRUE/FALSE), making the entire condition NULL, which is treated as FALSE.

**Solutions:**

```sql
-- Solution 1: Filter out NULLs
WHERE id NOT IN (SELECT id FROM table WHERE id IS NOT NULL)

-- Solution 2: Use NOT EXISTS (preferred)
WHERE NOT EXISTS (
    SELECT 1 FROM table t WHERE t.id = outer.id
)
```

## ANY Operator

**Syntax:**
```sql
WHERE column operator ANY (subquery)
-- operator can be: =, !=, >, <, >=, <=
```

**Meaning:**
| Expression | Equivalent |
|------------|------------|
| `> ANY (100, 200, 300)` | `> 100` (greater than minimum) |
| `< ANY (100, 200, 300)` | `< 300` (less than maximum) |
| `= ANY (100, 200, 300)` | Same as `IN` |

**Example: Find students with GPA higher than any CS major**

```sql
SELECT first_name, last_name, major, gpa
FROM students
WHERE gpa > ANY (
    SELECT gpa
    FROM students
    WHERE major = 'Computer Science' AND gpa IS NOT NULL
)
AND major != 'Computer Science';
```

**Result:**
| first_name | last_name | major | gpa |
|------------|-----------|-------|-----|
| Jane | Doe | Mathematics | 3.9 |
| Alice | Brown | Physics | 3.7 |

## ALL Operator

**Syntax:**
```sql
WHERE column operator ALL (subquery)
```

**Meaning:**
| Expression | Equivalent |
|------------|------------|
| `> ALL (100, 200, 300)` | `> 300` (greater than maximum) |
| `< ALL (100, 200, 300)` | `< 100` (less than minimum) |
| `!= ALL (100, 200, 300)` | Same as `NOT IN` |

**Example: Find student with highest GPA**

```sql
SELECT first_name, last_name, gpa
FROM students s1
WHERE gpa > ALL (
    SELECT gpa
    FROM students s2
    WHERE s2.student_id != s1.student_id AND gpa IS NOT NULL
)
AND gpa IS NOT NULL;
```

**Alternative (simpler):**
```sql
SELECT first_name, last_name, gpa
FROM students
WHERE gpa = (SELECT MAX(gpa) FROM students);
```

## Best Practices

**1. Use IN for simple membership tests (add DISTINCT if needed)**
```sql
WHERE student_id IN (SELECT DISTINCT student_id FROM enrollments)
```

**2. Always handle NULLs with NOT IN**
```sql
-- Add NULL filter or use NOT EXISTS
WHERE id NOT IN (SELECT id FROM table WHERE id IS NOT NULL)
```

**3. Prefer EXISTS for complex queries**
```sql
-- Often faster than IN
WHERE EXISTS (
    SELECT 1 FROM enrollments e 
    WHERE e.student_id = students.student_id
)
```

## Common Errors

**Error 1: Using = with multiple rows**
```sql
-- ERROR: subquery returns more than one row
WHERE gpa > (SELECT gpa FROM students WHERE major = 'Computer Science')

-- Fix: Use ANY or ALL
WHERE gpa > ALL (SELECT gpa FROM students WHERE major = 'Computer Science')
```

**Error 2: Forgetting NULL handling**
```sql
-- May return no rows
WHERE id NOT IN (SELECT id FROM table)

-- Fix
WHERE id NOT IN (SELECT id FROM table WHERE id IS NOT NULL)
```

