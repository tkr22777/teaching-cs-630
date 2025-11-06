# SQL Set Operators

## Overview

**Set operators** combine results from two or more SELECT statements. They treat query results as mathematical sets, allowing operations like union, intersection, and difference.

## Key Terms

**Set Operator**: Combines results from multiple queries (UNION, UNION ALL, INTERSECT, MINUS).

**UNION**: Combines results and removes duplicates.

**UNION ALL**: Combines results and keeps all duplicates.

**INTERSECT**: Returns only rows common to both queries.

**MINUS**: Returns rows from first query that are not in second query.

## Sample Database Schema

This module uses the university enrollment system.

**Note:** The complete database setup script is available in `00_initialization.md` in this directory.

## Set Operators Overview

| Operator | Description | Duplicates | Oracle SQL |
|----------|-------------|------------|------------|
| **UNION** | Combines results, removes duplicates | Removed | ✓ |
| **UNION ALL** | Combines results, keeps duplicates | Kept | ✓ |
| **INTERSECT** | Only common rows | Removed | ✓ |
| **MINUS** | Rows in first but not second | Removed | ✓ (EXCEPT in other databases) |

**Visual representation:**

```
Query A: {1, 2, 3, 4}
Query B: {3, 4, 5, 6}

UNION:      {1, 2, 3, 4, 5, 6}
INTERSECT:  {3, 4}
MINUS:      {1, 2}  (A - B)
```

## Universal Rules for Set Operators

1. **Same number of columns** in both queries
2. **Compatible data types** in corresponding columns
3. **Column names** come from first query
4. **ORDER BY** only at the end (applies to final result)

**Example of compatible types:**
```sql
-- Valid: NUMBER and NUMBER
SELECT student_id FROM students
UNION
SELECT instructor_id FROM instructors;

-- Valid: VARCHAR2 and VARCHAR2
SELECT first_name FROM students
UNION
SELECT instructor_name FROM instructors;

-- Invalid: NUMBER and VARCHAR2
SELECT student_id FROM students
UNION
SELECT instructor_name FROM instructors;  -- ERROR!
```

## UNION Operator

**Purpose:** Combine results from multiple queries and remove duplicates.

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
| department |
|------------|
| Chemistry |
| Computer Science |
| English |
| Mathematics |
| Physics |

**Note:** Duplicates like "Computer Science" appear only once.

## UNION ALL Operator

**Purpose:** Combine results and keep all duplicates (faster than UNION).

**Syntax:**
```sql
SELECT columns FROM table1
UNION ALL
SELECT columns FROM table2;
```

**UNION vs. UNION ALL:**

| Aspect | UNION | UNION ALL |
|--------|-------|-----------|
| **Duplicates** | Removed | Kept |
| **Performance** | Slower (sorts to remove dups) | Faster |
| **Use when** | Need unique results | All rows needed or no duplicates exist |

**Example: Count all courses and instructors by department (including duplicates)**

```sql
SELECT department, 'Course' AS type FROM courses
UNION ALL
SELECT department, 'Instructor' AS type FROM instructors
ORDER BY department, type;
```

**Result:**
| department | type |
|------------|------|
| Chemistry | Instructor |
| Computer Science | Course |
| Computer Science | Course |
| Computer Science | Course |
| Computer Science | Instructor |
| English | Course |
| Mathematics | Course |
| Mathematics | Instructor |
| Physics | Course |
| Physics | Instructor |

## INTERSECT Operator

**Purpose:** Return only rows that appear in both queries.

**Syntax:**
```sql
SELECT columns FROM table1
INTERSECT
SELECT columns FROM table2;
```

**Example: Find departments that have both courses and instructors**

```sql
SELECT department FROM courses
INTERSECT
SELECT department FROM instructors
ORDER BY department;
```

**Result:**
| department |
|------------|
| Computer Science |
| Mathematics |
| Physics |

**Explanation:** Chemistry (instructor only) and English (course only) are excluded.

## MINUS Operator

**Purpose:** Return rows from first query that are NOT in second query.

**Note:** Called `EXCEPT` in other databases (PostgreSQL, SQL Server), but Oracle uses `MINUS`.

**Syntax:**
```sql
SELECT columns FROM table1
MINUS
SELECT columns FROM table2;
```

**Example: Find departments with courses but no instructors**

```sql
SELECT department FROM courses
MINUS
SELECT department FROM instructors
ORDER BY department;
```

**Result:**
| department |
|------------|
| English |

**Order matters!** Reversing the queries gives different results:

```sql
-- Departments with instructors but no courses
SELECT department FROM instructors
MINUS
SELECT department FROM courses;
```

**Result:**
| department |
|------------|
| Chemistry |

## ORDER BY with Set Operators

**Rule:** ORDER BY can only appear at the very end and applies to the entire result.

**Example:**

```sql
SELECT first_name AS name, 'Student' AS type
FROM students
WHERE student_id <= 2
UNION
SELECT instructor_name AS name, 'Instructor' AS type
FROM instructors
WHERE instructor_id = 10
ORDER BY name;  -- Orders the combined result
```

**Result:**
| name | type |
|------|------|
| Dr. Johnson | Instructor |
| Jane | Student |
| John | Student |

**Invalid:**
```sql
-- ERROR: ORDER BY in middle
SELECT first_name FROM students ORDER BY first_name
UNION
SELECT instructor_name FROM instructors;
```

## Practical Example: Master Contact List

**Create a unified contact list from students and instructors:**

```sql
SELECT 
    first_name || ' ' || last_name AS full_name,
    email,
    'Student' AS role,
    major AS department
FROM students
UNION ALL
SELECT 
    instructor_name AS full_name,
    instructor_name || '@university.edu' AS email,
    'Instructor' AS role,
    department
FROM instructors
ORDER BY role, full_name;
```

## Performance Considerations

**Tips:**
1. **Use UNION ALL when possible** - Faster (no duplicate removal)
2. **Filter early** - Add WHERE clauses to reduce data before combining
3. **Index appropriately** - Indexes on columns used in WHERE and ORDER BY
4. **Consider alternatives** - Sometimes JOINs or CASE statements are faster

**Performance comparison:**
```sql
-- Slower: UNION removes duplicates
SELECT department FROM courses UNION SELECT department FROM instructors;

-- Faster: UNION ALL keeps all rows
SELECT department FROM courses UNION ALL SELECT department FROM instructors;
```

## Common Mistakes

**Mistake 1: Column count mismatch**
```sql
-- ERROR: Different number of columns
SELECT first_name, last_name FROM students
UNION
SELECT instructor_name FROM instructors;  -- Only one column!

-- Correct: Match column count
SELECT first_name, last_name FROM students
UNION
SELECT instructor_name, department FROM instructors;
```

**Mistake 2: Data type incompatibility**
```sql
-- ERROR: NUMBER vs VARCHAR2
SELECT student_id FROM students
UNION
SELECT instructor_name FROM instructors;

-- Correct: Same data types
SELECT CAST(student_id AS VARCHAR2(50)) FROM students
UNION
SELECT instructor_name FROM instructors;
```

**Mistake 3: ORDER BY in wrong place**
```sql
-- ERROR
SELECT first_name FROM students ORDER BY first_name
UNION
SELECT instructor_name FROM instructors;

-- Correct: ORDER BY at end
SELECT first_name FROM students
UNION
SELECT instructor_name FROM instructors
ORDER BY first_name;
```

