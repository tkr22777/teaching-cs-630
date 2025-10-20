# INNER JOIN

## Overview

An **INNER JOIN** returns only the rows where there is a match in both tables based on the join condition. Rows without matches in either table are excluded from the result.

INNER JOIN is the most commonly used join type because it returns only the data that has complete relationships.

## Syntax

```sql
SELECT columns
FROM table1
INNER JOIN table2 ON table1.column = table2.column;
```

**Alternative (INNER is optional):**
```sql
SELECT columns
FROM table1
JOIN table2 ON table1.column = table2.column;
```

**Note:** `JOIN` without any modifier defaults to `INNER JOIN`.

## How INNER JOIN Works

**Visual Representation:**

```
Table A          Table B          Result (A INNER JOIN B)
┌────┬────┐      ┌────┬────┐      ┌────┬────┬────┐
│ ID │Val │      │ ID │Val │      │ ID │ValA│ValB│
├────┼────┤      ├────┼────┤      ├────┼────┼────┤
│  1 │ A1 │◄────►│  1 │ B1 │─────►│  1 │ A1 │ B1 │
│  2 │ A2 │◄────►│  2 │ B2 │─────►│  2 │ A2 │ B2 │
│  3 │ A3 │      │  4 │ B4 │      └────┴────┴────┘
└────┴────┘      └────┴────┘      
   ↑                 ↑             Only matching rows
Row 3 excluded   Row 4 excluded    (IDs 1 and 2)
```

**Key Point:** Only rows with matching join keys appear in the result.

## Basic INNER JOIN Examples

### Example 1: Simple Two-Table Join

**Query:** Get student names with their enrollments.

```sql
SELECT s.student_id,
       s.first_name,
       s.last_name,
       e.course_id,
       e.semester,
       e.grade
FROM students s
INNER JOIN enrollments e ON s.student_id = e.student_id
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

**Explanation:**
- Returns 8 rows (8 enrollments with matching students)
- Student Charlie Davis (ID 5) is excluded because he has no enrollments
- Each enrollment is paired with its corresponding student information

### Example 2: Three-Table Join

**Query:** Get complete enrollment information with student and course names.

```sql
SELECT s.first_name,
       s.last_name,
       c.course_name,
       c.department,
       e.semester,
       e.grade
FROM students s
INNER JOIN enrollments e ON s.student_id = e.student_id
INNER JOIN courses c ON e.course_id = c.course_id
ORDER BY s.last_name, s.first_name, e.semester;
```

**Result:**
| first_name | last_name | course_name | department | semester | grade |
|------------|-----------|-------------|------------|----------|-------|
| Alice | Brown | Physics I | Physics | Spring 2024 | A |
| Jane | Doe | Calculus I | Mathematics | Fall 2023 | A |
| Jane | Doe | Introduction to Programming | Computer Science | Fall 2023 | A- |
| John | Smith | Introduction to Programming | Computer Science | Fall 2023 | A |
| John | Smith | Database Systems | Computer Science | Fall 2024 | NULL |
| John | Smith | Data Structures | Computer Science | Spring 2024 | B+ |
| Bob | Wilson | Introduction to Programming | Computer Science | Spring 2024 | B |
| Bob | Wilson | Data Structures | Computer Science | Spring 2024 | B+ |

**Explanation:**
- Joins three tables: students → enrollments → courses
- Only returns enrollments that have both a valid student AND a valid course
- Creates a complete view of who is taking what

### Example 3: Four-Table Join

**Query:** Include instructor information.

```sql
SELECT s.first_name || ' ' || s.last_name AS student,
       c.course_id,
       c.course_name,
       i.instructor_name,
       e.semester,
       e.grade
FROM students s
INNER JOIN enrollments e ON s.student_id = e.student_id
INNER JOIN courses c ON e.course_id = c.course_id
INNER JOIN instructors i ON c.instructor_id = i.instructor_id
ORDER BY i.instructor_name, s.last_name;
```

**Result:**
| student | course_id | course_name | instructor_name | semester | grade |
|---------|-----------|-------------|-----------------|----------|-------|
| Jane Doe | CS101 | Introduction to Programming | Dr. Johnson | Fall 2023 | A- |
| John Smith | CS101 | Introduction to Programming | Dr. Johnson | Fall 2023 | A |
| John Smith | CS201 | Data Structures | Dr. Johnson | Spring 2024 | B+ |
| John Smith | CS301 | Database Systems | Dr. Johnson | Fall 2024 | NULL |
| Bob Wilson | CS101 | Introduction to Programming | Dr. Johnson | Spring 2024 | B |
| Bob Wilson | CS201 | Data Structures | Dr. Johnson | Spring 2024 | B+ |
| Jane Doe | MATH101 | Calculus I | Dr. Lee | Fall 2023 | A |
| Alice Brown | PHYS101 | Physics I | Dr. Martinez | Spring 2024 | A |

**Explanation:**
- Joins four tables: students → enrollments → courses → instructors
- Only courses with assigned instructors appear
- ENG101 enrollments would be excluded (no instructor assigned)

## INNER JOIN with WHERE Clause

### Example 4: Filter After Joining

**Query:** Find Computer Science students in Computer Science courses.

```sql
SELECT s.first_name,
       s.last_name,
       c.course_id,
       c.course_name,
       e.grade
FROM students s
INNER JOIN enrollments e ON s.student_id = e.student_id
INNER JOIN courses c ON e.course_id = c.course_id
WHERE s.major = 'Computer Science'
  AND c.department = 'Computer Science'
ORDER BY s.last_name, c.course_id;
```

**Result:**
| first_name | last_name | course_id | course_name | grade |
|------------|-----------|-----------|-------------|-------|
| John | Smith | CS101 | Introduction to Programming | A |
| John | Smith | CS201 | Data Structures | B+ |
| John | Smith | CS301 | Database Systems | NULL |
| Bob | Wilson | CS101 | Introduction to Programming | B |
| Bob | Wilson | CS201 | Data Structures | B+ |

**Explanation:**
- Joins first, then filters
- Jane Doe's CS101 enrollment excluded (she's a Math major)
- Only CS majors enrolled in CS courses appear

## INNER JOIN with Aggregate Functions

### Example 6: Count Enrollments Per Student

**Query:** How many courses is each student taking?

```sql
SELECT s.student_id,
       s.first_name,
       s.last_name,
       COUNT(e.enrollment_id) AS course_count
FROM students s
INNER JOIN enrollments e ON s.student_id = e.student_id
GROUP BY s.student_id, s.first_name, s.last_name
ORDER BY course_count DESC;
```

**Result:**
| student_id | first_name | last_name | course_count |
|------------|------------|-----------|--------------|
| 1 | John | Smith | 3 |
| 2 | Jane | Doe | 2 |
| 3 | Bob | Wilson | 2 |
| 4 | Alice | Brown | 1 |

**Explanation:**
- Charlie Davis (ID 5) doesn't appear because INNER JOIN excludes students with no enrollments
- Counts only students who have at least one enrollment

## INNER JOIN with Calculations

## When to Use INNER JOIN

### ✅ Use INNER JOIN When:

1. **You need only complete relationships**
   - Orders with valid customers
   - Enrollments with both students and courses
   - Products with categories

2. **Excluding orphan records is acceptable**
   - Students without enrollments can be omitted
   - Courses with no enrollments can be omitted

3. **Performance is critical**
   - INNER JOIN is typically faster than OUTER JOINs
   - Smaller result sets

4. **Business logic requires both sides**
   - Sales report (only orders with products)
   - Class roster (only enrolled students)

### ❌ Avoid INNER JOIN When:

1. **You need all records from one table**
   - All students (including those not enrolled)
   - All products (including those never ordered)
   - Use LEFT JOIN instead

2. **Missing relationships are important**
   - Finding students without enrollments
   - Identifying products never sold
   - Use LEFT JOIN with WHERE IS NULL

## Common Patterns

### Pattern 1: Master-Detail Listing

```sql
-- List all students with their course count
SELECT s.student_id,
       s.first_name,
       s.last_name,
       COUNT(e.enrollment_id) AS enrollments
FROM students s
INNER JOIN enrollments e ON s.student_id = e.student_id
GROUP BY s.student_id, s.first_name, s.last_name;
```

### Pattern 2: Many-to-Many Through Junction Table

```sql
-- Students to Courses through Enrollments
SELECT s.first_name,
       s.last_name,
       c.course_name,
       c.credits
FROM students s
INNER JOIN enrollments e ON s.student_id = e.student_id
INNER JOIN courses c ON e.course_id = c.course_id;
```

### Pattern 3: Filtering Related Data

```sql
-- Students enrolled in high-credit courses
SELECT DISTINCT s.first_name,
                s.last_name,
                s.major
FROM students s
INNER JOIN enrollments e ON s.student_id = e.student_id
INNER JOIN courses c ON e.course_id = c.course_id
WHERE c.credits >= 4;
```

