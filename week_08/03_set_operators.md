# SQL Set Operators

## Overview

**Set operators** combine results from two or more SELECT statements. They treat query results as mathematical sets, allowing operations like union, intersection, and difference.

## Key Terms

**Set Operator**: Combines results from multiple queries.

**UNION**: Combines results, removes duplicates.

**UNION ALL**: Combines results, keeps duplicates.

**INTERSECT**: Returns only common rows.

**MINUS**: Returns rows in first query but not second.

## Set Operators Overview

| Operator            | Description                          | Duplicates | Oracle SQL                     |
| ------------------- | ------------------------------------ | ---------- | ------------------------------ |
| **UNION**     | Combines results, removes duplicates | Removed    | ✓                             |
| **UNION ALL** | Combines results, keeps duplicates   | Kept       | ✓                             |
| **INTERSECT** | Only common rows                     | Removed    | ✓                             |
| **MINUS**     | Rows in first but not second         | Removed    | ✓ (EXCEPT in other databases) |

**Visual:**
```
Query A: {1, 2, 3, 4}
Query B: {3, 4, 5, 6}

UNION:      {1, 2, 3, 4, 5, 6}
INTERSECT:  {3, 4}
MINUS:      {1, 2}
```

## Rules for Set Operators

1. Same number of columns in both queries
2. Compatible data types in corresponding columns
3. Column names come from first query

## UNION Operator

Combines results from multiple queries and removes duplicates.

**Syntax:**

```sql
SELECT columns FROM table1
UNION
SELECT columns FROM table2;
```

**Example: Get all departments from courses and instructors**

```sql
SELECT department FROM courses
UNION
SELECT department FROM instructors
ORDER BY department;
```

**Result:**

| department       |
| ---------------- |
| Chemistry        |
| Computer Science |
| English          |
| Mathematics      |
| Physics          |

---

## UNION ALL Operator

Keeps all duplicates (faster than UNION).

| Aspect | UNION | UNION ALL |
|--------|-------|-----------|
| **Duplicates** | Removed | Kept |
| **Performance** | Slower | Faster |

```sql
SELECT department FROM courses
UNION ALL
SELECT department FROM instructors;
```

---

## INTERSECT Operator

Returns only common rows.

```sql
SELECT department FROM courses
INTERSECT
SELECT department FROM instructors;
```

**Result:**
| department       |
| ---------------- |
| Computer Science |
| Mathematics      |
| Physics          |

---

## MINUS Operator

Returns rows in first query but not in second.

```sql
SELECT department FROM courses
MINUS
SELECT department FROM instructors;
```

**Result:**
| department |
| ---------- |
| English    |

**Note:** Order matters with MINUS. Reversing gives `Chemistry` (instructors without courses).
