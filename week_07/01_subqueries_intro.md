# Introduction to Subqueries

## Overview

A **subquery** (also called an inner query or nested query) is a query embedded within another SQL statement. Subqueries allow you to break complex queries into logical steps and perform operations that would otherwise require multiple separate queries or complex joins.

## Key Terms

**Subquery**: A SELECT statement nested inside another SQL statement.

**Outer Query**: The main query that contains the subquery.

**Inner Query**: The subquery that executes first and provides results to the outer query.

## What Are Subqueries?

Subqueries are queries nested within other queries. The subquery executes first, and its result is used by the outer query.

**Basic Structure:**

```sql
SELECT column1, column2
FROM table1
WHERE column1 = (SELECT column_x FROM table2 WHERE condition);
                 └─────── Subquery ───────────┘
```

**Common uses:**

- Find values above/below average
- Compare to maximum or minimum values
- Check if records exist in another table
- Filter data dynamically based on calculated values

## Types of Subqueries

**By Result Type:**

| Type                   | Returns         | Used With           |
| ---------------------- | --------------- | ------------------- |
| **Single-Row**   | One value       | =, >, <, >=, <=, != |
| **Multiple-Row** | Multiple values | IN, ANY, ALL        |

**By Execution:**

| Type                     | Execution                                        |
| ------------------------ | ------------------------------------------------ |
| **Non-Correlated** | Runs once (independent)                          |
| **Correlated**     | Runs once per outer row (references outer query) |

---

## Important Rules

**Subqueries must:**

- Be enclosed in parentheses: `WHERE id = (SELECT ...)`
- Return the right number of rows: use `=` for single-row, `IN` for multiple-row
- Match operators to result type

**Common mistake:**

```sql
-- ERROR: Using = when subquery returns multiple rows
WHERE student_id = (SELECT student_id FROM enrollments);

-- Fix: Use IN for multiple rows
WHERE student_id IN (SELECT student_id FROM enrollments);
```
