# DDL: DROP and TRUNCATE Statements

## Overview

This guide covers DROP and TRUNCATE commands for removing database objects and data in PostgreSQL. These are powerful commands that permanently delete data, so use them carefully.

## DROP Statement

The `DROP` statement permanently removes database objects (tables, indexes, views, etc.) from the database.

### DROP TABLE

#### Basic Syntax

```sql
DROP TABLE table_name;
DROP TABLE IF EXISTS table_name;  -- Doesn't error if table doesn't exist
```

### Example Setup

Let's create sample tables to demonstrate DROP operations:

```sql
CREATE TABLE temp_students (
    student_id SERIAL PRIMARY KEY,
    student_name VARCHAR(100)
);

CREATE TABLE temp_courses (
    course_id VARCHAR(10) PRIMARY KEY,
    course_name VARCHAR(100)
);

CREATE TABLE temp_enrollments (
    enrollment_id SERIAL PRIMARY KEY,
    student_id INTEGER REFERENCES temp_students(student_id),
    course_id VARCHAR(10) REFERENCES temp_courses(course_id)
);

-- Insert sample data
INSERT INTO temp_students (student_name) VALUES ('John Doe'), ('Jane Smith');
INSERT INTO temp_courses (course_id, course_name) VALUES ('CS101', 'Intro to CS');
INSERT INTO temp_enrollments (student_id, course_id) VALUES (1, 'CS101');
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

### Example 2: DROP TABLE with Dependencies (CASCADE)

**Option 1: Drop in correct order**
```sql
-- Drop dependent table first
DROP TABLE temp_enrollments;
-- Then drop the referenced table
DROP TABLE temp_students;
```

**Option 2: Use CASCADE**
```sql
-- Drops table and all dependent objects automatically
DROP TABLE temp_students CASCADE;
```

### Example 3: DROP IF EXISTS

**SQL Statement:**
```sql
-- Won't error even if table doesn't exist
DROP TABLE IF EXISTS non_existent_table;

-- Safe to run multiple times
DROP TABLE IF EXISTS temp_students;
DROP TABLE IF EXISTS temp_courses;
DROP TABLE IF EXISTS temp_enrollments;
```

**Result:** Tables are dropped if they exist; no error if they don't exist.

### Example 4: DROP Multiple Tables

**SQL Statement:**
```sql
-- Create test tables
CREATE TABLE test_a (id INTEGER);
CREATE TABLE test_b (id INTEGER);
CREATE TABLE test_c (id INTEGER);

-- Drop all at once
DROP TABLE test_a, test_b, test_c;
```

**Result:** All three tables are removed in a single statement.

## TRUNCATE Statement

The `TRUNCATE` statement removes all rows from a table quickly, but keeps the table structure.

### Basic Syntax

```sql
TRUNCATE TABLE table_name;
TRUNCATE TABLE table_name RESTART IDENTITY;  -- Resets auto-increment
TRUNCATE TABLE table_name CASCADE;            -- Truncates referencing tables too
```

### Sample Data for TRUNCATE Examples

```sql
-- Create and populate sample table
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    price NUMERIC(10, 2),
    stock_quantity INTEGER DEFAULT 0
);

INSERT INTO products (product_name, price, stock_quantity) VALUES
('Laptop', 999.99, 15),
('Mouse', 25.50, 100),
('Keyboard', 75.00, 50),
('Monitor', 299.99, 30),
('Webcam', 89.99, 45);
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

**Table Structure Still Exists:**
```sql
\d products  -- PostgreSQL command to describe table

-- Structure is intact, just empty
```

### Example 2: TRUNCATE vs DELETE

Let's compare TRUNCATE with DELETE:

**Setup:**
```sql
-- Recreate and populate products
INSERT INTO products (product_name, price, stock_quantity) VALUES
('Laptop', 999.99, 15),
('Mouse', 25.50, 100);
```

**Using DELETE:**
```sql
DELETE FROM products;
```
- Removes rows one by one
- Can use WHERE clause
- Triggers fire
- Can be rolled back
- Slower on large tables

**Using TRUNCATE:**
```sql
TRUNCATE TABLE products;
```
- Removes all rows at once
- Cannot use WHERE clause
- Triggers do NOT fire
- Cannot be rolled back (in most cases)
- Much faster on large tables

### Example 3: TRUNCATE with RESTART IDENTITY

When using SERIAL columns, TRUNCATE can reset the sequence:

**Without RESTART IDENTITY:**
```sql
-- Insert data
INSERT INTO products (product_name, price) VALUES ('Laptop', 999.99);
-- product_id = 1

TRUNCATE TABLE products;

INSERT INTO products (product_name, price) VALUES ('Mouse', 25.50);
-- product_id = 2 (continues from before)
```

**Products Table:**
| product_id | product_name | price | stock_quantity |
|------------|--------------|--------|----------------|
| 2 | Mouse | 25.50 | 0 |

**With RESTART IDENTITY:**
```sql
INSERT INTO products (product_name, price) VALUES ('Laptop', 999.99);
-- product_id = 3

TRUNCATE TABLE products RESTART IDENTITY;

INSERT INTO products (product_name, price) VALUES ('Keyboard', 75.00);
-- product_id = 1 (restarted)
```

**Products Table:**
| product_id | product_name | price | stock_quantity |
|------------|--------------|--------|----------------|
| 1 | Keyboard | 75.00 | 0 |

### Example 4: TRUNCATE with Foreign Keys

**Setup:**
```sql
CREATE TABLE categories (
    category_id SERIAL PRIMARY KEY,
    category_name VARCHAR(50)
);

CREATE TABLE items (
    item_id SERIAL PRIMARY KEY,
    item_name VARCHAR(100),
    category_id INTEGER REFERENCES categories(category_id)
);

INSERT INTO categories (category_name) VALUES ('Electronics'), ('Furniture');
INSERT INTO items (item_name, category_id) VALUES 
    ('Laptop', 1),
    ('Desk', 2);
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
ERROR: cannot truncate a table referenced in a foreign key constraint
DETAIL: Table "items" references "categories"
HINT: Truncate table "items" at the same time, or use TRUNCATE ... CASCADE
```

**Solution 1: TRUNCATE CASCADE**
```sql
TRUNCATE TABLE categories CASCADE;
```
**Result:** 
- Truncates `categories` table
- Automatically truncates `items` table (because it references categories)

**Both Tables After CASCADE:**

**Categories:**
| category_id | category_name |
|-------------|---------------|
| *(no rows)* | |

**Items:**
| item_id | item_name | category_id |
|---------|-----------|-------------|
| *(no rows)* | |

**Solution 2: TRUNCATE Multiple Tables**
```sql
TRUNCATE TABLE categories, items RESTART IDENTITY;
```
**Result:** Both tables are truncated and their sequences reset.

## DELETE vs TRUNCATE vs DROP

### Comparison Table

| Feature | DELETE | TRUNCATE | DROP |
|---------|--------|----------|------|
| Removes | Rows | All rows | Entire table |
| Table structure | Kept | Kept | Removed |
| WHERE clause | Supported | Not supported | N/A |
| Speed | Slower | Faster | Fastest |
| Rollback | Can rollback | Limited rollback | Can rollback (DDL transaction) |
| Triggers | Fires | Doesn't fire | N/A |
| Auto-increment | Continues | Can reset with RESTART IDENTITY | N/A |
| Foreign keys | Checks constraints | Requires CASCADE or drop dependents | Requires CASCADE or drop dependents |

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
- You want to reset auto-increment counters
- Performance is important

```sql
TRUNCATE TABLE products RESTART IDENTITY;
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
-- Remove all test data but keep tables
TRUNCATE TABLE test_enrollments, test_students, test_courses RESTART IDENTITY CASCADE;
```

### Example 2: Remove Temporary Tables

```sql
-- Remove tables created for one-time analysis
DROP TABLE IF EXISTS temp_analysis_results;
DROP TABLE IF EXISTS temp_calculations;
DROP TABLE IF EXISTS temp_staging_data;
```

### Example 3: Clear Log Table Daily

```sql
-- Clear old logs (in a scheduled job)
TRUNCATE TABLE application_logs RESTART IDENTITY;

-- Or delete old logs only
DELETE FROM application_logs 
WHERE log_date < CURRENT_DATE - INTERVAL '30 days';
```

### Example 4: Database Reset for Development

```sql
-- Complete database reset script
DROP TABLE IF EXISTS enrollments CASCADE;
DROP TABLE IF EXISTS courses CASCADE;
DROP TABLE IF EXISTS students CASCADE;

-- Recreate fresh tables
CREATE TABLE students (...);
CREATE TABLE courses (...);
CREATE TABLE enrollments (...);
```

## Quick Reference

```sql
-- DROP TABLE
DROP TABLE table_name;
DROP TABLE IF EXISTS table_name;
DROP TABLE table_name CASCADE;                    -- Drop with all dependencies
DROP TABLE table1, table2, table3;                -- Drop multiple tables

-- TRUNCATE TABLE  
TRUNCATE TABLE table_name;
TRUNCATE TABLE table_name RESTART IDENTITY;       -- Reset auto-increment
TRUNCATE TABLE table_name CASCADE;                -- Truncate referencing tables
TRUNCATE TABLE table1, table2;                    -- Truncate multiple tables

-- DELETE (for comparison)
DELETE FROM table_name;
DELETE FROM table_name WHERE condition;
```

