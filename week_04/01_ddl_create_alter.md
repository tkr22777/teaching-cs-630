# DDL: CREATE and ALTER Statements

## Overview

Data Definition Language (DDL) statements define and modify database structure. This guide covers CREATE and ALTER commands for creating and modifying tables in PostgreSQL.

## CREATE TABLE

The `CREATE TABLE` statement creates a new table in the database.

### Basic Syntax

```sql
CREATE TABLE table_name (
    column1 datatype constraints,
    column2 datatype constraints,
    ...
    table_constraints
);
```

### Example 1: Simple Table with Primary Key

**SQL Statement:**
```sql
CREATE TABLE students (
    student_id SERIAL PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE,
    enrollment_date DATE DEFAULT CURRENT_DATE
);
```

**Table Structure Created:**
| Column | Type | Constraints |
|--------|------|-------------|
| student_id | SERIAL | PRIMARY KEY |
| first_name | VARCHAR(50) | NOT NULL |
| last_name | VARCHAR(50) | NOT NULL |
| email | VARCHAR(100) | UNIQUE |
| enrollment_date | DATE | DEFAULT CURRENT_DATE |

### Example 2: Table with Foreign Key

**SQL Statement:**
```sql
CREATE TABLE courses (
    course_id VARCHAR(10) PRIMARY KEY,
    course_name VARCHAR(100) NOT NULL,
    credits INTEGER CHECK (credits > 0 AND credits <= 6),
    department VARCHAR(50) NOT NULL,
    instructor_id INTEGER,
    FOREIGN KEY (instructor_id) REFERENCES instructors(instructor_id)
);
```

**Table Structure Created:**
| Column | Type | Constraints |
|--------|------|-------------|
| course_id | VARCHAR(10) | PRIMARY KEY |
| course_name | VARCHAR(100) | NOT NULL |
| credits | INTEGER | CHECK (credits > 0 AND credits <= 6) |
| department | VARCHAR(50) | NOT NULL |
| instructor_id | INTEGER | FOREIGN KEY → instructors(instructor_id) |

### Example 3: Junction Table with Composite Primary Key

**SQL Statement:**
```sql
CREATE TABLE enrollments (
    student_id INTEGER,
    course_id VARCHAR(10),
    semester VARCHAR(20),
    grade VARCHAR(2),
    enrollment_date DATE DEFAULT CURRENT_DATE,
    PRIMARY KEY (student_id, course_id, semester),
    FOREIGN KEY (student_id) REFERENCES students(student_id),
    FOREIGN KEY (course_id) REFERENCES courses(course_id)
);
```

**Table Structure Created:**
| Column | Type | Constraints |
|--------|------|-------------|
| student_id | INTEGER | Part of PRIMARY KEY, FOREIGN KEY |
| course_id | VARCHAR(10) | Part of PRIMARY KEY, FOREIGN KEY |
| semester | VARCHAR(20) | Part of PRIMARY KEY |
| grade | VARCHAR(2) | |
| enrollment_date | DATE | DEFAULT CURRENT_DATE |

## PostgreSQL Data Types

### Common Data Types

| Category | Type | Description | Example |
|----------|------|-------------|---------|
| Integer | INTEGER | 4-byte integer | 42 |
| Integer | BIGINT | 8-byte integer | 9223372036854775807 |
| Integer | SERIAL | Auto-incrementing integer | 1, 2, 3... |
| Decimal | NUMERIC(p,s) | Exact decimal | NUMERIC(10,2) |
| Decimal | REAL | 4-byte floating point | 3.14159 |
| String | VARCHAR(n) | Variable-length string | 'Hello' |
| String | CHAR(n) | Fixed-length string | 'ABC' |
| String | TEXT | Unlimited length text | 'Long text...' |
| Date/Time | DATE | Calendar date | '2024-01-15' |
| Date/Time | TIMESTAMP | Date and time | '2024-01-15 10:30:00' |
| Boolean | BOOLEAN | True/false | TRUE, FALSE |

### Constraints

| Constraint | Purpose | Example |
|------------|---------|---------|
| PRIMARY KEY | Uniquely identifies each row | PRIMARY KEY |
| FOREIGN KEY | References another table | FOREIGN KEY (col) REFERENCES table(col) |
| NOT NULL | Column cannot be NULL | NOT NULL |
| UNIQUE | All values must be different | UNIQUE |
| CHECK | Custom validation | CHECK (age >= 18) |
| DEFAULT | Default value if none provided | DEFAULT 0 |

## ALTER TABLE

The `ALTER TABLE` statement modifies an existing table's structure.

### Sample Data for ALTER Examples

First, let's create and populate a sample table:

```sql
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    price NUMERIC(10, 2)
);

INSERT INTO products (product_name, price) VALUES
('Laptop', 999.99),
('Mouse', 25.50),
('Keyboard', 75.00);
```

**Current Products Table:**
| product_id | product_name | price |
|------------|--------------|--------|
| 1 | Laptop | 999.99 |
| 2 | Mouse | 25.50 |
| 3 | Keyboard | 75.00 |

### ALTER: Adding a Column

**SQL Statement:**
```sql
ALTER TABLE products 
ADD COLUMN stock_quantity INTEGER DEFAULT 0;
```

**Result - Updated Table Structure:**
| product_id | product_name | price | stock_quantity |
|------------|--------------|--------|----------------|
| 1 | Laptop | 999.99 | 0 |
| 2 | Mouse | 25.50 | 0 |
| 3 | Keyboard | 75.00 | 0 |

### ALTER: Modifying Column Data Type

**SQL Statement:**
```sql
ALTER TABLE products 
ALTER COLUMN price TYPE NUMERIC(12, 2);
```

**Effect:** Changes the price column from NUMERIC(10,2) to NUMERIC(12,2), allowing larger values.

### ALTER: Adding NOT NULL Constraint

**SQL Statement:**
```sql
-- First, ensure no NULL values exist
UPDATE products SET stock_quantity = 0 WHERE stock_quantity IS NULL;

-- Then add NOT NULL constraint
ALTER TABLE products 
ALTER COLUMN stock_quantity SET NOT NULL;
```

**Effect:** The stock_quantity column now requires a value (cannot be NULL).

### ALTER: Dropping a Column

**SQL Statement:**
```sql
ALTER TABLE products 
DROP COLUMN stock_quantity;
```

**Result - Updated Table:**
| product_id | product_name | price |
|------------|--------------|--------|
| 1 | Laptop | 999.99 |
| 2 | Mouse | 25.50 |
| 3 | Keyboard | 75.00 |

### ALTER: Adding Constraints

**Example 1: Add UNIQUE constraint**
```sql
ALTER TABLE products 
ADD CONSTRAINT unique_product_name UNIQUE (product_name);
```

**Example 2: Add CHECK constraint**
```sql
ALTER TABLE products 
ADD CONSTRAINT check_positive_price CHECK (price > 0);
```

**Example 3: Add Foreign Key**
```sql
-- Assuming we have a categories table
ALTER TABLE products 
ADD COLUMN category_id INTEGER,
ADD CONSTRAINT fk_category 
    FOREIGN KEY (category_id) REFERENCES categories(category_id);
```

### ALTER: Dropping Constraints

**SQL Statement:**
```sql
ALTER TABLE products 
DROP CONSTRAINT check_positive_price;
```

**Effect:** Removes the price validation constraint.

### ALTER: Renaming Column

**SQL Statement:**
```sql
ALTER TABLE products 
RENAME COLUMN product_name TO name;
```

**Result - Updated Table:**
| product_id | name | price |
|------------|------|--------|
| 1 | Laptop | 999.99 |
| 2 | Mouse | 25.50 |
| 3 | Keyboard | 75.00 |

### ALTER: Renaming Table

**SQL Statement:**
```sql
ALTER TABLE products 
RENAME TO inventory;
```

**Effect:** The table is now named `inventory` instead of `products`.

## Complete Example: Building a Related Schema

### Step 1: Create Departments Table

```sql
CREATE TABLE departments (
    dept_id SERIAL PRIMARY KEY,
    dept_name VARCHAR(50) NOT NULL UNIQUE,
    building VARCHAR(50)
);

INSERT INTO departments (dept_name, building) VALUES
('Computer Science', 'Science Hall'),
('Mathematics', 'Math Building'),
('Physics', 'Science Hall');
```

**Departments Table:**
| dept_id | dept_name | building |
|---------|-----------|----------|
| 1 | Computer Science | Science Hall |
| 2 | Mathematics | Math Building |
| 3 | Physics | Science Hall |

### Step 2: Create Instructors Table

```sql
CREATE TABLE instructors (
    instructor_id SERIAL PRIMARY KEY,
    instructor_name VARCHAR(100) NOT NULL,
    dept_id INTEGER,
    hire_date DATE,
    FOREIGN KEY (dept_id) REFERENCES departments(dept_id)
);

INSERT INTO instructors (instructor_name, dept_id, hire_date) VALUES
('Dr. Johnson', 1, '2018-08-15'),
('Dr. Lee', 2, '2019-01-10'),
('Dr. Martinez', 3, '2020-09-01');
```

**Instructors Table:**
| instructor_id | instructor_name | dept_id | hire_date |
|---------------|-----------------|---------|-----------|
| 1 | Dr. Johnson | 1 | 2018-08-15 |
| 2 | Dr. Lee | 2 | 2019-01-10 |
| 3 | Dr. Martinez | 3 | 2020-09-01 |

### Step 3: Add Department Column to Existing Courses Table

```sql
ALTER TABLE courses 
ADD COLUMN dept_id INTEGER,
ADD CONSTRAINT fk_courses_dept 
    FOREIGN KEY (dept_id) REFERENCES departments(dept_id);
```

