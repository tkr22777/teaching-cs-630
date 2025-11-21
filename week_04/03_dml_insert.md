# DML: INSERT Statements

## Overview

**DML (Data Manipulation Language)** commands work with the data inside tables. INSERT is how you add new rows - whether you're adding a single new customer, loading a batch of products, or copying data from another table.

## Basic INSERT Syntax

```sql
INSERT INTO table_name (column1, column2, ...) 
VALUES (value1, value2, ...);
```

## Sample Table

We'll use an employees table for our INSERT examples:

```sql
CREATE TABLE employees (
    employee_id INTEGER PRIMARY KEY,
    first_name VARCHAR2(50) NOT NULL,
    last_name VARCHAR2(50) NOT NULL,
    email VARCHAR2(100) UNIQUE,
    department VARCHAR2(50),
    salary NUMBER(10, 2),
    hire_date DATE DEFAULT SYSDATE
);
```

## Example 1: Insert Single Row with All Columns

**SQL Statement:**
```sql
INSERT INTO employees (employee_id, first_name, last_name, email, department, salary, hire_date)
VALUES (1, 'John', 'Doe', 'john.doe@company.com', 'Engineering', 75000.00, DATE '2024-01-15');
```

**Employees Table After INSERT:**
| employee_id | first_name | last_name | email | department | salary | hire_date |
|-------------|------------|-----------|-------|------------|---------|-----------|
| 1 | John | Doe | john.doe@company.com | Engineering | 75000.00 | 2024-01-15 |

## Example 2: Insert Without Specifying All Columns

**SQL Statement:**
```sql
INSERT INTO employees (employee_id, first_name, last_name, email, department)
VALUES (2, 'Jane', 'Smith', 'jane.smith@company.com', 'Marketing');
```

**Employees Table:**
| employee_id | first_name | last_name | email | department | salary | hire_date |
|-------------|------------|-----------|-------|------------|---------|-----------|
| 1 | John | Doe | john.doe@company.com | Engineering | 75000.00 | 2024-01-15 |
| 2 | Jane | Smith | jane.smith@company.com | Marketing | NULL | 2024-10-22 |

**Note:** 
- `salary` is NULL (not provided)
- `hire_date` uses DEFAULT value (SYSDATE)

When you have several rows to add at once, you can insert them one after another.

## Example 3: Insert Multiple Rows

**SQL Statement:**
```sql
-- Insert multiple rows (one statement per row)
INSERT INTO employees (employee_id, first_name, last_name, email, department, salary) 
VALUES (3, 'Bob', 'Wilson', 'bob.wilson@company.com', 'Engineering', 80000.00);

INSERT INTO employees (employee_id, first_name, last_name, email, department, salary) 
VALUES (4, 'Alice', 'Brown', 'alice.brown@company.com', 'Sales', 65000.00);

INSERT INTO employees (employee_id, first_name, last_name, email, department, salary) 
VALUES (5, 'Charlie', 'Davis', 'charlie.davis@company.com', 'Engineering', 72000.00);
```

**Employees Table:**
| employee_id | first_name | last_name | email | department | salary | hire_date |
|-------------|------------|-----------|-------|------------|---------|-----------|
| 1 | John | Doe | john.doe@company.com | Engineering | 75000.00 | 2024-01-15 |
| 2 | Jane | Smith | jane.smith@company.com | Marketing | NULL | 2024-10-22 |
| 3 | Bob | Wilson | bob.wilson@company.com | Engineering | 80000.00 | 2024-10-22 |
| 4 | Alice | Brown | alice.brown@company.com | Sales | 65000.00 | 2024-10-22 |
| 5 | Charlie | Davis | charlie.davis@company.com | Engineering | 72000.00 | 2024-10-22 |

Sometimes you legitimately don't have a value for a column. That's when you use NULL.

## Example 4: Insert with NULL Values

**SQL Statement:**
```sql
INSERT INTO employees (employee_id, first_name, last_name, email, department, salary)
VALUES (6, 'Emma', 'Johnson', 'emma.johnson@company.com', NULL, 55000.00);
```

**Result:**
| employee_id | first_name | last_name | email | department | salary | hire_date |
|-------------|------------|-----------|-------|------------|---------|-----------|
| 6 | Emma | Johnson | emma.johnson@company.com | NULL | 55000.00 | 2024-10-22 |

NULL is a valid value when a column allows it. You can also use the DEFAULT keyword to explicitly use a column's default value.

---

## INSERT ... SELECT

Often you'll need to copy or transform data from one table into another. INSERT with SELECT lets you do this without manually typing values.

### Example 6: INSERT from SELECT

Let's say we have a staging table with new hire data:

```sql
CREATE TABLE temp_new_hires (
    first_name VARCHAR2(50),
    last_name VARCHAR2(50),
    email VARCHAR2(100),
    dept VARCHAR2(50),
    salary NUMBER(10, 2)
);

INSERT INTO temp_new_hires VALUES ('Grace', 'Lee', 'grace.lee@company.com', 'Engineering', 78000.00);
INSERT INTO temp_new_hires VALUES ('Henry', 'Taylor', 'henry.taylor@company.com', 'Marketing', 67000.00);
```

**SQL Statement:**
```sql
INSERT INTO employees (employee_id, first_name, last_name, email, department, salary)
SELECT 
    ROWNUM + 7,  -- ROWNUM generates sequential numbers; add 7 to start at 8
    first_name,
    last_name,
    email,
    dept,
    salary
FROM temp_new_hires;
```

**Employees Table (New Rows):**
| employee_id | first_name | last_name | email | department | salary | hire_date |
|-------------|------------|-----------|-------|------------|---------|-----------|
| 8 | Grace | Lee | grace.lee@company.com | Engineering | 78000.00 | 2024-10-22 |
| 9 | Henry | Taylor | henry.taylor@company.com | Marketing | 67000.00 | 2024-10-22 |

This technique is powerful for data migrations, copying data between environments, or populating tables from query results.

## Summary

**INSERT** adds new rows to tables. You can insert single rows, use DEFAULT for default values, or use INSERT ... SELECT to copy data from another table.
