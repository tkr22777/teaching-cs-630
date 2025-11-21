# Advanced Join Patterns and Best Practices

## Overview

Advanced join patterns, composite joins, and performance optimization techniques.

## Composite Joins

### What is a Composite Join?

A **composite join** uses multiple columns in the join condition to uniquely identify matching rows.

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

| first_name | last_name | course_id | semester | year | section | room     | instructor_name | grade |
| ---------- | --------- | --------- | -------- | ---- | ------- | -------- | --------------- | ----- |
| Jane       | Doe       | CS101     | Fall     | 2023 | B       | Room 102 | Dr. Lee         | A-    |
| John       | Smith     | CS101     | Fall     | 2023 | A       | Room 101 | Dr. Johnson     | A     |
| John       | Smith     | CS201     | Spring   | 2024 | A       | Room 103 | Dr. Johnson     | B+    |
| Bob        | Wilson    | CS101     | Spring   | 2024 | A       | Room 101 | Dr. Johnson     | B     |

## Common Join Patterns

### Pattern 1: Many-to-Many Through Junction Table

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

| major            | students_in_major | courses_taken | total_enrollments | avg_credits |
| ---------------- | ----------------- | ------------- | ----------------- | ----------- |
| Computer Science | 2                 | 3             | 5                 | 3.6         |
| Mathematics      | 1                 | 2             | 2                 | 3.5         |
| Physics          | 1                 | 1             | 1                 | 4.0         |

### Pattern 2: Filtered Joins

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

**Result:**

| student_id | first_name | last_name | major            |
| ---------- | ---------- | --------- | ---------------- |
| 2          | Jane       | Doe       | Mathematics      |
| 1          | John       | Smith     | Computer Science |
| 3          | Bob        | Wilson    | Computer Science |
