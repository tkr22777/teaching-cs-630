# Advanced Join Patterns and Best Practices

## Overview

This guide covers advanced join patterns, composite joins, performance optimization, and best practices for writing efficient and maintainable SQL joins.

## Composite Joins

### What is a Composite Join?

A **composite join** uses multiple columns in the join condition. This is necessary when the relationship between tables requires more than one column to uniquely identify matching rows.

### When to Use Composite Joins

- Junction tables with composite keys
- Time-series data (date + time + location)
- Multi-tenant systems (tenant_id + record_id)
- Versioned data (id + version)

### Example 1: Course Sections with Composite Keys

**Setup:**
```sql
CREATE TABLE course_sections (
    course_id VARCHAR2(10),
    semester VARCHAR2(20),
    year INTEGER,
    section VARCHAR2(5),
    instructor_id INTEGER,
    room VARCHAR2(20),
    max_students INTEGER,
    PRIMARY KEY (course_id, semester, year, section)
);

CREATE TABLE section_enrollments (
    enrollment_id INTEGER PRIMARY KEY,
    student_id INTEGER,
    course_id VARCHAR2(10),
    semester VARCHAR2(20),
    year INTEGER,
    section VARCHAR2(5),
    grade VARCHAR2(5)
);

INSERT INTO course_sections VALUES
('CS101', 'Fall', 2023, 'A', 10, 'Room 101', 30),
('CS101', 'Fall', 2023, 'B', 11, 'Room 102', 30),
('CS101', 'Spring', 2024, 'A', 10, 'Room 101', 30),
('CS201', 'Spring', 2024, 'A', 10, 'Room 103', 25);

INSERT INTO section_enrollments (student_id, course_id, semester, year, section, grade) VALUES
(1, 'CS101', 'Fall', 2023, 'A', 'A'),
(2, 'CS101', 'Fall', 2023, 'B', 'A-'),
(3, 'CS101', 'Spring', 2024, 'A', 'B'),
(1, 'CS201', 'Spring', 2024, 'A', 'B+');
```

**Query:** Join enrollments with section details using composite key.

```sql
SELECT s.first_name,
       s.last_name,
       se.course_id,
       se.semester,
       se.year,
       se.section,
       cs.room,
       i.instructor_name,
       se.grade
FROM students s
INNER JOIN section_enrollments se ON s.student_id = se.student_id
INNER JOIN course_sections cs 
    ON se.course_id = cs.course_id
    AND se.semester = cs.semester
    AND se.year = cs.year
    AND se.section = cs.section
INNER JOIN instructors i ON cs.instructor_id = i.instructor_id
ORDER BY se.year, se.semester, s.last_name;
```

**Result:**
| first_name | last_name | course_id | semester | year | section | room | instructor_name | grade |
|------------|-----------|-----------|----------|------|---------|------|-----------------|-------|
| Jane | Doe | CS101 | Fall | 2023 | B | Room 102 | Dr. Lee | A- |
| John | Smith | CS101 | Fall | 2023 | A | Room 101 | Dr. Johnson | A |
| John | Smith | CS201 | Spring | 2024 | A | Room 103 | Dr. Johnson | B+ |
| Bob | Wilson | CS101 | Spring | 2024 | A | Room 101 | Dr. Johnson | B |

**Explanation:**
- Four columns required to uniquely identify a section
- Multiple AND conditions in ON clause
- Each enrollment matches its specific section
- Essential for systems with multiple offerings of same course

## Common Join Patterns (minimal set)

### Pattern 1: Master-Detail Relationship

**Use Case:** One parent record with multiple child records.

**Query:** Students with their enrollments and aggregate stats.

```sql
SELECT s.student_id,
       s.first_name || ' ' || s.last_name AS student_name,
       s.major,
       COUNT(e.enrollment_id) AS course_count,
       LISTAGG(c.course_name, ', ') WITHIN GROUP (ORDER BY c.course_name) AS courses,
       ROUND(AVG(
           CASE e.grade
               WHEN 'A' THEN 4.0
               WHEN 'A-' THEN 3.7
               WHEN 'B+' THEN 3.3
               WHEN 'B' THEN 3.0
               ELSE NULL
           END
       ), 2) AS gpa
FROM students s
LEFT JOIN enrollments e ON s.student_id = e.student_id
LEFT JOIN courses c ON e.course_id = c.course_id
GROUP BY s.student_id, s.first_name, s.last_name, s.major
ORDER BY gpa DESC NULLS LAST;
```

**Result:**
| student_id | student_name | major | course_count | courses | gpa |
|------------|--------------|-------|--------------|---------|-----|
| 4 | Alice Brown | Physics | 1 | Physics I | 4.00 |
| 2 | Jane Doe | Mathematics | 2 | Calculus I, Introduction to Programming | 3.85 |
| 1 | John Smith | Computer Science | 3 | Data Structures, Database Systems, Introduction to Programming | 3.65 |
| 3 | Bob Wilson | Computer Science | 2 | Data Structures, Introduction to Programming | 3.15 |
| 5 | Charlie Davis | NULL | 0 | NULL | NULL |

**Explanation:**
- LEFT JOIN preserves all students
- Aggregates detail records (enrollments)
- LISTAGG creates comma-separated list
- Calculates GPA from grades

### Pattern 2: Many-to-Many Through Junction Table

**Use Case:** Two entities connected via intermediate table.

**Query:** Complete many-to-many relationship with details.

```sql
SELECT s.major,
       COUNT(DISTINCT s.student_id) AS students_in_major,
       COUNT(DISTINCT c.course_id) AS courses_taken,
       COUNT(e.enrollment_id) AS total_enrollments,
       ROUND(AVG(c.credits), 1) AS avg_credits
FROM students s
INNER JOIN enrollments e ON s.student_id = e.student_id
INNER JOIN courses c ON e.course_id = c.course_id
WHERE s.major IS NOT NULL
GROUP BY s.major
ORDER BY total_enrollments DESC;
```

**Result:**
| major | students_in_major | courses_taken | total_enrollments | avg_credits |
|-------|-------------------|---------------|-------------------|-------------|
| Computer Science | 2 | 3 | 5 | 3.6 |
| Mathematics | 1 | 2 | 2 | 3.5 |
| Physics | 1 | 1 | 1 | 4.0 |

**Explanation:**
- Students ↔ Courses through Enrollments junction table
- Multiple levels of aggregation
- DISTINCT prevents double-counting

### Pattern 3: Finding Unmatched Records

**Use Case:** Identify records in one table without corresponding records in another.

**Query:** Find all orphan and missing records.

```sql
-- Students without enrollments
SELECT 'No Enrollments' AS issue_type,
       s.student_id,
       s.first_name || ' ' || s.last_name AS name,
       s.enrollment_date,
       NULL AS course_id
FROM students s
LEFT JOIN enrollments e ON s.student_id = e.student_id
WHERE e.enrollment_id IS NULL

UNION ALL

-- Courses without enrollments
SELECT 'No Enrollments' AS issue_type,
       NULL AS student_id,
       c.course_name AS name,
       NULL AS enrollment_date,
       c.course_id
FROM courses c
LEFT JOIN enrollments e ON c.course_id = e.course_id
WHERE e.enrollment_id IS NULL

UNION ALL

-- Courses without instructors
SELECT 'No Instructor' AS issue_type,
       NULL AS student_id,
       c.course_name AS name,
       NULL AS enrollment_date,
       c.course_id
FROM courses c
WHERE c.instructor_id IS NULL

ORDER BY issue_type, student_id, course_id;
```

**Result:**
| issue_type | student_id | name | enrollment_date | course_id |
|------------|------------|------|-----------------|-----------|
| No Enrollments | 5 | Charlie Davis | 2024-09-01 | NULL |
| No Enrollments | NULL | English Composition | NULL | ENG101 |
| No Instructor | NULL | English Composition | NULL | ENG101 |

**Explanation:**
- Uses UNION ALL to combine multiple checks
- LEFT JOIN + IS NULL pattern finds missing relationships
- Useful for data quality audits

###

**Use Case:** Get the most recent record for each entity.

**Query:** Most recent enrollment for each student.

```sql
WITH ranked_enrollments AS (
    SELECT e.*,
           c.course_name,
           ROW_NUMBER() OVER (PARTITION BY e.student_id ORDER BY e.enrollment_id DESC) AS rn
    FROM enrollments e
    JOIN courses c ON e.course_id = c.course_id
)
SELECT s.student_id,
       s.first_name,
       s.last_name,
       re.course_name AS latest_course,
       re.semester AS latest_semester,
       re.grade
FROM students s
INNER JOIN ranked_enrollments re ON s.student_id = re.student_id
WHERE re.rn = 1
ORDER BY s.student_id;
```

**Result:**
| student_id | first_name | last_name | latest_course | latest_semester | grade |
|------------|------------|-----------|---------------|-----------------|-------|
| 1 | John | Smith | Database Systems | Fall 2024 | NULL |
| 2 | Jane | Doe | Introduction to Programming | Fall 2023 | A- |
| 3 | Bob | Wilson | Data Structures | Spring 2024 | B+ |
| 4 | Alice | Brown | Physics I | Spring 2024 | A |

**Explanation:**
- ROW_NUMBER() ranks enrollments per student
- PARTITION BY creates separate ranking per student
- Filter WHERE rn = 1 gets the latest only
- Alternative: MAX(enrollment_id) with subquery

###

**Use Case:** Join based on complex conditions.

**Query:** Find grade improvements (students who retook courses).

```sql
SELECT s.first_name,
       s.last_name,
       c.course_name,
       e1.semester AS first_attempt_semester,
       e1.grade AS first_grade,
       e2.semester AS second_attempt_semester,
       e2.grade AS second_grade,
       CASE 
           WHEN e2.grade > e1.grade THEN 'Improved'
           WHEN e2.grade = e1.grade THEN 'Same'
           ELSE 'Declined'
       END AS grade_change
FROM enrollments e1
INNER JOIN enrollments e2 
    ON e1.student_id = e2.student_id
    AND e1.course_id = e2.course_id
    AND e1.enrollment_id < e2.enrollment_id
INNER JOIN students s ON e1.student_id = s.student_id
INNER JOIN courses c ON e1.course_id = c.course_id
ORDER BY s.last_name, c.course_name;
```

**Note:** Our sample data doesn't have retakes, but this pattern is useful when students retake courses.

### Pattern 4: Filtered Joins (Semi-Join Pattern)

**Use Case:** Filter main table based on existence in related table.

**Query:** Students who are enrolled in any Computer Science course.

```sql
SELECT DISTINCT s.student_id,
       s.first_name,
       s.last_name,
       s.major
FROM students s
INNER JOIN enrollments e ON s.student_id = e.student_id
INNER JOIN courses c ON e.course_id = c.course_id
WHERE c.department = 'Computer Science'
ORDER BY s.last_name;
```

**Alternative using EXISTS (often more efficient):**
```sql
SELECT s.student_id,
       s.first_name,
       s.last_name,
       s.major
FROM students s
WHERE EXISTS (
    SELECT 1
    FROM enrollments e
    JOIN courses c ON e.course_id = c.course_id
    WHERE e.student_id = s.student_id
      AND c.department = 'Computer Science'
)
ORDER BY s.last_name;
```

**Result:**
| student_id | first_name | last_name | major |
|------------|------------|-----------|-------|
| Jane | Doe | Mathematics |
| John | Smith | Computer Science |
| Bob | Wilson | Computer Science |

## Performance Tips

```sql
-- Index join columns (foreign keys)
CREATE INDEX idx_enrollments_student_id ON enrollments(student_id);
CREATE INDEX idx_enrollments_course_id ON enrollments(course_id);

-- Example query (check plan using your database's tools if desired)
SELECT s.first_name, c.course_name
FROM students s
JOIN enrollments e ON s.student_id = e.student_id
JOIN courses c ON e.course_id = c.course_id;

-- Filter early with WHERE
SELECT s.first_name, c.course_name
FROM students s
JOIN enrollments e ON s.student_id = e.student_id
JOIN courses c ON e.course_id = c.course_id
WHERE s.major = 'Computer Science';  -- Filter before joining when possible
```

