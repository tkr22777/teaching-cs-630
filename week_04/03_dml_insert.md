# DML: INSERT Statements

## Overview

The INSERT statement adds new rows to a table. This guide covers various methods of inserting data using Oracle SQL.

## Basic INSERT Syntax

```sql
INSERT INTO table_name (column1, column2, ...) 
VALUES (value1, value2, ...);
```

## Sample Table

Let's create a sample table for our examples:

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
- `employee_id` can be generated via identity columns (12c+) or sequences

## Example 3: Insert Multiple Rows

**SQL Statement:**
```sql
INSERT ALL
    INTO employees (employee_id, first_name, last_name, email, department, salary) VALUES (3, 'Bob', 'Wilson', 'bob.wilson@company.com', 'Engineering', 80000.00)
    INTO employees (employee_id, first_name, last_name, email, department, salary) VALUES (4, 'Alice', 'Brown', 'alice.brown@company.com', 'Sales', 65000.00)
    INTO employees (employee_id, first_name, last_name, email, department, salary) VALUES (5, 'Charlie', 'Davis', 'charlie.davis@company.com', 'Engineering', 72000.00)
SELECT * FROM dual;
```

**Employees Table:**
| employee_id | first_name | last_name | email | department | salary | hire_date |
|-------------|------------|-----------|-------|------------|---------|-----------|
| 1 | John | Doe | john.doe@company.com | Engineering | 75000.00 | 2024-01-15 |
| 2 | Jane | Smith | jane.smith@company.com | Marketing | NULL | 2024-10-22 |
| 3 | Bob | Wilson | bob.wilson@company.com | Engineering | 80000.00 | 2024-10-22 |
| 4 | Alice | Brown | alice.brown@company.com | Sales | 65000.00 | 2024-10-22 |
| 5 | Charlie | Davis | charlie.davis@company.com | Engineering | 72000.00 | 2024-10-22 |

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

## Example 5: INSERT with DEFAULT Keyword

**SQL Statement:**
```sql
INSERT INTO employees (employee_id, first_name, last_name, email, department, salary, hire_date)
VALUES (7, 'Frank', 'Miller', 'frank.miller@company.com', 'HR', 60000.00, DEFAULT);
```

**Result:**
| employee_id | first_name | last_name | email | department | salary | hire_date |
|-------------|------------|-----------|-------|------------|---------|-----------|
| 7 | Frank | Miller | frank.miller@company.com | HR | 60000.00 | 2024-10-22 |

## INSERT ... SELECT

You can insert data from another table using SELECT.

### Example Setup

```sql
-- Create a temporary table with employee data
CREATE TABLE temp_new_hires (
    name VARCHAR2(100),
    email VARCHAR2(100),
    dept VARCHAR2(50),
    salary NUMBER(10, 2)
);

INSERT INTO temp_new_hires VALUES ('Grace Lee', 'grace.lee@company.com', 'Engineering', 78000.00);
INSERT INTO temp_new_hires VALUES ('Henry Taylor', 'henry.taylor@company.com', 'Marketing', 67000.00);
```

**Temp_New_Hires Table:**
| name | email | dept | salary |
|------|-------|------|--------|
| Grace Lee | grace.lee@company.com | Engineering | 78000.00 |
| Henry Taylor | henry.taylor@company.com | Marketing | 67000.00 |

### Example 6: INSERT from SELECT

**SQL Statement:**
```sql
INSERT INTO employees (employee_id, first_name, last_name, email, department, salary)
SELECT 
    ROWNUM + 7,  -- Generate employee_id starting from 8
    SUBSTR(name, 1, INSTR(name, ' ') - 1) AS first_name,
    SUBSTR(name, INSTR(name, ' ') + 1) AS last_name,
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

<details>
<summary>Oracle-Specific Features: RETURNING and MERGE (UPSERT)</summary>

## INSERT ... RETURNING (Oracle)

Oracle supports the RETURNING clause to get values from inserted rows:

```sql
INSERT INTO employees (employee_id, first_name, last_name, email, department, salary)
VALUES (10, 'Isabel', 'Garcia', 'isabel.garcia@company.com', 'Sales', 71000.00)
RETURNING employee_id, hire_date INTO :emp_id, :hire_dt;
```

**Note:** The INTO clause requires bind variables (`:emp_id`, `:hire_dt`) which are typically used in PL/SQL blocks or application code.

## MERGE Statement (UPSERT in Oracle)

Oracle uses MERGE for insert-or-update operations:

```sql
-- Update if exists, insert if not
MERGE INTO employees e
USING (SELECT 1 AS employee_id, 
              'John' AS first_name, 
              'Doe' AS last_name,
              'john.doe@company.com' AS email,
              'Management' AS department,
              90000.00 AS salary
       FROM dual) src
ON (e.email = src.email)
WHEN MATCHED THEN
    UPDATE SET e.department = src.department, e.salary = src.salary
WHEN NOT MATCHED THEN
    INSERT (employee_id, first_name, last_name, email, department, salary)
    VALUES (src.employee_id, src.first_name, src.last_name, src.email, src.department, src.salary);
```

</details>

## Bulk INSERT Performance

### Example 11: Efficient Bulk Insert

**SQL Statement:**
```sql
-- Efficient: INSERT ALL for multiple rows
INSERT ALL
    INTO employees (employee_id, first_name, last_name, email, department, salary) VALUES (11, 'Person1', 'Last1', 'person1@company.com', 'Dept1', 50000)
    INTO employees (employee_id, first_name, last_name, email, department, salary) VALUES (12, 'Person2', 'Last2', 'person2@company.com', 'Dept2', 51000)
    INTO employees (employee_id, first_name, last_name, email, department, salary) VALUES (13, 'Person3', 'Last3', 'person3@company.com', 'Dept3', 52000)
    INTO employees (employee_id, first_name, last_name, email, department, salary) VALUES (14, 'Person4', 'Last4', 'person4@company.com', 'Dept4', 53000)
    INTO employees (employee_id, first_name, last_name, email, department, salary) VALUES (15, 'Person5', 'Last5', 'person5@company.com', 'Dept5', 54000)
SELECT * FROM dual;
```

## Constraint Enforcement Examples

### Foreign Key Constraint

```sql
-- Create related tables
CREATE TABLE departments (
    dept_id INTEGER PRIMARY KEY,
    dept_name VARCHAR2(50)
);

CREATE TABLE staff (
    staff_id INTEGER PRIMARY KEY,
    staff_name VARCHAR2(100),
    dept_id INTEGER REFERENCES departments(dept_id)
);

-- Insert parent record first
INSERT INTO departments (dept_id, dept_name) VALUES (1, 'Engineering');

-- Then insert child record
INSERT INTO staff (staff_id, staff_name, dept_id)
VALUES (1, 'John Smith', 1);  -- References valid dept_id
```

