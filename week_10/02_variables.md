# PL/SQL Variables and Data Types

## Overview

Variables are fundamental to PL/SQL programming. They store data temporarily during program execution and can hold various types of data including numbers, text, dates, and boolean values.

## Key Terms

**Variable**: A named storage location that holds a value that can change during program execution.

**Data Type**: Defines what kind of data a variable can hold (NUMBER, VARCHAR2, DATE, etc.).

**Declaration**: The statement that creates a variable and specifies its data type.

**Initialization**: Assigning an initial value to a variable when it is declared.

**Assignment Operator (:=)**: Used to assign values to variables.

**Constant**: A named value that cannot change after initialization.

**%TYPE**: Attribute that declares a variable with the same type as a database column.

**%ROWTYPE**: Attribute that declares a record variable with the same structure as a database row.

**Scope**: The region of code where a variable is accessible.

**NULL**: A special value representing the absence of a value.

## Sample Database Schema

This module uses the employee management system.

**Note:** The complete database setup script is available in `00_initialization.md` in this directory.

## Variable Declaration Syntax

Basic syntax for declaring variables:

```sql
variable_name datatype [NOT NULL] [:= initial_value];
```

**Examples:**

```sql
DECLARE
    v_emp_name VARCHAR2(50);
    v_salary NUMBER(10, 2) := 0;
    v_hire_date DATE;
    v_is_manager BOOLEAN := FALSE;
    c_tax_rate CONSTANT NUMBER := 0.25;
BEGIN
    NULL;
END;
/
```

## Scalar Data Types

Scalar data types hold a single value.

### Common Data Types

| Category | Data Type | Description | Example |
|----------|-----------|-------------|---------|
| **Numeric** | NUMBER | Fixed/floating-point | `NUMBER(10, 2)` |
| | INTEGER | Whole numbers | `INTEGER` |
| **Character** | VARCHAR2 | Variable-length string | `VARCHAR2(50)` |
| | CHAR | Fixed-length string | `CHAR(10)` |
| **Date/Time** | DATE | Date and time | `DATE` |
| | TIMESTAMP | Date with fractional seconds | `TIMESTAMP` |

**Example: Using Multiple Data Types**

```sql
DECLARE
    v_emp_count INTEGER;
    v_first_name VARCHAR2(50);
    v_last_name VARCHAR2(50);
    v_salary NUMBER(10, 2);
    v_hire_date DATE;
    v_bonus_rate NUMBER := 0.15;
BEGIN
    -- Query employee data
    SELECT first_name, last_name, salary, hire_date
    INTO v_first_name, v_last_name, v_salary, v_hire_date
    FROM employees
    WHERE emp_id = 101;
    
    -- Get department count
    SELECT COUNT(*)
    INTO v_emp_count
    FROM employees
    WHERE dept_id = 10;
    
    DBMS_OUTPUT.PUT_LINE('Employee: ' || v_first_name || ' ' || v_last_name);
    DBMS_OUTPUT.PUT_LINE('Salary: $' || v_salary);
    DBMS_OUTPUT.PUT_LINE('With Bonus: $' || ROUND(v_salary * (1 + v_bonus_rate), 2));
    DBMS_OUTPUT.PUT_LINE('Hire Date: ' || TO_CHAR(v_hire_date, 'Month DD, YYYY'));
    DBMS_OUTPUT.PUT_LINE('Dept Employees: ' || v_emp_count);
END;
/
```

**Output:**
```
Employee: Sarah Johnson
Salary: $95000
With Bonus: $109250
Hire Date: January 15, 2020
Dept Employees: 4
```

### Boolean Type

The BOOLEAN type holds TRUE, FALSE, or NULL values.

**Example:**

```sql
DECLARE
    v_salary NUMBER(10, 2);
    v_is_high_earner BOOLEAN;
    v_is_manager BOOLEAN;
    v_needs_review BOOLEAN;
    v_manager_id employees.manager_id%TYPE;
BEGIN
    SELECT salary, manager_id
    INTO v_salary, v_manager_id
    FROM employees
    WHERE emp_id = 101;
    
    -- Set boolean based on conditions
    v_is_high_earner := v_salary > 90000;
    v_is_manager := v_manager_id IS NULL;
    v_needs_review := v_is_high_earner AND v_is_manager;
    
    DBMS_OUTPUT.PUT_LINE('Salary: $' || v_salary);
    
    IF v_is_high_earner THEN
        DBMS_OUTPUT.PUT_LINE('Status: High Earner');
    END IF;
    
    IF v_is_manager THEN
        DBMS_OUTPUT.PUT_LINE('Role: Manager');
    END IF;
    
    IF v_needs_review THEN
        DBMS_OUTPUT.PUT_LINE('Action: Schedule compensation review');
    END IF;
END;
/
```

**Output:**
```
Salary: $95000
Status: High Earner
Role: Manager
Action: Schedule compensation review
```

## Variable Initialization

Variables can be initialized at declaration or assigned values later:

```sql
DECLARE
    v_status VARCHAR2(20) := 'Active';  -- Initialized at declaration
    v_counter INTEGER := 0;
    v_dept_name VARCHAR2(50);           -- Initialized later
    v_emp_count NUMBER;
BEGIN
    -- Initialize through query
    SELECT dept_name, COUNT(*)
    INTO v_dept_name, v_emp_count
    FROM departments d
    JOIN employees e ON d.dept_id = e.dept_id
    WHERE d.dept_id = 10
    GROUP BY dept_name;
    
    DBMS_OUTPUT.PUT_LINE('Status: ' || v_status);
    DBMS_OUTPUT.PUT_LINE('Counter: ' || v_counter);
    DBMS_OUTPUT.PUT_LINE('Department: ' || v_dept_name);
    DBMS_OUTPUT.PUT_LINE('Employee Count: ' || v_emp_count);
END;
/
```

**Output:**
```
Status: Active
Counter: 0
Department: Engineering
Employee Count: 4
```

## NOT NULL Constraint

Variables can be declared with NOT NULL constraint:

```sql
DECLARE
    v_company_name VARCHAR2(50) NOT NULL := 'TechCorp Inc.';
    v_min_salary NUMBER NOT NULL := 50000;
BEGIN
    DBMS_OUTPUT.PUT_LINE('Company: ' || v_company_name);
    DBMS_OUTPUT.PUT_LINE('Minimum Salary: $' || v_min_salary);
    
    -- This would cause an error:
    -- v_company_name := NULL;
END;
/
```

**Output:**
```
Company: TechCorp Inc.
Minimum Salary: $50000
```

**Note:** NOT NULL variables must be initialized at declaration.

## Constants

Constants are like variables but their values cannot change:

```sql
DECLARE
    c_company_name CONSTANT VARCHAR2(50) := 'TechCorp Inc.';
    c_max_raise CONSTANT NUMBER := 0.20;
    c_bonus_threshold CONSTANT NUMBER := 85000;
    
    v_current_salary NUMBER := 82000;
    v_proposed_raise NUMBER;
    v_new_salary NUMBER;
BEGIN
    v_proposed_raise := 0.15;  -- 15% raise
    
    IF v_proposed_raise > c_max_raise THEN
        v_proposed_raise := c_max_raise;
    END IF;
    
    v_new_salary := v_current_salary * (1 + v_proposed_raise);
    
    DBMS_OUTPUT.PUT_LINE('Company: ' || c_company_name);
    DBMS_OUTPUT.PUT_LINE('Current Salary: $' || v_current_salary);
    DBMS_OUTPUT.PUT_LINE('Approved Raise: ' || (v_proposed_raise * 100) || '%');
    DBMS_OUTPUT.PUT_LINE('New Salary: $' || v_new_salary);
    
    IF v_new_salary >= c_bonus_threshold THEN
        DBMS_OUTPUT.PUT_LINE('Eligible for bonus program');
    END IF;
END;
/
```

**Output:**
```
Company: TechCorp Inc.
Current Salary: $82000
Approved Raise: 15%
New Salary: $94300
Eligible for bonus program
```

## %TYPE Attribute

The %TYPE attribute declares a variable based on a column's data type:

```sql
DECLARE
    v_emp_id employees.emp_id%TYPE;
    v_first_name employees.first_name%TYPE;
    v_salary employees.salary%TYPE;
    v_hire_date employees.hire_date%TYPE;
BEGIN
    v_emp_id := 102;
    
    SELECT first_name, salary, hire_date
    INTO v_first_name, v_salary, v_hire_date
    FROM employees
    WHERE emp_id = v_emp_id;
    
    DBMS_OUTPUT.PUT_LINE('Employee ID: ' || v_emp_id);
    DBMS_OUTPUT.PUT_LINE('Name: ' || v_first_name);
    DBMS_OUTPUT.PUT_LINE('Salary: $' || v_salary);
    DBMS_OUTPUT.PUT_LINE('Hire Date: ' || TO_CHAR(v_hire_date, 'MM/DD/YYYY'));
END;
/
```

**Output:**
```
Employee ID: 102
Name: Mike
Salary: $85000
Hire Date: 03/20/2020
```

**Benefits of %TYPE:**
- Automatically matches column data type
- Adapts to schema changes
- Reduces coding errors
- Improves maintainability

## %ROWTYPE Attribute

The %ROWTYPE attribute declares a record variable with the structure of a table row:

```sql
DECLARE
    v_employee employees%ROWTYPE;
    v_department departments%ROWTYPE;
BEGIN
    -- Query entire row into record
    SELECT *
    INTO v_employee
    FROM employees
    WHERE emp_id = 103;
    
    -- Query department info
    SELECT *
    INTO v_department
    FROM departments
    WHERE dept_id = v_employee.dept_id;
    
    -- Access record fields using dot notation
    DBMS_OUTPUT.PUT_LINE('=== Employee Information ===');
    DBMS_OUTPUT.PUT_LINE('Name: ' || v_employee.first_name || ' ' || v_employee.last_name);
    DBMS_OUTPUT.PUT_LINE('Email: ' || v_employee.email);
    DBMS_OUTPUT.PUT_LINE('Salary: $' || v_employee.salary);
    DBMS_OUTPUT.PUT_LINE('Department: ' || v_department.dept_name);
    DBMS_OUTPUT.PUT_LINE('Location: ' || v_department.location);
END;
/
```

**Output:**
```
=== Employee Information ===
Name: Emily Davis
Email: emily.d@company.com
Salary: $78000
Department: Engineering
Location: Building A
```

**Explanation:** %ROWTYPE creates a record with fields matching all columns in the table. Access fields using `record_name.column_name` notation.

## Variable Scope

Variables are accessible only within their declared scope:

```sql
DECLARE
    v_outer NUMBER := 100;
BEGIN
    DBMS_OUTPUT.PUT_LINE('Outer block - v_outer: ' || v_outer);
    
    -- Inner block with local variable
    DECLARE
        v_inner NUMBER := 200;
        v_outer NUMBER := 300;  -- Shadows outer v_outer
    BEGIN
        DBMS_OUTPUT.PUT_LINE('Inner block - v_inner: ' || v_inner);
        DBMS_OUTPUT.PUT_LINE('Inner block - v_outer: ' || v_outer);
    END;
    
    -- Back in outer block
    DBMS_OUTPUT.PUT_LINE('Outer block - v_outer: ' || v_outer);
    -- v_inner is not accessible here
END;
/
```

**Output:**
```
Outer block - v_outer: 100
Inner block - v_inner: 200
Inner block - v_outer: 300
Outer block - v_outer: 100
```

## Working with NULL Values

NULL represents unknown or missing values. Any arithmetic operation with NULL results in NULL.

```sql
DECLARE
    v_salary NUMBER;
    v_bonus NUMBER := NULL;
    v_total NUMBER;
BEGIN
    SELECT salary INTO v_salary FROM employees WHERE emp_id = 105;
    
    v_total := v_salary + v_bonus;  -- Results in NULL!
    DBMS_OUTPUT.PUT_LINE('Without NVL: ' || NVL(TO_CHAR(v_total), 'NULL'));
    
    v_total := v_salary + NVL(v_bonus, 0);  -- Proper handling
    DBMS_OUTPUT.PUT_LINE('With NVL: $' || v_total);
END;
/
```

**Output:**
```
Without NVL: NULL
With NVL: $72000
```

## Practical Example: Salary Analysis

```sql
DECLARE
    v_emp_id employees.emp_id%TYPE := 102;
    v_employee employees%ROWTYPE;
    v_dept_name departments.dept_name%TYPE;
    v_years_employed NUMBER;
    v_annual_raise NUMBER;
    v_projected_salary NUMBER;
    
    c_annual_raise_rate CONSTANT NUMBER := 0.03;
BEGIN
    -- Get employee data
    SELECT e.*, d.dept_name
    INTO v_employee.emp_id, v_employee.first_name, v_employee.last_name,
         v_employee.email, v_employee.hire_date, v_employee.salary,
         v_employee.dept_id, v_employee.manager_id, v_dept_name
    FROM employees e
    JOIN departments d ON e.dept_id = d.dept_id
    WHERE e.emp_id = v_emp_id;
    
    -- Calculate years employed
    v_years_employed := TRUNC(MONTHS_BETWEEN(SYSDATE, v_employee.hire_date) / 12);
    
    -- Calculate projected salary with annual raises
    v_projected_salary := v_employee.salary * POWER(1 + c_annual_raise_rate, 3);
    
    -- Display analysis
    DBMS_OUTPUT.PUT_LINE('===== SALARY ANALYSIS =====');
    DBMS_OUTPUT.PUT_LINE('Employee: ' || v_employee.first_name || ' ' || v_employee.last_name);
    DBMS_OUTPUT.PUT_LINE('Department: ' || v_dept_name);
    DBMS_OUTPUT.PUT_LINE('Years Employed: ' || v_years_employed);
    DBMS_OUTPUT.PUT_LINE('Current Salary: $' || TO_CHAR(v_employee.salary, '999,999.99'));
    DBMS_OUTPUT.PUT_LINE('Projected Salary (3 years): $' || TO_CHAR(ROUND(v_projected_salary, 2), '999,999.99'));
    DBMS_OUTPUT.PUT_LINE('Total Increase: $' || TO_CHAR(ROUND(v_projected_salary - v_employee.salary, 2), '999,999.99'));
END;
/
```

**Output:**
```
===== SALARY ANALYSIS =====
Employee: Mike Chen
Department: Engineering
Years Employed: 4
Current Salary: $ 85,000.00
Projected Salary (3 years): $ 92,852.73
Total Increase: $  7,852.73
```

## Summary

**Key takeaways:**

1. **Variable Declaration** - Variables must be declared with a data type before use
2. **Data Types** - Common types include NUMBER, VARCHAR2, DATE, and BOOLEAN
3. **Initialization** - Variables can be initialized at declaration or in the executable section
4. **Constants** - Use CONSTANT keyword for values that should not change
5. **%TYPE** - Declares variables matching column data types
6. **%ROWTYPE** - Declares record variables matching entire table rows
7. **Scope** - Variables are accessible only within their declared block
8. **NULL Handling** - Use NVL or COALESCE to handle NULL values properly

Understanding variables and data types is essential for writing effective PL/SQL programs that store and manipulate data efficiently.

