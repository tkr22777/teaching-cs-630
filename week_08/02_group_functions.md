# SQL Group Functions (Aggregate Functions)

## Overview

**Group functions** (also called aggregate functions) operate on multiple rows and return a single result. They're essential for statistical analysis, reporting, and data summarization.

## Key Terms

**Group Function** (or Aggregate Function): Operates on multiple rows, returns a single value.

**GROUP BY**: Divides rows into groups for aggregation.

**HAVING**: Filters groups after aggregation.

## Common Group Functions

| Function | Description | NULL Handling |
|----------|-------------|---------------|
| **COUNT(*)** | Count all rows | Includes NULLs |
| **COUNT(column)** | Count non-NULL values | Ignores NULLs |
| **SUM(column)** | Total of values | Ignores NULLs |
| **AVG(column)** | Average of values | Ignores NULLs |
| **MAX(column)** | Maximum value | Ignores NULLs |
| **MIN(column)** | Minimum value | Ignores NULLs |

## COUNT Function

**Purpose:** Count rows or non-NULL values.

**Syntax:**
- `COUNT(*)` - Count all rows (including NULLs)
- `COUNT(column)` - Count non-NULL values in column
- `COUNT(DISTINCT column)` - Count unique non-NULL values

**Example:**

```sql
SELECT 
    COUNT(*) AS total_students,
    COUNT(major) AS students_with_major,
    COUNT(DISTINCT major) AS unique_majors
FROM students;
```

**Result:**
| total_students | students_with_major | unique_majors |
|----------------|---------------------|---------------|
| 5 | 4 | 3 |

---

## SUM Function

**Purpose:** Calculate total of numeric values.

**Syntax:** `SUM(column)`

**Example:**

```sql
SELECT SUM(credits) AS total_credits_offered
FROM courses;
```

**Result:**
| total_credits_offered |
|----------------------|
| 21 |

---

## AVG Function

**Purpose:** Calculate average of numeric values.

**Syntax:** `AVG(column)`

**Example:**

```sql
SELECT AVG(gpa) AS overall_avg_gpa,
       ROUND(AVG(gpa), 2) AS rounded_avg
FROM students
WHERE gpa IS NOT NULL;
```

**Result:**
| overall_avg_gpa | rounded_avg |
|-----------------|-------------|
| 3.48 | 3.48 |

---

## MAX and MIN Functions

**Purpose:** Find maximum or minimum value.

**Syntax:** `MAX(column)`, `MIN(column)`

**Works with:** Numbers, dates, strings (alphabetical order)

**Example:**

```sql
SELECT 
    MAX(gpa) AS highest_gpa,
    MIN(gpa) AS lowest_gpa,
    MAX(enrollment_date) AS most_recent_enrollment,
    MIN(last_name) AS first_alphabetically
FROM students
WHERE gpa IS NOT NULL;
```

**Result:**
| highest_gpa | lowest_gpa | most_recent_enrollment | first_alphabetically |
|-------------|------------|------------------------|----------------------|
| 3.9 | 2.8 | 2024-09-01 | Brown |

---

## GROUP BY Clause

Divides rows into groups and applies aggregate functions to each group.

**Rule:** Every column in SELECT (except aggregates) must be in GROUP BY.

**Example: Count students by major**

```sql
SELECT 
    major,
    COUNT(*) AS student_count,
    AVG(gpa) AS avg_gpa
FROM students
WHERE major IS NOT NULL
GROUP BY major
ORDER BY student_count DESC;
```

**Result:**
| major | student_count | avg_gpa |
|-------|---------------|---------|
| Computer Science | 2 | 3.5 |
| Mathematics | 1 | 3.9 |
| Physics | 1 | 3.7 |

---

## HAVING Clause

Filters groups after aggregation (WHERE filters rows before).

**Example: Find majors with average GPA above 3.5**

```sql
SELECT 
    major,
    COUNT(*) AS student_count,
    AVG(gpa) AS avg_gpa
FROM students
WHERE major IS NOT NULL
GROUP BY major
HAVING AVG(gpa) > 3.5
ORDER BY avg_gpa DESC;
```

**Result:**
| major | student_count | avg_gpa |
|-------|---------------|---------|
| Mathematics | 1 | 3.9 |
| Physics | 1 | 3.7 |

---

## WHERE vs. HAVING

| Aspect | WHERE | HAVING |
|--------|-------|--------|
| **Filters** | Rows (before grouping) | Groups (after grouping) |
| **Can use aggregates** | No | Yes |

**Explanation:**
- WHERE removes ENG101 (NULL instructor)
- GROUP BY creates groups
- HAVING keeps only departments with 2+ courses

## Advanced Patterns

### Pattern 1: Conditional Aggregation

### Pattern 1: Multiple Grouping Columns

**Group by semester and course:**

```sql
SELECT 
    semester,
    c.department,
    COUNT(*) AS enrollments,
    AVG(e.grade_points) AS avg_grade
FROM enrollments e
JOIN courses c ON e.course_id = c.course_id
WHERE e.grade IS NOT NULL
GROUP BY semester, c.department
ORDER BY semester, c.department;
```

**Result:**
| semester | department | enrollments | avg_grade |
|----------|------------|-------------|-----------|
| Fall 2023 | Computer Science | 2 | 3.85 |
| Fall 2023 | Mathematics | 1 | 4.0 |
| Spring 2024 | Computer Science | 2 | 3.15 |
| Spring 2024 | Physics | 1 | 4.0 |

---

## Common Mistakes

**Forgetting GROUP BY:**
```sql
-- ERROR
SELECT major, COUNT(*) FROM students;

-- Correct
SELECT major, COUNT(*) FROM students GROUP BY major;
```

**Using WHERE with aggregates:**
```sql
-- ERROR
WHERE AVG(gpa) > 3.5

-- Correct
HAVING AVG(gpa) > 3.5
```

