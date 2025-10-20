# LEFT JOIN (LEFT OUTER JOIN)

## Overview

A **LEFT JOIN** returns all rows from the left (first) table and the matching rows from the right (second) table. When there's no match, NULL values are returned for columns from the right table.

LEFT JOIN is essential when you need to preserve all records from your primary table, even if they don't have related records.

## Syntax

```sql
SELECT columns
FROM table1
LEFT JOIN table2 ON table1.column = table2.column;
```

**Alternative (equivalent):**
```sql
SELECT columns
FROM table1
LEFT OUTER JOIN table2 ON table1.column = table2.column;
```

**Note:** `LEFT JOIN` and `LEFT OUTER JOIN` are identical.

## How LEFT JOIN Works

**Visual Representation:**

```
Table A          Table B          Result (A LEFT JOIN B)
┌────┬────┐      ┌────┬────┐      ┌────┬────┬──────┐
│ ID │Val │      │ ID │Val │      │ ID │ValA│ ValB │
├────┼────┤      ├────┼────┤      ├────┼────┼──────┤
│  1 │ A1 │◄────►│  1 │ B1 │─────►│  1 │ A1 │  B1  │
│  2 │ A2 │◄────►│  2 │ B2 │─────►│  2 │ A2 │  B2  │
│  3 │ A3 │      │  4 │ B4 │   ┌─►│  3 │ A3 │ NULL │
└────┴────┘      └────┴────┘   │  └────┴────┴──────┘
   ↑                            │  All left rows kept,
Row 3 kept ─────────────────────┘  NULL when no match
```

**Key Point:** ALL rows from the left table appear in the result, with NULLs for unmatched right-side columns.

## Basic LEFT JOIN Examples

### Example 1: All Students with Their Enrollments

**Query:** List all students and their enrollments (including students with no enrollments).

```sql
SELECT s.student_id,
       s.first_name,
       s.last_name,
       e.course_id,
       e.semester,
       e.grade
FROM students s
LEFT JOIN enrollments e ON s.student_id = e.student_id
ORDER BY s.student_id, e.semester;
```

**Result:**
| student_id | first_name | last_name | course_id | semester | grade |
|------------|------------|-----------|-----------|----------|-------|
| 1 | John | Smith | CS101 | Fall 2023 | A |
| 1 | John | Smith | CS301 | Fall 2024 | NULL |
| 1 | John | Smith | CS201 | Spring 2024 | B+ |
| 2 | Jane | Doe | CS101 | Fall 2023 | A- |
| 2 | Jane | Doe | MATH101 | Fall 2023 | A |
| 3 | Bob | Wilson | CS101 | Spring 2024 | B |
| 3 | Bob | Wilson | CS201 | Spring 2024 | B+ |
| 4 | Alice | Brown | PHYS101 | Spring 2024 | A |
| 5 | Charlie | Davis | NULL | NULL | NULL |

**Explanation:**
- All 5 students appear in the result
- Charlie Davis (ID 5) has NULL for enrollment columns because he has no enrollments
- This is the key difference from INNER JOIN, which would exclude Charlie

### Example 2: All Courses with Enrollment Counts

**Query:** Show all courses with their enrollment counts.

```sql
SELECT c.course_id,
       c.course_name,
       c.department,
       COUNT(e.enrollment_id) AS enrollment_count
FROM courses c
LEFT JOIN enrollments e ON c.course_id = e.course_id
GROUP BY c.course_id, c.course_name, c.department
ORDER BY enrollment_count DESC, c.course_id;
```

**Result:**
| course_id | course_name | department | enrollment_count |
|-----------|-------------|------------|------------------|
| CS101 | Introduction to Programming | Computer Science | 3 |
| CS201 | Data Structures | Computer Science | 2 |
| CS301 | Database Systems | Computer Science | 1 |
| MATH101 | Calculus I | Mathematics | 1 |
| PHYS101 | Physics I | Physics | 1 |
| ENG101 | English Composition | English | 0 |

**Explanation:**
- All 6 courses appear, including ENG101 with 0 enrollments
- COUNT(e.enrollment_id) counts non-NULL values, so it correctly returns 0 for ENG101
- Important: COUNT(*) would return 1 for ENG101 (incorrect)

## Finding Missing Relationships

One of the most powerful uses of LEFT JOIN is finding "orphan" records.

### Example 3: Find Students with No Enrollments

**Query:** Identify students who haven't enrolled in any courses.

```sql
SELECT s.student_id,
       s.first_name,
       s.last_name,
       s.email,
       s.enrollment_date
FROM students s
LEFT JOIN enrollments e ON s.student_id = e.student_id
WHERE e.enrollment_id IS NULL;
```

**Result:**
| student_id | first_name | last_name | email | enrollment_date |
|------------|------------|-----------|-------|-----------------|
| 5 | Charlie | Davis | charlie.davis@university.edu | 2024-09-01 |

**Explanation:**
- LEFT JOIN ensures all students appear
- WHERE e.enrollment_id IS NULL filters for students with no enrollments
- This pattern is extremely useful for finding missing relationships

**Common Mistake:**
```sql
-- WRONG: This won't work as expected
WHERE e.course_id = NULL  -- NULL comparisons don't work with =
WHERE e.course_id != NULL -- Still wrong

-- CORRECT:
WHERE e.enrollment_id IS NULL
```

### Example 4: Find Courses with No Enrollments

**Query:** Which courses have no students enrolled?

```sql
SELECT c.course_id,
       c.course_name,
       c.department,
       c.credits
FROM courses c
LEFT JOIN enrollments e ON c.course_id = e.course_id
WHERE e.enrollment_id IS NULL;
```

**Result:**
| course_id | course_name | department | credits |
|-----------|-------------|------------|---------|
| ENG101 | English Composition | English | 3 |

**Explanation:**
- Shows courses that exist but have no student enrollments
- Useful for identifying unpopular courses or data integrity issues

## Multiple LEFT JOINs

### Example 5: Courses with Instructors and Enrollments

**Query:** Show all courses with their instructors and enrollment counts.

```sql
SELECT c.course_id,
       c.course_name,
       i.instructor_name,
       i.department AS instructor_dept,
       COUNT(e.enrollment_id) AS enrollment_count
FROM courses c
LEFT JOIN instructors i ON c.instructor_id = i.instructor_id
LEFT JOIN enrollments e ON c.course_id = e.course_id
GROUP BY c.course_id, c.course_name, i.instructor_name, i.department
ORDER BY enrollment_count DESC;
```

**Result:**
| course_id | course_name | instructor_name | instructor_dept | enrollment_count |
|-----------|-------------|-----------------|-----------------|------------------|
| CS101 | Introduction to Programming | Dr. Johnson | Computer Science | 3 |
| CS201 | Data Structures | Dr. Johnson | Computer Science | 2 |
| CS301 | Database Systems | Dr. Johnson | Computer Science | 1 |
| MATH101 | Calculus I | Dr. Lee | Mathematics | 1 |
| PHYS101 | Physics I | Dr. Martinez | Physics | 1 |
| ENG101 | English Composition | NULL | NULL | 0 |

**Explanation:**
- All courses appear (LEFT JOIN preserves courses)
- ENG101 has NULL for instructor (no assigned instructor)
- ENG101 has 0 enrollments
- Multiple LEFT JOINs can be chained together

### Example 6: All Students with Course and Instructor Info

**Query:** Complete student enrollment report.

```sql
SELECT s.student_id,
       s.first_name || ' ' || s.last_name AS student_name,
       s.major,
       c.course_id,
       c.course_name,
       i.instructor_name,
       e.semester,
       e.grade
FROM students s
LEFT JOIN enrollments e ON s.student_id = e.student_id
LEFT JOIN courses c ON e.course_id = c.course_id
LEFT JOIN instructors i ON c.instructor_id = i.instructor_id
ORDER BY s.student_id, e.semester;
```

**Result:**
| student_id | student_name | major | course_id | course_name | instructor_name | semester | grade |
|------------|--------------|-------|-----------|-------------|-----------------|----------|-------|
| 1 | John Smith | Computer Science | CS101 | Introduction to Programming | Dr. Johnson | Fall 2023 | A |
| 1 | John Smith | Computer Science | CS301 | Database Systems | Dr. Johnson | Fall 2024 | NULL |
| 1 | John Smith | Computer Science | CS201 | Data Structures | Dr. Johnson | Spring 2024 | B+ |
| 2 | Jane Doe | Mathematics | CS101 | Introduction to Programming | Dr. Johnson | Fall 2023 | A- |
| 2 | Jane Doe | Mathematics | MATH101 | Calculus I | Dr. Lee | Fall 2023 | A |
| 3 | Bob Wilson | Computer Science | CS101 | Introduction to Programming | Dr. Johnson | Spring 2024 | B |
| 3 | Bob Wilson | Computer Science | CS201 | Data Structures | Dr. Johnson | Spring 2024 | B+ |
| 4 | Alice Brown | Physics | PHYS101 | Physics I | Dr. Martinez | Spring 2024 | A |
| 5 | Charlie Davis | NULL | NULL | NULL | NULL | NULL | NULL |

**Explanation:**
- ALL students appear, including Charlie Davis with no enrollments
- Complete information chain: student → enrollment → course → instructor
- NULLs appear where relationships don't exist

## LEFT JOIN with Aggregation

### Example 7: Student Enrollment Statistics

**Query:** Get enrollment counts for all students (including those with zero enrollments).

```sql
SELECT s.student_id,
       s.first_name,
       s.last_name,
       s.major,
       COUNT(e.enrollment_id) AS course_count,
       COALESCE(SUM(c.credits), 0) AS total_credits
FROM students s
LEFT JOIN enrollments e ON s.student_id = e.student_id
LEFT JOIN courses c ON e.course_id = c.course_id
GROUP BY s.student_id, s.first_name, s.last_name, s.major
ORDER BY course_count DESC;
```

**Result:**
| student_id | first_name | last_name | major | course_count | total_credits |
|------------|------------|-----------|-------|--------------|---------------|
| 1 | John | Smith | Computer Science | 3 | 10 |
| 2 | Jane | Doe | Mathematics | 2 | 7 |
| 3 | Bob | Wilson | Computer Science | 2 | 7 |
| 4 | Alice | Brown | Physics | 1 | 4 |
| 5 | Charlie | Davis | NULL | 0 | 0 |

**Explanation:**
- ALL students appear, including Charlie with 0 courses
- COUNT(e.enrollment_id) correctly returns 0 for Charlie (counts non-NULL values)
- COALESCE converts NULL to 0 for total_credits
- This is the key difference from INNER JOIN: complete roster

###

**Query:** Calculate average grade for all students (excluding those with no grades).

```sql
SELECT s.student_id,
       s.first_name,
       s.last_name,
       COUNT(e.enrollment_id) AS courses_taken,
       COUNT(CASE WHEN e.grade IS NOT NULL THEN 1 END) AS graded_courses,
       ROUND(AVG(
           CASE e.grade
               WHEN 'A' THEN 4.0
               WHEN 'A-' THEN 3.7
               WHEN 'B+' THEN 3.3
               WHEN 'B' THEN 3.0
               WHEN 'B-' THEN 2.7
               ELSE NULL
           END
       ), 2) AS gpa
FROM students s
LEFT JOIN enrollments e ON s.student_id = e.student_id
GROUP BY s.student_id, s.first_name, s.last_name
ORDER BY gpa DESC NULLS LAST;
```

**Result:**
| student_id | first_name | last_name | courses_taken | graded_courses | gpa |
|------------|------------|-----------|---------------|----------------|-----|
| 4 | Alice | Brown | 1 | 1 | 4.00 |
| 2 | Jane | Doe | 2 | 2 | 3.85 |
| 1 | John | Smith | 3 | 2 | 3.65 |
| 3 | Bob | Wilson | 2 | 2 | 3.15 |
| 5 | Charlie | Davis | 0 | 0 | NULL |

**Explanation:**
- All students appear, including Charlie (GPA is NULL for no courses)
- Distinguishes between courses_taken and graded_courses
- John has 3 enrollments but only 2 graded (CS301 grade is NULL)
- NULLS LAST ensures students with no GPA appear at the end

## LEFT JOIN vs INNER JOIN Comparison

### Example 9: Side-by-Side Comparison

**INNER JOIN Query:**
```sql
SELECT s.first_name, s.last_name, COUNT(e.enrollment_id) AS courses
FROM students s
INNER JOIN enrollments e ON s.student_id = e.student_id
GROUP BY s.student_id, s.first_name, s.last_name;
```

**INNER JOIN Result:**
| first_name | last_name | courses |
|------------|-----------|---------|
| John | Smith | 3 |
| Jane | Doe | 2 |
| Bob | Wilson | 2 |
| Alice | Brown | 1 |

**LEFT JOIN Query:**
```sql
SELECT s.first_name, s.last_name, COUNT(e.enrollment_id) AS courses
FROM students s
LEFT JOIN enrollments e ON s.student_id = e.student_id
GROUP BY s.student_id, s.first_name, s.last_name;
```

**LEFT JOIN Result:**
| first_name | last_name | courses |
|------------|-----------|---------|
| John | Smith | 3 |
| Jane | Doe | 2 |
| Bob | Wilson | 2 |
| Alice | Brown | 1 |
| Charlie | Davis | 0 |

**Key Difference:** Charlie Davis appears with LEFT JOIN but not with INNER JOIN.

## When to Use LEFT JOIN

### ✅ Use LEFT JOIN When:

1. **You need all records from the left table**
   - Complete student roster
   - All products (including those never sold)
   - All customers (including those with no orders)

2. **Finding missing relationships is important**
   - Students not enrolled
   - Products never ordered
   - Customers with no purchases

3. **Reporting requires complete lists**
   - All employees (including those with no sales)
   - All inventory items (including out-of-stock)

4. **Aggregations need to include zeros**
   - Course enrollment counts (including 0)
   - Sales by product (including products with $0 sales)

### ❌ Avoid LEFT JOIN When:

1. **You only need matched data**
   - Use INNER JOIN for better performance
   - Smaller result sets

2. **NULL values complicate logic**
   - If you don't need unmatched rows, use INNER JOIN

## Common Patterns

### Pattern 1: Roster with Optional Details

```sql
-- All students with their enrollment info (if any)
SELECT s.first_name, s.last_name, c.course_name, e.grade
FROM students s
LEFT JOIN enrollments e ON s.student_id = e.student_id
LEFT JOIN courses c ON e.course_id = c.course_id;
```

### Pattern 2: Find Missing Relationships

```sql
-- Students who haven't enrolled
SELECT s.*
FROM students s
LEFT JOIN enrollments e ON s.student_id = e.student_id
WHERE e.enrollment_id IS NULL;
```

### Pattern 3: Aggregation with Complete List

```sql
-- All students with course counts (including 0)
SELECT s.first_name, s.last_name, COUNT(e.enrollment_id) AS courses
FROM students s
LEFT JOIN enrollments e ON s.student_id = e.student_id
GROUP BY s.student_id, s.first_name, s.last_name;
```

### Pattern 4: COALESCE for Default Values

```sql
-- Replace NULL with meaningful defaults
SELECT s.first_name,
       s.last_name,
       COALESCE(c.course_name, 'Not Enrolled') AS course,
       COALESCE(e.grade, 'N/A') AS grade
FROM students s
LEFT JOIN enrollments e ON s.student_id = e.student_id
LEFT JOIN courses c ON e.course_id = c.course_id;
```

## Important Notes

### NULL Handling

```sql
-- Count NULL-safe
COUNT(e.enrollment_id)  -- Counts only non-NULL values ✓

-- Sum NULL-safe
COALESCE(SUM(c.credits), 0)  -- Converts NULL to 0 ✓

-- Filtering NULL
WHERE e.enrollment_id IS NULL  -- Find unmatched rows ✓
WHERE e.enrollment_id = NULL   -- WRONG! Always false ✗
```

### Performance Considerations

- LEFT JOIN can be slower than INNER JOIN (more rows to process)
- Index foreign key columns for better performance
- Filter early with WHERE when possible
- Consider if you really need all left-table rows

