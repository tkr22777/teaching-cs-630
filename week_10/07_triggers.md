# PL/SQL Triggers (Introduction)

## Overview

A **trigger** is a stored PL/SQL block that automatically executes (fires) in response to specific events on a table or view. Triggers are powerful tools for enforcing business rules, maintaining data integrity, and automating database tasks.

## Key Terms

**Trigger**: Stored PL/SQL block that automatically fires when a specific event occurs.

**DML Trigger**: Trigger that fires on INSERT, UPDATE, or DELETE operations.

**Timing**: BEFORE or AFTER the triggering event occurs.

**Event**: The action that causes the trigger to fire (INSERT, UPDATE, DELETE).

**Row-Level Trigger**: Fires once for each row affected by the DML statement.

**Statement-Level Trigger**: Fires once for the entire DML statement.

**:NEW**: Pseudo-record holding the new values for row-level triggers.

**:OLD**: Pseudo-record holding the old values for row-level triggers.

**Trigger Body**: The PL/SQL code that executes when the trigger fires.

**BEFORE Trigger**: Fires before the DML operation executes.

**AFTER Trigger**: Fires after the DML operation executes.

**Mutating Table**: A table currently being modified by a DML statement.

## Sample Database Schema

This module uses the employee management system.

**Note:** The complete database setup script is available in `00_initialization.md` in this directory.

## Why Use Triggers?

### Common Use Cases:

1. **Audit Trails** - Track who changed data and when
2. **Data Validation** - Enforce complex business rules
3. **Automatic Values** - Auto-populate columns
4. **Referential Integrity** - Maintain relationships
5. **Derived Data** - Update calculated values
6. **Security** - Restrict operations based on conditions

## Trigger Syntax

**Basic Syntax:**

```sql
CREATE OR REPLACE TRIGGER trigger_name
{BEFORE | AFTER} {INSERT | UPDATE | DELETE} [OR ...]
ON table_name
[FOR EACH ROW]
[WHEN (condition)]
DECLARE
    -- declarations (optional)
BEGIN
    -- trigger body
EXCEPTION
    -- exception handling (optional)
END;
/
```

## BEFORE Triggers

BEFORE triggers fire before the DML operation executes. They can modify :NEW values.

### Example: Automatic Timestamp

```sql
CREATE OR REPLACE TRIGGER trg_emp_before_insert
BEFORE INSERT ON employees
FOR EACH ROW
BEGIN
    -- Set hire_date to current date if not provided
    IF :NEW.hire_date IS NULL THEN
        :NEW.hire_date := SYSDATE;
    END IF;
    
    DBMS_OUTPUT.PUT_LINE('BEFORE INSERT trigger fired for employee: ' || :NEW.first_name);
END;
/
```

**Testing the Trigger:**

```sql
BEGIN
    INSERT INTO employees (emp_id, first_name, last_name, email, salary, dept_id)
    VALUES (109, 'John', 'Smith', 'john.smith@company.com', 70000, 10);
    
    DBMS_OUTPUT.PUT_LINE('Employee inserted successfully');
    
    ROLLBACK;  -- Preserve original data
END;
/
```

**Output:**
```
BEFORE INSERT trigger fired for employee: John
Employee inserted successfully
```

### Example: Data Validation

```sql
CREATE OR REPLACE TRIGGER trg_emp_salary_validation
BEFORE INSERT OR UPDATE OF salary ON employees
FOR EACH ROW
DECLARE
    v_min_salary CONSTANT NUMBER := 50000;
    v_max_salary CONSTANT NUMBER := 200000;
BEGIN
    -- Validate salary range
    IF :NEW.salary < v_min_salary THEN
        RAISE_APPLICATION_ERROR(-20001, 
            'Salary $' || :NEW.salary || ' is below minimum $' || v_min_salary);
    END IF;
    
    IF :NEW.salary > v_max_salary THEN
        RAISE_APPLICATION_ERROR(-20002,
            'Salary $' || :NEW.salary || ' exceeds maximum $' || v_max_salary);
    END IF;
    
    DBMS_OUTPUT.PUT_LINE('Salary validation passed: $' || :NEW.salary);
END;
/
```

**Testing the Trigger:**

```sql
-- Test 1: Valid salary
BEGIN
    UPDATE employees
    SET salary = 80000
    WHERE emp_id = 108;
    
    DBMS_OUTPUT.PUT_LINE('Update successful');
    ROLLBACK;
END;
/

-- Test 2: Invalid salary (too low)
BEGIN
    UPDATE employees
    SET salary = 45000
    WHERE emp_id = 108;
EXCEPTION
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);
END;
/
```

**Output:**
```
Salary validation passed: $80000
Update successful

Error: ORA-20001: Salary $45000 is below minimum $50000
```

### Example: Automatic Email Generation

```sql
CREATE OR REPLACE TRIGGER trg_emp_email_generation
BEFORE INSERT ON employees
FOR EACH ROW
BEGIN
    -- Generate email if not provided
    IF :NEW.email IS NULL THEN
        :NEW.email := LOWER(:NEW.first_name || '.' || :NEW.last_name || '@company.com');
        DBMS_OUTPUT.PUT_LINE('Auto-generated email: ' || :NEW.email);
    END IF;
END;
/
```

**Testing the Trigger:**

```sql
BEGIN
    INSERT INTO employees (emp_id, first_name, last_name, salary, dept_id)
    VALUES (110, 'Jane', 'Wilson', 75000, 20);
    
    -- Query to see the generated email
    DECLARE
        v_email VARCHAR2(100);
    BEGIN
        SELECT email INTO v_email FROM employees WHERE emp_id = 110;
        DBMS_OUTPUT.PUT_LINE('Email in database: ' || v_email);
    END;
    
    ROLLBACK;
END;
/
```

**Output:**
```
Auto-generated email: jane.wilson@company.com
Email in database: jane.wilson@company.com
```

## AFTER Triggers

AFTER triggers fire after the DML operation completes. They can't modify :NEW values but are useful for auditing and cascading changes.

### Example: Audit Trail

First, create an audit table:

```sql
CREATE TABLE employee_audit (
    audit_id NUMBER PRIMARY KEY,
    emp_id NUMBER,
    operation VARCHAR2(10),
    old_salary NUMBER,
    new_salary NUMBER,
    changed_by VARCHAR2(50),
    change_date DATE
);

CREATE SEQUENCE audit_seq START WITH 1;
```

Now create the trigger:

```sql
CREATE OR REPLACE TRIGGER trg_emp_salary_audit
AFTER UPDATE OF salary ON employees
FOR EACH ROW
BEGIN
    -- Log salary changes
    INSERT INTO employee_audit (
        audit_id, emp_id, operation, 
        old_salary, new_salary, 
        changed_by, change_date
    ) VALUES (
        audit_seq.NEXTVAL, 
        :NEW.emp_id, 
        'UPDATE',
        :OLD.salary, 
        :NEW.salary, 
        USER, 
        SYSDATE
    );
    
    DBMS_OUTPUT.PUT_LINE('Audit record created for employee ' || :NEW.emp_id);
END;
/
```

**Testing the Trigger:**

```sql
BEGIN
    -- Update salary
    UPDATE employees
    SET salary = 87000
    WHERE emp_id = 102;
    
    -- View audit records
    DECLARE
        CURSOR audit_cur IS
            SELECT * FROM employee_audit WHERE emp_id = 102;
    BEGIN
        DBMS_OUTPUT.PUT_LINE('=== AUDIT RECORDS ===');
        FOR rec IN audit_cur LOOP
            DBMS_OUTPUT.PUT_LINE('Employee: ' || rec.emp_id);
            DBMS_OUTPUT.PUT_LINE('Operation: ' || rec.operation);
            DBMS_OUTPUT.PUT_LINE('Old Salary: $' || rec.old_salary);
            DBMS_OUTPUT.PUT_LINE('New Salary: $' || rec.new_salary);
            DBMS_OUTPUT.PUT_LINE('Changed By: ' || rec.changed_by);
            DBMS_OUTPUT.PUT_LINE('Date: ' || TO_CHAR(rec.change_date, 'YYYY-MM-DD HH24:MI:SS'));
        END LOOP;
    END;
    
    ROLLBACK;
END;
/
```

**Output:**
```
Audit record created for employee 102
=== AUDIT RECORDS ===
Employee: 102
Operation: UPDATE
Old Salary: $85000
New Salary: $87000
Changed By: SYSTEM
Date: 2024-11-12 10:45:30
```


## Conditional Triggers (WHEN Clause)

The WHEN clause restricts when a row-level trigger fires:

```sql
CREATE OR REPLACE TRIGGER trg_emp_high_salary_alert
AFTER UPDATE OF salary ON employees
FOR EACH ROW
WHEN (NEW.salary > 100000)
BEGIN
    DBMS_OUTPUT.PUT_LINE('=== HIGH SALARY ALERT ===');
    DBMS_OUTPUT.PUT_LINE('Employee: ' || :NEW.first_name || ' ' || :NEW.last_name);
    DBMS_OUTPUT.PUT_LINE('New Salary: $' || :NEW.salary);
    DBMS_OUTPUT.PUT_LINE('This salary requires executive approval');
END;
/
```

**Testing the Trigger:**

```sql
BEGIN
    -- This won't fire the trigger (salary <= 100000)
    UPDATE employees
    SET salary = 90000
    WHERE emp_id = 108;
    
    DBMS_OUTPUT.PUT_LINE('Update 1 complete (no alert)');
    DBMS_OUTPUT.PUT_LINE('');
    
    -- This will fire the trigger (salary > 100000)
    UPDATE employees
    SET salary = 105000
    WHERE emp_id = 101;
    
    ROLLBACK;
END;
/
```

**Output:**
```
Update 1 complete (no alert)

=== HIGH SALARY ALERT ===
Employee: Sarah Johnson
New Salary: $105000
This salary requires executive approval
```

## Multiple Events in One Trigger

You can handle multiple events in a single trigger:

```sql
CREATE OR REPLACE TRIGGER trg_emp_all_operations
BEFORE INSERT OR UPDATE OR DELETE ON employees
FOR EACH ROW
BEGIN
    IF INSERTING THEN
        DBMS_OUTPUT.PUT_LINE('Inserting employee: ' || :NEW.first_name || ' ' || :NEW.last_name);
        IF :NEW.hire_date IS NULL THEN
            :NEW.hire_date := SYSDATE;
        END IF;
    ELSIF UPDATING THEN
        DBMS_OUTPUT.PUT_LINE('Updating employee ID: ' || :OLD.emp_id);
        DBMS_OUTPUT.PUT_LINE('  Old salary: $' || :OLD.salary);
        DBMS_OUTPUT.PUT_LINE('  New salary: $' || :NEW.salary);
    ELSIF DELETING THEN
        DBMS_OUTPUT.PUT_LINE('Deleting employee: ' || :OLD.first_name || ' ' || :OLD.last_name);
    END IF;
END;
/
```

## Practical Example: Salary Validation System

```sql
-- Create comprehensive salary trigger
CREATE OR REPLACE TRIGGER trg_salary_validation
BEFORE UPDATE OF salary ON employees
FOR EACH ROW
DECLARE
    v_raise_pct NUMBER;
    v_max_raise CONSTANT NUMBER := 0.20;  -- 20% max
BEGIN
    -- Calculate raise percentage
    v_raise_pct := (:NEW.salary - :OLD.salary) / :OLD.salary;
    
    -- Validation: No salary reduction
    IF :NEW.salary < :OLD.salary THEN
        RAISE_APPLICATION_ERROR(-20003, 
            'Salary reduction not allowed without HR approval');
    END IF;
    
    -- Validation: Maximum raise limit
    IF v_raise_pct > v_max_raise THEN
        RAISE_APPLICATION_ERROR(-20004,
            'Raise of ' || ROUND(v_raise_pct * 100, 1) || '% exceeds maximum of ' || 
            (v_max_raise * 100) || '%');
    END IF;
    
    DBMS_OUTPUT.PUT_LINE('Salary change validated for employee ' || :NEW.emp_id);
    DBMS_OUTPUT.PUT_LINE('Raise percentage: ' || ROUND(v_raise_pct * 100, 2) || '%');
END;
/
```

**Testing the System:**

```sql
BEGIN
    DBMS_OUTPUT.PUT_LINE('=== SALARY VALIDATION TEST ===');
    
    -- Test: Valid raise
    UPDATE employees
    SET salary = salary * 1.10
    WHERE emp_id = 108;
    
    ROLLBACK;
END;
/
```

**Output:**
```
=== SALARY VALIDATION TEST ===
Salary change validated for employee 108
Raise percentage: 10%
```

## Trigger Management

### Viewing Triggers

```sql
-- View all triggers on a table
SELECT trigger_name, trigger_type, triggering_event, status
FROM user_triggers
WHERE table_name = 'EMPLOYEES';
```

### Disabling/Enabling Triggers

```sql
-- Disable a trigger
ALTER TRIGGER trg_emp_salary_validation DISABLE;

-- Enable a trigger
ALTER TRIGGER trg_emp_salary_validation ENABLE;

-- Disable all triggers on a table
ALTER TABLE employees DISABLE ALL TRIGGERS;

-- Enable all triggers on a table
ALTER TABLE employees ENABLE ALL TRIGGERS;
```

### Dropping Triggers

```sql
DROP TRIGGER trg_emp_salary_validation;
```

## Best Practices

**DO:**
- Keep triggers simple - complex logic should be in procedures
- Document thoroughly - triggers can be hard to debug
- Use for automation - timestamps, audit trails, data validation
- Test extensively - triggers fire on every DML operation

**DON'T:**
- Avoid long transactions - keep trigger code fast
- Don't modify the same table - causes mutating table errors
- Don't hide business logic - makes code hard to understand
- Don't overuse - too many triggers hurt performance

## Summary

**Key takeaways:**

1. **Triggers** - Automatically execute in response to DML events
2. **Timing** - BEFORE triggers can modify data, AFTER triggers cannot
3. **:NEW and :OLD** - Access new and old values in row-level triggers
4. **Use Cases** - Audit trails, validation, automation
5. **Best Practices** - Keep simple, document well, test thoroughly
6. **Management** - Can be disabled, enabled, or dropped as needed

Triggers are powerful tools for enforcing business rules and automating database operations, but should be used judiciously to maintain system performance and clarity.

