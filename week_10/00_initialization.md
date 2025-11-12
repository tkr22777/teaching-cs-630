# Database Initialization - Week 9: PL/SQL

## Overview

This initialization script sets up the database used throughout all Week 9 PL/SQL lessons. The database represents an employee management system with departments, employees, projects, and salary history.

## Why PL/SQL? (Motivation)

PL/SQL is Oracle’s procedural extension to SQL. It lets you run logic where the data lives.

- Performance: send a block once, reduce app–DB round trips and context switches
- Reliability: transactions, exceptions, and validation close to the data
- Reuse and security: procedures/functions/packages encapsulate rules and can be permissioned
- Automation: triggers enforce policies (audit, defaults) without app changes
- Practical for CS/DS/BA: fast batch processing, data quality checks, computed metrics, controlled updates

## Database Schema

The database consists of four interconnected tables:

- **Departments**: Department information
- **Employees**: Employee records with salary and department assignment
- **Projects**: Project information with budgets
- **Salary_History**: Historical record of salary changes

## Setup Script

```sql
-- Clean up existing tables if they exist
BEGIN
   EXECUTE IMMEDIATE 'DROP TABLE salary_history CASCADE CONSTRAINTS';
EXCEPTION WHEN OTHERS THEN NULL;
END;
/

BEGIN
   EXECUTE IMMEDIATE 'DROP TABLE employees CASCADE CONSTRAINTS';
EXCEPTION WHEN OTHERS THEN NULL;
END;
/

BEGIN
   EXECUTE IMMEDIATE 'DROP TABLE departments CASCADE CONSTRAINTS';
EXCEPTION WHEN OTHERS THEN NULL;
END;
/

BEGIN
   EXECUTE IMMEDIATE 'DROP TABLE projects CASCADE CONSTRAINTS';
EXCEPTION WHEN OTHERS THEN NULL;
END;
/

-- Create Departments table
CREATE TABLE departments (
    dept_id INTEGER PRIMARY KEY,
    dept_name VARCHAR2(50) NOT NULL,
    location VARCHAR2(50),
    budget NUMBER(12, 2)
);

-- Create Employees table
CREATE TABLE employees (
    emp_id INTEGER PRIMARY KEY,
    first_name VARCHAR2(50) NOT NULL,
    last_name VARCHAR2(50) NOT NULL,
    email VARCHAR2(100) UNIQUE,
    hire_date DATE DEFAULT SYSDATE,
    salary NUMBER(10, 2),
    dept_id INTEGER,
    manager_id INTEGER,
    CONSTRAINT fk_emp_dept FOREIGN KEY (dept_id) REFERENCES departments(dept_id),
    CONSTRAINT fk_emp_mgr FOREIGN KEY (manager_id) REFERENCES employees(emp_id)
);

-- Create Projects table
CREATE TABLE projects (
    project_id INTEGER PRIMARY KEY,
    project_name VARCHAR2(100) NOT NULL,
    start_date DATE,
    end_date DATE,
    budget NUMBER(12, 2),
    dept_id INTEGER,
    status VARCHAR2(20) DEFAULT 'Active',
    CONSTRAINT fk_proj_dept FOREIGN KEY (dept_id) REFERENCES departments(dept_id),
    CONSTRAINT chk_status CHECK (status IN ('Active', 'Completed', 'On Hold'))
);

-- Create Salary_History table
CREATE TABLE salary_history (
    history_id INTEGER PRIMARY KEY,
    emp_id INTEGER NOT NULL,
    old_salary NUMBER(10, 2),
    new_salary NUMBER(10, 2),
    change_date DATE DEFAULT SYSDATE,
    reason VARCHAR2(100),
    CONSTRAINT fk_sal_emp FOREIGN KEY (emp_id) REFERENCES employees(emp_id)
);

-- Insert Departments
INSERT INTO departments (dept_id, dept_name, location, budget) VALUES
(10, 'Engineering', 'Building A', 500000);
INSERT INTO departments (dept_id, dept_name, location, budget) VALUES
(20, 'Sales', 'Building B', 300000);
INSERT INTO departments (dept_id, dept_name, location, budget) VALUES
(30, 'Marketing', 'Building B', 250000);
INSERT INTO departments (dept_id, dept_name, location, budget) VALUES
(40, 'HR', 'Building C', 150000);

-- Insert Employees
INSERT INTO employees (emp_id, first_name, last_name, email, hire_date, salary, dept_id, manager_id) VALUES
(101, 'Sarah', 'Johnson', 'sarah.j@company.com', DATE '2020-01-15', 95000, 10, NULL);
INSERT INTO employees (emp_id, first_name, last_name, email, hire_date, salary, dept_id, manager_id) VALUES
(102, 'Mike', 'Chen', 'mike.c@company.com', DATE '2020-03-20', 85000, 10, 101);
INSERT INTO employees (emp_id, first_name, last_name, email, hire_date, salary, dept_id, manager_id) VALUES
(103, 'Emily', 'Davis', 'emily.d@company.com', DATE '2021-06-10', 78000, 10, 101);
INSERT INTO employees (emp_id, first_name, last_name, email, hire_date, salary, dept_id, manager_id) VALUES
(104, 'James', 'Wilson', 'james.w@company.com', DATE '2019-09-05', 88000, 20, NULL);
INSERT INTO employees (emp_id, first_name, last_name, email, hire_date, salary, dept_id, manager_id) VALUES
(105, 'Lisa', 'Brown', 'lisa.b@company.com', DATE '2021-01-12', 72000, 20, 104);
INSERT INTO employees (emp_id, first_name, last_name, email, hire_date, salary, dept_id, manager_id) VALUES
(106, 'David', 'Martinez', 'david.m@company.com', DATE '2022-04-18', 68000, 30, NULL);
INSERT INTO employees (emp_id, first_name, last_name, email, hire_date, salary, dept_id, manager_id) VALUES
(107, 'Jennifer', 'Taylor', 'jennifer.t@company.com', DATE '2020-11-30', 65000, 40, NULL);
INSERT INTO employees (emp_id, first_name, last_name, email, hire_date, salary, dept_id, manager_id) VALUES
(108, 'Robert', 'Anderson', 'robert.a@company.com', DATE '2023-01-05', 62000, 10, 101);

-- Insert Projects
INSERT INTO projects (project_id, project_name, start_date, end_date, budget, dept_id, status) VALUES
(1001, 'Website Redesign', DATE '2024-01-01', DATE '2024-06-30', 150000, 10, 'Active');
INSERT INTO projects (project_id, project_name, start_date, end_date, budget, dept_id, status) VALUES
(1002, 'Mobile App Development', DATE '2024-02-15', DATE '2024-12-31', 250000, 10, 'Active');
INSERT INTO projects (project_id, project_name, start_date, end_date, budget, dept_id, status) VALUES
(1003, 'Sales Campaign Q1', DATE '2024-01-01', DATE '2024-03-31', 80000, 20, 'Completed');
INSERT INTO projects (project_id, project_name, start_date, end_date, budget, dept_id, status) VALUES
(1004, 'Brand Refresh', DATE '2024-03-01', DATE '2024-08-31', 120000, 30, 'Active');
INSERT INTO projects (project_id, project_name, start_date, end_date, budget, dept_id, status) VALUES
(1005, 'Training Program', DATE '2024-01-15', NULL, 50000, 40, 'On Hold');

-- Insert Salary History
INSERT INTO salary_history (history_id, emp_id, old_salary, new_salary, change_date, reason) VALUES
(1, 102, 80000, 85000, DATE '2023-01-15', 'Annual raise');
INSERT INTO salary_history (history_id, emp_id, old_salary, new_salary, change_date, reason) VALUES
(2, 103, 75000, 78000, DATE '2023-06-10', 'Performance bonus');
INSERT INTO salary_history (history_id, emp_id, old_salary, new_salary, change_date, reason) VALUES
(3, 105, 68000, 72000, DATE '2023-01-12', 'Promotion');

COMMIT;
```

## Summary

This script creates a complete employee management database with:
- 4 departments with budgets and locations
- 8 employees with varying salaries and management relationships
- 5 projects with different statuses (Active, Completed, On Hold)
- 3 salary history records tracking compensation changes

The data includes various scenarios useful for demonstrating PL/SQL concepts, including:
- NULL values (some end_dates, manager_ids)
- Hierarchical relationships (employee-manager structure)
- Referential integrity constraints
- Date-based data for temporal queries

