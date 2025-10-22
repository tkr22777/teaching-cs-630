# Views using standard SQL

## Overview

A view is a virtual table based on a SELECT query. It doesn't store data itself but provides a way to simplify complex queries, improve security, and create abstractions over base tables.

## Why Use Views?

1. **Simplify Complex Queries** - Hide complexity behind simple names
2. **Security** - Restrict access to specific columns or rows
3. **Data Abstraction** - Hide underlying table structure changes
4. **Reusability** - Define query once, use many times
5. **Consistency** - Ensure same logic used across applications

## Sample Data

<details>
<summary>Click to expand: Database setup script</summary>

```sql
CREATE TABLE employees (
    employee_id INTEGER PRIMARY KEY,
    first_name VARCHAR2(50),
    last_name VARCHAR2(50),
    email VARCHAR2(100),
    department VARCHAR2(50),
    salary NUMBER(10, 2),
    hire_date DATE,
    manager_id INTEGER
);

CREATE TABLE departments (
    dept_id INTEGER PRIMARY KEY,
    dept_name VARCHAR2(50),
    location VARCHAR2(50),
    budget NUMBER(12, 2)
);

INSERT INTO departments (dept_name, location, budget) VALUES
('Engineering', 'San Francisco', 500000),
('Sales', 'New York', 300000),
('Marketing', 'Los Angeles', 200000),
('HR', 'Chicago', 150000);

INSERT INTO employees (first_name, last_name, email, department, salary, hire_date, manager_id) VALUES
('John', 'Doe', 'john.doe@company.com', 'Engineering', 95000, '2020-01-15', NULL),
('Jane', 'Smith', 'jane.smith@company.com', 'Engineering', 85000, '2020-03-20', 1),
('Bob', 'Johnson', 'bob.j@company.com', 'Sales', 75000, '2021-06-10', NULL),
('Alice', 'Williams', 'alice.w@company.com', 'Sales', 70000, '2021-08-15', 3),
('Charlie', 'Brown', 'charlie.b@company.com', 'Marketing', 65000, '2022-01-20', NULL),
('Diana', 'Davis', 'diana.d@company.com', 'HR', 60000, '2022-03-25', NULL),
('Eve', 'Miller', 'eve.m@company.com', 'Engineering', 90000, '2022-07-01', 1),
('Frank', 'Wilson', 'frank.w@company.com', 'Sales', 72000, '2023-02-14', 3);
```

</details>

**Employees Table:**
| employee_id | first_name | last_name | email | department | salary | hire_date | manager_id |
|-------------|------------|-----------|-------|------------|--------|-----------|------------|
| 1 | John | Doe | john.doe@company.com | Engineering | 95000 | 2020-01-15 | NULL |
| 2 | Jane | Smith | jane.smith@company.com | Engineering | 85000 | 2020-03-20 | 1 |
| 3 | Bob | Johnson | bob.j@company.com | Sales | 75000 | 2021-06-10 | NULL |
| 4 | Alice | Williams | alice.w@company.com | Sales | 70000 | 2021-08-15 | 3 |
| 5 | Charlie | Brown | charlie.b@company.com | Marketing | 65000 | 2022-01-20 | NULL |
| 6 | Diana | Davis | diana.d@company.com | HR | 60000 | 2022-03-25 | NULL |
| 7 | Eve | Miller | eve.m@company.com | Engineering | 90000 | 2022-07-01 | 1 |
| 8 | Frank | Wilson | frank.w@company.com | Sales | 72000 | 2023-02-14 | 3 |

## Creating Views

### Basic Syntax

```sql
CREATE VIEW view_name AS
SELECT columns
FROM tables
WHERE conditions;
```

### Example 1: Simple View

**SQL Statement:**
```sql
CREATE VIEW engineering_employees AS
SELECT employee_id,
       first_name,
       last_name,
       email,
       salary,
       hire_date
FROM employees
WHERE department = 'Engineering';
```

**Using the View:**
```sql
SELECT * FROM engineering_employees;
```

**Result:**
| employee_id | first_name | last_name | email | salary | hire_date |
|-------------|------------|-----------|-------|--------|-----------|
| 1 | John | Doe | john.doe@company.com | 95000 | 2020-01-15 |
| 2 | Jane | Smith | jane.smith@company.com | 85000 | 2020-03-20 |
| 7 | Eve | Miller | eve.m@company.com | 90000 | 2022-07-01 |

### Example 2: View with Calculated Columns

**SQL Statement:**
```sql
CREATE VIEW employee_summary AS
SELECT employee_id,
       first_name || ' ' || last_name AS full_name,
       email,
       department,
       salary,
       salary * 0.15 AS estimated_tax,
       salary * 0.85 AS net_salary,
       EXTRACT(YEAR FROM AGE(CURRENT_DATE, hire_date)) AS years_employed
FROM employees;
```

**Using the View:**
```sql
SELECT full_name, department, net_salary, years_employed
FROM employee_summary
WHERE years_employed > 2
ORDER BY net_salary DESC;
```

**Result:**
| full_name | department | net_salary | years_employed |
|-----------|------------|------------|----------------|
| John Doe | Engineering | 80750.00 | 4 |
| Jane Smith | Engineering | 72250.00 | 4 |
| Bob Johnson | Sales | 63750.00 | 3 |

###

**SQL Statement:**
```sql
CREATE VIEW employee_with_manager AS
SELECT e.employee_id,
       e.first_name || ' ' || e.last_name AS employee_name,
       e.department,
       e.salary,
       m.first_name || ' ' || m.last_name AS manager_name
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.employee_id;
```

**Using the View:**
```sql
SELECT * FROM employee_with_manager
ORDER BY department, employee_name;
```

**Result:**
| employee_id | employee_name | department | salary | manager_name |
|-------------|---------------|------------|--------|--------------|
| 1 | John Doe | Engineering | 95000 | NULL |
| 2 | Jane Smith | Engineering | 85000 | John Doe |
| 7 | Eve Miller | Engineering | 90000 | John Doe |
| 6 | Diana Davis | HR | 60000 | NULL |
| 5 | Charlie Brown | Marketing | 65000 | NULL |
| 4 | Alice Williams | Sales | 70000 | Bob Johnson |
| 3 | Bob Johnson | Sales | 75000 | NULL |
| 8 | Frank Wilson | Sales | 72000 | Bob Johnson |

### Example 3: View with Aggregation

**SQL Statement:**
```sql
CREATE VIEW department_stats AS
SELECT department,
       COUNT(*) AS employee_count,
       AVG(salary) AS avg_salary,
       MIN(salary) AS min_salary,
       MAX(salary) AS max_salary,
       SUM(salary) AS total_payroll
FROM employees
GROUP BY department;
```

**Using the View:**
```sql
SELECT * FROM department_stats
ORDER BY total_payroll DESC;
```

**Result:**
| department | employee_count | avg_salary | min_salary | max_salary | total_payroll |
|------------|----------------|------------|------------|------------|---------------|
| Engineering | 3 | 90000.00 | 85000 | 95000 | 270000 |
| Sales | 3 | 72333.33 | 70000 | 75000 | 217000 |
| Marketing | 1 | 65000.00 | 65000 | 65000 | 65000 |
| HR | 1 | 60000.00 | 60000 | 60000 | 60000 |

###

**SQL Statement:**
```sql
-- View that hides sensitive salary information
CREATE VIEW public_employee_directory AS
SELECT employee_id,
       first_name,
       last_name,
       email,
       department,
       hire_date
FROM employees;
-- Note: salary is excluded
```

**Using the View:**
```sql
SELECT * FROM public_employee_directory
WHERE department = 'Sales';
```

**Result:**
| employee_id | first_name | last_name | email | department | hire_date |
|-------------|------------|-----------|-------|------------|-----------|
| 3 | Bob | Johnson | bob.j@company.com | Sales | 2021-06-10 |
| 4 | Alice | Williams | alice.w@company.com | Sales | 2021-08-15 |
| 8 | Frank | Wilson | frank.w@company.com | Sales | 2023-02-14 |

## Using Views

Views can be queried just like regular tables:

### Filtering a View

**SQL Statement:**
```sql
SELECT full_name, net_salary
FROM employee_summary
WHERE department = 'Engineering'
  AND net_salary > 80000
ORDER BY net_salary DESC;
```

**Result:**
| full_name | net_salary |
|-----------|------------|
| Eve Miller | 76500.00 |
| John Doe | 80750.00 |

### Joining Views

**SQL Statement:**
```sql
SELECT es.full_name,
       es.department,
       ds.employee_count,
       ds.avg_salary
FROM employee_summary es
JOIN department_stats ds ON es.department = ds.department
WHERE es.salary > ds.avg_salary;
```

**Result (employees earning above department average):**
| full_name | department | employee_count | avg_salary |
|-----------|------------|----------------|------------|
| John Doe | Engineering | 3 | 90000.00 |
| Bob Johnson | Sales | 3 | 72333.33 |

## Modifying Views

### CREATE OR REPLACE VIEW

**SQL Statement:**
```sql
-- Update existing view
CREATE OR REPLACE VIEW engineering_employees AS
SELECT employee_id,
       first_name,
       last_name,
       email,
       salary,
       hire_date,
       manager_id  -- Added new column
FROM employees
WHERE department = 'Engineering'
  AND salary >= 80000;  -- Added salary filter
```

**Using Updated View:**
```sql
SELECT * FROM engineering_employees;
```

**Result (now includes manager_id and filters by salary):**
| employee_id | first_name | last_name | email | salary | hire_date | manager_id |
|-------------|------------|-----------|-------|--------|-----------|------------|
| 1 | John | Doe | john.doe@company.com | 95000 | 2020-01-15 | NULL |
| 2 | Jane | Smith | jane.smith@company.com | 85000 | 2020-03-20 | 1 |
| 7 | Eve | Miller | eve.m@company.com | 90000 | 2022-07-01 | 1 |

### Dropping Views

**SQL Statement:**
```sql
DROP VIEW engineering_employees;
```

**Safe Version:**
```sql
DROP VIEW IF EXISTS engineering_employees;
```

###

**SQL Statement:**
```sql
DROP VIEW IF EXISTS view1, view2, view3;
```

## Updatable Views (brief)

Some views allow INSERT, UPDATE, DELETE operations.

### Requirements for Updatable Views:

- Based on a single table
- No aggregate functions (COUNT, SUM, AVG, etc.)
- No DISTINCT, GROUP BY, HAVING
- No set operations (UNION, INTERSECT, EXCEPT)
- No window functions

### Example: Simple Updatable View

**SQL Statement:**
```sql
CREATE VIEW active_employees AS
SELECT employee_id,
       first_name,
       last_name,
       email,
       department
FROM employees
WHERE hire_date >= '2022-01-01';
```

**Update Through View:**
```sql
UPDATE active_employees
SET department = 'Engineering'
WHERE employee_id = 5;
```

**Verify:**
```sql
SELECT employee_id, first_name, last_name, department
FROM employees
WHERE employee_id = 5;
```

**Result:**
| employee_id | first_name | last_name | department |
|-------------|------------|-----------|------------|
| 5 | Charlie | Brown | Engineering |

###

**SQL Statement:**
```sql
INSERT INTO active_employees (first_name, last_name, email, department)
VALUES ('Grace', 'Lee', 'grace.l@company.com', 'HR');
```

**Result:** New row is inserted into base `employees` table.

###

**SQL Statement:**
```sql
CREATE VIEW dept_summary AS
SELECT department,
       COUNT(*) AS emp_count,
       AVG(salary) AS avg_sal
FROM employees
GROUP BY department;

-- This will FAIL
UPDATE dept_summary SET emp_count = 10 WHERE department = 'Sales';
-- ERROR: cannot update view "dept_summary"
```

## Materialized Views

Materialized views store query results physically (unlike regular views).

### Creating Materialized Views

**SQL Statement:**
```sql
CREATE MATERIALIZED VIEW mv_department_summary AS
SELECT department,
       COUNT(*) AS employee_count,
       AVG(salary) AS avg_salary,
       SUM(salary) AS total_payroll
FROM employees
GROUP BY department;
```

**Using Materialized View:**
```sql
SELECT * FROM mv_department_summary;
```

### Refreshing Materialized Views

<details>
<summary>PostgreSQL (primary) and MySQL note</summary>

PostgreSQL supports materialized views and concurrent refresh when a unique index exists.

```sql
-- PostgreSQL: Refresh data (blocks reads)
REFRESH MATERIALIZED VIEW mv_department_summary;

-- PostgreSQL: Refresh without blocking reads (requires unique index)
REFRESH MATERIALIZED VIEW CONCURRENTLY mv_department_summary;
```

MySQL does not have native materialized views; emulate with tables + scheduled refresh.

</details>

### Materialized View with Index

**SQL Statement:**
```sql
-- PostgreSQL: Create materialized view
CREATE MATERIALIZED VIEW mv_employee_stats AS
SELECT department,
       EXTRACT(YEAR FROM hire_date) AS hire_year,
       COUNT(*) AS employees_hired,
       AVG(salary) AS avg_starting_salary
FROM employees
GROUP BY department, EXTRACT(YEAR FROM hire_date);

-- PostgreSQL: Create unique index for concurrent refresh
CREATE UNIQUE INDEX idx_mv_emp_stats 
ON mv_employee_stats (department, hire_year);

-- PostgreSQL: Refresh concurrently
REFRESH MATERIALIZED VIEW CONCURRENTLY mv_employee_stats;
```

### View vs Materialized View

| Feature | View | Materialized View |
|---------|------|-------------------|
| Storage | No data stored | Data physically stored |
| Performance | Executes query each time | Reads stored data (faster) |
| Data freshness | Always current | Needs manual refresh |
| Space usage | Minimal | Can be significant |
| Best for | Real-time data, simple queries | Complex queries, reports, dashboards |

## Advanced View Technique

### View with UNION

**SQL Statement:**
```sql
CREATE VIEW all_contacts AS
SELECT employee_id AS id,
       first_name || ' ' || last_name AS name,
       email,
       'Employee' AS contact_type
FROM employees
UNION ALL
SELECT customer_id AS id,
       customer_name AS name,
       customer_email AS email,
       'Customer' AS contact_type
FROM customers;
```

###

## Viewing View Definitions

### Example 17: List All Views

**SQL Statement:**
```sql
SELECT table_name, view_definition
FROM information_schema.views
WHERE table_schema = 'public'
ORDER BY table_name;
```

### Example 18: Get Specific View Definition

**SQL Statement:**
```sql
SELECT pg_get_viewdef('employee_summary', true);
```

