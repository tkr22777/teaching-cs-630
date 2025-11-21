# SELF JOIN and NATURAL JOIN

## SELF JOIN

### Overview

A **SELF JOIN** is a join where a table is joined with itself. This powerful technique is used to compare rows within the same table or to represent hierarchical relationships.

**Key Requirement:** Table aliases are MANDATORY for self joins to distinguish between the two instances of the same table.

### Visual: How SELF JOIN Works

```text
Same Table Used Twice (Employees)

Alias 'e' (Employee Role)          Alias 'm' (Manager Role)
┌──────────┬───────┬────────────┐ ┌──────────┬───────┬────────────┐
│ emp_id   │ name  │ manager_id │ │ emp_id   │ name  │ manager_id │
├──────────┼───────┼────────────┤ ├──────────┼───────┼────────────┤
│ 1        │ Alice │ NULL       │ │ 1        │ Alice │ NULL       │
│ 2        │ Bob   │ 1          │ │ 2        │ Bob   │ 1          │
│ 3        │ Carol │ 1          │ │ 3        │ Carol │ 1          │
└──────────┴───────┴────────────┘ └──────────┴───────┴────────────┘
            ↓                                ↑
         manager_id = 1  ───────────────► emp_id = 1 (MATCH!)
     
     
                ↓ SELF JOIN ON e.manager_id = m.emp_id ↓
            
                        Result
            ┌──────────┬────────────┬──────────┐
            │ employee │ emp_id     │ manager  │
            ├──────────┼────────────┼──────────┤
            │ Bob      │ 2          │ Alice    │  ← Bob's manager_id(1) matches Alice's emp_id(1)
            │ Carol    │ 3          │ Alice    │  ← Carol's manager_id(1) matches Alice's emp_id(1)
            └──────────┴────────────┴──────────┘
```

**How SELF JOIN matches rows:** The same table is used twice with different aliases (`e` for employees, `m` for managers). The join condition `e.manager_id = m.emp_id` links each employee's manager ID to the corresponding manager's employee ID within the same table. This reveals hierarchical relationships: Bob's `manager_id` of 1 matches Alice's `emp_id` of 1, showing Bob reports to Alice.

### Syntax

```sql
SELECT columns
FROM table1 t1
JOIN table1 t2 ON t1.column = t2.column;
```

**Note:** You can use any join type (INNER, LEFT, RIGHT) with self joins.

**How it works:** Join a table to itself using different aliases to compare rows within the same table. Commonly used for hierarchical data (employee-manager relationships).

---

## Common Use Case: Hierarchical Data (Employee-Manager)

**Setup:**

```sql
CREATE TABLE employees_org (
    employee_id INTEGER PRIMARY KEY,
    employee_name VARCHAR2(100),
    position VARCHAR2(50),
    manager_id INTEGER,
    salary NUMBER(10, 2)
);

INSERT INTO employees_org (employee_id, employee_name, position, manager_id, salary) VALUES
(1, 'Alice Johnson', 'CEO', NULL, 150000),
(2, 'Bob Smith', 'VP Engineering', 1, 120000),
(3, 'Carol White', 'VP Sales', 1, 115000),
(4, 'David Brown', 'Senior Engineer', 2, 95000),
(5, 'Eve Davis', 'Engineer', 2, 85000),
(6, 'Frank Miller', 'Sales Rep', 3, 70000),
(7, 'Grace Lee', 'Junior Engineer', 4, 75000);
```

**Employees Table:**

| employee_id | employee_name | position        | manager_id | salary |
| ----------- | ------------- | --------------- | ---------- | ------ |
| 1           | Alice Johnson | CEO             | NULL       | 150000 |
| 2           | Bob Smith     | VP Engineering  | 1          | 120000 |
| 3           | Carol White   | VP Sales        | 1          | 115000 |
| 4           | David Brown   | Senior Engineer | 2          | 95000  |
| 5           | Eve Davis     | Engineer        | 2          | 85000  |
| 6           | Frank Miller  | Sales Rep       | 3          | 70000  |
| 7           | Grace Lee     | Junior Engineer | 4          | 75000  |

### Example 1: List Employees with Their Managers

**Query:**

```sql
SELECT e.employee_name AS employee,
       e.position AS employee_position,
       m.employee_name AS manager,
       m.position AS manager_position
FROM employees_org e
LEFT JOIN employees_org m ON e.manager_id = m.employee_id
ORDER BY e.employee_id;
```

**Result:**

| employee      | employee_position | manager       | manager_position |
| ------------- | ----------------- | ------------- | ---------------- |
| Alice Johnson | CEO               | NULL          | NULL             |
| Bob Smith     | VP Engineering    | Alice Johnson | CEO              |
| Carol White   | VP Sales          | Alice Johnson | CEO              |
| David Brown   | Senior Engineer   | Bob Smith     | VP Engineering   |
| Eve Davis     | Engineer          | Bob Smith     | VP Engineering   |
| Frank Miller  | Sales Rep         | Carol White   | VP Sales         |
| Grace Lee     | Junior Engineer   | David Brown   | Senior Engineer  |

**Explanation:**

- `e` alias represents employees
- `m` alias represents managers (same table)
- LEFT JOIN includes CEO (who has no manager)
- Self join links employee's manager_id to manager's employee_id
