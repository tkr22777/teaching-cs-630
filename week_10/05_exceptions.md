# PL/SQL Exception Handling

## Overview

**Exceptions** are runtime errors that occur during program execution. PL/SQL provides a robust exception handling mechanism that allows you to detect and respond to errors gracefully, preventing program crashes and providing meaningful error messages.

## Key Terms

**Exception**: An error condition that occurs during program execution.

**Exception Handler**: Code block that processes exceptions.

**Predefined Exception**: Built-in Oracle exception with a standard name (e.g., NO_DATA_FOUND).

**User-Defined Exception**: Custom exception declared by the programmer.

**RAISE Statement**: Statement that explicitly triggers an exception.

**EXCEPTION Section**: Part of a PL/SQL block where exception handlers are defined.

**WHEN OTHERS**: Catch-all exception handler that handles any exception not explicitly caught.

**SQLCODE**: Function that returns the error code of the exception.

**SQLERRM**: Function that returns the error message of the exception.

**Exception Propagation**: Process of passing exceptions up to outer blocks when not handled locally.

## Sample Database Schema

This module uses the employee management system.

**Note:** The complete database setup script is available in `00_initialization.md` in this directory.

## Exception Handling Basics

When an exception occurs, normal execution stops and control transfers to the EXCEPTION section.

### Basic Syntax

```sql
DECLARE
    -- declarations
BEGIN
    -- executable statements
EXCEPTION
    WHEN exception_name THEN
        -- handler statements
    WHEN another_exception THEN
        -- handler statements
    WHEN OTHERS THEN
        -- catch-all handler
END;
```

## Predefined Exceptions

Oracle provides many predefined exceptions for common errors:

| Exception Name | Error Code | Description |
|----------------|------------|-------------|
| **NO_DATA_FOUND** | ORA-01403 | SELECT INTO returns no rows |
| **TOO_MANY_ROWS** | ORA-01422 | SELECT INTO returns multiple rows |
| **ZERO_DIVIDE** | ORA-01476 | Division by zero |
| **VALUE_ERROR** | ORA-06502 | Arithmetic, conversion, or constraint error |

### Example: NO_DATA_FOUND Exception

```sql
DECLARE
    v_emp_name VARCHAR2(100);
    v_salary NUMBER;
BEGIN
    -- Try to find employee that doesn't exist
    SELECT first_name || ' ' || last_name, salary
    INTO v_emp_name, v_salary
    FROM employees
    WHERE emp_id = 999;  -- Non-existent ID
    
    DBMS_OUTPUT.PUT_LINE('Employee: ' || v_emp_name);
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('Error: Employee not found');
        DBMS_OUTPUT.PUT_LINE('Please verify the employee ID');
END;
/
```

**Output:**
```
Error: Employee not found
Please verify the employee ID
```

**Explanation:** Without exception handling, this would crash with an unhandled exception. The handler catches the error and displays a user-friendly message.

### Example: TOO_MANY_ROWS Exception

```sql
DECLARE
    v_dept_name VARCHAR2(50);
    v_emp_count NUMBER;
BEGIN
    -- This will fail if multiple departments exist
    SELECT dept_name
    INTO v_dept_name
    FROM departments;  -- Returns multiple rows!
    
    DBMS_OUTPUT.PUT_LINE('Department: ' || v_dept_name);
EXCEPTION
    WHEN TOO_MANY_ROWS THEN
        DBMS_OUTPUT.PUT_LINE('Error: Multiple departments found');
        DBMS_OUTPUT.PUT_LINE('Query returned more than one row');
        
        -- Get actual count
        SELECT COUNT(*)
        INTO v_emp_count
        FROM departments;
        DBMS_OUTPUT.PUT_LINE('Total departments: ' || v_emp_count);
END;
/
```

**Output:**
```
Error: Multiple departments found
Query returned more than one row
Total departments: 4
```

### Example: ZERO_DIVIDE Exception

```sql
DECLARE
    v_total_budget NUMBER;
    v_emp_count NUMBER := 0;
    v_budget_per_emp NUMBER;
BEGIN
    SELECT budget INTO v_total_budget FROM departments WHERE dept_id = 40;
    
    v_budget_per_emp := v_total_budget / v_emp_count;  -- Error!
    
    DBMS_OUTPUT.PUT_LINE('Budget per employee: $' || v_budget_per_emp);
EXCEPTION
    WHEN ZERO_DIVIDE THEN
        DBMS_OUTPUT.PUT_LINE('Error: Cannot divide by zero');
END;
/
```

**Output:**
```
Error: Cannot divide by zero
```

## Multiple Exception Handlers

You can handle different exceptions with specific handlers:

```sql
DECLARE
    v_emp_id NUMBER := 101;
    v_salary NUMBER;
    v_bonus_rate NUMBER := 0;
    v_bonus NUMBER;
BEGIN
    -- Get employee salary
    SELECT salary
    INTO v_salary
    FROM employees
    WHERE emp_id = v_emp_id;
    
    -- Calculate bonus (might cause division by zero)
    v_bonus := v_salary / v_bonus_rate;
    
    DBMS_OUTPUT.PUT_LINE('Bonus: $' || v_bonus);
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('Error: Employee ID ' || v_emp_id || ' not found');
    WHEN ZERO_DIVIDE THEN
        DBMS_OUTPUT.PUT_LINE('Error: Bonus rate cannot be zero');
    WHEN VALUE_ERROR THEN
        DBMS_OUTPUT.PUT_LINE('Error: Invalid numeric value');
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Error: An unexpected error occurred');
        DBMS_OUTPUT.PUT_LINE('Error code: ' || SQLCODE);
        DBMS_OUTPUT.PUT_LINE('Error message: ' || SQLERRM);
END;
/
```

**Output:**
```
Error: Bonus rate cannot be zero
```

## WHEN OTHERS Handler

WHEN OTHERS catches all exceptions not explicitly handled:

```sql
DECLARE
    v_emp_id NUMBER := 101;
    v_result NUMBER;
BEGIN
    -- Some operation that might fail in various ways
    SELECT salary / (emp_id - 101)
    INTO v_result
    FROM employees
    WHERE emp_id = v_emp_id;
    
    DBMS_OUTPUT.PUT_LINE('Result: ' || v_result);
EXCEPTION
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('An error occurred:');
        DBMS_OUTPUT.PUT_LINE('  Error Code: ' || SQLCODE);
        DBMS_OUTPUT.PUT_LINE('  Error Message: ' || SQLERRM);
END;
/
```

**Output:**
```
An error occurred:
  Error Code: -1476
  Error Message: ORA-01476: divisor is equal to zero
```

**Best Practice:** Always place WHEN OTHERS as the last handler, as it catches all remaining exceptions.

## User-Defined Exceptions

You can create custom exceptions for application-specific errors:

### Declaring User-Defined Exceptions

```sql
DECLARE
    -- Declare custom exception
    e_salary_too_low EXCEPTION;
    
    v_emp_id NUMBER := 108;
    v_salary NUMBER;
    c_min_salary CONSTANT NUMBER := 70000;
BEGIN
    -- Get employee salary
    SELECT salary
    INTO v_salary
    FROM employees
    WHERE emp_id = v_emp_id;
    
    -- Check business rule
    IF v_salary < c_min_salary THEN
        RAISE e_salary_too_low;
    END IF;
    
    DBMS_OUTPUT.PUT_LINE('Employee salary: $' || v_salary);
    DBMS_OUTPUT.PUT_LINE('Status: Meets minimum requirement');
EXCEPTION
    WHEN e_salary_too_low THEN
        DBMS_OUTPUT.PUT_LINE('Error: Salary below minimum threshold');
        DBMS_OUTPUT.PUT_LINE('Current salary: $' || v_salary);
        DBMS_OUTPUT.PUT_LINE('Minimum required: $' || c_min_salary);
        DBMS_OUTPUT.PUT_LINE('Adjustment needed: $' || (c_min_salary - v_salary));
END;
/
```

**Output:**
```
Error: Salary below minimum threshold
Current salary: $62000
Minimum required: $70000
Adjustment needed: $8000
```

### Example: Multiple Custom Exceptions

```sql
DECLARE
    -- Custom exceptions
    e_budget_exceeded EXCEPTION;
    e_insufficient_staff EXCEPTION;
    
    v_dept_id NUMBER := 10;
    v_new_salary NUMBER := 100000;
    v_current_total NUMBER;
    v_budget NUMBER;
    v_emp_count NUMBER;
BEGIN
    -- Get department info
    SELECT d.budget, COALESCE(SUM(e.salary), 0), COUNT(e.emp_id)
    INTO v_budget, v_current_total, v_emp_count
    FROM departments d
    LEFT JOIN employees e ON d.dept_id = e.dept_id
    WHERE d.dept_id = v_dept_id
    GROUP BY d.budget;
    
    -- Check business rules
    IF v_current_total + v_new_salary > v_budget THEN
        RAISE e_budget_exceeded;
    END IF;
    
    IF v_emp_count < 2 THEN
        RAISE e_insufficient_staff;
    END IF;
    
    DBMS_OUTPUT.PUT_LINE('Validation passed');
    DBMS_OUTPUT.PUT_LINE('New hire can be added to department ' || v_dept_id);
EXCEPTION
    WHEN e_budget_exceeded THEN
        DBMS_OUTPUT.PUT_LINE('Error: Budget exceeded');
        DBMS_OUTPUT.PUT_LINE('Current total: $' || v_current_total);
        DBMS_OUTPUT.PUT_LINE('New salary: $' || v_new_salary);
        DBMS_OUTPUT.PUT_LINE('Budget: $' || v_budget);
        DBMS_OUTPUT.PUT_LINE('Overage: $' || (v_current_total + v_new_salary - v_budget));
    WHEN e_insufficient_staff THEN
        DBMS_OUTPUT.PUT_LINE('Error: Department needs more staff before hiring');
        DBMS_OUTPUT.PUT_LINE('Current staff: ' || v_emp_count);
END;
/
```

**Output:**
```
Validation passed
New hire can be added to department 10
```

## RAISE_APPLICATION_ERROR

RAISE_APPLICATION_ERROR allows you to create custom error codes and messages:

### Syntax

```sql
RAISE_APPLICATION_ERROR(error_code, error_message);
```

- Error codes must be between -20000 and -20999
- Error message can be up to 2048 bytes

**Example: Custom Error with Code**

```sql
DECLARE
    v_emp_id NUMBER := 101;
    v_new_salary NUMBER := 50000;
    v_current_salary NUMBER;
    c_min_salary CONSTANT NUMBER := 60000;
BEGIN
    -- Get current salary
    SELECT salary
    INTO v_current_salary
    FROM employees
    WHERE emp_id = v_emp_id;
    
    -- Validate new salary
    IF v_new_salary < c_min_salary THEN
        RAISE_APPLICATION_ERROR(-20001, 
            'Salary $' || v_new_salary || ' is below minimum $' || c_min_salary);
    END IF;
    
    IF v_new_salary < v_current_salary THEN
        RAISE_APPLICATION_ERROR(-20002,
            'New salary cannot be lower than current salary of $' || v_current_salary);
    END IF;
    
    DBMS_OUTPUT.PUT_LINE('Salary update approved');
EXCEPTION
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Error Code: ' || SQLCODE);
        DBMS_OUTPUT.PUT_LINE('Error Message: ' || SQLERRM);
END;
/
```

**Output:**
```
Error Code: -20002
Error Message: ORA-20002: New salary cannot be lower than current salary of $95000
```


## Practical Example: Salary Update with Validation

```sql
DECLARE
    -- Custom exceptions
    e_invalid_employee EXCEPTION;
    e_invalid_salary EXCEPTION;
    e_excessive_raise EXCEPTION;
    
    v_emp_id NUMBER := 108;
    v_new_salary NUMBER := 70000;
    v_current_salary NUMBER;
    v_raise_amount NUMBER;
    v_raise_percent NUMBER;
    
    c_max_raise_percent CONSTANT NUMBER := 0.20;  -- 20% max
BEGIN
    DBMS_OUTPUT.PUT_LINE('=== SALARY UPDATE VALIDATION ===');
    DBMS_OUTPUT.PUT_LINE('Employee ID: ' || v_emp_id);
    DBMS_OUTPUT.PUT_LINE('Proposed Salary: $' || v_new_salary);
    DBMS_OUTPUT.PUT_LINE('');
    
    -- Get current salary
    BEGIN
        SELECT salary
        INTO v_current_salary
        FROM employees
        WHERE emp_id = v_emp_id;
    EXCEPTION
        WHEN NO_DATA_FOUND THEN
            RAISE e_invalid_employee;
    END;
    
    -- Validate new salary
    IF v_new_salary <= 0 THEN
        RAISE e_invalid_salary;
    END IF;
    
    -- Calculate raise
    v_raise_amount := v_new_salary - v_current_salary;
    v_raise_percent := v_raise_amount / v_current_salary;
    
    -- Check raise limit
    IF v_raise_percent > c_max_raise_percent THEN
        RAISE e_excessive_raise;
    END IF;
    
    -- All validations passed
    DBMS_OUTPUT.PUT_LINE('VALIDATION PASSED');
    DBMS_OUTPUT.PUT_LINE('Current Salary: $' || v_current_salary);
    DBMS_OUTPUT.PUT_LINE('New Salary: $' || v_new_salary);
    DBMS_OUTPUT.PUT_LINE('Raise Amount: $' || v_raise_amount);
    DBMS_OUTPUT.PUT_LINE('Raise Percent: ' || ROUND(v_raise_percent * 100, 2) || '%');
    DBMS_OUTPUT.PUT_LINE('Status: APPROVED');
    
EXCEPTION
    WHEN e_invalid_employee THEN
        DBMS_OUTPUT.PUT_LINE('ERROR: Employee ID ' || v_emp_id || ' not found');
    WHEN e_invalid_salary THEN
        DBMS_OUTPUT.PUT_LINE('ERROR: Invalid salary amount');
    WHEN e_excessive_raise THEN
        DBMS_OUTPUT.PUT_LINE('ERROR: Raise exceeds maximum allowed');
        DBMS_OUTPUT.PUT_LINE('Requested raise: ' || ROUND(v_raise_percent * 100, 2) || '%');
        DBMS_OUTPUT.PUT_LINE('Maximum allowed: ' || (c_max_raise_percent * 100) || '%');
        DBMS_OUTPUT.PUT_LINE('Maximum salary: $' || ROUND(v_current_salary * (1 + c_max_raise_percent), 2));
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('ERROR: Unexpected error occurred');
        DBMS_OUTPUT.PUT_LINE(SQLERRM);
END;
/
```

**Output:**
```
=== SALARY UPDATE VALIDATION ===
Employee ID: 108
Proposed Salary: $70000

VALIDATION PASSED
Current Salary: $62000
New Salary: $70000
Raise Amount: $8000
Raise Percent: 12.9%
Status: APPROVED
```


## Best Practices for Exception Handling

1. **Be Specific** - Handle specific exceptions rather than relying only on WHEN OTHERS
2. **Provide Meaningful Messages** - Give clear context about what went wrong
3. **Log Error Details** - Use SQLCODE and SQLERRM for troubleshooting
4. **Don't Hide Errors** - Avoid empty WHEN OTHERS blocks that silently fail

## Summary

**Key takeaways:**

1. **Exception Handling** - Catches and responds to runtime errors gracefully
2. **Predefined Exceptions** - Oracle provides standard exceptions for common errors
3. **User-Defined Exceptions** - Create custom exceptions for business logic errors
4. **WHEN OTHERS** - Catch-all handler for unexpected errors
5. **RAISE_APPLICATION_ERROR** - Create custom error codes (-20000 to -20999)
6. **SQLCODE and SQLERRM** - Provide error code and message information

Proper exception handling makes PL/SQL programs more robust, maintainable, and user-friendly by preventing crashes and providing clear error information.

