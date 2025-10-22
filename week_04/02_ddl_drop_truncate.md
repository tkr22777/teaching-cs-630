# DDL: DROP and TRUNCATE Statements

## Overview

This guide covers DROP and TRUNCATE commands for removing database objects and data using Oracle SQL. These are powerful commands that permanently delete data, so use them carefully.

## DROP Statement

The `DROP` statement permanently removes database objects (tables, indexes, views, etc.) from the database.

### DROP TABLE

#### Basic Syntax

```sql
DROP TABLE table_name;
DROP TABLE table_name CASCADE CONSTRAINTS;  -- Drops table and dependent constraints
```

**Note:** Oracle does not support `IF EXISTS` in DROP TABLE. To avoid errors, use PL/SQL blocks or handle exceptions in your application code.

### Example Setup

Let's create sample tables to demonstrate DROP operations:

```sql
CREATE TABLE temp_students (
    student_id INTEGER PRIMARY KEY,
    student_name VARCHAR2(100)
);

CREATE TABLE temp_courses (
    course_id VARCHAR2(10) PRIMARY KEY,
    course_name VARCHAR2(100)
);

CREATE TABLE temp_enrollments (
    enrollment_id INTEGER PRIMARY KEY,
    student_id INTEGER REFERENCES temp_students(student_id),
    course_id VARCHAR2(10) REFERENCES temp_courses(course_id)
);

-- Insert sample data
INSERT INTO temp_students (student_id, student_name) VALUES (1, 'John Doe');
INSERT INTO temp_students (student_id, student_name) VALUES (2, 'Jane Smith');
INSERT INTO temp_courses (course_id, course_name) VALUES ('CS101', 'Intro to CS');
INSERT INTO temp_enrollments (enrollment_id, student_id, course_id) VALUES (1, 1, 'CS101');
```

**Temp_Students Table:**
| student_id | student_name |
|------------|--------------|
| 1 | John Doe |
| 2 | Jane Smith |

**Temp_Courses Table:**
| course_id | course_name |
|-----------|-------------|
| CS101 | Intro to CS |

**Temp_Enrollments Table:**
| enrollment_id | student_id | course_id |
|---------------|------------|-----------|
| 1 | 1 | CS101 |

### Example 1: Simple DROP TABLE

**SQL Statement:**
```sql
DROP TABLE temp_enrollments;
```

**Result:** The `temp_enrollments` table is permanently deleted from the database.

**Verification:**
```sql
SELECT * FROM temp_enrollments;
-- Error: relation "temp_enrollments" does not exist
```

### Example 2: DROP TABLE with Dependencies (CASCADE CONSTRAINTS)

```sql
-- Drops table and all dependent constraints automatically
DROP TABLE temp_students CASCADE CONSTRAINTS;
```

**Result:** The table and all foreign key constraints referencing it are dropped.

### Example 3: Dropping Multiple Tables

**SQL Statement:**
```sql
-- Drop tables in correct order (child tables first)
DROP TABLE temp_enrollments;
DROP TABLE temp_courses;
DROP TABLE temp_students;
```

**Result:** All tables are dropped successfully when done in the correct dependency order.

###

## TRUNCATE Statement

The `TRUNCATE` statement removes all rows from a table quickly, but keeps the table structure.

### Basic Syntax

```sql
TRUNCATE TABLE table_name;
TRUNCATE TABLE table_name DROP STORAGE;      -- Deallocates freed space
TRUNCATE TABLE table_name REUSE STORAGE;     -- Keeps allocated space (default)
```

**Note:** Oracle does not have `RESTART IDENTITY` like PostgreSQL. To reset sequences, you must manually reset them after truncating.

### Sample Data for TRUNCATE Examples

```sql
-- Create and populate sample table
CREATE TABLE products (
    product_id INTEGER PRIMARY KEY,
    product_name VARCHAR2(100) NOT NULL,
    price NUMBER(10, 2),
    stock_quantity INTEGER DEFAULT 0
);

INSERT INTO products (product_id, product_name, price, stock_quantity) VALUES (1, 'Laptop', 999.99, 15);
INSERT INTO products (product_id, product_name, price, stock_quantity) VALUES (2, 'Mouse', 25.50, 100);
INSERT INTO products (product_id, product_name, price, stock_quantity) VALUES (3, 'Keyboard', 75.00, 50);
INSERT INTO products (product_id, product_name, price, stock_quantity) VALUES (4, 'Monitor', 299.99, 30);
INSERT INTO products (product_id, product_name, price, stock_quantity) VALUES (5, 'Webcam', 89.99, 45);
```

**Products Table (Before TRUNCATE):**
| product_id | product_name | price | stock_quantity |
|------------|--------------|--------|----------------|
| 1 | Laptop | 999.99 | 15 |
| 2 | Mouse | 25.50 | 100 |
| 3 | Keyboard | 75.00 | 50 |
| 4 | Monitor | 299.99 | 30 |
| 5 | Webcam | 89.99 | 45 |

### Example 1: Basic TRUNCATE

**SQL Statement:**
```sql
TRUNCATE TABLE products;
```

**Products Table (After TRUNCATE):**
| product_id | product_name | price | stock_quantity |
|------------|--------------|--------|----------------|
| *(no rows)* | | | |

**Verification:**
```sql
SELECT COUNT(*) FROM products;
```
**Result:**
| count |
|-------|
| 0 |

**Table Structure Still Exists:** The table definition remains, only data is removed.

###

### Example 3: Resetting Sequences After TRUNCATE

When using sequences for auto-incrementing keys, you must manually reset them after truncating:

**Example with Sequence:**
```sql
-- Create sequence for product_id
CREATE SEQUENCE product_seq START WITH 1 INCREMENT BY 1;

-- Create table using sequence
CREATE TABLE products_seq (
    product_id INTEGER PRIMARY KEY,
    product_name VARCHAR2(100),
    price NUMBER(10, 2)
);

-- Insert using sequence
INSERT INTO products_seq VALUES (product_seq.NEXTVAL, 'Laptop', 999.99);
INSERT INTO products_seq VALUES (product_seq.NEXTVAL, 'Mouse', 25.50);

-- Truncate table
TRUNCATE TABLE products_seq;

-- Reset sequence to start from 1 again
ALTER SEQUENCE product_seq RESTART START WITH 1;

-- Next insert will use product_id = 1
INSERT INTO products_seq VALUES (product_seq.NEXTVAL, 'Keyboard', 75.00);
```

**Products_Seq Table After Reset:**
| product_id | product_name | price |
|------------|--------------|--------|
| 1 | Keyboard | 75.00 |

### Example 2: TRUNCATE with Foreign Keys

**Setup:**
```sql
CREATE TABLE categories (
    category_id INTEGER PRIMARY KEY,
    category_name VARCHAR2(50)
);

CREATE TABLE items (
    item_id INTEGER PRIMARY KEY,
    item_name VARCHAR2(100),
    category_id INTEGER REFERENCES categories(category_id)
);

INSERT INTO categories (category_id, category_name) VALUES (1, 'Electronics');
INSERT INTO categories (category_id, category_name) VALUES (2, 'Furniture');
INSERT INTO items (item_id, item_name, category_id) VALUES (1, 'Laptop', 1);
INSERT INTO items (item_id, item_name, category_id) VALUES (2, 'Desk', 2);
```

**Categories Table:**
| category_id | category_name |
|-------------|---------------|
| 1 | Electronics |
| 2 | Furniture |

**Items Table:**
| item_id | item_name | category_id |
|---------|-----------|-------------|
| 1 | Laptop | 1 |
| 2 | Desk | 2 |

**Attempt to TRUNCATE Referenced Table:**
```sql
TRUNCATE TABLE categories;
```

**Error:**
```
ORA-02266: unique/primary keys in table referenced by enabled foreign keys
```

**Solution 1: Disable and Re-enable Constraints**
```sql
-- Disable foreign key constraint
ALTER TABLE items DISABLE CONSTRAINT items_category_fk;

-- Truncate both tables
TRUNCATE TABLE categories;
TRUNCATE TABLE items;

-- Re-enable foreign key constraint
ALTER TABLE items ENABLE CONSTRAINT items_category_fk;
```

**Solution 2: Truncate in Correct Order**
```sql
-- Truncate child table first
TRUNCATE TABLE items;
-- Then truncate parent table
TRUNCATE TABLE categories;
```

**Both Tables After Truncate:**

**Categories:**
| category_id | category_name |
|-------------|---------------|
| *(no rows)* | |

**Items:**
| item_id | item_name | category_id |
|---------|-----------|-------------|
| *(no rows)* | |

## DELETE vs TRUNCATE vs DROP

### Comparison Table

| Feature | DELETE | TRUNCATE | DROP |
|---------|--------|----------|------|
| Removes | Rows | All rows | Entire table |
| Table structure | Kept | Kept | Removed |
| WHERE clause | Supported | Not supported | N/A |
| Speed | Slower | Faster | Fastest |
| Rollback | Can rollback | Cannot rollback (DDL) | Cannot rollback (DDL) |
| Triggers | Fires | Doesn't fire | N/A |
| Sequences | Continues | Must manually reset | N/A |
| Foreign keys | Checks constraints | Must disable or truncate child first | Use CASCADE CONSTRAINTS |

### When to Use Each

**Use DELETE when:**
- You need to remove specific rows (with WHERE clause)
- You need triggers to fire
- You want transaction safety

```sql
DELETE FROM products WHERE price < 10;
```

**Use TRUNCATE when:**
- You need to remove ALL rows quickly
- Performance is important
- You can manually reset sequences if needed

```sql
TRUNCATE TABLE products;
```

**Use DROP when:**
- You want to completely remove the table
- The table is no longer needed
- You're restructuring the database

```sql
DROP TABLE products;
```

## Practical Examples

### Example 1: Clean Up Test Data

```sql
-- Remove all test data but keep tables (truncate in correct order)
TRUNCATE TABLE test_enrollments;
TRUNCATE TABLE test_students;
TRUNCATE TABLE test_courses;
```

### Example 2: Remove Temporary Tables

```sql
-- Remove tables created for one-time analysis
DROP TABLE temp_staging_data;
DROP TABLE temp_calculations;
DROP TABLE temp_analysis_results;
```

### Example 3: Clear Log Table Daily

```sql
-- Clear old logs (in a scheduled job)
TRUNCATE TABLE application_logs;

-- Or delete old logs only
DELETE FROM application_logs 
WHERE log_date < SYSDATE - 30;
```

### Example 4: Database Reset for Development

```sql
-- Complete database reset script (drop in correct order)
DROP TABLE enrollments CASCADE CONSTRAINTS;
DROP TABLE courses CASCADE CONSTRAINTS;
DROP TABLE students CASCADE CONSTRAINTS;

-- Recreate fresh tables
CREATE TABLE students (...);
CREATE TABLE courses (...);
CREATE TABLE enrollments (...);
```

