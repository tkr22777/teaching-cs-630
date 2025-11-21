# DDL: DROP and TRUNCATE Statements

## Overview

This guide covers DROP and TRUNCATE commands. **DROP removes the table itself; TRUNCATE empties the table but keeps it.** These are powerful commands that permanently delete data, so use them carefully.

## DROP Statement

When you need to completely remove a table from your database - structure and all data - you'll use DROP. This is permanent and can't be undone, so you'll typically use this when removing test tables, obsolete tables, or during development when restructuring your database.

### DROP TABLE

#### Basic Syntax

```sql
DROP TABLE table_name;
DROP TABLE table_name CASCADE CONSTRAINTS;  -- Drops table and dependent constraints
```

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

When a table is referenced by foreign keys in other tables, you need `CASCADE CONSTRAINTS` to drop it. This automatically removes those dependent foreign key constraints.

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

The key is to drop child tables (those with foreign keys) before parent tables (those being referenced).

---

## TRUNCATE Statement

When you want to keep a table but remove all its data quickly, use TRUNCATE. This is much faster than DELETE for clearing entire tables, and it's commonly used when refreshing data or clearing test data between runs.

### Basic Syntax

```sql
TRUNCATE TABLE table_name;
```

### Sample Data for TRUNCATE Examples

We'll use a simple products table with some sample data:

```sql
CREATE TABLE products (
    product_id INTEGER PRIMARY KEY,
    product_name VARCHAR2(100) NOT NULL,
    price NUMBER(10, 2)
);

INSERT INTO products VALUES (1, 'Laptop', 999.99);
INSERT INTO products VALUES (2, 'Mouse', 25.50);
INSERT INTO products VALUES (3, 'Keyboard', 75.00);
```

**Products Table (Before TRUNCATE):**
| product_id | product_name | price |
|------------|--------------|--------|
| 1 | Laptop | 999.99 |
| 2 | Mouse | 25.50 |
| 3 | Keyboard | 75.00 |

### Example 1: Basic TRUNCATE

**SQL Statement:**
```sql
TRUNCATE TABLE products;
```

**Products Table (After TRUNCATE):**
| product_id | product_name | price |
|------------|--------------|--------|
| *(no rows)* | | |

**Verification:**
```sql
SELECT COUNT(*) FROM products;
```
**Result:**
| count |
|-------|
| 0 |

**Table Structure Still Exists:** The table definition remains, only data is removed. You can immediately start inserting new data without recreating the table.

Foreign keys add a complication - you can't truncate a parent table if child tables reference it. Let's see how to handle this.

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

**Solution: Truncate in Correct Order**
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

---

## DELETE vs TRUNCATE vs DROP

| Command  | What It Does                  | Use When                                   |
|----------|-------------------------------|--------------------------------------------|
| DELETE   | Removes specific rows         | You need a WHERE clause to select rows     |
| TRUNCATE | Removes all rows, keeps table | Clearing all data quickly                  |
| DROP     | Removes entire table          | Table no longer needed                     |

## Summary

**DROP** removes entire table structure and data permanently. **TRUNCATE** removes all data but keeps table structure. **DELETE** removes specific rows (covered in DML section).
