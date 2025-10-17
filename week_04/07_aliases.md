# Aliases in SQL

## Overview

Aliases are temporary names assigned to tables or columns to make queries more readable and concise. This guide covers both column and table aliases in PostgreSQL.

## Sample Data

```sql
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(100),
    city VARCHAR(50)
);

CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
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

### Example 2: Alias Without AS Keyword

**SQL Statement:**
```sql
SELECT first_name fname,
       last_name lname
FROM customers;
```

**Result:**
| fname | lname |
|-------|-------|
| John | Doe |
| Jane | Smith |
| Bob | Johnson |
| Alice | Williams |

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

### Example 4: Alias with Calculations

**SQL Statement:**
```sql
SELECT order_id,
       total_amount,
       total_amount * 0.1 AS tax,
       total_amount * 1.1 AS total_with_tax
FROM orders;
```

**Result:**
| order_id | total_amount | tax | total_with_tax |
|----------|--------------|-----|----------------|
| 1 | 150.00 | 15.00 | 165.00 |
| 2 | 200.50 | 20.05 | 220.55 |
| 3 | 75.25 | 7.53 | 82.78 |
| 4 | 300.00 | 30.00 | 330.00 |
| 5 | 120.75 | 12.08 | 132.83 |

### Example 5: Alias with Aggregate Functions

**SQL Statement:**
```sql
SELECT customer_id,
       COUNT(*) AS order_count,
       SUM(total_amount) AS total_spent,
       AVG(total_amount) AS average_order_value,
       MAX(total_amount) AS largest_order
FROM orders
GROUP BY customer_id;
```

**Result:**
| customer_id | order_count | total_spent | average_order_value | largest_order |
|-------------|-------------|-------------|---------------------|---------------|
| 1 | 3 | 471.25 | 157.08 | 200.50 |
| 2 | 1 | 75.25 | 75.25 | 75.25 |
| 3 | 1 | 300.00 | 300.00 | 300.00 |

### Example 6: Aliases with Spaces (Use Quotes)

**SQL Statement:**
```sql
SELECT first_name AS "First Name",
       last_name AS "Last Name",
       city AS "City of Residence"
FROM customers;
```

**Result:**
| First Name | Last Name | City of Residence |
|------------|-----------|-------------------|
| John | Doe | New York |
| Jane | Smith | Los Angeles |
| Bob | Johnson | Chicago |
| Alice | Williams | Houston |

**Note:** When alias contains spaces, use double quotes. However, it's better to avoid spaces in aliases.

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

### Example 7: Simple Table Alias

**SQL Statement:**
```sql
SELECT c.first_name, 
       c.last_name, 
       c.email
FROM customers c
WHERE c.city = 'New York';
```

**Result:**
| first_name | last_name | email |
|------------|-----------|-------|
| John | Doe | john.doe@email.com |

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

### Example 9: Multiple Table Aliases

**SQL Statement:**
```sql
SELECT c.customer_id,
       c.first_name || ' ' || c.last_name AS customer_name,
       COUNT(o.order_id) AS total_orders,
       COALESCE(SUM(o.total_amount), 0) AS total_spent
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.first_name, c.last_name
ORDER BY total_spent DESC;
```

**Result:**
| customer_id | customer_name | total_orders | total_spent |
|-------------|---------------|--------------|-------------|
| 1 | John Doe | 3 | 471.25 |
| 3 | Bob Johnson | 1 | 300.00 |
| 2 | Jane Smith | 1 | 75.25 |
| 4 | Alice Williams | 0 | 0.00 |

### Example 10: Self-Join with Aliases

**Setup:**
```sql
CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
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

## Combined Column and Table Aliases

### Example 11: Complex Query with Both Alias Types

**SQL Statement:**
```sql
SELECT c.customer_id AS id,
       c.first_name || ' ' || c.last_name AS full_name,
       c.city AS location,
       COUNT(o.order_id) AS number_of_orders,
       COALESCE(SUM(o.total_amount), 0) AS lifetime_value,
       ROUND(COALESCE(AVG(o.total_amount), 0), 2) AS avg_order_value
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.first_name, c.last_name, c.city
HAVING COUNT(o.order_id) > 0
ORDER BY lifetime_value DESC;
```

**Result:**
| id | full_name | location | number_of_orders | lifetime_value | avg_order_value |
|----|-----------|----------|------------------|----------------|-----------------|
| 1 | John Doe | New York | 3 | 471.25 | 157.08 |
| 3 | Bob Johnson | Chicago | 1 | 300.00 | 300.00 |
| 2 | Jane Smith | Los Angeles | 1 | 75.25 | 75.25 |

### Example 12: Subquery with Aliases

**SQL Statement:**
```sql
SELECT customer_summary.name,
       customer_summary.order_count,
       customer_summary.total
FROM (
    SELECT c.first_name || ' ' || c.last_name AS name,
           COUNT(o.order_id) AS order_count,
           SUM(o.total_amount) AS total
    FROM customers c
    JOIN orders o ON c.customer_id = o.customer_id
    GROUP BY c.customer_id, c.first_name, c.last_name
) AS customer_summary
WHERE customer_summary.total > 100
ORDER BY customer_summary.total DESC;
```

**Result:**
| name | order_count | total |
|------|-------------|-------|
| John Doe | 3 | 471.25 |
| Bob Johnson | 1 | 300.00 |

## When to Use Aliases

### Use Column Aliases When:

1. **Creating calculated fields**
```sql
SELECT quantity * unit_price AS line_total
FROM order_items;
```

2. **Concatenating strings**
```sql
SELECT first_name || ' ' || last_name AS full_name
FROM customers;
```

3. **Using aggregate functions**
```sql
SELECT COUNT(*) AS total_orders,
       SUM(amount) AS total_revenue
FROM orders;
```

4. **Improving readability**
```sql
SELECT customer_id AS cust_id,
       order_date AS date_ordered
FROM orders;
```

### Use Table Aliases When:

1. **Simplifying long table names**
```sql
SELECT cust.name
FROM customer_information cust;
```

2. **Joining multiple tables**
```sql
SELECT c.name, o.date
FROM customers c
JOIN orders o ON c.id = o.customer_id;
```

3. **Self-joins (REQUIRED)**
```sql
SELECT e1.name, e2.name AS manager
FROM employees e1
JOIN employees e2 ON e1.manager_id = e2.id;
```

4. **Subqueries**
```sql
SELECT *
FROM (SELECT * FROM orders WHERE status = 'active') AS active_orders;
```

## Best Practices

### 1. Use Meaningful Aliases

```sql
-- Good: Clear and descriptive
SELECT c.first_name AS customer_first_name,
       o.order_date AS purchase_date
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id;

-- Avoid: Single letters that don't match table names
SELECT a.first_name,
       b.order_date
FROM customers a
JOIN orders b ON a.customer_id = b.customer_id;
```

### 2. Keep Table Aliases Short but Meaningful

```sql
-- Good: First letter(s) of table name
FROM customers c
FROM orders o
FROM order_items oi

-- Avoid: Random or confusing aliases
FROM customers xyz
FROM orders abc
```

### 3. Consistent Naming Convention

```sql
-- Good: Consistent style
SELECT COUNT(*) AS order_count,
       SUM(amount) AS total_amount,
       AVG(amount) AS average_amount
FROM orders;

-- Avoid: Inconsistent style
SELECT COUNT(*) AS orderCount,
       SUM(amount) AS TotalAmount,
       AVG(amount) AS avg_amt
FROM orders;
```

## Common Patterns

### Pattern 1: Aggregate with Descriptive Aliases

```sql
SELECT DATE_TRUNC('month', order_date) AS month,
       COUNT(*) AS orders_count,
       SUM(total_amount) AS monthly_revenue,
       AVG(total_amount) AS avg_order_size,
       MAX(total_amount) AS largest_order
FROM orders
GROUP BY DATE_TRUNC('month', order_date)
ORDER BY month;
```

### Pattern 2: Complex Calculations

```sql
SELECT product_name,
       quantity,
       unit_price,
       quantity * unit_price AS subtotal,
       quantity * unit_price * 0.08 AS sales_tax,
       quantity * unit_price * 1.08 AS total
FROM order_items;
```

### Pattern 3: Multiple Joins with Aliases

```sql
SELECT c.first_name || ' ' || c.last_name AS customer,
       o.order_id AS order_number,
       o.order_date AS date,
       p.product_name AS product,
       oi.quantity AS qty,
       oi.unit_price AS price
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
WHERE o.status = 'completed'
ORDER BY o.order_date DESC;
```

