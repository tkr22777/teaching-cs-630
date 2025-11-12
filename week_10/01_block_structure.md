# PL/SQL Block Structure

## Overview

**PL/SQL** (Procedural Language/SQL) extends SQL by adding procedural programming capabilities. It allows you to write programs with variables, conditions, loops, and error handling while seamlessly integrating SQL statements.

## Key Terms

**PL/SQL Block**: The basic unit of PL/SQL code, containing declarations, executable statements, and exception handlers.

**Anonymous Block**: A PL/SQL block that is not stored in the database and executes immediately.

**Declaration Section**: The part where variables, constants, and cursors are declared (optional).

**Executable Section**: The part containing the program logic and SQL statements (required).

**Exception Section**: The part that handles runtime errors (optional).

**DBMS_OUTPUT**: Oracle package used to display output from PL/SQL programs.

## What is PL/SQL?

PL/SQL combines the data manipulation power of SQL with the processing power of procedural languages.

### Benefits of PL/SQL:

1. **Better Performance** - Multiple SQL statements sent as a single block
2. **Modular Programming** - Code organized into procedures, functions, and packages
3. **Error Handling** - Built-in exception handling mechanism
4. **Portability** - Runs on any platform that supports Oracle
5. **Integration** - Seamless SQL integration within procedural code

## Sample Database Schema

This module uses the employee management system.

**Note:** The complete database setup script is available in `00_initialization.md` in this directory.

## Basic Block Structure

A PL/SQL block has three sections:

```sql
DECLARE
    -- Declaration section (OPTIONAL)
    -- Variables, constants, cursors declared here
BEGIN
    -- Executable section (REQUIRED)
    -- Program logic and SQL statements here
EXCEPTION
    -- Exception section (OPTIONAL)
    -- Error handling code here
END;
/
```

### Sections Explained:

1. **DECLARE** - Where you define variables, constants, cursors
2. **BEGIN** - Where your program logic executes
3. **EXCEPTION** - Where you handle errors
4. **END;** - Marks the end of the block
5. **/** - Forward slash executes the block

## Anonymous Blocks

An **anonymous block** executes immediately and is not stored in the database.

### Example 1: Simple Anonymous Block

```sql
BEGIN
    DBMS_OUTPUT.PUT_LINE('Hello, PL/SQL!');
END;
/
```

**Output:**
```
Hello, PL/SQL!
```

**Note:** You may need to enable output display:
```sql
SET SERVEROUTPUT ON;
```

### Example 2: Block with Variables

```sql
DECLARE
    v_message VARCHAR2(50);
BEGIN
    v_message := 'Welcome to PL/SQL Programming';
    DBMS_OUTPUT.PUT_LINE(v_message);
END;
/
```

**Output:**
```
Welcome to PL/SQL Programming
```

**Explanation:** 
- `v_message` is declared as a variable
- `:=` is the assignment operator
- `DBMS_OUTPUT.PUT_LINE` displays the message

## Block Structure Components

### Declaration Section

Variables and constants are declared before use:

```sql
DECLARE
    v_emp_name VARCHAR2(50);
    v_salary NUMBER(10, 2);
    v_hire_date DATE;
    c_bonus_rate CONSTANT NUMBER := 0.10;  -- Constant value
BEGIN
    -- Use variables here
    NULL;  -- Placeholder for now
END;
/
```

**Key Points:**
- Variable names typically start with `v_`
- Constant names typically start with `c_`
- Constants must be initialized and cannot change
- `NULL;` is a statement that does nothing (placeholder)

### Executable Section

The executable section contains program logic:

```sql
DECLARE
    v_emp_count NUMBER;
BEGIN
    -- SQL query to count employees
    SELECT COUNT(*)
    INTO v_emp_count
    FROM employees;
    
    DBMS_OUTPUT.PUT_LINE('Total Employees: ' || v_emp_count);
END;
/
```

**Output:**
```
Total Employees: 8
```

**Explanation:**
- `SELECT ... INTO` assigns query result to variable
- `||` concatenates strings
- Every SQL statement must end with semicolon

## Practical Examples

### Example 1: Query and Display Data

```sql
DECLARE
    v_first_name VARCHAR2(50);
    v_last_name VARCHAR2(50);
    v_salary NUMBER(10, 2);
BEGIN
    -- Query single employee
    SELECT first_name, last_name, salary
    INTO v_first_name, v_last_name, v_salary
    FROM employees
    WHERE emp_id = 101;
    
    -- Display results
    DBMS_OUTPUT.PUT_LINE('Employee: ' || v_first_name || ' ' || v_last_name);
    DBMS_OUTPUT.PUT_LINE('Salary: $' || v_salary);
END;
/
```

**Output:**
```
Employee: Sarah Johnson
Salary: $95000
```

**Explanation:** The SELECT INTO statement retrieves data from the database and stores it in PL/SQL variables. Each column in the SELECT clause corresponds to a variable in the INTO clause.

### Example 2: Perform Calculations

```sql
DECLARE
    v_salary NUMBER(10, 2);
    v_annual_bonus NUMBER(10, 2);
    c_bonus_rate CONSTANT NUMBER := 0.15;
BEGIN
    -- Get employee salary
    SELECT salary
    INTO v_salary
    FROM employees
    WHERE emp_id = 102;
    
    -- Calculate bonus
    v_annual_bonus := v_salary * c_bonus_rate;
    
    -- Display results
    DBMS_OUTPUT.PUT_LINE('Current Salary: $' || v_salary);
    DBMS_OUTPUT.PUT_LINE('Annual Bonus (15%): $' || v_annual_bonus);
    DBMS_OUTPUT.PUT_LINE('Total Compensation: $' || (v_salary + v_annual_bonus));
END;
/
```

**Output:**
```
Current Salary: $85000
Annual Bonus (15%): $12750
Total Compensation: $97750
```

**Explanation:** This block demonstrates variable usage, constant declaration, arithmetic operations, and formatted output.

### Example 3: Using %TYPE Attribute

The `%TYPE` attribute declares a variable with the same data type as a column:

```sql
DECLARE
    v_emp_name employees.first_name%TYPE;
    v_emp_salary employees.salary%TYPE;
    v_dept_name departments.dept_name%TYPE;
BEGIN
    -- Query employee and department
    SELECT e.first_name, e.salary, d.dept_name
    INTO v_emp_name, v_emp_salary, v_dept_name
    FROM employees e
    JOIN departments d ON e.dept_id = d.dept_id
    WHERE e.emp_id = 103;
    
    DBMS_OUTPUT.PUT_LINE('Employee: ' || v_emp_name);
    DBMS_OUTPUT.PUT_LINE('Department: ' || v_dept_name);
    DBMS_OUTPUT.PUT_LINE('Salary: $' || v_emp_salary);
END;
/
```

**Output:**
```
Employee: Emily
Department: Engineering
Salary: $78000
```

**Benefits of %TYPE:**
- Automatically matches column data types
- Code adapts to database schema changes
- Reduces maintenance effort

### Example 4: Basic DML Operations

```sql
DECLARE
    v_emp_id employees.emp_id%TYPE := 108;
    v_new_salary NUMBER(10, 2) := 65000;
    v_old_salary NUMBER(10, 2);
BEGIN
    -- Get current salary
    SELECT salary
    INTO v_old_salary
    FROM employees
    WHERE emp_id = v_emp_id;
    
    -- Update salary
    UPDATE employees
    SET salary = v_new_salary
    WHERE emp_id = v_emp_id;
    
    -- Display changes
    DBMS_OUTPUT.PUT_LINE('Employee ID: ' || v_emp_id);
    DBMS_OUTPUT.PUT_LINE('Old Salary: $' || v_old_salary);
    DBMS_OUTPUT.PUT_LINE('New Salary: $' || v_new_salary);
    DBMS_OUTPUT.PUT_LINE('Increase: $' || (v_new_salary - v_old_salary));
    
    -- Rollback to preserve original data
    ROLLBACK;
END;
/
```

**Output:**
```
Employee ID: 108
Old Salary: $62000
New Salary: $65000
Increase: $3000
```

**Explanation:** This demonstrates executing DML (UPDATE) within PL/SQL. The ROLLBACK statement is used here to preserve the original data for future examples.

## Block Nesting

PL/SQL blocks can be nested within other blocks:

```sql
DECLARE
    v_outer_var VARCHAR2(20) := 'Outer';
BEGIN
    DBMS_OUTPUT.PUT_LINE('Outer block: ' || v_outer_var);
    
    -- Nested block
    DECLARE
        v_inner_var VARCHAR2(20) := 'Inner';
    BEGIN
        DBMS_OUTPUT.PUT_LINE('Inner block: ' || v_inner_var);
        DBMS_OUTPUT.PUT_LINE('Can access outer: ' || v_outer_var);
    END;
    
    DBMS_OUTPUT.PUT_LINE('Back in outer block');
END;
/
```

**Output:**
```
Outer block: Outer
Inner block: Inner
Can access outer: Outer
Back in outer block
```

**Scope Rules:**
- Inner blocks can access outer block variables
- Outer blocks cannot access inner block variables
- Inner variables can shadow outer variables with the same name

## Common Patterns

### Pattern 1: Query-Process-Display

```sql
DECLARE
    v_dept_budget NUMBER(12, 2);
    v_emp_count NUMBER;
    v_budget_per_emp NUMBER(10, 2);
BEGIN
    -- Query data
    SELECT d.budget, COUNT(e.emp_id)
    INTO v_dept_budget, v_emp_count
    FROM departments d
    LEFT JOIN employees e ON d.dept_id = e.dept_id
    WHERE d.dept_id = 10
    GROUP BY d.budget;
    
    -- Process
    v_budget_per_emp := v_dept_budget / v_emp_count;
    
    -- Display
    DBMS_OUTPUT.PUT_LINE('Department Budget: $' || v_dept_budget);
    DBMS_OUTPUT.PUT_LINE('Number of Employees: ' || v_emp_count);
    DBMS_OUTPUT.PUT_LINE('Budget Per Employee: $' || ROUND(v_budget_per_emp, 2));
END;
/
```

**Output:**
```
Department Budget: $500000
Number of Employees: 4
Budget Per Employee: $125000
```

## Summary

**Key takeaways:**

1. **Block Structure** - PL/SQL code is organized into blocks with DECLARE, BEGIN, EXCEPTION, and END sections
2. **Anonymous Blocks** - Execute immediately and are not stored in the database
3. **Variables** - Must be declared before use in the DECLARE section
4. **%TYPE Attribute** - Automatically matches column data types
5. **SQL Integration** - SQL statements can be embedded directly in PL/SQL code
6. **DBMS_OUTPUT** - Used to display output from PL/SQL programs

PL/SQL block structure provides the foundation for writing more complex programs with loops, conditions, cursors, and error handling.

