# Multiple-Row Subqueries

## Overview

A **multiple-row subquery** returns zero or more rows. These require special operators: IN, NOT IN, ANY, or ALL (EXISTS covered separately).

## Key Terms

**Multiple-Row Subquery**: A subquery returning multiple rows (requires IN, ANY, or ALL operators).

## Multiple-Row Operators

When working with multiple-row subqueries, you need special operators that can handle lists of values:

| Operator         | Description                            | Example                      |
| ---------------- | -------------------------------------- | ---------------------------- |
| **IN**     | Matches any value in list              | `WHERE id IN (1, 2, 3)`    |
| **NOT IN** | Does not match any value               | `WHERE id NOT IN (4, 5)`   |
| **ANY**    | Compare to any value (>, <, =, etc.)   | `WHERE salary > ANY (...)` |
| **ALL**    | Compare to all values (all must match) | `WHERE salary > ALL (...)` |

**Quick Reference:**

- `= ANY` is same as `IN`
- `!= ALL` is same as `NOT IN`
- `> ANY` means "greater than minimum"
- `> ALL` means "greater than maximum"

## IN Operator

**Visual: How IN Works**

```text
Step 1: Subquery returns multiple student IDs
        ┌─────────────┐
        │ student_id  │
        ├─────────────┤
        │ 1           │  ← John
        │ 2           │  ← Jane
        │ 3           │  ← Bob
        └─────────────┘

Step 2: Check each student against this list
      
students table              Subquery result
┌────┬───────┐             ┌────┐
│ id │ name  │             │ id │
├────┼───────┤             ├────┤
│ 1  │ John  │ ──match──► │ 1  │ ✓ Include
│ 2  │ Jane  │ ──match──► │ 2  │ ✓ Include
│ 3  │ Bob   │ ──match──► │ 3  │ ✓ Include
│ 4  │ Alice │ ──no───────┘     │ ✗ Exclude
│ 5  │ Charlie│ ──no───────     │ ✗ Exclude
└────┴───────┘
```

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

| first_name | last_name | major            |
| ---------- | --------- | ---------------- |
| John       | Smith     | Computer Science |
| Jane       | Doe       | Mathematics      |
| Bob        | Wilson    | Computer Science |

---

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
| ---------- | --------- |
| Charlie    | Davis     |

## ANY Operator

**Syntax:**

```sql
WHERE column operator ANY (subquery)
-- operator can be: =, !=, >, <, >=, <=
```

**How ANY works:**

| Expression                | Equivalent                       |
| ------------------------- | -------------------------------- |
| `> ANY (100, 200, 300)` | `> 100` (greater than minimum) |
| `< ANY (100, 200, 300)` | `< 300` (less than maximum)    |
| `= ANY (100, 200, 300)` | Same as `IN`                   |

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

| first_name | last_name | major       | gpa |
| ---------- | --------- | ----------- | --- |
| Jane       | Doe       | Mathematics | 3.9 |
| Alice      | Brown     | Physics     | 3.7 |

---

## ALL Operator

**Syntax:**

```sql
WHERE column operator ALL (subquery)
```

**How ALL works:**

| Expression                 | Equivalent                       |
| -------------------------- | -------------------------------- |
| `> ALL (100, 200, 300)`  | `> 300` (greater than maximum) |
| `< ALL (100, 200, 300)`  | `< 100` (less than minimum)    |
| `!= ALL (100, 200, 300)` | Same as `NOT IN`               |

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

---

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
