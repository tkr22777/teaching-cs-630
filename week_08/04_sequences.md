# SQL Sequences

## Overview

A **sequence** is a database object that generates unique sequential numbers, commonly used for primary keys. Sequences are independent of tables and can be shared across multiple tables.

## Key Terms

**Sequence**: Database object that generates unique sequential numbers.

**NEXTVAL**: Pseudo-column that returns the next value from a sequence.

**CURRVAL**: Pseudo-column that returns the current value of a sequence.

**INCREMENT BY**: Step size for sequence progression.

**START WITH**: Initial value of the sequence.

**CACHE**: Number of sequence values pre-allocated in memory for performance.

**CYCLE**: Option to restart sequence after reaching max/min value.

## Sample Database Schema

This module uses the university enrollment system. If you haven't set it up yet:

<details>
<summary>Click to expand: Database setup script</summary>

```sql
-- Create Students table
CREATE TABLE students (
    student_id INTEGER PRIMARY KEY,
    first_name VARCHAR2(50) NOT NULL,
    last_name VARCHAR2(50) NOT NULL,
    email VARCHAR2(100) UNIQUE NOT NULL,
    major VARCHAR2(50),
    enrollment_date DATE DEFAULT SYSDATE,
    gpa NUMBER(3, 2)
);

-- Create Instructors table
CREATE TABLE instructors (
    instructor_id INTEGER PRIMARY KEY,
    instructor_name VARCHAR2(100) NOT NULL,
    department VARCHAR2(50),
    hire_date DATE
);

-- Create Courses table
CREATE TABLE courses (
    course_id VARCHAR2(10) PRIMARY KEY,
    course_name VARCHAR2(100) NOT NULL,
    department VARCHAR2(50),
    credits INTEGER,
    instructor_id INTEGER REFERENCES instructors(instructor_id)
);

-- Create Enrollments table (junction table)
CREATE TABLE enrollments (
    enrollment_id INTEGER PRIMARY KEY,
    student_id INTEGER REFERENCES students(student_id),
    course_id VARCHAR2(10) REFERENCES courses(course_id),
    semester VARCHAR2(20),
    grade VARCHAR2(5),
    grade_points NUMBER(3, 2)
);

-- Insert Students
INSERT INTO students (student_id, first_name, last_name, email, major, enrollment_date, gpa) VALUES
(1, 'John', 'Smith', 'john.smith@university.edu', 'Computer Science', DATE '2023-09-01', 3.8);
INSERT INTO students (student_id, first_name, last_name, email, major, enrollment_date, gpa) VALUES
(2, 'Jane', 'Doe', 'jane.doe@university.edu', 'Mathematics', DATE '2023-09-01', 3.9);
INSERT INTO students (student_id, first_name, last_name, email, major, enrollment_date, gpa) VALUES
(3, 'Bob', 'Wilson', 'bob.wilson@university.edu', 'Computer Science', DATE '2024-01-15', 3.2);
INSERT INTO students (student_id, first_name, last_name, email, major, enrollment_date, gpa) VALUES
(4, 'Alice', 'Brown', 'alice.brown@university.edu', 'Physics', DATE '2024-01-15', 3.7);
INSERT INTO students (student_id, first_name, last_name, email, major, enrollment_date, gpa) VALUES
(5, 'Charlie', 'Davis', 'charlie.davis@university.edu', NULL, DATE '2024-09-01', 2.8);

-- Insert Instructors
INSERT INTO instructors (instructor_id, instructor_name, department, hire_date) VALUES
(10, 'Dr. Johnson', 'Computer Science', DATE '2018-08-15');
INSERT INTO instructors (instructor_id, instructor_name, department, hire_date) VALUES
(11, 'Dr. Lee', 'Mathematics', DATE '2019-01-10');
INSERT INTO instructors (instructor_id, instructor_name, department, hire_date) VALUES
(12, 'Dr. Martinez', 'Physics', DATE '2020-09-01');
INSERT INTO instructors (instructor_id, instructor_name, department, hire_date) VALUES
(13, 'Dr. Taylor', 'Chemistry', DATE '2021-06-15');

-- Insert Courses
INSERT INTO courses (course_id, course_name, department, credits, instructor_id) VALUES
('CS101', 'Introduction to Programming', 'Computer Science', 3, 10);
INSERT INTO courses (course_id, course_name, department, credits, instructor_id) VALUES
('CS201', 'Data Structures', 'Computer Science', 4, 10);
INSERT INTO courses (course_id, course_name, department, credits, instructor_id) VALUES
('MATH101', 'Calculus I', 'Mathematics', 4, 11);
INSERT INTO courses (course_id, course_name, department, credits, instructor_id) VALUES
('PHYS101', 'Physics I', 'Physics', 4, 12);
INSERT INTO courses (course_id, course_name, department, credits, instructor_id) VALUES
('CS301', 'Database Systems', 'Computer Science', 3, 10);
INSERT INTO courses (course_id, course_name, department, credits, instructor_id) VALUES
('ENG101', 'English Composition', 'English', 3, NULL);

-- Insert Enrollments
INSERT INTO enrollments (enrollment_id, student_id, course_id, semester, grade, grade_points) VALUES
(101, 1, 'CS101', 'Fall 2023', 'A', 4.0);
INSERT INTO enrollments (enrollment_id, student_id, course_id, semester, grade, grade_points) VALUES
(102, 1, 'CS201', 'Spring 2024', 'B+', 3.3);
INSERT INTO enrollments (enrollment_id, student_id, course_id, semester, grade, grade_points) VALUES
(103, 2, 'MATH101', 'Fall 2023', 'A', 4.0);
INSERT INTO enrollments (enrollment_id, student_id, course_id, semester, grade, grade_points) VALUES
(104, 2, 'CS101', 'Fall 2023', 'A-', 3.7);
INSERT INTO enrollments (enrollment_id, student_id, course_id, semester, grade, grade_points) VALUES
(105, 3, 'CS101', 'Spring 2024', 'B', 3.0);
INSERT INTO enrollments (enrollment_id, student_id, course_id, semester, grade, grade_points) VALUES
(106, 3, 'CS201', 'Spring 2024', 'B+', 3.3);
INSERT INTO enrollments (enrollment_id, student_id, course_id, semester, grade, grade_points) VALUES
(107, 4, 'PHYS101', 'Spring 2024', 'A', 4.0);
INSERT INTO enrollments (enrollment_id, student_id, course_id, semester, grade, grade_points) VALUES
(108, 1, 'CS301', 'Fall 2024', NULL, NULL);

COMMIT;
```

</details>

## Why Use Sequences?

**Benefits:**
- **Unique values guaranteed** across all sessions
- **Independent of tables** - can be shared
- **Better performance** than querying MAX(id) + 1
- **No locks** on tables during generation
- **Centralized control** over number generation

## Creating Sequences

**Basic Syntax:**
```sql
CREATE SEQUENCE sequence_name
    START WITH initial_value
    INCREMENT BY step_value
    MAXVALUE max_value | NOMAXVALUE
    MINVALUE min_value | NOMINVALUE
    CACHE cache_size | NOCACHE
    CYCLE | NOCYCLE;
```

**Common Parameters:**

| Parameter | Description | Default |
|-----------|-------------|---------|
| `START WITH` | Initial value | 1 |
| `INCREMENT BY` | Step size | 1 |
| `MAXVALUE` | Maximum value | 10^27 |
| `CACHE` | Pre-allocated values | 20 |
| `CYCLE` | Restart after max/min | NOCYCLE |

**Example 1: Simple sequence for student IDs**

```sql
CREATE SEQUENCE student_id_seq
    START WITH 100
    INCREMENT BY 1
    NOCACHE
    NOCYCLE;
```

**Example 2: Sequence with caching (better performance)**

```sql
CREATE SEQUENCE enrollment_id_seq
    START WITH 1000
    INCREMENT BY 1
    CACHE 50
    NOCYCLE;
```

Caching improves performance by pre-allocating values in memory.

**Example 3: Descending sequence**

```sql
CREATE SEQUENCE countdown_seq
    START WITH 1000
    INCREMENT BY -1
    MINVALUE 1
    NOCYCLE;
```

## Using Sequences

### NEXTVAL - Get Next Value

**Syntax:** `sequence_name.NEXTVAL`

**Purpose:** Advances sequence and returns the next value.

**Example: Insert with sequence**

```sql
-- Create a sequence
CREATE SEQUENCE test_student_seq
    START WITH 100
    INCREMENT BY 1;

-- Use in INSERT
INSERT INTO students (student_id, first_name, last_name, email, major, enrollment_date, gpa)
VALUES (test_student_seq.NEXTVAL, 'New', 'Student', 'new.student@university.edu', 'Engineering', SYSDATE, 3.5);

-- Use in SELECT to see next value
SELECT test_student_seq.NEXTVAL FROM DUAL;
```

**Important:** Each call to NEXTVAL increments the sequence.

### CURRVAL - Get Current Value

**Syntax:** `sequence_name.CURRVAL`

**Purpose:** Returns the last value generated by NEXTVAL in the current session.

**Rules:**
- Must call NEXTVAL first in session
- Returns same value until NEXTVAL is called again
- Session-specific (each session has its own CURRVAL)

**Example: Use in parent-child insert**

```sql
-- Create sequence
CREATE SEQUENCE order_seq START WITH 1000;

-- Parent insert
INSERT INTO orders (order_id, customer_name, order_date)
VALUES (order_seq.NEXTVAL, 'John Doe', SYSDATE);

-- Child inserts using same order_id
INSERT INTO order_items (item_id, order_id, product_name, quantity)
VALUES (1, order_seq.CURRVAL, 'Product A', 2);

INSERT INTO order_items (item_id, order_id, product_name, quantity)
VALUES (2, order_seq.CURRVAL, 'Product B', 1);
```

CURRVAL ensures all items get the same order_id.

## Viewing Sequence Information

**Query user sequences:**

```sql
SELECT sequence_name, min_value, max_value, increment_by, last_number
FROM user_sequences
ORDER BY sequence_name;
```

**Example result:**
| sequence_name | min_value | max_value | increment_by | last_number |
|---------------|-----------|-----------|--------------|-------------|
| ENROLLMENT_ID_SEQ | 1 | 9999999... | 1 | 1050 |
| STUDENT_ID_SEQ | 1 | 9999999... | 1 | 105 |

## Modifying Sequences

**Syntax:**
```sql
ALTER SEQUENCE sequence_name
    INCREMENT BY new_step
    MAXVALUE new_max | NOMAXVALUE
    MINVALUE new_min | NOMINVALUE
    CACHE new_cache | NOCACHE
    CYCLE | NOCYCLE;
```

**Example: Change increment and cache**

```sql
ALTER SEQUENCE student_id_seq
    INCREMENT BY 5
    CACHE 100;
```

**Note:** You CANNOT alter `START WITH`. To reset, you must drop and recreate.

## Dropping Sequences

**Syntax:**
```sql
DROP SEQUENCE sequence_name;
```

**Example:**

```sql
DROP SEQUENCE test_student_seq;
```

**Warning:** Dropping a sequence does NOT affect existing data, but you cannot generate new values.

## Practical Examples

### Example 1: Auto-Increment Primary Key

```sql
-- Create sequence
CREATE SEQUENCE new_enrollment_seq
    START WITH 200
    INCREMENT BY 1
    CACHE 20;

-- Insert using sequence
INSERT INTO enrollments (enrollment_id, student_id, course_id, semester)
VALUES (new_enrollment_seq.NEXTVAL, 5, 'CS101', 'Fall 2024');

INSERT INTO enrollments (enrollment_id, student_id, course_id, semester)
VALUES (new_enrollment_seq.NEXTVAL, 5, 'CS201', 'Fall 2024');
```

### Example 2: Generate Batch Numbers

```sql
-- Useful for tracking groups of records
CREATE SEQUENCE batch_number_seq
    START WITH 1
    INCREMENT BY 1;

-- Insert multiple records with same batch
INSERT INTO import_logs (log_id, batch_number, import_date)
SELECT 
    ROWNUM,
    batch_number_seq.NEXTVAL,
    SYSDATE
FROM DUAL
CONNECT BY LEVEL <= 5;
```

## Sequence Gaps

**Why gaps occur:**
- Rollback of transaction (sequence value is not rolled back)
- Multiple sessions using same sequence
- Cached values lost during database restart

**Example:**
```sql
-- Get value 101
INSERT INTO students (...) VALUES (student_id_seq.NEXTVAL, ...);

-- Rollback
ROLLBACK;

-- Next INSERT gets 102 (101 is lost)
INSERT INTO students (...) VALUES (student_id_seq.NEXTVAL, ...);
```

**Why gaps are acceptable:**
- Primary keys only need to be unique, not consecutive
- Performance benefit outweighs gap concern
- Sequences guarantee uniqueness, not continuity

## Identity Columns (Oracle 12c+)

Modern alternative to sequences - database automatically manages ID generation.

**Syntax:**
```sql
CREATE TABLE new_students (
    student_id NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    first_name VARCHAR2(50),
    last_name VARCHAR2(50)
);
```

**Insert (no need to specify ID):**
```sql
INSERT INTO new_students (first_name, last_name)
VALUES ('Test', 'User');
```

**Identity vs. Sequence:**

| Aspect | Sequence | Identity Column |
|--------|----------|-----------------|
| **Syntax** | Separate object | Part of table definition |
| **Usage** | Manual (NEXTVAL) | Automatic |
| **Sharing** | Can share across tables | One per column |
| **Oracle Version** | All versions | 12c+ |

**Recommendation:** Use Identity columns for new development (simpler), use sequences for backward compatibility or shared number generation.

## Summary

**Key Points:**

1. **Sequences generate unique sequential numbers** for primary keys and other unique identifiers
2. **NEXTVAL** advances and returns next value; **CURRVAL** returns current value in session
3. **CREATE SEQUENCE** with START WITH, INCREMENT BY, CACHE options
4. **Sequences are independent** of tables and can be shared
5. **ALTER SEQUENCE** to modify (but cannot change START WITH)
6. **Gaps are normal** due to rollbacks, caching, and multiple sessions
7. **Identity columns** (Oracle 12c+) provide simpler auto-increment
8. **Best practices**: Use caching for performance, accept gaps, use identity columns for new projects

Sequences provide efficient, reliable unique number generation for database applications.
