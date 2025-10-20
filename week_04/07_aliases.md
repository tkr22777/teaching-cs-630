# Aliases in SQL

## Overview

Aliases are temporary names assigned to tables or columns to make queries more readable and concise. This guide covers both column and table aliases using standard SQL.

## Sample Data

<details>
<summary>Click to expand: Database setup script</summary>

```sql
CREATE TABLE customers (
    customer_id INTEGER PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(100),
    city VARCHAR(50)
);

CREATE TABLE orders (
    order_id INTEGER PRIMARY KEY,
    customer_id INTEGER REFERENCES customers(customer_id),
    order_date DATE,
    total_amount NUMERIC(10, 2),
    status VARCHAR(20)
);

INSERT INTO customers (first_name, last_name, email, city) VALUES
('John', 'Doe', 'john.doe@email.com', 'New York'),
('Jane', 'Smith', 'jane.smith@email.com', 'Los Angeles'),
('Bob', 'Johnson', 'bob.j@email.com', 'Chicago'),
('Alice', 'Williams', 'alice.w@email.com', 'Houston');

INSERT INTO orders (customer_id, order_date, total_amount, status) VALUES
(1, '2024-10-01', 150.00, 'completed'),
(1, '2024-10-05', 200.50, 'completed'),
(2, '2024-10-03', 75.25, 'completed'),
(3, '2024-10-07', 300.00, 'pending'),
(1, '2024-10-10', 120.75, 'shipped');
```

</details>

**Customers Table:**
| customer_id | first_name | last_name | email | city |
|-------------|------------|-----------|-------|------|
| 1 | John | Doe | john.doe@email.com | New York |
| 2 | Jane | Smith | jane.smith@email.com | Los Angeles |
| 3 | Bob | Johnson | bob.j@email.com | Chicago |
| 4 | Alice | Williams | alice.w@email.com | Houston |

**Orders Table:**
| order_id | customer_id | order_date | total_amount | status |
|----------|-------------|------------|--------------|--------|
| 1 | 1 | 2024-10-01 | 150.00 | completed |
| 2 | 1 | 2024-10-05 | 200.50 | completed |
| 3 | 2 | 2024-10-03 | 75.25 | completed |
| 4 | 3 | 2024-10-07 | 300.00 | pending |
| 5 | 1 | 2024-10-10 | 120.75 | shipped |

## Column Aliases

Column aliases give temporary names to columns in the SELECT statement.

### Syntax

```sql
SELECT column_name AS alias_name
FROM table_name;

-- AS keyword is optional
SELECT column_name alias_name
FROM table_name;
```

### Example 1: Simple Column Alias

**SQL Statement:**
```sql
SELECT first_name AS fname,
       last_name AS lname,
       email AS email_address
FROM customers;
```

**Result:**
| fname | lname | email_address |
|-------|-------|---------------|
| John | Doe | john.doe@email.com |
| Jane | Smith | jane.smith@email.com |
| Bob | Johnson | bob.j@email.com |
| Alice | Williams | alice.w@email.com |

### Example 3: Calculated Column with Alias

**SQL Statement:**
```sql
SELECT first_name || ' ' || last_name AS full_name,
       email AS contact_email,
       'Customer' AS record_type
FROM customers;
```

**Result:**
| full_name | contact_email | record_type |
|-----------|---------------|-------------|
| John Doe | john.doe@email.com | Customer |
| Jane Smith | jane.smith@email.com | Customer |
| Bob Johnson | bob.j@email.com | Customer |
| Alice Williams | alice.w@email.com | Customer |

###

## Table Aliases

Table aliases provide shorter names for tables, making queries more readable.

### Syntax

```sql
SELECT columns
FROM table_name alias_name;

-- Or with AS keyword
SELECT columns
FROM table_name AS alias_name;
```

### Example 8: Table Aliases in Joins

**SQL Statement:**
```sql
SELECT c.first_name,
       c.last_name,
       o.order_id,
       o.order_date,
       o.total_amount
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE o.status = 'completed'
ORDER BY c.last_name, o.order_date;
```

**Result:**
| first_name | last_name | order_id | order_date | total_amount |
|------------|-----------|----------|------------|--------------|
| John | Doe | 1 | 2024-10-01 | 150.00 |
| John | Doe | 2 | 2024-10-05 | 200.50 |
| Jane | Smith | 3 | 2024-10-03 | 75.25 |

###

### Example 10: Self-Join with Aliases

**Setup:**
```sql
CREATE TABLE employees (
    employee_id INTEGER PRIMARY KEY,
    employee_name VARCHAR(100),
    manager_id INTEGER
);

INSERT INTO employees (employee_name, manager_id) VALUES
('Alice', NULL),
('Bob', 1),
('Charlie', 1),
('David', 2);
```

**SQL Statement:**
```sql
SELECT e.employee_name AS employee,
       m.employee_name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.employee_id
ORDER BY e.employee_id;
```

**Result:**
| employee | manager |
|----------|---------|
| Alice | NULL |
| Bob | Alice |
| Charlie | Alice |
| David | Bob |

**Explanation:** Table aliases (e and m) are REQUIRED for self-joins to distinguish between the two instances of the same table.

##

