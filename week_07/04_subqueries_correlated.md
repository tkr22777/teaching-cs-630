# Correlated Subqueries

## Overview

A **correlated subquery** references columns from the outer query and executes once for each row processed by the outer query.

## Key Terms

**Correlated Subquery**: A subquery that references columns from the outer query and executes repeatedly for each outer row.

**Table Alias**: Required shorthand name for tables (e.g., `s`, `s2`) to distinguish between inner and outer references.

## How Correlated Subqueries Work

**Visual: Row-by-Row Execution**

```text
Query: Find students with GPA above their major's average

Outer Table (s)                        Inner Table (s2 - same table!)
┌────┬──────┬──────┬─────┐            ┌────┬──────┬──────┬─────┐
│ id │ name │ major│ gpa │            │ id │ name │ major│ gpa │
├────┼──────┼──────┼─────┤            ├────┼──────┼──────┼─────┤
│ 1  │ John │ CS   │ 3.8 │◄───┐       │ 1  │ John │ CS   │ 3.8 │
│ 2  │ Jane │ Math │ 3.9 │    │       │ 2  │ Jane │ Math │ 3.9 │
│ 3  │ Bob  │ CS   │ 3.2 │    │       │ 3  │ Bob  │ CS   │ 3.2 │
└────┴──────┴──────┴─────┘    │       └────┴──────┴──────┴─────┘
                               │              ▲
Step 1: Process Row 1          │              │
(John, CS, 3.8)                └──────────────┘
                          s.major='CS' filters s2
                                   ↓
                          Filter: WHERE s2.major='CS'
                                   ↓
                          s2 rows: John(3.8), Bob(3.2)
                                   ↓
                          Calculate: AVG(3.8, 3.2) = 3.5
                                   ↓
                          Compare: 3.8 > 3.5? ✓ YES → Include John


Step 2: Process Row 2 (Jane, Math, 3.9)
                          s.major='Math' filters s2
                                   ↓
                          Filter: WHERE s2.major='Math'
                                   ↓
                          s2 rows: Jane(3.9)
                                   ↓
                          Calculate: AVG(3.9) = 3.9
                                   ↓
                          Compare: 3.9 > 3.9? ✗ NO → Exclude Jane


Step 3: Process Row 3 (Bob, CS, 3.2)
                          s.major='CS' filters s2
                                   ↓
                          Filter: WHERE s2.major='CS'
                                   ↓
                          s2 rows: John(3.8), Bob(3.2)
                                   ↓
                          Calculate: AVG(3.8, 3.2) = 3.5
                                   ↓
                          Compare: 3.2 > 3.5? ✗ NO → Exclude Bob


Result: 1 row (John) out of 3 rows
```

**Key point:** Subquery runs 3 times (once per outer row), using each row's major to filter and calculate a different average.

**Characteristics:**
- Reference outer query columns
- Execute once per outer row (slower)
- **Require table aliases** to distinguish outer and inner tables

**Example:**
```sql
SELECT s.first_name, s.gpa
FROM students s
WHERE s.gpa > (
    SELECT AVG(s2.gpa)
    FROM students s2
    WHERE s2.major = s.major  -- References outer query's s.major
);
```

---

## Common Patterns

Here are the most useful patterns for correlated subqueries:

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

---

## Common Mistake

**Missing table aliases causes ambiguity:**

```sql
-- ERROR: Which table's major?
WHERE gpa > (SELECT AVG(gpa) FROM students WHERE major = major)

-- Correct: Use aliases
WHERE s1.gpa > (SELECT AVG(s2.gpa) FROM students s2 WHERE s2.major = s1.major)
```

## Summary

**Use correlated subqueries for:**
- Row-specific comparisons (GPA above major average)
- Calculated columns (enrollment count per course)

**Important:** Correlated subqueries are slower (run once per row). Consider JOINs for better performance when possible.

