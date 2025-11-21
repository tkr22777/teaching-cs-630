# Introduction to SQL Joins

## Overview

**Joins** combine rows from two or more tables based on related columns. In normalized databases, data is split across multiple tables to avoid duplication. Joins let you query this related data and present results as if the data were in a single table.

### Visual: How Joins Combine Tables

**Note:** This is a simplified example for clarity. In practice, one student can have multiple enrollments.

```text
Table A (Students)              Table B (Enrollments)
┌─────────────┬─────────┐      ┌─────────────┬───────────┐
│ student_id  │ name    │      │ student_id  │ course_id │
├─────────────┼─────────┤      ├─────────────┼───────────┤
│ 1           │ John    │      │ 1           │ CS101     │
│ 2           │ Jane    │      │ 2           │ MATH101   │
│ 3           │ Bob     │      │ 3           │ CS201     │
│ 4           │ Alice   │      │ 4           │ PHYS101   │
└─────────────┴─────────┘      └─────────────┴───────────┘
                    ↓                   ↓
                    └─── JOIN ON student_id ───┘
                                ↓
                        Combined Result
            ┌─────────────┬─────────┬───────────┐
            │ student_id  │ name    │ course_id │
            ├─────────────┼─────────┼───────────┤
            │ 1           │ John    │ CS101     │
            │ 2           │ Jane    │ MATH101   │
            │ 3           │ Bob     │ CS201     │
            │ 4           │ Alice   │ PHYS101   │
            └─────────────┴─────────┴───────────┘
```

**How Joins Work:** Two separate tables are combined through a join condition (matching `student_id` values). Rows with matching join keys are combined into a single result row. The join condition identifies which rows from each table should be matched together.

## Join Syntax

```sql
SELECT columns
FROM table1
JOIN table2 ON table1.column = table2.column;
```

---

## Sample Database Schema

For all examples in this course module, we'll use a university enrollment system.

**Note:** The complete database setup script is in `00_initialization.md`.

---

### Students Table

| student_id | first_name | last_name | email                        | major            | enrollment_date |
| ---------- | ---------- | --------- | ---------------------------- | ---------------- | --------------- |
| 1          | John       | Smith     | john.smith@university.edu    | Computer Science | 2023-09-01      |
| 2          | Jane       | Doe       | jane.doe@university.edu      | Mathematics      | 2023-09-01      |
| 3          | Bob        | Wilson    | bob.wilson@university.edu    | Computer Science | 2024-01-15      |
| 4          | Alice      | Brown     | alice.brown@university.edu   | Physics          | 2024-01-15      |
| 5          | Charlie    | Davis     | charlie.davis@university.edu | NULL             | 2024-09-01      |

### Instructors Table

| instructor_id | instructor_name | department       | hire_date  |
| ------------- | --------------- | ---------------- | ---------- |
| 10            | Dr. Johnson     | Computer Science | 2018-08-15 |
| 11            | Dr. Lee         | Mathematics      | 2019-01-10 |
| 12            | Dr. Martinez    | Physics          | 2020-09-01 |
| 13            | Dr. Taylor      | Chemistry        | 2021-06-15 |

### Courses Table

| course_id | course_name                 | department       | credits | instructor_id |
| --------- | --------------------------- | ---------------- | ------- | ------------- |
| CS101     | Introduction to Programming | Computer Science | 3       | 10            |
| CS201     | Data Structures             | Computer Science | 4       | 10            |
| MATH101   | Calculus I                  | Mathematics      | 4       | 11            |
| PHYS101   | Physics I                   | Physics          | 4       | 12            |
| CS301     | Database Systems            | Computer Science | 3       | 10            |
| ENG101    | English Composition         | English          | 3       | NULL          |

### Enrollments Table

| enrollment_id | student_id | course_id | semester    | grade |
| ------------- | ---------- | --------- | ----------- | ----- |
| 101           | 1          | CS101     | Fall 2023   | A     |
| 102           | 1          | CS201     | Spring 2024 | B+    |
| 103           | 2          | MATH101   | Fall 2023   | A     |
| 104           | 2          | CS101     | Fall 2023   | A-    |
| 105           | 3          | CS101     | Spring 2024 | B     |
| 106           | 3          | CS201     | Spring 2024 | B+    |
| 107           | 4          | PHYS101   | Spring 2024 | A     |
| 108           | 1          | CS301     | Fall 2024   | NULL  |

---

## Simple Join Example

Let's see a basic join in action:

**Query:** Get student names with their enrolled course names.

```sql
SELECT s.first_name,
       s.last_name,
       c.course_name,
       e.grade
FROM students s
JOIN enrollments e ON s.student_id = e.student_id
JOIN courses c ON e.course_id = c.course_id
ORDER BY s.last_name, s.first_name;
```

**Result:**

| first_name | last_name | course_name                 | grade |
| ---------- | --------- | --------------------------- | ----- |
| Alice      | Brown     | Physics I                   | A     |
| Jane       | Doe       | Calculus I                  | A     |
| Jane       | Doe       | Introduction to Programming | A-    |
| John       | Smith     | Introduction to Programming | A     |
| John       | Smith     | Data Structures             | B+    |
| John       | Smith     | Database Systems            | NULL  |
| Bob        | Wilson    | Introduction to Programming | B     |
| Bob        | Wilson    | Data Structures             | B+    |

**What happened:** We joined three tables to combine student names, enrollments, and course names. Only students with enrollments appear (Charlie Davis has no enrollments, so he's excluded).
