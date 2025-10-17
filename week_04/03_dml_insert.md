# DML: INSERT Statements

## Overview

The INSERT statement adds new rows to a table. This guide covers various methods of inserting data in PostgreSQL.

## Basic INSERT Syntax

```sql
INSERT INTO table_name (column1, column2, ...) 
VALUES (value1, value2, ...);
```

## Sample Table

Let's create a sample table for our examples:

```sql
CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE,
    department VARCHAR(50),
    salary NUMERIC(10, 2),
    hire_date DATE DEFAULT CURRENT_DATE
);
```

## Example 1: Insert Single Row with All Columns

**SQL Statement:**
```sql
INSERT INTO employees (first_name, last_name, email, department, salary, hire_date)
VALUES ('John', 'Doe', 'john.doe@company.com', 'Engineering', 75000.00, '2024-01-15');
```

**Employees Table After INSERT:**
| employee_id | first_name | last_name | email | department | salary | hire_date |
|-------------|------------|-----------|-------|------------|---------|-----------|
| 1 | John | Doe | john.doe@company.com | Engineering | 75000.00 | 2024-01-15 |

## Example 2: Insert Without Specifying All Columns

**SQL Statement:**
```sql
INSERT INTO employees (first_name, last_name, email, department)
VALUES ('Jane', 'Smith', 'jane.smith@company.com', 'Marketing');
```

**Employees Table:**
| employee_id | first_name | last_name | email | department | salary | hire_date |
|-------------|------------|-----------|-------|------------|---------|-----------|
| 1 | John | Doe | john.doe@company.com | Engineering | 75000.00 | 2024-01-15 |
| 2 | Jane | Smith | jane.smith@company.com | Marketing | NULL | 2024-10-17 |

**Note:** 
- `salary` is NULL (not provided)
- `hire_date` uses DEFAULT value (CURRENT_DATE)
- `employee_id` auto-increments (SERIAL)

## Example 3: Insert Multiple Rows

**SQL Statement:**
```sql
INSERT INTO employees (first_name, last_name, email, department, salary) 
VALUES 
    ('Bob', 'Wilson', 'bob.wilson@company.com', 'Engineering', 80000.00),
    ('Alice', 'Brown', 'alice.brown@company.com', 'Sales', 65000.00),
    ('Charlie', 'Davis', 'charlie.davis@company.com', 'Engineering', 72000.00);
```

**Employees Table:**
| employee_id | first_name | last_name | email | department | salary | hire_date |
|-------------|------------|-----------|-------|------------|---------|-----------|
| 1 | John | Doe | john.doe@company.com | Engineering | 75000.00 | 2024-01-15 |
| 2 | Jane | Smith | jane.smith@company.com | Marketing | NULL | 2024-10-17 |
| 3 | Bob | Wilson | bob.wilson@company.com | Engineering | 80000.00 | 2024-10-17 |
| 4 | Alice | Brown | alice.brown@company.com | Sales | 65000.00 | 2024-10-17 |
| 5 | Charlie | Davis | charlie.davis@company.com | Engineering | 72000.00 | 2024-10-17 |

## Example 4: Insert with NULL Values

**SQL Statement:**
```sql
INSERT INTO employees (first_name, last_name, email, department, salary)
VALUES ('Emma', 'Johnson', 'emma.johnson@company.com', NULL, 55000.00);
```

**Result:**
| employee_id | first_name | last_name | email | department | salary | hire_date |
|-------------|------------|-----------|-------|------------|---------|-----------|
| 6 | Emma | Johnson | emma.johnson@company.com | NULL | 55000.00 | 2024-10-17 |

## Example 5: INSERT with DEFAULT Keyword

**SQL Statement:**
```sql
INSERT INTO employees (first_name, last_name, email, department, salary, hire_date)
VALUES ('Frank', 'Miller', 'frank.miller@company.com', 'HR', 60000.00, DEFAULT);
```

**Result:**
| employee_id | first_name | last_name | email | department | salary | hire_date |
|-------------|------------|-----------|-------|------------|---------|-----------|
| 7 | Frank | Miller | frank.miller@company.com | HR | 60000.00 | 2024-10-17 |

## INSERT ... SELECT

You can insert data from another table using SELECT.

### Example Setup

```sql
-- Create a temporary table with employee data
CREATE TABLE temp_new_hires (
    name VARCHAR(100),
    email VARCHAR(100),
    dept VARCHAR(50),
    salary NUMERIC(10, 2)
);

INSERT INTO temp_new_hires VALUES
    ('Grace Lee', 'grace.lee@company.com', 'Engineering', 78000.00),
    ('Henry Taylor', 'henry.taylor@company.com', 'Marketing', 67000.00);
```

**Temp_New_Hires Table:**
| name | email | dept | salary |
|------|-------|------|--------|
| Grace Lee | grace.lee@company.com | Engineering | 78000.00 |
| Henry Taylor | henry.taylor@company.com | Marketing | 67000.00 |

### Example 6: INSERT from SELECT

**SQL Statement:**
```sql
INSERT INTO employees (first_name, last_name, email, department, salary)
SELECT 
    SPLIT_PART(name, ' ', 1) AS first_name,
    SPLIT_PART(name, ' ', 2) AS last_name,
    email,
    dept,
    salary
FROM temp_new_hires;
```

**Employees Table (New Rows):**
| employee_id | first_name | last_name | email | department | salary | hire_date |
|-------------|------------|-----------|-------|------------|---------|-----------|
| 8 | Grace | Lee | grace.lee@company.com | Engineering | 78000.00 | 2024-10-17 |
| 9 | Henry | Taylor | henry.taylor@company.com | Marketing | 67000.00 | 2024-10-17 |

## INSERT ... RETURNING

PostgreSQL allows you to return data from inserted rows.

### Example 7: RETURNING Clause

**SQL Statement:**
```sql
INSERT INTO employees (first_name, last_name, email, department, salary)
VALUES ('Isabel', 'Garcia', 'isabel.garcia@company.com', 'Sales', 71000.00)
RETURNING employee_id, first_name, last_name, hire_date;
```

**Query Result (Returned Data):**
| employee_id | first_name | last_name | hire_date |
|-------------|------------|-----------|-----------|
| 10 | Isabel | Garcia | 2024-10-17 |

### Example 8: RETURNING with Multiple Rows

**SQL Statement:**
```sql
INSERT INTO employees (first_name, last_name, email, department, salary)
VALUES 
    ('Jack', 'Anderson', 'jack.anderson@company.com', 'Engineering', 82000.00),
    ('Kate', 'Martinez', 'kate.martinez@company.com', 'HR', 64000.00)
RETURNING employee_id, first_name || ' ' || last_name AS full_name, salary;
```

**Query Result:**
| employee_id | full_name | salary |
|-------------|-----------|--------|
| 11 | Jack Anderson | 82000.00 |
| 12 | Kate Martinez | 64000.00 |

## INSERT with ON CONFLICT (UPSERT)

PostgreSQL supports INSERT ... ON CONFLICT for handling duplicate key conflicts.

### Example 9: INSERT with ON CONFLICT DO NOTHING

**SQL Statement:**
```sql
-- Try to insert duplicate email (will conflict with UNIQUE constraint)
INSERT INTO employees (first_name, last_name, email, department, salary)
VALUES ('John', 'Duplicate', 'john.doe@company.com', 'Sales', 70000.00)
ON CONFLICT (email) DO NOTHING;
```

**Result:** No row inserted (email already exists), no error thrown.

### Example 10: INSERT with ON CONFLICT DO UPDATE

**SQL Statement:**
```sql
-- Update if email exists, insert if it doesn't
INSERT INTO employees (first_name, last_name, email, department, salary)
VALUES ('John', 'Doe', 'john.doe@company.com', 'Management', 90000.00)
ON CONFLICT (email) 
DO UPDATE SET 
    department = EXCLUDED.department,
    salary = EXCLUDED.salary;
```

**Before:**
| employee_id | first_name | last_name | email | department | salary |
|-------------|------------|-----------|-------|------------|--------|
| 1 | John | Doe | john.doe@company.com | Engineering | 75000.00 |

**After:**
| employee_id | first_name | last_name | email | department | salary |
|-------------|------------|-----------|-------|------------|--------|
| 1 | John | Doe | john.doe@company.com | Management | 90000.00 |

## Bulk INSERT Performance

### Example 11: Efficient Bulk Insert

**SQL Statement:**
```sql
-- Efficient: Single multi-value INSERT
INSERT INTO employees (first_name, last_name, email, department, salary)
VALUES 
    ('Person1', 'Last1', 'person1@company.com', 'Dept1', 50000),
    ('Person2', 'Last2', 'person2@company.com', 'Dept2', 51000),
    ('Person3', 'Last3', 'person3@company.com', 'Dept3', 52000),
    ('Person4', 'Last4', 'person4@company.com', 'Dept4', 53000),
    ('Person5', 'Last5', 'person5@company.com', 'Dept5', 54000);
```

## Constraint Enforcement Examples

### Foreign Key Constraint

```sql
-- Create related tables
CREATE TABLE departments (
    dept_id SERIAL PRIMARY KEY,
    dept_name VARCHAR(50)
);

CREATE TABLE staff (
    staff_id SERIAL PRIMARY KEY,
    staff_name VARCHAR(100),
    dept_id INTEGER REFERENCES departments(dept_id)
);

-- Insert parent record first
INSERT INTO departments (dept_name) VALUES ('Engineering');

-- Then insert child record
INSERT INTO staff (staff_name, dept_id)
VALUES ('John Smith', 1);  -- References valid dept_id
```

