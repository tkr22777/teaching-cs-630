# PL/SQL Procedures, Functions, and Packages

## Overview

**Stored procedures** and **functions** are named PL/SQL blocks stored in the database. They promote code reusability, modularity, and easier maintenance. **Packages** group related procedures and functions together, providing organization and additional features like encapsulation.

## Key Terms

**Stored Procedure**: Named PL/SQL block that performs an action, stored in the database.

**Function**: Named PL/SQL block that returns a single value, stored in the database.

**Parameter**: Value passed to or from a procedure or function.

**IN Parameter**: Input parameter passed to a procedure/function (default mode).

**OUT Parameter**: Output parameter that returns a value from a procedure/function.

**IN OUT Parameter**: Parameter that can be both input and output.

**RETURN Statement**: Statement that returns a value from a function.

**Package**: Collection of related procedures, functions, variables, and cursors.

**Package Specification**: Public interface declaring what's available in the package.

**Package Body**: Implementation containing the actual code.

**Overloading**: Defining multiple procedures/functions with the same name but different parameters.

## Sample Database Schema

This module uses the employee management system.

**Note:** The complete database setup script is available in `00_initialization.md` in this directory.

## Stored Procedures

Procedures perform actions but don't return values directly. They can have multiple OUT parameters.

### Creating a Procedure

**Syntax:**

```sql
CREATE OR REPLACE PROCEDURE procedure_name (
    parameter1 mode datatype,
    parameter2 mode datatype
) IS
    -- declarations
BEGIN
    -- executable statements
EXCEPTION
    -- exception handlers
END procedure_name;
/
```

### Example: Simple Procedure

```sql
CREATE OR REPLACE PROCEDURE display_employee (
    p_emp_id IN NUMBER
) IS
    v_first_name VARCHAR2(50);
    v_last_name VARCHAR2(50);
    v_salary NUMBER;
    v_dept_name VARCHAR2(50);
BEGIN
    SELECT e.first_name, e.last_name, e.salary, d.dept_name
    INTO v_first_name, v_last_name, v_salary, v_dept_name
    FROM employees e
    JOIN departments d ON e.dept_id = d.dept_id
    WHERE e.emp_id = p_emp_id;
    
    DBMS_OUTPUT.PUT_LINE('=== EMPLOYEE INFORMATION ===');
    DBMS_OUTPUT.PUT_LINE('Name: ' || v_first_name || ' ' || v_last_name);
    DBMS_OUTPUT.PUT_LINE('Department: ' || v_dept_name);
    DBMS_OUTPUT.PUT_LINE('Salary: $' || v_salary);
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('Error: Employee ' || p_emp_id || ' not found');
END display_employee;
/
```

**Calling the Procedure:**

```sql
BEGIN
    display_employee(101);
END;
/
```

**Output:**
```
=== EMPLOYEE INFORMATION ===
Name: Sarah Johnson
Department: Engineering
Salary: $95000
```

### Example: Procedure with OUT Parameters

```sql
CREATE OR REPLACE PROCEDURE get_employee_stats (
    p_dept_id IN NUMBER,
    p_emp_count OUT NUMBER,
    p_avg_salary OUT NUMBER,
    p_total_salary OUT NUMBER
) IS
BEGIN
    SELECT COUNT(*), 
           NVL(AVG(salary), 0), 
           NVL(SUM(salary), 0)
    INTO p_emp_count, p_avg_salary, p_total_salary
    FROM employees
    WHERE dept_id = p_dept_id;
EXCEPTION
    WHEN OTHERS THEN
        p_emp_count := 0;
        p_avg_salary := 0;
        p_total_salary := 0;
END get_employee_stats;
/
```

**Calling the Procedure:**

```sql
DECLARE
    v_count NUMBER;
    v_avg NUMBER;
    v_total NUMBER;
BEGIN
    get_employee_stats(10, v_count, v_avg, v_total);
    
    DBMS_OUTPUT.PUT_LINE('Department 10 Statistics:');
    DBMS_OUTPUT.PUT_LINE('  Employees: ' || v_count);
    DBMS_OUTPUT.PUT_LINE('  Average Salary: $' || ROUND(v_avg, 2));
    DBMS_OUTPUT.PUT_LINE('  Total Salary: $' || v_total);
END;
/
```

**Output:**
```
Department 10 Statistics:
  Employees: 4
  Average Salary: $80000
  Total Salary: $320000
```


## Functions

Functions return a single value and can be used in SQL statements.

### Creating a Function

**Syntax:**

```sql
CREATE OR REPLACE FUNCTION function_name (
    parameter1 mode datatype,
    parameter2 mode datatype
) RETURN return_datatype IS
    -- declarations
BEGIN
    -- executable statements
    RETURN value;
EXCEPTION
    -- exception handlers
END function_name;
/
```

### Example: Simple Function

```sql
CREATE OR REPLACE FUNCTION get_employee_salary (
    p_emp_id IN NUMBER
) RETURN NUMBER IS
    v_salary NUMBER;
BEGIN
    SELECT salary
    INTO v_salary
    FROM employees
    WHERE emp_id = p_emp_id;
    
    RETURN v_salary;
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        RETURN NULL;
END get_employee_salary;
/
```

**Calling the Function:**

```sql
-- In SQL query
SELECT emp_id, 
       first_name, 
       last_name,
       get_employee_salary(emp_id) AS salary
FROM employees
WHERE emp_id <= 103;
```

**Output:**

| emp_id | first_name | last_name | salary |
|--------|------------|-----------|--------|
| 101 | Sarah | Johnson | 95000 |
| 102 | Mike | Chen | 85000 |
| 103 | Emily | Davis | 78000 |

### Example: Function with Calculation

```sql
CREATE OR REPLACE FUNCTION calculate_annual_compensation (
    p_emp_id IN NUMBER,
    p_bonus_rate IN NUMBER DEFAULT 0.10
) RETURN NUMBER IS
    v_salary NUMBER;
    v_annual_comp NUMBER;
BEGIN
    SELECT salary
    INTO v_salary
    FROM employees
    WHERE emp_id = p_emp_id;
    
    v_annual_comp := v_salary * (1 + p_bonus_rate);
    
    RETURN ROUND(v_annual_comp, 2);
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        RETURN 0;
END calculate_annual_compensation;
/
```

**Calling the Function:**

```sql
SELECT emp_id,
       first_name || ' ' || last_name AS name,
       salary,
       calculate_annual_compensation(emp_id, 0.15) AS total_comp
FROM employees
WHERE dept_id = 10
ORDER BY salary DESC;
```

**Output:**

| emp_id | name | salary | total_comp |
|--------|------|--------|------------|
| 101 | Sarah Johnson | 95000 | 109250 |
| 102 | Mike Chen | 85000 | 97750 |
| 103 | Emily Davis | 78000 | 89700 |
| 108 | Robert Anderson | 62000 | 71300 |

### Example: Function with Multiple Parameters

```sql
CREATE OR REPLACE FUNCTION get_department_budget_status (
    p_dept_id IN NUMBER
) RETURN VARCHAR2 IS
    v_budget NUMBER;
    v_total_salary NUMBER;
    v_remaining NUMBER;
    v_utilization NUMBER;
BEGIN
    SELECT d.budget, NVL(SUM(e.salary), 0)
    INTO v_budget, v_total_salary
    FROM departments d
    LEFT JOIN employees e ON d.dept_id = e.dept_id
    WHERE d.dept_id = p_dept_id
    GROUP BY d.budget;
    
    v_remaining := v_budget - v_total_salary;
    v_utilization := (v_total_salary / v_budget) * 100;
    
    IF v_remaining < 0 THEN
        RETURN 'OVER BUDGET';
    ELSIF v_utilization > 90 THEN
        RETURN 'CRITICAL';
    ELSIF v_utilization > 75 THEN
        RETURN 'HIGH';
    ELSIF v_utilization > 50 THEN
        RETURN 'MODERATE';
    ELSE
        RETURN 'LOW';
    END IF;
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        RETURN 'UNKNOWN';
END get_department_budget_status;
/
```

**Calling the Function:**

```sql
SELECT dept_id,
       dept_name,
       budget,
       get_department_budget_status(dept_id) AS status
FROM departments
ORDER BY dept_id;
```

**Output:**

| dept_id | dept_name | budget | status |
|---------|-----------|--------|--------|
| 10 | Engineering | 500000 | MODERATE |
| 20 | Sales | 300000 | MODERATE |
| 30 | Marketing | 250000 | LOW |
| 40 | HR | 150000 | LOW |

## Packages

Packages group related procedures and functions together, providing better organization and performance.

### Package Components

1. **Package Specification** - Public interface (what users can access)
2. **Package Body** - Implementation (actual code)

### Creating a Package

**Package Specification:**

```sql
CREATE OR REPLACE PACKAGE employee_pkg IS
    -- Public procedures
    PROCEDURE hire_employee(
        p_emp_id IN NUMBER,
        p_first_name IN VARCHAR2,
        p_last_name IN VARCHAR2,
        p_email IN VARCHAR2,
        p_salary IN NUMBER,
        p_dept_id IN NUMBER
    );
    
    PROCEDURE fire_employee(
        p_emp_id IN NUMBER
    );
    
    PROCEDURE give_raise(
        p_emp_id IN NUMBER,
        p_raise_percent IN NUMBER
    );
    
    -- Public functions
    FUNCTION get_employee_count(
        p_dept_id IN NUMBER DEFAULT NULL
    ) RETURN NUMBER;
    
    FUNCTION calculate_years_employed(
        p_emp_id IN NUMBER
    ) RETURN NUMBER;
    
    -- Public constant
    c_standard_raise CONSTANT NUMBER := 0.05;
END employee_pkg;
/
```

**Package Body:**

```sql
CREATE OR REPLACE PACKAGE BODY employee_pkg IS
    
    -- Private procedure (not in spec, only used internally)
    PROCEDURE log_action(
        p_action IN VARCHAR2,
        p_emp_id IN NUMBER
    ) IS
    BEGIN
        DBMS_OUTPUT.PUT_LINE('[LOG] ' || p_action || ' - Employee ' || p_emp_id || 
                            ' at ' || TO_CHAR(SYSDATE, 'YYYY-MM-DD HH24:MI:SS'));
    END log_action;
    
    -- Public procedure implementation
    PROCEDURE hire_employee(
        p_emp_id IN NUMBER,
        p_first_name IN VARCHAR2,
        p_last_name IN VARCHAR2,
        p_email IN VARCHAR2,
        p_salary IN NUMBER,
        p_dept_id IN NUMBER
    ) IS
    BEGIN
        INSERT INTO employees (emp_id, first_name, last_name, email, salary, dept_id, hire_date)
        VALUES (p_emp_id, p_first_name, p_last_name, p_email, p_salary, p_dept_id, SYSDATE);
        
        log_action('HIRE', p_emp_id);
        DBMS_OUTPUT.PUT_LINE('Employee ' || p_first_name || ' ' || p_last_name || ' hired successfully');
    EXCEPTION
        WHEN DUP_VAL_ON_INDEX THEN
            DBMS_OUTPUT.PUT_LINE('Error: Employee ID ' || p_emp_id || ' already exists');
        WHEN OTHERS THEN
            DBMS_OUTPUT.PUT_LINE('Error hiring employee: ' || SQLERRM);
    END hire_employee;
    
    PROCEDURE fire_employee(
        p_emp_id IN NUMBER
    ) IS
        v_emp_name VARCHAR2(100);
    BEGIN
        SELECT first_name || ' ' || last_name
        INTO v_emp_name
        FROM employees
        WHERE emp_id = p_emp_id;
        
        DELETE FROM employees WHERE emp_id = p_emp_id;
        
        log_action('FIRE', p_emp_id);
        DBMS_OUTPUT.PUT_LINE('Employee ' || v_emp_name || ' (ID: ' || p_emp_id || ') removed');
    EXCEPTION
        WHEN NO_DATA_FOUND THEN
            DBMS_OUTPUT.PUT_LINE('Error: Employee ID ' || p_emp_id || ' not found');
    END fire_employee;
    
    PROCEDURE give_raise(
        p_emp_id IN NUMBER,
        p_raise_percent IN NUMBER
    ) IS
        v_old_salary NUMBER;
        v_new_salary NUMBER;
    BEGIN
        SELECT salary
        INTO v_old_salary
        FROM employees
        WHERE emp_id = p_emp_id;
        
        v_new_salary := v_old_salary * (1 + p_raise_percent);
        
        UPDATE employees
        SET salary = v_new_salary
        WHERE emp_id = p_emp_id;
        
        log_action('RAISE', p_emp_id);
        DBMS_OUTPUT.PUT_LINE('Raise applied: $' || v_old_salary || ' → $' || ROUND(v_new_salary, 2));
    EXCEPTION
        WHEN NO_DATA_FOUND THEN
            DBMS_OUTPUT.PUT_LINE('Error: Employee ID ' || p_emp_id || ' not found');
    END give_raise;
    
    FUNCTION get_employee_count(
        p_dept_id IN NUMBER DEFAULT NULL
    ) RETURN NUMBER IS
        v_count NUMBER;
    BEGIN
        IF p_dept_id IS NULL THEN
            SELECT COUNT(*)
            INTO v_count
            FROM employees;
        ELSE
            SELECT COUNT(*)
            INTO v_count
            FROM employees
            WHERE dept_id = p_dept_id;
        END IF;
        
        RETURN v_count;
    END get_employee_count;
    
    FUNCTION calculate_years_employed(
        p_emp_id IN NUMBER
    ) RETURN NUMBER IS
        v_hire_date DATE;
        v_years NUMBER;
    BEGIN
        SELECT hire_date
        INTO v_hire_date
        FROM employees
        WHERE emp_id = p_emp_id;
        
        v_years := TRUNC(MONTHS_BETWEEN(SYSDATE, v_hire_date) / 12);
        
        RETURN v_years;
    EXCEPTION
        WHEN NO_DATA_FOUND THEN
            RETURN NULL;
    END calculate_years_employed;
    
END employee_pkg;
/
```

### Using the Package

```sql
BEGIN
    -- Use package procedures
    DBMS_OUTPUT.PUT_LINE('=== TESTING EMPLOYEE PACKAGE ===');
    DBMS_OUTPUT.PUT_LINE('');
    
    -- Get employee count
    DBMS_OUTPUT.PUT_LINE('Total employees: ' || employee_pkg.get_employee_count);
    DBMS_OUTPUT.PUT_LINE('Engineering employees: ' || employee_pkg.get_employee_count(10));
    DBMS_OUTPUT.PUT_LINE('');
    
    -- Give raise using package constant
    DBMS_OUTPUT.PUT_LINE('Applying standard raise (' || (employee_pkg.c_standard_raise * 100) || '%)');
    employee_pkg.give_raise(108, employee_pkg.c_standard_raise);
    DBMS_OUTPUT.PUT_LINE('');
    
    -- Calculate years employed
    DBMS_OUTPUT.PUT_LINE('Employee 101 years of service: ' || 
                        employee_pkg.calculate_years_employed(101));
    
    -- Rollback to preserve data
    ROLLBACK;
END;
/
```

**Output:**
```
=== TESTING EMPLOYEE PACKAGE ===

Total employees: 8
Engineering employees: 4

Applying standard raise (5%)
[LOG] RAISE - Employee 108 at 2024-11-12 10:30:45
Raise applied: $62000 → $65100

Employee 101 years of service: 4
```


## Summary

**Key takeaways:**

1. **Procedures** - Named blocks that perform actions, can have multiple OUT parameters
2. **Functions** - Named blocks that return a single value, can be used in SQL
3. **Parameters** - IN (input), OUT (output)
4. **Packages** - Group related procedures and functions for better organization
5. **Package Specification** - Public interface showing what's available
6. **Package Body** - Implementation containing actual code
7. **Benefits** - Code reusability, modularity, encapsulation, easier maintenance

Procedures, functions, and packages are essential for building modular, maintainable, and reusable PL/SQL code in enterprise applications.

