# Advanced SQL - Joining Tables Study Guide

## Key Terms (Glossary)

### Join Fundamentals
- **Join**: An operation that combines rows from two or more tables based on a related column
- **Join Condition**: The criteria used to match rows from different tables (typically using ON clause)
- **Cartesian Product**: The result of combining every row from one table with every row from another table
- **Equijoin**: A join that uses equality comparison in the join condition

### Join Types
- **INNER JOIN**: Returns only the rows that have matching values in both tables
- **LEFT JOIN (LEFT OUTER JOIN)**: Returns all rows from the left table and matching rows from the right table; NULLs for non-matches
- **RIGHT JOIN (RIGHT OUTER JOIN)**: Returns all rows from the right table and matching rows from the left table; NULLs for non-matches
- **FULL OUTER JOIN**: Returns all rows from both tables; NULLs where there are no matches
- **CROSS JOIN**: Returns the Cartesian product of both tables (all possible combinations)
- **SELF JOIN**: A table joined with itself
- **NATURAL JOIN**: Automatically joins tables based on columns with the same name

### Advanced Concepts
- **Foreign Key**: A column that references the primary key of another table, establishing relationships
- **One-to-Many Relationship**: One row in a table can relate to multiple rows in another table
- **Many-to-Many Relationship**: Multiple rows in one table can relate to multiple rows in another table
- **Join Table (Junction Table)**: A table used to implement many-to-many relationships
- **Composite Join**: A join condition that involves multiple columns
- **Outer Query**: The main query that contains a subquery
- **Table Alias**: A temporary name assigned to a table in a query

## Introduction to Joins

### Why Use Joins?

In relational databases, data is organized into multiple related tables to reduce redundancy and improve data integrity (normalization). Joins allow us to:

1. **Retrieve related data** from multiple tables in a single query
2. **Combine information** that is logically connected but physically separated
3. **Maintain data integrity** while avoiding redundancy
4. **Enable complex queries** across normalized database structures

### Basic Join Syntax

**ANSI SQL Standard Syntax:**
```sql
SELECT columns
FROM table1
JOIN table2 ON table1.column = table2.column;
```

**Older Syntax (still supported):**
```sql
SELECT columns
FROM table1, table2
WHERE table1.column = table2.column;
```

**Note:** The ANSI standard syntax (using JOIN keyword) is preferred as it's more readable and separates the join condition from filtering conditions.

## Sample Database Schema

For all examples in this guide, we'll use the following database schema:

### Students Table
| student_id | first_name | last_name | email | major | enrollment_date |
|------------|------------|-----------|-------|-------|-----------------|
| 1 | John | Smith | john.smith@university.edu | Computer Science | 2023-09-01 |
| 2 | Jane | Doe | jane.doe@university.edu | Mathematics | 2023-09-01 |
| 3 | Bob | Wilson | bob.wilson@university.edu | Computer Science | 2024-01-15 |
| 4 | Alice | Brown | alice.brown@university.edu | Physics | 2024-01-15 |
| 5 | Charlie | Davis | charlie.davis@university.edu | NULL | 2024-09-01 |

### Courses Table
| course_id | course_name | department | credits | instructor_id |
|-----------|-------------|------------|---------|---------------|
| CS101 | Introduction to Programming | Computer Science | 3 | 10 |
| CS201 | Data Structures | Computer Science | 4 | 10 |
| MATH101 | Calculus I | Mathematics | 4 | 11 |
| PHYS101 | Physics I | Physics | 4 | 12 |
| CS301 | Database Systems | Computer Science | 3 | 10 |
| ENG101 | English Composition | English | 3 | NULL |

### Enrollments Table
| enrollment_id | student_id | course_id | semester | grade |
|---------------|------------|-----------|----------|-------|
| 101 | 1 | CS101 | Fall 2023 | A |
| 102 | 1 | CS201 | Spring 2024 | B+ |
| 103 | 2 | MATH101 | Fall 2023 | A |
| 104 | 2 | CS101 | Fall 2023 | A- |
| 105 | 3 | CS101 | Spring 2024 | B |
| 106 | 3 | CS201 | Spring 2024 | B+ |
| 107 | 4 | PHYS101 | Spring 2024 | A |
| 108 | 1 | CS301 | Fall 2024 | NULL |

### Instructors Table
| instructor_id | instructor_name | department | hire_date |
|---------------|-----------------|------------|-----------|
| 10 | Dr. Johnson | Computer Science | 2018-08-15 |
| 11 | Dr. Lee | Mathematics | 2019-01-10 |
| 12 | Dr. Martinez | Physics | 2020-09-01 |
| 13 | Dr. Taylor | Chemistry | 2021-06-15 |

## INNER JOIN

### What is an INNER JOIN?

An **INNER JOIN** returns only the rows where there is a match in both tables based on the join condition. If a row in one table doesn't have a corresponding match in the other table, it will be excluded from the result.

### Syntax

```sql
SELECT columns
FROM table1
INNER JOIN table2 ON table1.column = table2.column;
```

**Note:** The `INNER` keyword is optional; `JOIN` alone defaults to `INNER JOIN`.

### Example 1: Basic INNER JOIN

**Query:** Find all student enrollments with student names and course names.

```sql
SELECT s.first_name, 
       s.last_name, 
       c.course_name, 
       e.semester, 
       e.grade
FROM students s
INNER JOIN enrollments e ON s.student_id = e.student_id
INNER JOIN courses c ON e.course_id = c.course_id
ORDER BY s.last_name, s.first_name;
```

**Result:**
| first_name | last_name | course_name | semester | grade |
|------------|-----------|-------------|----------|-------|
| Alice | Brown | Physics I | Spring 2024 | A |
| Jane | Doe | Calculus I | Fall 2023 | A |
| Jane | Doe | Introduction to Programming | Fall 2023 | A- |
| John | Smith | Introduction to Programming | Fall 2023 | A |
| John | Smith | Data Structures | Spring 2024 | B+ |
| John | Smith | Database Systems | Fall 2024 | NULL |
| Bob | Wilson | Introduction to Programming | Spring 2024 | B |
| Bob | Wilson | Data Structures | Spring 2024 | B+ |

**Explanation:** This query returns 8 rows because:
- Student Charlie Davis (ID 5) has no enrollments, so he's excluded
- Course ENG101 has no enrollments, so it's excluded
- Only students who have enrollments AND courses that have enrollments are included

### Example 2: INNER JOIN with WHERE Clause

**Query:** Find Computer Science students enrolled in Computer Science courses.

```sql
SELECT s.first_name, 
       s.last_name, 
       c.course_name,
       e.grade
FROM students s
INNER JOIN enrollments e ON s.student_id = e.student_id
INNER JOIN courses c ON e.course_id = c.course_id
WHERE s.major = 'Computer Science' 
  AND c.department = 'Computer Science'
ORDER BY s.last_name;
```

**Result:**
| first_name | last_name | course_name | grade |
|------------|-----------|-------------|-------|
| John | Smith | Introduction to Programming | A |
| John | Smith | Data Structures | B+ |
| John | Smith | Database Systems | NULL |
| Bob | Wilson | Introduction to Programming | B |
| Bob | Wilson | Data Structures | B+ |

**Explanation:** Only John Smith and Bob Wilson are CS majors enrolled in CS courses. Jane Doe is not a CS major, so her CS101 enrollment is excluded.

### Example 3: INNER JOIN with Aggregate Functions

**Query:** Count the number of enrollments per student.

```sql
SELECT s.student_id,
       s.first_name || ' ' || s.last_name AS full_name,
       COUNT(e.enrollment_id) AS enrollment_count
FROM students s
INNER JOIN enrollments e ON s.student_id = e.student_id
GROUP BY s.student_id, s.first_name, s.last_name
ORDER BY enrollment_count DESC;
```

**Result:**
| student_id | full_name | enrollment_count |
|------------|-----------|------------------|
| 1 | John Smith | 3 |
| 2 | Jane Doe | 2 |
| 3 | Bob Wilson | 2 |
| 4 | Alice Brown | 1 |

**Explanation:** Charlie Davis (ID 5) doesn't appear because he has no enrollments (INNER JOIN excludes him).

## LEFT JOIN (LEFT OUTER JOIN)

### What is a LEFT JOIN?

A **LEFT JOIN** returns all rows from the left (first) table and the matching rows from the right (second) table. If there's no match, NULL values are returned for columns from the right table.

### Syntax

```sql
SELECT columns
FROM table1
LEFT JOIN table2 ON table1.column = table2.column;
```

**Note:** `LEFT JOIN` and `LEFT OUTER JOIN` are equivalent.

### Example 1: Basic LEFT JOIN

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

**Explanation:** All 5 students appear in the result. Charlie Davis has NULLs for enrollment columns because he has no enrollments.

### Example 2: LEFT JOIN to Find Missing Relationships

**Query:** Find students who have NOT enrolled in any courses.

```sql
SELECT s.student_id,
       s.first_name,
       s.last_name,
       s.enrollment_date
FROM students s
LEFT JOIN enrollments e ON s.student_id = e.student_id
WHERE e.enrollment_id IS NULL;
```

**Result:**
| student_id | first_name | last_name | enrollment_date |
|------------|------------|-----------|-----------------|
| 5 | Charlie | Davis | 2024-09-01 |

**Explanation:** This is a common pattern to find "orphan" records. The WHERE clause filters for rows where the right table has NULL, meaning no match was found.

### Example 3: LEFT JOIN with COUNT

**Query:** Count enrollments for all students (including those with zero enrollments).

```sql
SELECT s.student_id,
       s.first_name || ' ' || s.last_name AS full_name,
       COUNT(e.enrollment_id) AS enrollment_count
FROM students s
LEFT JOIN enrollments e ON s.student_id = e.student_id
GROUP BY s.student_id, s.first_name, s.last_name
ORDER BY enrollment_count DESC;
```

**Result:**
| student_id | full_name | enrollment_count |
|------------|-----------|------------------|
| 1 | John Smith | 3 |
| 2 | Jane Doe | 2 |
| 3 | Bob Wilson | 2 |
| 4 | Alice Brown | 1 |
| 5 | Charlie Davis | 0 |

**Explanation:** Unlike the INNER JOIN example, this includes Charlie Davis with a count of 0. Note: `COUNT(e.enrollment_id)` counts non-NULL values, so it correctly returns 0 for Charlie.

### Example 4: Multiple LEFT JOINs

**Query:** List all courses with their instructor names and enrollment counts.

```sql
SELECT c.course_id,
       c.course_name,
       i.instructor_name,
       COUNT(e.enrollment_id) AS enrollment_count
FROM courses c
LEFT JOIN instructors i ON c.instructor_id = i.instructor_id
LEFT JOIN enrollments e ON c.course_id = e.course_id
GROUP BY c.course_id, c.course_name, i.instructor_name
ORDER BY enrollment_count DESC;
```

**Result:**
| course_id | course_name | instructor_name | enrollment_count |
|-----------|-------------|-----------------|------------------|
| CS101 | Introduction to Programming | Dr. Johnson | 3 |
| CS201 | Data Structures | Dr. Johnson | 2 |
| MATH101 | Calculus I | Dr. Lee | 1 |
| PHYS101 | Physics I | Dr. Martinez | 1 |
| CS301 | Database Systems | Dr. Johnson | 1 |
| ENG101 | English Composition | NULL | 0 |

**Explanation:** All courses appear, including ENG101 which has no enrollments and no assigned instructor.

## RIGHT JOIN (RIGHT OUTER JOIN)

### What is a RIGHT JOIN?

A **RIGHT JOIN** returns all rows from the right (second) table and the matching rows from the left (first) table. If there's no match, NULL values are returned for columns from the left table.

### Syntax

```sql
SELECT columns
FROM table1
RIGHT JOIN table2 ON table1.column = table2.column;
```

**Note:** `RIGHT JOIN` and `RIGHT OUTER JOIN` are equivalent. Most databases support RIGHT JOIN, but it's less commonly used than LEFT JOIN.

### Example 1: Basic RIGHT JOIN

**Query:** List all courses and their enrollments (including courses with no enrollments).

```sql
SELECT c.course_id,
       c.course_name,
       e.student_id,
       e.semester,
       e.grade
FROM enrollments e
RIGHT JOIN courses c ON e.course_id = c.course_id
ORDER BY c.course_id, e.semester;
```

**Result:**
| course_id | course_name | student_id | semester | grade |
|-----------|-------------|------------|----------|-------|
| CS101 | Introduction to Programming | 1 | Fall 2023 | A |
| CS101 | Introduction to Programming | 2 | Fall 2023 | A- |
| CS101 | Introduction to Programming | 3 | Spring 2024 | B |
| CS201 | Data Structures | 1 | Spring 2024 | B+ |
| CS201 | Data Structures | 3 | Spring 2024 | B+ |
| CS301 | Database Systems | 1 | Fall 2024 | NULL |
| ENG101 | English Composition | NULL | NULL | NULL |
| MATH101 | Calculus I | 2 | Fall 2023 | A |
| PHYS101 | Physics I | 4 | Spring 2024 | A |

**Explanation:** All 6 courses appear. ENG101 has NULL for enrollment columns because no one is enrolled.

### LEFT JOIN vs RIGHT JOIN

These two queries are equivalent:

```sql
-- Using LEFT JOIN
SELECT s.first_name, e.course_id
FROM students s
LEFT JOIN enrollments e ON s.student_id = e.student_id;

-- Using RIGHT JOIN (reversed table order)
SELECT s.first_name, e.course_id
FROM enrollments e
RIGHT JOIN students s ON e.student_id = s.student_id;
```

**Best Practice:** Most developers prefer LEFT JOIN because it's more intuitive to read left-to-right. RIGHT JOIN can usually be rewritten as LEFT JOIN by reversing the table order.

## FULL OUTER JOIN

### What is a FULL OUTER JOIN?

A **FULL OUTER JOIN** returns all rows from both tables. Where there's a match, it combines the rows. Where there's no match, it includes the row with NULLs for the missing side.

### Syntax

```sql
SELECT columns
FROM table1
FULL OUTER JOIN table2 ON table1.column = table2.column;
```

**Note:** Not all databases support FULL OUTER JOIN (notably MySQL doesn't). PostgreSQL, Oracle, and SQL Server do support it.

### Example 1: FULL OUTER JOIN

**Query:** Show all students and all courses, with enrollments where they exist.

```sql
SELECT s.student_id,
       s.first_name,
       s.last_name,
       c.course_id,
       c.course_name
FROM students s
FULL OUTER JOIN enrollments e ON s.student_id = e.student_id
FULL OUTER JOIN courses c ON e.course_id = c.course_id
ORDER BY s.student_id, c.course_id;
```

**Result (Partial - showing key examples):**
| student_id | first_name | last_name | course_id | course_name |
|------------|------------|-----------|-----------|-------------|
| 1 | John | Smith | CS101 | Introduction to Programming |
| 1 | John | Smith | CS201 | Data Structures |
| 1 | John | Smith | CS301 | Database Systems |
| 2 | Jane | Doe | CS101 | Introduction to Programming |
| 2 | Jane | Doe | MATH101 | Calculus I |
| 3 | Bob | Wilson | CS101 | Introduction to Programming |
| 3 | Bob | Wilson | CS201 | Data Structures |
| 4 | Alice | Brown | PHYS101 | Physics I |
| 5 | Charlie | Davis | NULL | NULL |
| NULL | NULL | NULL | ENG101 | English Composition |

**Explanation:** 
- Charlie Davis appears with NULL course info (no enrollments)
- ENG101 appears with NULL student info (no enrollments)
- All enrollments are shown with both student and course information

### Simulating FULL OUTER JOIN in MySQL

Since MySQL doesn't support FULL OUTER JOIN, you can simulate it using UNION:

```sql
SELECT s.student_id, s.first_name, s.last_name, e.course_id
FROM students s
LEFT JOIN enrollments e ON s.student_id = e.student_id

UNION

SELECT s.student_id, s.first_name, s.last_name, e.course_id
FROM students s
RIGHT JOIN enrollments e ON s.student_id = e.student_id;
```

## CROSS JOIN

### What is a CROSS JOIN?

A **CROSS JOIN** produces the Cartesian product of two tables, meaning every row from the first table is combined with every row from the second table. If table1 has M rows and table2 has N rows, the result will have M × N rows.

### Syntax

```sql
-- Explicit syntax
SELECT columns
FROM table1
CROSS JOIN table2;

-- Implicit syntax (older style)
SELECT columns
FROM table1, table2;
```

### Example 1: Basic CROSS JOIN

**Query:** Generate all possible student-course combinations.

```sql
SELECT s.first_name,
       s.last_name,
       c.course_id,
       c.course_name
FROM students s
CROSS JOIN courses c
ORDER BY s.student_id, c.course_id
LIMIT 12;
```

**Result (First 12 rows):**
| first_name | last_name | course_id | course_name |
|------------|-----------|-----------|-------------|
| John | Smith | CS101 | Introduction to Programming |
| John | Smith | CS201 | Data Structures |
| John | Smith | CS301 | Database Systems |
| John | Smith | ENG101 | English Composition |
| John | Smith | MATH101 | Calculus I |
| John | Smith | PHYS101 | Physics I |
| Jane | Doe | CS101 | Introduction to Programming |
| Jane | Doe | CS201 | Data Structures |
| Jane | Doe | CS301 | Database Systems |
| Jane | Doe | ENG101 | English Composition |
| Jane | Doe | MATH101 | Calculus I |
| Jane | Doe | PHYS101 | Physics I |

**Explanation:** With 5 students and 6 courses, this produces 30 total rows (5 × 6 = 30).

### Example 2: CROSS JOIN with WHERE Clause

**Query:** Find all possible CS student and CS course pairings.

```sql
SELECT s.first_name,
       s.last_name,
       c.course_id,
       c.course_name
FROM students s
CROSS JOIN courses c
WHERE s.major = 'Computer Science' 
  AND c.department = 'Computer Science'
ORDER BY s.last_name, c.course_id;
```

**Result:**
| first_name | last_name | course_id | course_name |
|------------|-----------|-----------|-------------|
| John | Smith | CS101 | Introduction to Programming |
| John | Smith | CS201 | Data Structures |
| John | Smith | CS301 | Database Systems |
| Bob | Wilson | CS101 | Introduction to Programming |
| Bob | Wilson | CS201 | Data Structures |
| Bob | Wilson | CS301 | Database Systems |

**Explanation:** 2 CS students × 3 CS courses = 6 rows.

### When to Use CROSS JOIN

CROSS JOIN is useful for:
1. **Generating test data** or all possible combinations
2. **Creating calendar tables** (combining dates with time slots)
3. **Price calculations** (products × discount rates)
4. **Scheduling** (employees × shifts)

**Warning:** CROSS JOINs can produce very large result sets. Use with caution and always consider if you really need all combinations.

## SELF JOIN

### What is a SELF JOIN?

A **SELF JOIN** is a join where a table is joined with itself. This is useful for comparing rows within the same table or representing hierarchical relationships.

### Syntax

```sql
SELECT columns
FROM table1 t1
JOIN table1 t2 ON t1.column = t2.column;
```

**Note:** Table aliases are REQUIRED for self joins to distinguish between the two instances of the same table.

### Example 1: Find Students in the Same Major

First, let's add sample data to demonstrate:

```sql
-- Additional context: Students table with majors
```

**Query:** Find pairs of students who share the same major.

```sql
SELECT s1.first_name || ' ' || s1.last_name AS student1,
       s2.first_name || ' ' || s2.last_name AS student2,
       s1.major
FROM students s1
JOIN students s2 ON s1.major = s2.major
WHERE s1.student_id < s2.student_id
  AND s1.major IS NOT NULL
ORDER BY s1.major, s1.student_id;
```

**Result:**
| student1 | student2 | major |
|----------|----------|-------|
| John Smith | Bob Wilson | Computer Science |

**Explanation:** 
- John and Bob are both CS majors
- The condition `s1.student_id < s2.student_id` prevents duplicate pairs and self-pairing
- Charlie Davis is excluded because his major is NULL

### Example 2: Hierarchical Data

Let's add an employee table to demonstrate organizational hierarchy:

**Employees Table:**
| employee_id | employee_name | position | manager_id |
|-------------|---------------|----------|------------|
| 1 | Alice Johnson | CEO | NULL |
| 2 | Bob Smith | VP Engineering | 1 |
| 3 | Carol White | VP Sales | 1 |
| 4 | David Brown | Engineer | 2 |
| 5 | Eve Davis | Engineer | 2 |
| 6 | Frank Miller | Sales Rep | 3 |

**Query:** List each employee with their manager's name.

```sql
SELECT e.employee_name AS employee,
       e.position,
       m.employee_name AS manager,
       m.position AS manager_position
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.employee_id
ORDER BY e.employee_id;
```

**Result:**
| employee | position | manager | manager_position |
|----------|----------|---------|------------------|
| Alice Johnson | CEO | NULL | NULL |
| Bob Smith | VP Engineering | Alice Johnson | CEO |
| Carol White | VP Sales | Alice Johnson | CEO |
| David Brown | Engineer | Bob Smith | VP Engineering |
| Eve Davis | Engineer | Bob Smith | VP Engineering |
| Frank Miller | Sales Rep | Carol White | VP Sales |

**Explanation:** 
- Alice Johnson (CEO) has no manager, hence NULL
- Each other employee is matched with their manager
- LEFT JOIN ensures employees without managers still appear

### Example 3: Find Students Who Enrolled in Same Course

**Query:** Find pairs of students enrolled in the same course.

```sql
SELECT s1.first_name || ' ' || s1.last_name AS student1,
       s2.first_name || ' ' || s2.last_name AS student2,
       c.course_name
FROM enrollments e1
JOIN enrollments e2 ON e1.course_id = e2.course_id
JOIN students s1 ON e1.student_id = s1.student_id
JOIN students s2 ON e2.student_id = s2.student_id
JOIN courses c ON e1.course_id = c.course_id
WHERE e1.student_id < e2.student_id
ORDER BY c.course_name, s1.last_name;
```

**Result:**
| student1 | student2 | course_name |
|----------|----------|-------------|
| Jane Doe | John Smith | Introduction to Programming |
| John Smith | Bob Wilson | Introduction to Programming |
| Jane Doe | Bob Wilson | Introduction to Programming |
| John Smith | Bob Wilson | Data Structures |

**Explanation:** 
- CS101 has 3 students, producing 3 pairs (1-2, 1-3, 2-3)
- CS201 has 2 students, producing 1 pair
- Other courses have only 1 student each, so no pairs

## NATURAL JOIN

### What is a NATURAL JOIN?

A **NATURAL JOIN** automatically joins tables based on columns with the same name in both tables. It's a convenience feature but can be risky in production code.

### Syntax

```sql
SELECT columns
FROM table1
NATURAL JOIN table2;
```

### Example

Assuming we have tables with matching column names:

```sql
-- This will join on any columns with matching names
SELECT *
FROM students
NATURAL JOIN enrollments;
```

### ⚠️ Warning: Use with Caution

**Problems with NATURAL JOIN:**
1. **Implicit join conditions** make code harder to understand
2. **Schema changes** (adding a column with a matching name) can break queries
3. **Unexpected results** if multiple columns match
4. **Poor maintainability** - explicit joins are clearer

**Best Practice:** Use explicit JOIN with ON clause instead:

```sql
-- Preferred: Explicit join
SELECT *
FROM students s
JOIN enrollments e ON s.student_id = e.student_id;

-- Avoid: Natural join
SELECT *
FROM students
NATURAL JOIN enrollments;
```

## Composite Joins

### What is a Composite Join?

A **composite join** uses multiple columns in the join condition. This is necessary when the relationship between tables requires more than one column to uniquely identify matching rows.

### Example: Many-to-Many with Composite Key

Let's add a student_courses table with a composite key:

**Student_Courses Table:**
| student_id | course_id | semester | year | section |
|------------|-----------|----------|------|---------|
| 1 | CS101 | Fall | 2023 | A |
| 2 | CS101 | Fall | 2023 | B |
| 3 | CS101 | Spring | 2024 | A |

**Course_Sections Table:**
| course_id | semester | year | section | instructor | room |
|-----------|----------|------|---------|------------|------|
| CS101 | Fall | 2023 | A | Dr. Johnson | Room 101 |
| CS101 | Fall | 2023 | B | Dr. Lee | Room 102 |
| CS101 | Spring | 2024 | A | Dr. Johnson | Room 101 |

**Query:** Join student enrollments with section details.

```sql
SELECT s.first_name,
       s.last_name,
       sc.course_id,
       sc.semester,
       sc.year,
       cs.instructor,
       cs.room
FROM students s
JOIN student_courses sc ON s.student_id = sc.student_id
JOIN course_sections cs ON sc.course_id = cs.course_id
                        AND sc.semester = cs.semester
                        AND sc.year = cs.year
                        AND sc.section = cs.section
ORDER BY sc.semester, sc.year, s.last_name;
```

**Result:**
| first_name | last_name | course_id | semester | year | instructor | room |
|------------|-----------|-----------|----------|------|------------|------|
| Jane | Doe | CS101 | Fall | 2023 | Dr. Lee | Room 102 |
| John | Smith | CS101 | Fall | 2023 | Dr. Johnson | Room 101 |
| Bob | Wilson | CS101 | Spring | 2024 | Dr. Johnson | Room 101 |

**Explanation:** The join requires matching four columns (course_id, semester, year, section) to correctly link students with their specific course sections.

## Join Performance and Best Practices

### Performance Considerations

1. **Index Join Columns**
```sql
-- Create indexes on foreign key columns
CREATE INDEX idx_enrollments_student ON enrollments(student_id);
CREATE INDEX idx_enrollments_course ON enrollments(course_id);
CREATE INDEX idx_courses_instructor ON courses(instructor_id);
```

2. **Join Order Matters**
   - Start with the table that will filter the most rows
   - Use WHERE clauses to reduce data before joining when possible

3. **Avoid Cartesian Products**
```sql
-- Bad: Missing join condition creates Cartesian product
SELECT * 
FROM students, courses;  -- Returns 5 × 6 = 30 rows!

-- Good: Proper join condition
SELECT * 
FROM students s
JOIN enrollments e ON s.student_id = e.student_id
JOIN courses c ON e.course_id = c.course_id;
```

### Best Practices

1. **Always Use Table Aliases**
```sql
-- Good: Clear and readable
SELECT s.first_name, e.grade
FROM students s
JOIN enrollments e ON s.student_id = e.student_id;

-- Avoid: Unclear and verbose
SELECT students.first_name, enrollments.grade
FROM students
JOIN enrollments ON students.student_id = enrollments.student_id;
```

2. **Specify Column Names**
```sql
-- Good: Explicit columns
SELECT s.student_id, s.first_name, e.course_id
FROM students s
JOIN enrollments e ON s.student_id = e.student_id;

-- Avoid: SELECT * in joins
SELECT *
FROM students s
JOIN enrollments e ON s.student_id = e.student_id;
```

3. **Use ANSI JOIN Syntax**
```sql
-- Modern (Preferred)
SELECT s.first_name, e.grade
FROM students s
JOIN enrollments e ON s.student_id = e.student_id
WHERE e.grade = 'A';

-- Old style (Avoid)
SELECT s.first_name, e.grade
FROM students s, enrollments e
WHERE s.student_id = e.student_id
  AND e.grade = 'A';
```

4. **Comment Complex Joins**
```sql
-- Find students with better grades than class average
SELECT s.first_name, e.grade
FROM students s
JOIN enrollments e ON s.student_id = e.student_id
JOIN (
    -- Subquery: Calculate average grade per course
    SELECT course_id, AVG(grade_value) AS avg_grade
    FROM enrollments
    GROUP BY course_id
) avg_grades ON e.course_id = avg_grades.course_id
WHERE e.grade_value > avg_grades.avg_grade;
```

## Common Join Patterns

### Pattern 1: Master-Detail Relationship

**Use Case:** One parent record with multiple child records

```sql
-- Students (master) with their enrollments (detail)
SELECT s.first_name, s.last_name, 
       COUNT(e.enrollment_id) AS total_courses,
       AVG(CASE e.grade
           WHEN 'A' THEN 4.0
           WHEN 'B+' THEN 3.5
           WHEN 'B' THEN 3.0
           WHEN 'A-' THEN 3.7
           ELSE 0
       END) AS gpa
FROM students s
LEFT JOIN enrollments e ON s.student_id = e.student_id
GROUP BY s.student_id, s.first_name, s.last_name;
```

### Pattern 2: Many-to-Many Through Junction Table

**Use Case:** Two tables connected through a third table

```sql
-- Students to Courses through Enrollments
SELECT s.first_name, s.last_name,
       c.course_name, c.department
FROM students s
JOIN enrollments e ON s.student_id = e.student_id
JOIN courses c ON e.course_id = c.course_id
WHERE s.major = c.department;
```

### Pattern 3: Finding Unmatched Records

**Use Case:** Records in one table without corresponding records in another

```sql
-- Courses with no enrollments
SELECT c.course_id, c.course_name, c.department
FROM courses c
LEFT JOIN enrollments e ON c.course_id = e.course_id
WHERE e.enrollment_id IS NULL;
```

**Result:**
| course_id | course_name | department |
|-----------|-------------|------------|
| ENG101 | English Composition | English |

### Pattern 4: Latest Record per Group

**Use Case:** Get the most recent record for each entity

```sql
-- Most recent enrollment for each student
SELECT s.first_name, s.last_name,
       e.course_id, e.semester, e.grade
FROM students s
JOIN enrollments e ON s.student_id = e.student_id
JOIN (
    SELECT student_id, MAX(enrollment_id) AS latest_enrollment
    FROM enrollments
    GROUP BY student_id
) latest ON e.student_id = latest.student_id 
        AND e.enrollment_id = latest.latest_enrollment;
```

## Summary

### Key Takeaways:

1. **INNER JOIN:**
   - Returns only matching rows from both tables
   - Most common join type
   - Use when you need data that exists in both tables

2. **LEFT JOIN:**
   - Returns all rows from left table, matches from right
   - Use to include all records from the primary table
   - Great for finding missing relationships

3. **RIGHT JOIN:**
   - Returns all rows from right table, matches from left
   - Less common, usually rewritten as LEFT JOIN
   - Mirror of LEFT JOIN

4. **FULL OUTER JOIN:**
   - Returns all rows from both tables
   - Use when you need everything regardless of matches
   - Not supported in all databases (e.g., MySQL)

5. **CROSS JOIN:**
   - Cartesian product of both tables
   - Use sparingly - can produce huge result sets
   - Useful for generating combinations

6. **SELF JOIN:**
   - Table joined with itself
   - Requires table aliases
   - Great for hierarchical data or comparing rows within same table

7. **Best Practices:**
   - Always use table aliases for readability
   - Index join columns for performance
   - Use explicit ANSI JOIN syntax
   - Avoid SELECT * in joins
   - Test join results for correctness
   - Document complex join logic

### Join Selection Guide:

| Need | Use |
|------|-----|
| Only matched records | INNER JOIN |
| All from left + matches from right | LEFT JOIN |
| All from right + matches from left | RIGHT JOIN |
| Everything from both tables | FULL OUTER JOIN |
| All possible combinations | CROSS JOIN |
| Compare rows in same table | SELF JOIN |
| Hierarchical relationships | SELF JOIN |

### SQL Command Quick Reference:

```sql
-- INNER JOIN
SELECT * FROM table1 
JOIN table2 ON table1.id = table2.id;

-- LEFT JOIN
SELECT * FROM table1 
LEFT JOIN table2 ON table1.id = table2.id;

-- RIGHT JOIN
SELECT * FROM table1 
RIGHT JOIN table2 ON table1.id = table2.id;

-- FULL OUTER JOIN
SELECT * FROM table1 
FULL OUTER JOIN table2 ON table1.id = table2.id;

-- CROSS JOIN
SELECT * FROM table1 
CROSS JOIN table2;

-- SELF JOIN
SELECT * FROM table1 t1
JOIN table1 t2 ON t1.id = t2.parent_id;

-- Multiple Joins
SELECT * FROM table1 t1
JOIN table2 t2 ON t1.id = t2.t1_id
JOIN table3 t3 ON t2.id = t3.t2_id;

-- Composite Join
SELECT * FROM table1 t1
JOIN table2 t2 ON t1.col1 = t2.col1 
              AND t1.col2 = t2.col2;
```

This foundation in SQL joins provides the essential skills for combining data from multiple related tables. Practice these concepts with real databases to master complex queries and data relationships!

