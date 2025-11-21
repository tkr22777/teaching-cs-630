# Views using standard SQL

## Overview

A **view** is a saved query that acts like a virtual table. "Virtual" means the view doesn't store data - it runs the query each time you use it. Instead of writing the same complex query repeatedly, you create a view once and then query it like a regular table. Views simplify complex queries, restrict access to specific columns, and ensure consistency.

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

| employee_id | first_name | last_name | email                  | department  | salary | hire_date  | manager_id |
| ----------- | ---------- | --------- | ---------------------- | ----------- | ------ | ---------- | ---------- |
| 1           | John       | Doe       | john.doe@company.com   | Engineering | 95000  | 2020-01-15 | NULL       |
| 2           | Jane       | Smith     | jane.smith@company.com | Engineering | 85000  | 2020-03-20 | 1          |
| 3           | Bob        | Johnson   | bob.j@company.com      | Sales       | 75000  | 2021-06-10 | NULL       |
| 4           | Alice      | Williams  | alice.w@company.com    | Sales       | 70000  | 2021-08-15 | 3          |
| 5           | Charlie    | Brown     | charlie.b@company.com  | Marketing   | 65000  | 2022-01-20 | NULL       |
| 6           | Diana      | Davis     | diana.d@company.com    | HR          | 60000  | 2022-03-25 | NULL       |
| 7           | Eve        | Miller    | eve.m@company.com      | Engineering | 90000  | 2022-07-01 | 1          |
| 8           | Frank      | Wilson    | frank.w@company.com    | Sales       | 72000  | 2023-02-14 | 3          |

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

| employee_id | first_name | last_name | email                  | salary | hire_date  |
| ----------- | ---------- | --------- | ---------------------- | ------ | ---------- |
| 1           | John       | Doe       | john.doe@company.com   | 95000  | 2020-01-15 |
| 2           | Jane       | Smith     | jane.smith@company.com | 85000  | 2020-03-20 |
| 7           | Eve        | Miller    | eve.m@company.com      | 90000  | 2022-07-01 |

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
       FLOOR(MONTHS_BETWEEN(SYSDATE, hire_date) / 12) AS years_employed
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

| full_name   | department  | net_salary | years_employed |
| ----------- | ----------- | ---------- | -------------- |
| John Doe    | Engineering | 80750.00   | 4              |
| Jane Smith  | Engineering | 72250.00   | 4              |
| Bob Johnson | Sales       | 63750.00   | 3              |

---

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

| employee_id | employee_name  | department  | salary | manager_name |
| ----------- | -------------- | ----------- | ------ | ------------ |
| 1           | John Doe       | Engineering | 95000  | NULL         |
| 2           | Jane Smith     | Engineering | 85000  | John Doe     |
| 7           | Eve Miller     | Engineering | 90000  | John Doe     |
| 6           | Diana Davis    | HR          | 60000  | NULL         |
| 5           | Charlie Brown  | Marketing   | 65000  | NULL         |
| 4           | Alice Williams | Sales       | 70000  | Bob Johnson  |
| 3           | Bob Johnson    | Sales       | 75000  | NULL         |
| 8           | Frank Wilson   | Sales       | 72000  | Bob Johnson  |

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

| department  | employee_count | avg_salary | min_salary | max_salary | total_payroll |
| ----------- | -------------- | ---------- | ---------- | ---------- | ------------- |
| Engineering | 3              | 90000.00   | 85000      | 95000      | 270000        |
| Sales       | 3              | 72333.33   | 70000      | 75000      | 217000        |
| Marketing   | 1              | 65000.00   | 65000      | 65000      | 65000         |
| HR          | 1              | 60000.00   | 60000      | 60000      | 60000         |

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

| employee_id | first_name | last_name | email               | department | hire_date  |
| ----------- | ---------- | --------- | ------------------- | ---------- | ---------- |
| 3           | Bob        | Johnson   | bob.j@company.com   | Sales      | 2021-06-10 |
| 4           | Alice      | Williams  | alice.w@company.com | Sales      | 2021-08-15 |
| 8           | Frank      | Wilson    | frank.w@company.com | Sales      | 2023-02-14 |

---

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

| employee_id | first_name | last_name | email                  | salary | hire_date  | manager_id |
| ----------- | ---------- | --------- | ---------------------- | ------ | ---------- | ---------- |
| 1           | John       | Doe       | john.doe@company.com   | 95000  | 2020-01-15 | NULL       |
| 2           | Jane       | Smith     | jane.smith@company.com | 85000  | 2020-03-20 | 1          |
| 7           | Eve        | Miller    | eve.m@company.com      | 90000  | 2022-07-01 | 1          |

### Dropping Views

**SQL Statement:**

```sql
DROP VIEW engineering_employees;
```

**Safe Version (if unsure whether view exists):**

```sql
DROP VIEW IF EXISTS engineering_employees;
```
