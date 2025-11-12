# PL/SQL Control Structures

## Overview

Control structures allow you to control the flow of program execution. They enable conditional logic (IF statements), repetitive operations (loops), and sequential execution. These are essential for building complex business logic in PL/SQL programs.

## Key Terms

**Control Structure**: Statement that determines the flow of program execution.

**Conditional Statement**: Statement that executes code based on a condition (IF-THEN-ELSE).

**Loop**: Statement that repeats a block of code multiple times.

**LOOP**: Basic loop that continues until explicitly exited.

**WHILE LOOP**: Loop that continues while a condition is TRUE.

**FOR LOOP**: Loop that iterates a specific number of times.

**EXIT Statement**: Terminates a loop immediately.

**CONTINUE Statement**: Skips the current iteration and continues with the next.

**Nested Loop**: A loop inside another loop.

**Iteration**: One execution of the loop body.

## Sample Database Schema

This module uses the employee management system.

**Note:** The complete database setup script is available in `00_initialization.md` in this directory.

## IF Statement

The IF statement executes code conditionally based on boolean expressions.

### IF-THEN Syntax

```sql
IF condition THEN
    -- statements
END IF;
```

**Example: Simple Condition**

```sql
DECLARE
    v_salary NUMBER;
    v_emp_name VARCHAR2(100);
BEGIN
    SELECT first_name || ' ' || last_name, salary
    INTO v_emp_name, v_salary
    FROM employees
    WHERE emp_id = 101;
    
    DBMS_OUTPUT.PUT_LINE('Employee: ' || v_emp_name);
    DBMS_OUTPUT.PUT_LINE('Salary: $' || v_salary);
    
    IF v_salary > 90000 THEN
        DBMS_OUTPUT.PUT_LINE('Status: High Earner');
    END IF;
END;
/
```

**Output:**
```
Employee: Sarah Johnson
Salary: $95000
Status: High Earner
```

### IF-THEN-ELSE Syntax

```sql
IF condition THEN
    -- statements when TRUE
ELSE
    -- statements when FALSE
END IF;
```

**Example: Binary Choice**

```sql
DECLARE
    v_salary NUMBER;
    v_bonus NUMBER;
BEGIN
    SELECT salary
    INTO v_salary
    FROM employees
    WHERE emp_id = 108;
    
    IF v_salary >= 70000 THEN
        v_bonus := v_salary * 0.10;
        DBMS_OUTPUT.PUT_LINE('Bonus Rate: 10%');
    ELSE
        v_bonus := v_salary * 0.05;
        DBMS_OUTPUT.PUT_LINE('Bonus Rate: 5%');
    END IF;
    
    DBMS_OUTPUT.PUT_LINE('Salary: $' || v_salary);
    DBMS_OUTPUT.PUT_LINE('Bonus: $' || v_bonus);
END;
/
```

**Output:**
```
Bonus Rate: 5%
Salary: $62000
Bonus: $3100
```

### IF-THEN-ELSIF-ELSE Syntax

```sql
IF condition1 THEN
    -- statements when condition1 is TRUE
ELSIF condition2 THEN
    -- statements when condition2 is TRUE
ELSIF condition3 THEN
    -- statements when condition3 is TRUE
ELSE
    -- statements when all conditions are FALSE
END IF;
```

**Example: Multiple Conditions**

```sql
DECLARE
    v_salary NUMBER;
    v_performance_rating VARCHAR2(20);
    v_raise_pct NUMBER;
BEGIN
    SELECT salary
    INTO v_salary
    FROM employees
    WHERE emp_id = 103;
    
    IF v_salary >= 90000 THEN
        v_performance_rating := 'Exceptional';
        v_raise_pct := 0.08;
    ELSIF v_salary >= 75000 THEN
        v_performance_rating := 'Excellent';
        v_raise_pct := 0.06;
    ELSIF v_salary >= 65000 THEN
        v_performance_rating := 'Good';
        v_raise_pct := 0.04;
    ELSE
        v_performance_rating := 'Needs Improvement';
        v_raise_pct := 0.02;
    END IF;
    
    DBMS_OUTPUT.PUT_LINE('Current Salary: $' || v_salary);
    DBMS_OUTPUT.PUT_LINE('Rating: ' || v_performance_rating);
    DBMS_OUTPUT.PUT_LINE('Raise Percentage: ' || (v_raise_pct * 100) || '%');
    DBMS_OUTPUT.PUT_LINE('New Salary: $' || ROUND(v_salary * (1 + v_raise_pct), 2));
END;
/
```

**Output:**
```
Current Salary: $78000
Rating: Excellent
Raise Percentage: 6%
New Salary: $82680
```

## CASE Statement

CASE provides an alternative to IF-ELSIF for multiple conditions.

### Simple CASE

```sql
DECLARE
    v_dept_id NUMBER := 10;
    v_dept_category VARCHAR2(20);
BEGIN
    v_dept_category := CASE v_dept_id
        WHEN 10 THEN 'Technical'
        WHEN 20 THEN 'Sales'
        WHEN 30 THEN 'Marketing'
        WHEN 40 THEN 'Support'
        ELSE 'Other'
    END;
    
    DBMS_OUTPUT.PUT_LINE('Department ID: ' || v_dept_id);
    DBMS_OUTPUT.PUT_LINE('Category: ' || v_dept_category);
END;
/
```

**Output:**
```
Department ID: 10
Category: Technical
```

### Searched CASE

```sql
DECLARE
    v_emp_id NUMBER := 105;
    v_salary NUMBER;
    v_hire_date DATE;
    v_employee_level VARCHAR2(20);
BEGIN
    SELECT salary, hire_date
    INTO v_salary, v_hire_date
    FROM employees
    WHERE emp_id = v_emp_id;
    
    v_employee_level := CASE
        WHEN v_salary >= 90000 AND MONTHS_BETWEEN(SYSDATE, v_hire_date) >= 60 THEN 'Senior'
        WHEN v_salary >= 75000 AND MONTHS_BETWEEN(SYSDATE, v_hire_date) >= 36 THEN 'Mid-Level'
        WHEN v_salary >= 60000 THEN 'Junior'
        ELSE 'Entry-Level'
    END;
    
    DBMS_OUTPUT.PUT_LINE('Employee ID: ' || v_emp_id);
    DBMS_OUTPUT.PUT_LINE('Salary: $' || v_salary);
    DBMS_OUTPUT.PUT_LINE('Level: ' || v_employee_level);
END;
/
```

**Output:**
```
Employee ID: 105
Salary: $72000
Level: Junior
```

## Basic LOOP

The basic LOOP repeats statements until explicitly terminated with EXIT.

### Syntax

```sql
LOOP
    -- statements
    EXIT WHEN condition;
END LOOP;
```

**Example:**

```sql
DECLARE
    v_counter NUMBER := 1;
BEGIN
    LOOP
        DBMS_OUTPUT.PUT_LINE('Counter: ' || v_counter);
        v_counter := v_counter + 1;
        
        EXIT WHEN v_counter > 5;
    END LOOP;
    
    DBMS_OUTPUT.PUT_LINE('Loop completed');
END;
/
```

**Output:**
```
Counter: 1
Counter: 2
Counter: 3
Counter: 4
Counter: 5
Loop completed
```

**Explanation:** The loop continues until the EXIT WHEN condition becomes TRUE.

## WHILE LOOP

WHILE LOOP executes statements as long as a condition remains TRUE.

### Syntax

```sql
WHILE condition LOOP
    -- statements
END LOOP;
```

**Example:**

```sql
DECLARE
    v_amount NUMBER := 10000;
    v_target NUMBER := 15000;
    v_years NUMBER := 0;
BEGIN
    WHILE v_amount < v_target LOOP
        v_years := v_years + 1;
        v_amount := v_amount * 1.05;  -- 5% growth
        DBMS_OUTPUT.PUT_LINE('Year ' || v_years || ': $' || ROUND(v_amount, 2));
    END LOOP;
    
    DBMS_OUTPUT.PUT_LINE('Target reached in ' || v_years || ' years');
END;
/
```

**Output:**
```
Year 1: $10500
Year 2: $11025
Year 3: $11576.25
Year 4: $12155.06
Year 5: $12762.82
Year 6: $13400.96
Year 7: $14071.01
Year 8: $14774.55
Year 9: $15513.28
Target reached in 9 years
```

## FOR LOOP

FOR LOOP iterates a predetermined number of times.

### Syntax

```sql
FOR counter IN [REVERSE] lower_bound..upper_bound LOOP
    -- statements
END LOOP;
```

**Example: Simple Iteration**

```sql
DECLARE
    v_total NUMBER := 0;
BEGIN
    FOR i IN 1..10 LOOP
        v_total := v_total + i;
        DBMS_OUTPUT.PUT_LINE('i = ' || i || ', Total = ' || v_total);
    END LOOP;
    
    DBMS_OUTPUT.PUT_LINE('Sum of 1 to 10: ' || v_total);
END;
/
```

**Output:**
```
i = 1, Total = 1
i = 2, Total = 3
i = 3, Total = 6
i = 4, Total = 10
i = 5, Total = 15
i = 6, Total = 21
i = 7, Total = 28
i = 8, Total = 36
i = 9, Total = 45
i = 10, Total = 55
Sum of 1 to 10: 55
```

**Example: REVERSE Loop**

```sql
BEGIN
    DBMS_OUTPUT.PUT_LINE('Countdown:');
    
    FOR i IN REVERSE 1..5 LOOP
        DBMS_OUTPUT.PUT_LINE(i);
    END LOOP;
    
    DBMS_OUTPUT.PUT_LINE('Liftoff!');
END;
/
```

**Output:**
```
Countdown:
5
4
3
2
1
Liftoff!
```

**Example: Generate Salary Projections**

```sql
DECLARE
    v_current_salary NUMBER := 70000;
    v_annual_raise NUMBER := 0.04;  -- 4% annual raise
BEGIN
    DBMS_OUTPUT.PUT_LINE('=== 5-Year Salary Projection ===');
    DBMS_OUTPUT.PUT_LINE('Starting Salary: $' || v_current_salary);
    DBMS_OUTPUT.PUT_LINE('Annual Raise: ' || (v_annual_raise * 100) || '%');
    DBMS_OUTPUT.PUT_LINE('---');
    
    FOR year IN 1..5 LOOP
        v_current_salary := v_current_salary * (1 + v_annual_raise);
        DBMS_OUTPUT.PUT_LINE('Year ' || year || ': $' || ROUND(v_current_salary, 2));
    END LOOP;
END;
/
```

**Output:**
```
=== 5-Year Salary Projection ===
Starting Salary: $70000
Annual Raise: 4%
---
Year 1: $72800
Year 2: $75712
Year 3: $78740.48
Year 4: $81890.1
Year 5: $85165.7
```

## CONTINUE Statement

CONTINUE skips the current iteration and moves to the next.

**Example: Skip Even Numbers**

```sql
DECLARE
    v_sum_odd NUMBER := 0;
BEGIN
    FOR i IN 1..10 LOOP
        -- Skip even numbers
        CONTINUE WHEN MOD(i, 2) = 0;
        
        v_sum_odd := v_sum_odd + i;
        DBMS_OUTPUT.PUT_LINE('Processing odd number: ' || i);
    END LOOP;
    
    DBMS_OUTPUT.PUT_LINE('---');
    DBMS_OUTPUT.PUT_LINE('Sum of odd numbers (1-10): ' || v_sum_odd);
END;
/
```

**Output:**
```
Processing odd number: 1
Processing odd number: 3
Processing odd number: 5
Processing odd number: 7
Processing odd number: 9
---
Sum of odd numbers (1-10): 25
```

## Practical Example: Department Budget Analysis

```sql
DECLARE
    v_dept_id departments.dept_id%TYPE;
    v_dept_name departments.dept_name%TYPE;
    v_budget departments.budget%TYPE;
    v_emp_count NUMBER;
    v_total_salary NUMBER;
    v_remaining_budget NUMBER;
    v_dept_status VARCHAR2(20);
BEGIN
    DBMS_OUTPUT.PUT_LINE('=== DEPARTMENT BUDGET ANALYSIS ===');
    DBMS_OUTPUT.PUT_LINE('');
    
    -- Loop through each department
    FOR dept IN (SELECT dept_id, dept_name, budget FROM departments ORDER BY dept_id) LOOP
        v_dept_id := dept.dept_id;
        v_dept_name := dept.dept_name;
        v_budget := dept.budget;
        
        -- Count employees and sum salaries
        SELECT COUNT(*), NVL(SUM(salary), 0)
        INTO v_emp_count, v_total_salary
        FROM employees
        WHERE dept_id = v_dept_id;
        
        -- Calculate remaining budget
        v_remaining_budget := v_budget - v_total_salary;
        
        -- Determine status
        IF v_remaining_budget < 0 THEN
            v_dept_status := 'OVER BUDGET';
        ELSIF v_remaining_budget < v_budget * 0.1 THEN
            v_dept_status := 'LOW BUDGET';
        ELSE
            v_dept_status := 'HEALTHY';
        END IF;
        
        -- Display results
        DBMS_OUTPUT.PUT_LINE('Department: ' || v_dept_name);
        DBMS_OUTPUT.PUT_LINE('  Employees: ' || v_emp_count);
        DBMS_OUTPUT.PUT_LINE('  Total Budget: $' || TO_CHAR(v_budget, '999,999'));
        DBMS_OUTPUT.PUT_LINE('  Total Salaries: $' || TO_CHAR(v_total_salary, '999,999'));
        DBMS_OUTPUT.PUT_LINE('  Remaining: $' || TO_CHAR(v_remaining_budget, '999,999'));
        DBMS_OUTPUT.PUT_LINE('  Status: ' || v_dept_status);
        DBMS_OUTPUT.PUT_LINE('');
    END LOOP;
END;
/
```

**Output:**
```
=== DEPARTMENT BUDGET ANALYSIS ===

Department: Engineering
  Employees: 4
  Total Budget: $500,000
  Total Salaries: $320,000
  Remaining: $180,000
  Status: HEALTHY

Department: Sales
  Employees: 2
  Total Budget: $300,000
  Total Salaries: $160,000
  Remaining: $140,000
  Status: HEALTHY

Department: Marketing
  Employees: 1
  Total Budget: $250,000
  Total Salaries: $ 68,000
  Remaining: $182,000
  Status: HEALTHY

Department: HR
  Employees: 1
  Total Budget: $150,000
  Total Salaries: $ 65,000
  Remaining: $ 85,000
  Status: HEALTHY
```

**Explanation:** This example demonstrates a cursor FOR loop (which we'll explore more in the next module) combined with conditional logic to analyze each department's budget status.

## Summary

**Key takeaways:**

1. **IF Statements** - Execute code conditionally using IF-THEN, IF-THEN-ELSE, and IF-THEN-ELSIF
2. **CASE Statements** - Alternative to multiple IF-ELSIF conditions for cleaner code
3. **Basic LOOP** - Repeats until EXIT statement is encountered
4. **WHILE LOOP** - Repeats while a condition is TRUE
5. **FOR LOOP** - Iterates a specific number of times with automatic counter management
6. **CONTINUE** - Skips current iteration and moves to next
7. **Nested Loops** - Loops within loops for complex iterations
8. **Loop Control** - Use EXIT and CONTINUE to control loop flow

Control structures are fundamental to building complex business logic and processing data efficiently in PL/SQL programs.

