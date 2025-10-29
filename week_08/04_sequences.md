# Sequences

## Overview

A **sequence** is a database object that generates unique numeric values automatically. Sequences are commonly used to create primary key values, ensuring each new row gets a unique identifier without manual intervention or application logic. They provide a reliable, high-performance mechanism for generating sequential numbers in multi-user environments.

## Key Terms

**Sequence**: A database object that generates a series of unique numbers according to specified rules.

**NEXTVAL**: Pseudo-column that generates and returns the next value from a sequence.

**CURRVAL**: Pseudo-column that returns the current value of a sequence (last value generated in this session).

**INCREMENT BY**: Specifies the interval between sequence numbers.

**START WITH**: Specifies the first sequence number to be generated.

**MAXVALUE/MINVALUE**: Upper and lower bounds for sequence values.

**CYCLE**: Option to restart sequence from beginning when limit is reached.

**CACHE**: Number of sequence values pre-allocated in memory for performance.

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

### Benefits

1. **Automatic unique values** - No manual tracking needed
2. **Concurrency safe** - Multiple users can get unique values simultaneously
3. **Performance** - Faster than application-generated IDs
4. **Centralized** - Single source of truth for ID generation
5. **Independent** - Not tied to specific tables or transactions
6. **Scalable** - Handles high-volume insertions efficiently

### Alternatives to Sequences

| Method | Pros | Cons |
|--------|------|------|
| **Sequence** | Fast, concurrent, reliable | Oracle-specific syntax |
| **IDENTITY Column** | Built into table definition | Oracle 12c+ only |
| **MAX(id)+1** | Simple logic | Slow, concurrency issues |
| **Application-generated** | Cross-database | Network overhead, gaps |
| **UUID/GUID** | Globally unique | Large storage, not sequential |

## Creating Sequences

### Basic Syntax

```sql
CREATE SEQUENCE sequence_name
    [START WITH n]
    [INCREMENT BY n]
    [MAXVALUE n | NOMAXVALUE]
    [MINVALUE n | NOMINVALUE]
    [CYCLE | NOCYCLE]
    [CACHE n | NOCACHE]
    [ORDER | NOORDER];
```

### Parameter Descriptions

| Parameter | Description | Default |
|-----------|-------------|---------|
| **START WITH** | Initial value | MINVALUE for ascending, MAXVALUE for descending |
| **INCREMENT BY** | Value to add each time | 1 |
| **MINVALUE** | Minimum value | 1 for ascending, -10^26 for descending |
| **MAXVALUE** | Maximum value | 10^27 for ascending, -1 for descending |
| **CYCLE** | Restart when limit reached | NOCYCLE |
| **CACHE** | Pre-allocate values in memory | 20 |
| **ORDER** | Guarantee order in RAC | NOORDER |

### Example 1: Simple Sequence

```sql
CREATE SEQUENCE student_seq
    START WITH 1000
    INCREMENT BY 1
    NOCYCLE
    NOCACHE;
```

**Explanation:**
- Starts at 1000
- Increments by 1 (1000, 1001, 1002, ...)
- No cycling (stops at maximum)
- No caching (each value generated from disk)

### Example 2: Sequence with Caching

```sql
CREATE SEQUENCE enrollment_seq
    START WITH 1
    INCREMENT BY 1
    MAXVALUE 999999
    NOCYCLE
    CACHE 50;
```

**Explanation:**
- Starts at 1
- Caches 50 values for performance
- Stops at 999999 (no cycling)

### Example 3: Descending Sequence

```sql
CREATE SEQUENCE countdown_seq
    START WITH 100
    INCREMENT BY -1
    MINVALUE 1
    NOCYCLE
    NOCACHE;
```

**Explanation:**
- Starts at 100
- Decrements by 1 (100, 99, 98, ...)
- Stops at 1

### Example 4: Sequence with Gaps

```sql
CREATE SEQUENCE order_seq
    START WITH 10000
    INCREMENT BY 10
    NOCYCLE
    CACHE 20;
```

**Explanation:**
- Starts at 10000
- Increments by 10 (10000, 10010, 10020, ...)
- Leaves gaps for manual adjustment

### Example 5: Cycling Sequence

```sql
CREATE SEQUENCE rotation_seq
    START WITH 1
    INCREMENT BY 1
    MAXVALUE 5
    CYCLE
    NOCACHE;
```

**Explanation:**
- Cycles through 1, 2, 3, 4, 5, then repeats
- Useful for round-robin assignments

## Using Sequences

### NEXTVAL - Get Next Value

**Syntax:**
```sql
sequence_name.NEXTVAL
```

**Example: Insert with Sequence**

```sql
INSERT INTO students (student_id, first_name, last_name, email, major)
VALUES (student_seq.NEXTVAL, 'Sarah', 'Johnson', 'sarah.j@university.edu', 'Biology');
```

**Example: Get Value in SELECT**

```sql
SELECT student_seq.NEXTVAL AS next_id FROM DUAL;
```

**Result:**
| next_id |
|---------|
| 1000 |

**Example: Multiple Inserts**

```sql
INSERT INTO students (student_id, first_name, last_name, email)
VALUES (student_seq.NEXTVAL, 'Tom', 'Brown', 'tom.b@university.edu');

INSERT INTO students (student_id, first_name, last_name, email)
VALUES (student_seq.NEXTVAL, 'Lisa', 'White', 'lisa.w@university.edu');
-- Tom gets 1001, Lisa gets 1002
```

### CURRVAL - Get Current Value

**Syntax:**
```sql
sequence_name.CURRVAL
```

**Important:** CURRVAL can only be called after NEXTVAL in the same session.

**Example: Use Current Value**

```sql
-- Generate new student ID
INSERT INTO students (student_id, first_name, last_name, email)
VALUES (student_seq.NEXTVAL, 'Mike', 'Davis', 'mike.d@university.edu');

-- Use same ID for related record
INSERT INTO student_profiles (student_id, bio, photo_url)
VALUES (student_seq.CURRVAL, 'Computer Science major', '/photos/mike.jpg');
```

**Example: Return Inserted ID**

```sql
INSERT INTO students (student_id, first_name, last_name, email)
VALUES (student_seq.NEXTVAL, 'Emma', 'Wilson', 'emma.w@university.edu');

SELECT student_seq.CURRVAL AS new_student_id FROM DUAL;
```

**Result:**
| new_student_id |
|----------------|
| 1003 |

**Error Example:**

```sql
-- ERROR: sequence not yet defined in this session
SELECT student_seq.CURRVAL FROM DUAL;
-- Must call NEXTVAL first!
```

## Sequence Operations

### Viewing Sequence Information

```sql
-- Query user sequences
SELECT 
    sequence_name,
    min_value,
    max_value,
    increment_by,
    last_number,
    cache_size,
    cycle_flag
FROM user_sequences;
```

**Result:**
| sequence_name | min_value | max_value | increment_by | last_number | cache_size | cycle_flag |
|---------------|-----------|-----------|--------------|-------------|------------|------------|
| STUDENT_SEQ | 1 | 10^27 | 1 | 1000 | 0 | N |
| ENROLLMENT_SEQ | 1 | 999999 | 1 | 150 | 50 | N |

### Current Sequence Value (Across Sessions)

```sql
-- See what next value will be
SELECT sequence_name, last_number 
FROM user_sequences 
WHERE sequence_name = 'STUDENT_SEQ';
```

**Note:** `last_number` shows the last allocated value (may be cached, not yet used).

## Modifying Sequences

### ALTER SEQUENCE

You can modify most sequence properties except START WITH.

**Syntax:**
```sql
ALTER SEQUENCE sequence_name
    [INCREMENT BY n]
    [MAXVALUE n | NOMAXVALUE]
    [MINVALUE n | NOMINVALUE]
    [CYCLE | NOCYCLE]
    [CACHE n | NOCACHE];
```

**Example: Change Increment**

```sql
ALTER SEQUENCE student_seq
    INCREMENT BY 5;
```

**Example: Change Cache Size**

```sql
ALTER SEQUENCE enrollment_seq
    CACHE 100;  -- Increase cache for better performance
```

**Example: Enable Cycling**

```sql
ALTER SEQUENCE rotation_seq
    CYCLE;
```

**Example: Change Maximum Value**

```sql
ALTER SEQUENCE order_seq
    MAXVALUE 999999;
```

### Reset Sequence (Workaround)

You cannot directly reset START WITH, but you can recreate:

```sql
-- Drop and recreate
DROP SEQUENCE student_seq;

CREATE SEQUENCE student_seq
    START WITH 1
    INCREMENT BY 1;
```

**Alternative - Adjust with ALTER:**

```sql
-- To reset to specific value (e.g., 5000)
-- First, find current value
SELECT last_number FROM user_sequences WHERE sequence_name = 'STUDENT_SEQ';
-- Result: 1050

-- Calculate adjustment needed: 5000 - 1050 = 3950
ALTER SEQUENCE student_seq INCREMENT BY 3950;
SELECT student_seq.NEXTVAL FROM DUAL;  -- Advances to 5000

-- Reset increment to original
ALTER SEQUENCE student_seq INCREMENT BY 1;
```

## Dropping Sequences

### DROP SEQUENCE

```sql
DROP SEQUENCE sequence_name;
```

**Example:**

```sql
DROP SEQUENCE old_sequence;
```

**Important:** Once dropped, all references to the sequence will fail. Ensure no tables or applications depend on it.

## Practical Examples

### Example 1: Basic Auto-Increment Primary Key

```sql
-- Create sequence
CREATE SEQUENCE course_seq START WITH 100 INCREMENT BY 1 NOCACHE;

-- Create table
CREATE TABLE courses (
    course_id NUMBER PRIMARY KEY,
    course_name VARCHAR2(100) NOT NULL,
    credits NUMBER
);

-- Insert with sequence
INSERT INTO courses (course_id, course_name, credits)
VALUES (course_seq.NEXTVAL, 'Advanced Databases', 3);

INSERT INTO courses (course_id, course_name, credits)
VALUES (course_seq.NEXTVAL, 'Machine Learning', 4);
```

### Example 2: Order Number Generation

```sql
-- Create sequence for order numbers
CREATE SEQUENCE order_number_seq
    START WITH 20240001
    INCREMENT BY 1
    CACHE 100;

-- Use in order table
INSERT INTO orders (order_id, customer_id, order_date, order_number)
VALUES (orders_seq.NEXTVAL, 42, SYSDATE, order_number_seq.NEXTVAL);
```

### Example 3: Parent-Child Relationship

```sql
-- Create sequence
CREATE SEQUENCE transaction_seq START WITH 1 INCREMENT BY 1 CACHE 50;

-- Insert parent record
INSERT INTO transactions (transaction_id, transaction_date, amount)
VALUES (transaction_seq.NEXTVAL, SYSDATE, 100.00);

-- Insert child records with same ID
INSERT INTO transaction_details (transaction_id, item_id, quantity)
VALUES (transaction_seq.CURRVAL, 'ITEM001', 2);

INSERT INTO transaction_details (transaction_id, item_id, quantity)
VALUES (transaction_seq.CURRVAL, 'ITEM002', 1);
```

### Example 4: Multi-Table Insert

```sql
-- Single sequence for multiple related tables
CREATE SEQUENCE entity_seq START WITH 1000 INCREMENT BY 1;

-- Insert into different tables
INSERT INTO customers (customer_id, customer_name)
VALUES (entity_seq.NEXTVAL, 'ABC Corp');

INSERT INTO suppliers (supplier_id, supplier_name)
VALUES (entity_seq.NEXTVAL, 'XYZ Supplies');

-- Both get unique IDs from same sequence
```

### Example 5: Batch Number Generation

```sql
-- Generate batch of IDs
CREATE SEQUENCE batch_seq START WITH 1 INCREMENT BY 1 CACHE 20;

-- Generate 5 IDs
SELECT batch_seq.NEXTVAL AS id FROM DUAL CONNECT BY LEVEL <= 5;
```

**Result:**
| id |
|----|
| 1 |
| 2 |
| 3 |
| 4 |
| 5 |

## Sequence Gaps

### Why Gaps Occur

Gaps in sequence values are normal and expected:

1. **Rollback** - Transaction uses NEXTVAL but rolls back
2. **Caching** - Cached values lost on database restart
3. **Multiple tables** - Same sequence used across tables
4. **Failed inserts** - Constraint violations after NEXTVAL called

### Example: Gap from Rollback

```sql
INSERT INTO students (student_id, first_name, last_name, email)
VALUES (student_seq.NEXTVAL, 'Test', 'User', 'invalid-email');
-- Value 1005 used

ROLLBACK;
-- Value 1005 is lost forever

INSERT INTO students (student_id, first_name, last_name, email)
VALUES (student_seq.NEXTVAL, 'Real', 'User', 'real@university.edu');
-- Gets value 1006 (1005 is skipped)
```

### Why Gaps Are Acceptable

- **Performance** - No locking needed, high concurrency
- **Uniqueness** - Only requirement for primary keys
- **Predictability** - Sequences never duplicate values

## Identity Columns (Oracle 12c+)

An alternative to sequences with simpler syntax.

### Syntax

```sql
CREATE TABLE table_name (
    id NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    other_columns ...
);
```

### Example: Identity Column

```sql
CREATE TABLE departments (
    dept_id NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    dept_name VARCHAR2(50) NOT NULL,
    building VARCHAR2(50)
);

-- Insert without specifying dept_id
INSERT INTO departments (dept_name, building)
VALUES ('Computer Science', 'Science Hall');

INSERT INTO departments (dept_name, building)
VALUES ('Mathematics', 'Math Building');
-- dept_id automatically assigned (1, 2, ...)
```

### Identity Options

```sql
CREATE TABLE employees (
    emp_id NUMBER GENERATED BY DEFAULT AS IDENTITY (
        START WITH 1000
        INCREMENT BY 1
        CACHE 20
    ) PRIMARY KEY,
    emp_name VARCHAR2(100)
);
```

**GENERATED ALWAYS:** Cannot manually insert values
**GENERATED BY DEFAULT:** Allows manual insertion if needed

### Comparison: Sequence vs. Identity

| Aspect | Sequence | Identity Column |
|--------|----------|-----------------|
| **Oracle Version** | All versions | 12c+ |
| **Flexibility** | Can use across tables | Tied to one column |
| **Syntax** | Requires NEXTVAL | Automatic |
| **Manual Values** | Always allowed | Only with BY DEFAULT |
| **Reusability** | High | Low |

## Common Mistakes and Solutions

### Mistake 1: Forgetting NEXTVAL

**Problem:**
```sql
INSERT INTO students (student_id, first_name, last_name)
VALUES (student_seq, 'John', 'Doe');
-- ERROR: invalid number
```

**Solution:**
```sql
INSERT INTO students (student_id, first_name, last_name)
VALUES (student_seq.NEXTVAL, 'John', 'Doe');
```

### Mistake 2: Using CURRVAL Before NEXTVAL

**Problem:**
```sql
-- In a new session
SELECT student_seq.CURRVAL FROM DUAL;
-- ERROR: sequence not yet defined in this session
```

**Solution:**
```sql
-- Call NEXTVAL first
SELECT student_seq.NEXTVAL FROM DUAL;
-- Now CURRVAL works
SELECT student_seq.CURRVAL FROM DUAL;
```

### Mistake 3: Expecting No Gaps

**Problem:** Gaps occur naturally; don't use sequences if gaps are unacceptable.

**Solution:** If gapless numbers are required, use application logic with locking:
```sql
-- Lock the table
SELECT MAX(order_num) + 1 INTO next_num FROM orders FOR UPDATE;
INSERT INTO orders (order_num, ...) VALUES (next_num, ...);
COMMIT;
```

**Note:** This is slower and less scalable than sequences.

### Mistake 4: Using MAX(id)+1

**Problem:**
```sql
-- Concurrency issues, slow performance
INSERT INTO students (student_id, first_name, last_name)
VALUES ((SELECT MAX(student_id)+1 FROM students), 'Jane', 'Doe');
```

**Solution:** Use a sequence instead:
```sql
INSERT INTO students (student_id, first_name, last_name)
VALUES (student_seq.NEXTVAL, 'Jane', 'Doe');
```

## Performance Tips

### Best Practices

1. **Use appropriate CACHE size** - Larger cache for high-volume tables
2. **NOCACHE for low-volume** - Avoids wasting numbers on restart
3. **One sequence per table** - Unless intentionally sharing
4. **Pre-allocate if needed** - Generate IDs in batch for bulk operations
5. **Monitor sequence exhaustion** - Check MAXVALUE limits

### Cache Size Guidelines

| Insert Volume | Recommended Cache |
|---------------|-------------------|
| Low (< 100/day) | NOCACHE or 20 |
| Medium (100s/day) | 50-100 |
| High (1000s/day) | 100-1000 |
| Very High (millions/day) | 1000+ |

### Example: Optimize for Performance

```sql
-- High-volume table
CREATE SEQUENCE transaction_seq
    START WITH 1
    INCREMENT BY 1
    CACHE 1000;    -- Large cache for performance

-- Low-volume table
CREATE SEQUENCE audit_seq
    START WITH 1
    INCREMENT BY 1
    NOCACHE;       -- No cache to avoid waste
```

## Summary

**Key Takeaways:**

1. **Sequences generate unique numeric values automatically**, ideal for primary keys and ensuring uniqueness across concurrent users.

2. **NEXTVAL generates the next value** while CURRVAL returns the current value (only after NEXTVAL has been called in the session).

3. **Key parameters**: START WITH (initial value), INCREMENT BY (step size), MAXVALUE/MINVALUE (bounds), CYCLE (restart option), CACHE (performance).

4. **Gaps are normal and expected** due to rollbacks, caching, database restarts, or failed inserts - sequences prioritize performance over gapless sequences.

5. **Common pattern**: `CREATE SEQUENCE → INSERT with NEXTVAL → Use CURRVAL if needed for related records`.

6. **Identity columns (12c+)** provide simpler syntax for single-column auto-increment but are less flexible than traditional sequences.

7. **Best practices**: Use appropriate cache size for volume, one sequence per table (usually), monitor for approaching MAXVALUE, use NOCACHE for low-volume tables.

8. **Cannot directly reset START WITH** - must drop and recreate sequence or use ALTER with calculated INCREMENT BY.

Sequences are essential database objects for generating unique identifiers efficiently in multi-user environments without application-level coordination.

