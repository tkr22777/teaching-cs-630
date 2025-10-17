# SELF JOIN and NATURAL JOIN

## SELF JOIN

### Overview

A **SELF JOIN** is a join where a table is joined with itself. This powerful technique is used to compare rows within the same table or to represent hierarchical relationships.

**Key Requirement:** Table aliases are MANDATORY for self joins to distinguish between the two instances of the same table.

### Syntax

```sql
SELECT columns
FROM table1 t1
JOIN table1 t2 ON t1.column = t2.column;
```

**Note:** You can use any join type (INNER, LEFT, RIGHT) with self joins.

### How SELF JOIN Works

**Conceptual Representation:**

```
Original Table:              Self Join (Conceptual):
┌────┬────┬─────┐            ┌─────────┬─────────┐
│ ID │Name│MgrID│            │Table t1 │Table t2 │
├────┼────┼─────┤            ├─────────┼─────────┤
│  1 │Ann │ NULL│            │ Emp Row │ Mgr Row │
│  2 │Bob │  1  │───────────►│    2    │    1    │
│  3 │Cal │  1  │───────────►│    3    │    1    │
│  4 │Dan │  2  │───────────►│    4    │    2    │
└────┴────┴─────┘            └─────────┴─────────┘
                             Same table, two aliases
```

## Common Use Cases

### Use Case 1: Hierarchical Data (Employee-Manager)

**Setup:**
```sql
CREATE TABLE employees_org (
    employee_id SERIAL PRIMARY KEY,
    employee_name VARCHAR(100),
    position VARCHAR(50),
    manager_id INTEGER,
    salary NUMERIC(10, 2)
);

INSERT INTO employees_org (employee_id, employee_name, position, manager_id, salary) VALUES
(1, 'Alice Johnson', 'CEO', NULL, 150000),
(2, 'Bob Smith', 'VP Engineering', 1, 120000),
(3, 'Carol White', 'VP Sales', 1, 115000),
(4, 'David Brown', 'Senior Engineer', 2, 95000),
(5, 'Eve Davis', 'Engineer', 2, 85000),
(6, 'Frank Miller', 'Sales Rep', 3, 70000),
(7, 'Grace Lee', 'Junior Engineer', 4, 75000);
```

**Employees Table:**
| employee_id | employee_name | position | manager_id | salary |
|-------------|---------------|----------|------------|--------|
| 1 | Alice Johnson | CEO | NULL | 150000 |
| 2 | Bob Smith | VP Engineering | 1 | 120000 |
| 3 | Carol White | VP Sales | 1 | 115000 |
| 4 | David Brown | Senior Engineer | 2 | 95000 |
| 5 | Eve Davis | Engineer | 2 | 85000 |
| 6 | Frank Miller | Sales Rep | 3 | 70000 |
| 7 | Grace Lee | Junior Engineer | 4 | 75000 |

### Example 1: List Employees with Their Managers

**Query:**
```sql
SELECT e.employee_name AS employee,
       e.position AS employee_position,
       m.employee_name AS manager,
       m.position AS manager_position
FROM employees_org e
LEFT JOIN employees_org m ON e.manager_id = m.employee_id
ORDER BY e.employee_id;
```

**Result:**
| employee | employee_position | manager | manager_position |
|----------|-------------------|---------|------------------|
| Alice Johnson | CEO | NULL | NULL |
| Bob Smith | VP Engineering | Alice Johnson | CEO |
| Carol White | VP Sales | Alice Johnson | CEO |
| David Brown | Senior Engineer | Bob Smith | VP Engineering |
| Eve Davis | Engineer | Bob Smith | VP Engineering |
| Frank Miller | Sales Rep | Carol White | VP Sales |
| Grace Lee | Junior Engineer | David Brown | Senior Engineer |

**Explanation:**
- `e` alias represents employees
- `m` alias represents managers (same table)
- LEFT JOIN includes CEO (who has no manager)
- Self join links employee's manager_id to manager's employee_id

### Example 2: Find Direct Reports

**Query:** List managers and their direct reports.

```sql
SELECT m.employee_name AS manager,
       m.position AS manager_position,
       COUNT(e.employee_id) AS direct_reports,
       STRING_AGG(e.employee_name, ', ' ORDER BY e.employee_name) AS report_names
FROM employees_org m
LEFT JOIN employees_org e ON m.employee_id = e.manager_id
GROUP BY m.employee_id, m.employee_name, m.position
HAVING COUNT(e.employee_id) > 0
ORDER BY direct_reports DESC;
```

**Result:**
| manager | manager_position | direct_reports | report_names |
|---------|------------------|----------------|--------------|
| Alice Johnson | CEO | 2 | Bob Smith, Carol White |
| Bob Smith | VP Engineering | 2 | David Brown, Eve Davis |
| Carol White | VP Sales | 1 | Frank Miller |
| David Brown | Senior Engineer | 1 | Grace Lee |

**Explanation:**
- Reverses the relationship to show managers with their reports
- Uses aggregation to count and list direct reports
- HAVING filters to show only managers with reports

### Example 3: Salary Comparison with Manager

**Query:** Compare each employee's salary with their manager's salary.

```sql
SELECT e.employee_name,
       e.position,
       e.salary AS employee_salary,
       m.employee_name AS manager,
       m.salary AS manager_salary,
       m.salary - e.salary AS salary_diff,
       ROUND((e.salary::NUMERIC / m.salary * 100), 1) AS pct_of_manager_salary
FROM employees_org e
INNER JOIN employees_org m ON e.manager_id = m.employee_id
ORDER BY salary_diff DESC;
```

**Result:**
| employee_name | position | employee_salary | manager | manager_salary | salary_diff | pct_of_manager_salary |
|---------------|----------|-----------------|---------|----------------|-------------|----------------------|
| Frank Miller | Sales Rep | 70000 | Carol White | 115000 | 45000 | 60.9 |
| Grace Lee | Junior Engineer | 75000 | David Brown | 95000 | 20000 | 78.9 |
| Eve Davis | Engineer | 85000 | Bob Smith | 120000 | 35000 | 70.8 |
| David Brown | Senior Engineer | 95000 | Bob Smith | 120000 | 25000 | 79.2 |
| Bob Smith | VP Engineering | 120000 | Alice Johnson | 150000 | 30000 | 80.0 |
| Carol White | VP Sales | 115000 | Alice Johnson | 150000 | 35000 | 76.7 |

**Explanation:**
- INNER JOIN excludes CEO (no manager to compare)
- Calculates difference and percentage
- Shows compensation structure relative to management

## Use Case 2: Finding Pairs or Relationships

### Example 4: Find Students in the Same Major

**Query:**
```sql
SELECT s1.first_name || ' ' || s1.last_name AS student1,
       s2.first_name || ' ' || s2.last_name AS student2,
       s1.major
FROM students s1
INNER JOIN students s2 ON s1.major = s2.major
WHERE s1.student_id < s2.student_id  -- Avoid duplicates and self-pairing
  AND s1.major IS NOT NULL
ORDER BY s1.major, s1.student_id;
```

**Result:**
| student1 | student2 | major |
|----------|----------|-------|
| John Smith | Bob Wilson | Computer Science |

**Explanation:**
- Joins students table with itself on same major
- `s1.student_id < s2.student_id` prevents:
  - Self-pairing (John with John)
  - Duplicate pairs (John-Bob and Bob-John)
- Only shows unique pairs of students sharing a major

### Example 5: Find Students Who Took the Same Course

**Query:**
```sql
SELECT s1.first_name || ' ' || s1.last_name AS student1,
       s2.first_name || ' ' || s2.last_name AS student2,
       c.course_name,
       e1.semester AS student1_semester,
       e2.semester AS student2_semester
FROM enrollments e1
INNER JOIN enrollments e2 ON e1.course_id = e2.course_id
INNER JOIN students s1 ON e1.student_id = s1.student_id
INNER JOIN students s2 ON e2.student_id = s2.student_id
INNER JOIN courses c ON e1.course_id = c.course_id
WHERE e1.student_id < e2.student_id  -- Avoid duplicates
ORDER BY c.course_name, s1.last_name;
```

**Result:**
| student1 | student2 | course_name | student1_semester | student2_semester |
|----------|----------|-------------|-------------------|-------------------|
| Jane Doe | John Smith | Introduction to Programming | Fall 2023 | Fall 2023 |
| John Smith | Bob Wilson | Introduction to Programming | Fall 2023 | Spring 2024 |
| Jane Doe | Bob Wilson | Introduction to Programming | Fall 2023 | Spring 2024 |
| John Smith | Bob Wilson | Data Structures | Spring 2024 | Spring 2024 |

**Explanation:**
- Self join on enrollments table
- Finds all pairs of students who took the same course
- Shows different semesters when applicable
- CS101 had 3 students, producing 3 pairs (1-2, 1-3, 2-3)

## Use Case 3: Comparing Rows Within Same Table

### Example 6: Find Courses with Similar Enrollments

**Query:** Find pairs of courses with similar enrollment counts (within 1 of each other).

```sql
WITH course_counts AS (
    SELECT c.course_id,
           c.course_name,
           COUNT(e.enrollment_id) AS enrollment_count
    FROM courses c
    LEFT JOIN enrollments e ON c.course_id = e.course_id
    GROUP BY c.course_id, c.course_name
)
SELECT cc1.course_id AS course1_id,
       cc1.course_name AS course1,
       cc1.enrollment_count AS course1_enrollments,
       cc2.course_id AS course2_id,
       cc2.course_name AS course2,
       cc2.enrollment_count AS course2_enrollments,
       ABS(cc1.enrollment_count - cc2.enrollment_count) AS enrollment_diff
FROM course_counts cc1
INNER JOIN course_counts cc2 
    ON cc1.course_id < cc2.course_id
    AND ABS(cc1.enrollment_count - cc2.enrollment_count) <= 1
ORDER BY enrollment_diff, cc1.course_id;
```

**Result:**
| course1_id | course1 | course1_enrollments | course2_id | course2 | course2_enrollments | enrollment_diff |
|------------|---------|---------------------|------------|---------|---------------------|-----------------|
| ENG101 | English Composition | 0 | MATH101 | Calculus I | 1 | 1 |
| ENG101 | English Composition | 0 | PHYS101 | Physics I | 1 | 1 |
| ENG101 | English Composition | 0 | CS301 | Database Systems | 1 | 1 |
| MATH101 | Calculus I | 1 | PHYS101 | Physics I | 1 | 0 |
| MATH101 | Calculus I | 1 | CS301 | Database Systems | 1 | 0 |
| PHYS101 | Physics I | 1 | CS301 | Database Systems | 1 | 0 |
| CS201 | Data Structures | 2 | CS101 | Introduction to Programming | 3 | 1 |

**Explanation:**
- CTE calculates enrollment counts
- Self join finds course pairs with similar enrollments
- Uses ABS() for absolute difference
- Useful for identifying courses that could be combined or compared

## Advanced Self Join Patterns

### Example 7: Multi-Level Hierarchy

**Query:** Show employee, manager, and manager's manager.

```sql
SELECT e.employee_name AS employee,
       m1.employee_name AS manager,
       m2.employee_name AS managers_manager
FROM employees_org e
LEFT JOIN employees_org m1 ON e.manager_id = m1.employee_id
LEFT JOIN employees_org m2 ON m1.manager_id = m2.employee_id
WHERE e.manager_id IS NOT NULL
ORDER BY e.employee_id;
```

**Result:**
| employee | manager | managers_manager |
|----------|---------|------------------|
| Bob Smith | Alice Johnson | NULL |
| Carol White | Alice Johnson | NULL |
| David Brown | Bob Smith | Alice Johnson |
| Eve Davis | Bob Smith | Alice Johnson |
| Frank Miller | Carol White | Alice Johnson |
| Grace Lee | David Brown | Bob Smith |

**Explanation:**
- Multiple self joins traverse hierarchy upward
- Shows 3 levels: employee → manager → grand-manager
- Can extend further for deeper hierarchies

## NATURAL JOIN

### Overview

A **NATURAL JOIN** automatically joins tables based on columns with the same name. It's a convenience feature but comes with significant risks.

### Syntax

```sql
SELECT columns
FROM table1
NATURAL JOIN table2;
```

### How NATURAL JOIN Works

The database automatically:
1. Finds columns with matching names in both tables
2. Creates equality conditions for all matching columns
3. Returns rows where all matching columns have equal values

### Example 8: NATURAL JOIN Demo

**Setup:**
```sql
CREATE TABLE table_a (
    id INTEGER,
    name VARCHAR(50),
    value INTEGER
);

CREATE TABLE table_b (
    id INTEGER,
    value INTEGER,
    description VARCHAR(50)
);

INSERT INTO table_a VALUES (1, 'Alice', 100), (2, 'Bob', 200);
INSERT INTO table_b VALUES (1, 100, 'First'), (2, 300, 'Second');
```

**NATURAL JOIN:**
```sql
SELECT *
FROM table_a
NATURAL JOIN table_b;
```

**Result:**
| id | name | value | description |
|----|------|-------|-------------|
| 1 | Alice | 100 | First |

**Explanation:**
- Joins on both `id` AND `value` (both columns match)
- Only row 1 matches on both columns
- Row 2 excluded because values don't match (200 ≠ 300)

**Equivalent explicit JOIN:**
```sql
SELECT *
FROM table_a a
INNER JOIN table_b b ON a.id = b.id AND a.value = b.value;
```

## Why NATURAL JOIN is Dangerous

### Problem 1: Implicit Join Conditions

**Issue:** Not clear what columns are being joined.

```sql
-- What columns are being joined?
SELECT *
FROM students
NATURAL JOIN enrollments;
```

**Better:** Explicit join
```sql
SELECT *
FROM students s
JOIN enrollments e ON s.student_id = e.student_id;
```

### Problem 2: Schema Changes Break Queries

**Scenario:** Someone adds a `department` column to both tables.

**Before:**
```sql
NATURAL JOIN  -- Joins on student_id only
```

**After adding department column:**
```sql
NATURAL JOIN  -- Now joins on student_id AND department!
-- Suddenly returns different results or no results!
```

### Problem 3: Unexpected Column Matches

**Example:**
```sql
-- Accidentally matches on 'year' column present in both tables
-- but meaning different things (graduation year vs. course year)
SELECT *
FROM students
NATURAL JOIN courses;
```

### Problem 4: Poor Readability and Maintenance

**Issues:**
- Future developers don't know what the join condition is
- Must examine schema to understand the query
- Changes to table structure can silently break queries
- Difficult to debug

## Best Practices

### ✅ DO: Use Explicit Joins

```sql
-- Good: Clear and maintainable
SELECT s.first_name, s.last_name, e.course_id, e.grade
FROM students s
INNER JOIN enrollments e ON s.student_id = e.student_id;
```

### ❌ DON'T: Use NATURAL JOIN

```sql
-- Bad: Implicit and risky
SELECT *
FROM students
NATURAL JOIN enrollments;
```

### ✅ DO: Use Table Aliases in Self Joins

```sql
-- Good: Clear distinction between instances
SELECT e.employee_name, m.employee_name AS manager
FROM employees_org e
LEFT JOIN employees_org m ON e.manager_id = m.employee_id;
```

### ❌ DON'T: Use Meaningless Aliases

```sql
-- Bad: Confusing
SELECT a.employee_name, b.employee_name
FROM employees_org a
JOIN employees_org b ON a.manager_id = b.employee_id;
-- Which is which?
```

### ✅ DO: Prevent Duplicate Pairs in Self Joins

```sql
-- Good: Avoid duplicates with <
WHERE s1.student_id < s2.student_id
```

### ❌ DON'T: Create Duplicate Pairs

```sql
-- Bad: Creates duplicates
WHERE s1.student_id != s2.student_id
-- John-Bob AND Bob-John both appear
```

## Summary

### SELF JOIN Key Takeaways:
- Join a table with itself
- **Table aliases are REQUIRED**
- Useful for hierarchical data (employees, categories, etc.)
- Comparing rows within same table
- Finding pairs or relationships
- Use `id1 < id2` to avoid duplicate pairs
- Can use any join type (INNER, LEFT, RIGHT)

### NATURAL JOIN Key Takeaways:
- **Avoid in production code**
- Joins on all columns with matching names
- Implicit join conditions are risky
- Schema changes can break queries silently
- Poor maintainability and readability
- Always use explicit JOIN with ON clause instead

### Quick Reference:

```sql
-- SELF JOIN: Employee-Manager
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.employee_id;

-- SELF JOIN: Find Pairs (avoid duplicates)
SELECT s1.name, s2.name, s1.category
FROM table s1
JOIN table s2 ON s1.category = s2.category
WHERE s1.id < s2.id;

-- SELF JOIN: Multi-Level Hierarchy
SELECT e.name, m1.name AS manager, m2.name AS grand_manager
FROM employees e
LEFT JOIN employees m1 ON e.manager_id = m1.id
LEFT JOIN employees m2 ON m1.manager_id = m2.id;

-- NATURAL JOIN (AVOID)
SELECT * FROM table1 NATURAL JOIN table2;  -- Don't use!

-- Explicit JOIN (PREFERRED)
SELECT * FROM table1 t1
JOIN table2 t2 ON t1.id = t2.id;  -- Use this instead
```

