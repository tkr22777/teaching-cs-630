# PL/SQL Cursors

## Overview

A **cursor** is a pointer to a context area that holds the results of a SQL query. Cursors allow you to process query results one row at a time, providing fine-grained control over data retrieval and manipulation.

## Key Terms

**Cursor**: A database object that points to the result set of a query.

**Implicit Cursor**: Automatically created by Oracle for single-row queries and DML statements.

**Explicit Cursor**: Manually declared and controlled by the programmer for multi-row queries.

**Cursor Attributes**: Properties that provide information about cursor state (%FOUND, %NOTFOUND, %ROWCOUNT, %ISOPEN).

**OPEN**: Statement that executes the cursor's query and establishes the result set.

**FETCH**: Statement that retrieves one row from the cursor's result set.

**CLOSE**: Statement that releases the cursor's resources.

**Cursor FOR Loop**: Simplified syntax that automatically opens, fetches, and closes a cursor.

**Cursor Variable**: REF CURSOR type that can point to different queries.

**Result Set**: The collection of rows returned by a cursor's query.

## Sample Database Schema

This module uses the employee management system.

**Note:** The complete database setup script is available in `00_initialization.md` in this directory.

## Implicit Cursors

Oracle automatically creates implicit cursors for all SQL statements. You don't need to declare or manage them explicitly.

### Implicit Cursor Attributes

| Attribute | Description |
|-----------|-------------|
| **SQL%FOUND** | TRUE if the last SQL statement affected one or more rows |
| **SQL%NOTFOUND** | TRUE if the last SQL statement affected zero rows |
| **SQL%ROWCOUNT** | Number of rows affected by the last SQL statement |
| **SQL%ISOPEN** | Always FALSE for implicit cursors (closes automatically) |

**Example: Using Implicit Cursor Attributes**

```sql
DECLARE
    v_emp_id NUMBER := 108;
    v_new_salary NUMBER := 65000;
BEGIN
    -- Update employee salary
    UPDATE employees
    SET salary = v_new_salary
    WHERE emp_id = v_emp_id;
    
    -- Check if update was successful
    IF SQL%FOUND THEN
        DBMS_OUTPUT.PUT_LINE('Employee ' || v_emp_id || ' salary updated');
        DBMS_OUTPUT.PUT_LINE('Rows affected: ' || SQL%ROWCOUNT);
    ELSE
        DBMS_OUTPUT.PUT_LINE('Employee ' || v_emp_id || ' not found');
    END IF;
    
    ROLLBACK;  -- Preserve original data
END;
/
```

**Output:**
```
Employee 108 salary updated
Rows affected: 1
```

**Example: Checking INSERT Results**

```sql
DECLARE
    v_rows_inserted NUMBER;
BEGIN
    -- Insert new project
    INSERT INTO projects (project_id, project_name, start_date, budget, dept_id, status)
    VALUES (1006, 'Data Migration', SYSDATE, 100000, 10, 'Active');
    
    v_rows_inserted := SQL%ROWCOUNT;
    
    DBMS_OUTPUT.PUT_LINE('Projects inserted: ' || v_rows_inserted);
    
    ROLLBACK;  -- Preserve original data
END;
/
```

**Output:**
```
Projects inserted: 1
```

## Explicit Cursors

Explicit cursors give you complete control over multi-row query processing.

### Explicit Cursor Lifecycle

1. **DECLARE** - Define the cursor and its query
2. **OPEN** - Execute the query and establish result set
3. **FETCH** - Retrieve rows one at a time
4. **CLOSE** - Release resources

### Basic Explicit Cursor Syntax

```sql
DECLARE
    -- Cursor declaration
    CURSOR cursor_name IS
        SELECT statement;
    
    -- Variables to hold fetched data
    variable_declarations;
BEGIN
    -- Open cursor
    OPEN cursor_name;
    
    -- Fetch rows
    LOOP
        FETCH cursor_name INTO variables;
        EXIT WHEN cursor_name%NOTFOUND;
        -- Process row
    END LOOP;
    
    -- Close cursor
    CLOSE cursor_name;
END;
```

### Example: Basic Explicit Cursor

```sql
DECLARE
    -- Declare cursor
    CURSOR emp_cursor IS
        SELECT emp_id, first_name, last_name, salary
        FROM employees
        ORDER BY salary DESC;
    
    -- Variables for fetched data
    v_emp_id employees.emp_id%TYPE;
    v_first_name employees.first_name%TYPE;
    v_last_name employees.last_name%TYPE;
    v_salary employees.salary%TYPE;
BEGIN
    DBMS_OUTPUT.PUT_LINE('=== TOP EARNERS ===');
    
    -- Open cursor
    OPEN emp_cursor;
    
    -- Fetch rows
    LOOP
        FETCH emp_cursor INTO v_emp_id, v_first_name, v_last_name, v_salary;
        EXIT WHEN emp_cursor%NOTFOUND;
        
        DBMS_OUTPUT.PUT_LINE(v_emp_id || ': ' || v_first_name || ' ' || 
                            v_last_name || ' - $' || v_salary);
    END LOOP;
    
    -- Close cursor
    CLOSE emp_cursor;
    
    DBMS_OUTPUT.PUT_LINE('Total employees: ' || emp_cursor%ROWCOUNT);
END;
/
```

**Output:**
```
=== TOP EARNERS ===
101: Sarah Johnson - $95000
104: James Wilson - $88000
102: Mike Chen - $85000
103: Emily Davis - $78000
105: Lisa Brown - $72000
106: David Martinez - $68000
107: Jennifer Taylor - $65000
108: Robert Anderson - $62000
Total employees: 8
```

**Explanation:** The cursor retrieves all employees ordered by salary. The FETCH statement loads one row at a time, and %NOTFOUND signals when no more rows exist.

### Explicit Cursor Attributes

| Attribute | Description |
|-----------|-------------|
| **cursor_name%FOUND** | TRUE if the last FETCH returned a row |
| **cursor_name%NOTFOUND** | TRUE if the last FETCH did not return a row |
| **cursor_name%ROWCOUNT** | Number of rows fetched so far |
| **cursor_name%ISOPEN** | TRUE if cursor is open |

**Example: Using Cursor Attributes**

```sql
DECLARE
    CURSOR dept_cursor IS
        SELECT dept_id, dept_name, budget
        FROM departments
        ORDER BY budget DESC;
    
    v_dept_id departments.dept_id%TYPE;
    v_dept_name departments.dept_name%TYPE;
    v_budget departments.budget%TYPE;
BEGIN
    OPEN dept_cursor;
    
    DBMS_OUTPUT.PUT_LINE('=== DEPARTMENTS BY BUDGET ===');
    
    LOOP
        FETCH dept_cursor INTO v_dept_id, v_dept_name, v_budget;
        EXIT WHEN dept_cursor%NOTFOUND;
        
        DBMS_OUTPUT.PUT_LINE('Dept ' || v_dept_id || ': ' || v_dept_name || 
                            ' - $' || TO_CHAR(v_budget, '999,999'));
    END LOOP;
    
    DBMS_OUTPUT.PUT_LINE('Total rows fetched: ' || dept_cursor%ROWCOUNT);
    CLOSE dept_cursor;
END;
/
```

**Output:**
```
=== DEPARTMENTS BY BUDGET ===
Dept 10: Engineering - $500,000
Dept 20: Sales - $300,000
Dept 30: Marketing - $250,000
Dept 40: HR - $150,000
Total rows fetched: 4
```

## Cursor FOR Loop

Cursor FOR loops provide simplified syntax by automatically opening, fetching, and closing cursors.

### Syntax

```sql
FOR record_name IN cursor_name LOOP
    -- statements using record_name.column_name
END LOOP;
```

**Example: Simple Cursor FOR Loop**

```sql
DECLARE
    CURSOR emp_cursor IS
        SELECT first_name, last_name, salary, dept_id
        FROM employees
        WHERE dept_id = 10
        ORDER BY salary DESC;
BEGIN
    DBMS_OUTPUT.PUT_LINE('=== ENGINEERING DEPARTMENT ===');
    
    FOR emp_rec IN emp_cursor LOOP
        DBMS_OUTPUT.PUT_LINE(emp_rec.first_name || ' ' || emp_rec.last_name || 
                            ' - $' || emp_rec.salary);
    END LOOP;
END;
/
```

**Output:**
```
=== ENGINEERING DEPARTMENT ===
Sarah Johnson - $95000
Mike Chen - $85000
Emily Davis - $78000
Robert Anderson - $62000
```

**Benefits of Cursor FOR Loop:**
- No need to OPEN, FETCH, or CLOSE explicitly
- Automatic loop termination
- Cleaner, more readable code
- Record variable automatically declared

### Cursor FOR Loop with Inline Query

You can define the cursor inline without a separate declaration:

```sql
BEGIN
    DBMS_OUTPUT.PUT_LINE('=== ACTIVE PROJECTS ===');
    
    FOR proj IN (SELECT project_name, budget, status 
                 FROM projects 
                 WHERE status = 'Active'
                 ORDER BY budget DESC) LOOP
        DBMS_OUTPUT.PUT_LINE(proj.project_name || ' - $' || 
                            TO_CHAR(proj.budget, '999,999'));
    END LOOP;
END;
/
```

**Output:**
```
=== ACTIVE PROJECTS ===
Mobile App Development - $250,000
Website Redesign - $150,000
Brand Refresh - $120,000
```

## Parameterized Cursors

Cursors can accept parameters to make them more flexible and reusable.

### Syntax

```sql
CURSOR cursor_name (parameter_name datatype) IS
    SELECT statement WHERE column = parameter_name;
```

**Example: Cursor with Parameters**

```sql
DECLARE
    CURSOR emp_by_dept_cursor (p_dept_id NUMBER) IS
        SELECT emp_id, first_name, last_name, salary
        FROM employees
        WHERE dept_id = p_dept_id
        ORDER BY salary DESC;
    
    v_dept_id NUMBER;
    v_dept_name VARCHAR2(50);
BEGIN
    -- Process Engineering Department
    SELECT dept_id, dept_name 
    INTO v_dept_id, v_dept_name
    FROM departments 
    WHERE dept_id = 10;
    
    DBMS_OUTPUT.PUT_LINE('=== ' || v_dept_name || ' Department ===');
    FOR emp IN emp_by_dept_cursor(v_dept_id) LOOP
        DBMS_OUTPUT.PUT_LINE(emp.first_name || ' ' || emp.last_name || ' - $' || emp.salary);
    END LOOP;
    
    DBMS_OUTPUT.PUT_LINE('');
    
    -- Process Sales Department
    SELECT dept_id, dept_name 
    INTO v_dept_id, v_dept_name
    FROM departments 
    WHERE dept_id = 20;
    
    DBMS_OUTPUT.PUT_LINE('=== ' || v_dept_name || ' Department ===');
    FOR emp IN emp_by_dept_cursor(v_dept_id) LOOP
        DBMS_OUTPUT.PUT_LINE(emp.first_name || ' ' || emp.last_name || ' - $' || emp.salary);
    END LOOP;
END;
/
```

**Output:**
```
=== Engineering Department ===
Sarah Johnson - $95000
Mike Chen - $85000
Emily Davis - $78000
Robert Anderson - $62000

=== Sales Department ===
James Wilson - $88000
Lisa Brown - $72000
```

**Explanation:** The same cursor is reused with different department IDs, demonstrating parameter flexibility.

## Cursors with %ROWTYPE

Using %ROWTYPE simplifies variable declarations when fetching entire rows:

```sql
DECLARE
    CURSOR emp_cursor IS
        SELECT * FROM employees WHERE dept_id = 10;
    
    emp_record emp_cursor%ROWTYPE;  -- Matches cursor's row structure
BEGIN
    OPEN emp_cursor;
    
    DBMS_OUTPUT.PUT_LINE('=== EMPLOYEE DETAILS ===');
    
    LOOP
        FETCH emp_cursor INTO emp_record;
        EXIT WHEN emp_cursor%NOTFOUND;
        
        DBMS_OUTPUT.PUT_LINE('ID: ' || emp_record.emp_id);
        DBMS_OUTPUT.PUT_LINE('Name: ' || emp_record.first_name || ' ' || emp_record.last_name);
        DBMS_OUTPUT.PUT_LINE('Email: ' || emp_record.email);
        DBMS_OUTPUT.PUT_LINE('Salary: $' || emp_record.salary);
        DBMS_OUTPUT.PUT_LINE('---');
    END LOOP;
    
    CLOSE emp_cursor;
END;
/
```

**Output:**
```
=== EMPLOYEE DETAILS ===
ID: 101
Name: Sarah Johnson
Email: sarah.j@company.com
Salary: $95000
---
ID: 102
Name: Mike Chen
Email: mike.c@company.com
Salary: $85000
---
ID: 103
Name: Emily Davis
Email: emily.d@company.com
Salary: $78000
---
ID: 108
Name: Robert Anderson
Email: robert.a@company.com
Salary: $62000
---
```

## Practical Example: Department Statistics

```sql
DECLARE
    CURSOR dept_stats_cursor IS
        SELECT d.dept_id, d.dept_name, d.budget,
               COUNT(e.emp_id) AS emp_count,
               NVL(SUM(e.salary), 0) AS total_salary,
               NVL(AVG(e.salary), 0) AS avg_salary,
               NVL(MAX(e.salary), 0) AS max_salary,
               NVL(MIN(e.salary), 0) AS min_salary
        FROM departments d
        LEFT JOIN employees e ON d.dept_id = e.dept_id
        GROUP BY d.dept_id, d.dept_name, d.budget
        ORDER BY d.dept_id;
    
    v_remaining_budget NUMBER;
    v_budget_utilization NUMBER;
BEGIN
    DBMS_OUTPUT.PUT_LINE('=== DEPARTMENT STATISTICS REPORT ===');
    DBMS_OUTPUT.PUT_LINE('');
    
    FOR dept IN dept_stats_cursor LOOP
        v_remaining_budget := dept.budget - dept.total_salary;
        v_budget_utilization := (dept.total_salary / dept.budget) * 100;
        
        DBMS_OUTPUT.PUT_LINE('Department: ' || dept.dept_name || ' (ID: ' || dept.dept_id || ')');
        DBMS_OUTPUT.PUT_LINE('  Employees: ' || dept.emp_count);
        
        IF dept.emp_count > 0 THEN
            DBMS_OUTPUT.PUT_LINE('  Avg Salary: $' || TO_CHAR(ROUND(dept.avg_salary, 2), '999,999'));
            DBMS_OUTPUT.PUT_LINE('  Salary Range: $' || TO_CHAR(dept.min_salary, '999,999') || 
                                ' - $' || TO_CHAR(dept.max_salary, '999,999'));
        END IF;
        
        DBMS_OUTPUT.PUT_LINE('  Total Budget: $' || TO_CHAR(dept.budget, '999,999'));
        DBMS_OUTPUT.PUT_LINE('  Total Salaries: $' || TO_CHAR(dept.total_salary, '999,999'));
        DBMS_OUTPUT.PUT_LINE('  Remaining: $' || TO_CHAR(v_remaining_budget, '999,999'));
        DBMS_OUTPUT.PUT_LINE('  Utilization: ' || ROUND(v_budget_utilization, 1) || '%');
        DBMS_OUTPUT.PUT_LINE('');
    END LOOP;
END;
/
```

**Output:**
```
=== DEPARTMENT STATISTICS REPORT ===

Department: Engineering (ID: 10)
  Employees: 4
  Avg Salary: $ 80,000
  Salary Range: $ 62,000 - $ 95,000
  Total Budget: $500,000
  Total Salaries: $320,000
  Remaining: $180,000
  Utilization: 64%

Department: Sales (ID: 20)
  Employees: 2
  Avg Salary: $ 80,000
  Salary Range: $ 72,000 - $ 88,000
  Total Budget: $300,000
  Total Salaries: $160,000
  Remaining: $140,000
  Utilization: 53.3%

Department: Marketing (ID: 30)
  Employees: 1
  Avg Salary: $ 68,000
  Salary Range: $ 68,000 - $ 68,000
  Total Budget: $250,000
  Total Salaries: $ 68,000
  Remaining: $182,000
  Utilization: 27.2%

Department: HR (ID: 40)
  Employees: 1
  Avg Salary: $ 65,000
  Salary Range: $ 65,000 - $ 65,000
  Total Budget: $150,000
  Total Salaries: $ 65,000
  Remaining: $ 85,000
  Utilization: 43.3%
```

## Summary

**Key takeaways:**

1. **Implicit Cursors** - Automatically created for single-row queries and DML statements
2. **Explicit Cursors** - Manually controlled for multi-row query processing
3. **Cursor Lifecycle** - DECLARE → OPEN → FETCH → CLOSE
4. **Cursor Attributes** - %FOUND, %NOTFOUND, %ROWCOUNT, %ISOPEN provide cursor state information
5. **Cursor FOR Loop** - Simplified syntax that automatically manages cursor lifecycle
6. **Parameterized Cursors** - Accept parameters for flexible, reusable queries
7. **%ROWTYPE** - Simplifies variable declaration for fetching entire rows

Cursors are essential for processing multi-row query results in PL/SQL, providing the flexibility needed for complex data manipulation and reporting tasks.

