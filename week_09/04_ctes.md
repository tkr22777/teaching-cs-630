# Common Table Expressions (CTEs)

## Overview

A **Common Table Expression (CTE)** is a named temporary result set that exists only during query execution. CTEs make complex queries more readable, maintainable, and can be referenced multiple times within the same query.

## Key Terms

**CTE (Common Table Expression)**: Named temporary result set defined using WITH clause.

**WITH Clause**: Keyword that introduces one or more CTEs.

**Non-Recursive CTE**: Standard CTE that doesn't reference itself.

**Recursive CTE**: CTE that references itself to process hierarchical data.

**Anchor Member**: Initial query in recursive CTE.

**Recursive Member**: Query that references the CTE itself.

**Subquery**: Query nested inside another query (inline).

**Derived Table**: Subquery used in FROM clause.

## Sample Database Schema

This module uses the e-commerce system.

**Note:** The complete database setup script is available in `00_initialization.md` in this directory.

## Why Use CTEs?

CTEs solve several problems:

**Readability**: Break complex queries into logical, named steps
```sql
-- Without CTE (hard to read)
SELECT * FROM (
    SELECT * FROM (
        SELECT customer_id, SUM(total_amount) as total
        FROM orders GROUP BY customer_id
    ) WHERE total > 1000
) WHERE customer_id < 1005;

-- With CTE (clear and readable)
WITH customer_totals AS (
    SELECT customer_id, SUM(total_amount) as total
    FROM orders GROUP BY customer_id
)
SELECT * FROM customer_totals
WHERE total > 1000 AND customer_id < 1005;
```

**Reusability**: Reference the same result set multiple times
**Recursion**: Process hierarchical data (org charts, bill of materials)
**Maintainability**: Easier to modify and debug

## Basic CTE Syntax

```sql
WITH cte_name AS (
    SELECT columns
    FROM tables
    WHERE conditions
)
SELECT *
FROM cte_name;
```

## Simple CTE Example

Find high-value customers:

```sql
WITH high_value_customers AS (
    SELECT 
        customer_id,
        first_name,
        last_name,
        credit_limit
    FROM customers
    WHERE credit_limit > 4000
)
SELECT 
    hvc.customer_id,
    hvc.first_name,
    hvc.last_name,
    COUNT(o.order_id) AS order_count,
    COALESCE(SUM(o.total_amount), 0) AS total_spent
FROM high_value_customers hvc
LEFT JOIN orders o ON hvc.customer_id = o.customer_id
WHERE o.status != 'Cancelled' OR o.status IS NULL
GROUP BY hvc.customer_id, hvc.first_name, hvc.last_name
ORDER BY total_spent DESC;
```

**Output:**
```
CUSTOMER_ID | FIRST_NAME | LAST_NAME | ORDER_COUNT | TOTAL_SPENT
------------|------------|-----------|-------------|-------------
1003        | Carol      | Williams  | 2           | 2059.95
1005        | Emma       | Davis     | 2           | 1789.96
1001        | Alice      | Johnson   | 2           | 1619.96
1006        | Frank      | Miller    | 0           | 0
```

## Multiple CTEs

Chain multiple CTEs together:

```sql
WITH 
-- CTE 1: Calculate customer order totals
customer_totals AS (
    SELECT 
        customer_id,
        COUNT(order_id) AS order_count,
        SUM(total_amount) AS total_spent
    FROM orders
    WHERE status != 'Cancelled'
    GROUP BY customer_id
),
-- CTE 2: Calculate average order value
customer_averages AS (
    SELECT 
        customer_id,
        order_count,
        total_spent,
        ROUND(total_spent / order_count, 2) AS avg_order_value
    FROM customer_totals
    WHERE order_count > 0
),
-- CTE 3: Categorize customers
customer_segments AS (
    SELECT 
        ca.*,
        CASE 
            WHEN ca.total_spent >= 2000 THEN 'VIP'
            WHEN ca.total_spent >= 1000 THEN 'Gold'
            WHEN ca.total_spent >= 500 THEN 'Silver'
            ELSE 'Bronze'
        END AS segment
    FROM customer_averages ca
)
SELECT 
    c.customer_id,
    c.first_name || ' ' || c.last_name AS customer_name,
    cs.order_count,
    cs.total_spent,
    cs.avg_order_value,
    cs.segment
FROM customers c
JOIN customer_segments cs ON c.customer_id = cs.customer_id
ORDER BY cs.total_spent DESC;
```

**Output:**
```
CUSTOMER_ID | CUSTOMER_NAME  | ORDER_COUNT | TOTAL_SPENT | AVG_ORDER_VALUE | SEGMENT
------------|----------------|-------------|-------------|-----------------|--------
1003        | Carol Williams | 2           | 2059.95     | 1029.98         | VIP
1005        | Emma Davis     | 2           | 1789.96     | 894.98          | Gold
1001        | Alice Johnson  | 2           | 1619.96     | 809.98          | Gold
1002        | Bob Smith      | 1           | 649.98      | 649.98          | Silver
1004        | David Brown    | 1           | 449.98      | 449.98          | Bronze
```

**Explanation:** Each CTE builds on the previous ones, creating a logical pipeline of transformations.

## CTEs vs Subqueries

### CTE Version (Readable)

```sql
WITH order_summary AS (
    SELECT 
        o.customer_id,
        COUNT(*) AS order_count,
        SUM(o.total_amount) AS total
    FROM orders o
    WHERE o.status = 'Delivered'
    GROUP BY o.customer_id
)
SELECT 
    c.first_name,
    c.last_name,
    os.order_count,
    os.total
FROM customers c
JOIN order_summary os ON c.customer_id = os.customer_id
WHERE os.total > 1000;
```

### Subquery Version (Harder to Read)

```sql
SELECT 
    c.first_name,
    c.last_name,
    order_stats.order_count,
    order_stats.total
FROM customers c
JOIN (
    SELECT 
        o.customer_id,
        COUNT(*) AS order_count,
        SUM(o.total_amount) AS total
    FROM orders o
    WHERE o.status = 'Delivered'
    GROUP BY o.customer_id
) order_stats ON c.customer_id = order_stats.customer_id
WHERE order_stats.total > 1000;
```

Both produce the same result, but CTE is clearer.

## Practical Example: Product Analytics

Comprehensive product analysis using CTEs:

```sql
WITH 
-- Step 1: Calculate product sales statistics
product_sales AS (
    SELECT 
        p.product_id,
        p.product_name,
        p.category,
        p.price,
        COUNT(od.detail_id) AS times_ordered,
        SUM(od.quantity) AS total_quantity_sold,
        SUM(od.quantity * od.unit_price) AS total_revenue
    FROM products p
    LEFT JOIN order_details od ON p.product_id = od.product_id
    LEFT JOIN orders o ON od.order_id = o.order_id
    WHERE o.status != 'Cancelled' OR o.status IS NULL
    GROUP BY p.product_id, p.product_name, p.category, p.price
),
-- Step 2: Calculate category totals
category_totals AS (
    SELECT 
        category,
        SUM(total_revenue) AS category_revenue
    FROM product_sales
    GROUP BY category
),
-- Step 3: Calculate metrics and rankings
product_metrics AS (
    SELECT 
        ps.*,
        ct.category_revenue,
        ROUND(ps.total_revenue * 100.0 / NULLIF(ct.category_revenue, 0), 2) AS pct_of_category_revenue,
        RANK() OVER (PARTITION BY ps.category ORDER BY ps.total_revenue DESC) AS revenue_rank_in_category,
        RANK() OVER (ORDER BY ps.total_revenue DESC) AS revenue_rank_overall
    FROM product_sales ps
    LEFT JOIN category_totals ct ON ps.category = ct.category
)
SELECT 
    product_id,
    product_name,
    category,
    price,
    COALESCE(times_ordered, 0) AS times_ordered,
    COALESCE(total_quantity_sold, 0) AS units_sold,
    COALESCE(total_revenue, 0) AS revenue,
    COALESCE(pct_of_category_revenue, 0) AS pct_of_category,
    revenue_rank_in_category,
    revenue_rank_overall
FROM product_metrics
ORDER BY category, revenue_rank_in_category;
```

**Output:**
```
PRODUCT_ID | PRODUCT_NAME         | CATEGORY    | PRICE   | TIMES_ORDERED | UNITS_SOLD | REVENUE | PCT_OF_CATEGORY | RANK_IN_CATEGORY | RANK_OVERALL
-----------|----------------------|-------------|---------|---------------|------------|---------|-----------------|------------------|-------------
2001       | Laptop Pro 15        | Electronics | 1299.99 | 3             | 3          | 3899.97 | 79.38           | 1                | 1
2006       | Monitor 27"          | Electronics | 349.99  | 2             | 2          | 699.98  | 14.25           | 2                | 3
2010       | Headphones Wireless  | Electronics | 159.99  | 1             | 1          | 159.99  | 3.26            | 3                | 7
2007       | Keyboard Mechanical  | Electronics | 89.99   | 1             | 1          | 89.99   | 1.83            | 4                | 9
2002       | Wireless Mouse       | Electronics | 29.99   | 2             | 3          | 59.98   | 1.22            | 5                | 10
2003       | USB-C Cable          | Electronics | 12.99   | 1             | 5          | 12.99   | 0.26            | 6                | 11
2009       | Webcam HD            | Electronics | 79.99   | 0             | 0          | 0       | 0               | 7                | 12
2005       | Standing Desk        | Furniture   | 399.99  | 3             | 3          | 1199.97 | 70.58           | 1                | 2
2004       | Office Chair         | Furniture   | 249.99  | 3             | 3          | 749.97  | 44.12           | 2                | 4
2008       | Desk Lamp            | Furniture   | 39.99   | 2             | 2          | 79.98   | 4.70            | 3                | 8
```

**Insights:**
- Laptop Pro 15 is the top revenue generator overall and in Electronics
- Electronics category generates more revenue than Furniture
- Webcam HD has never been ordered

## Recursive CTEs

Recursive CTEs process hierarchical data like org charts, bill of materials, or folder structures.

### Syntax

```sql
WITH RECURSIVE cte_name (columns) AS (
    -- Anchor member (base case)
    SELECT initial_data
    
    UNION ALL
    
    -- Recursive member (references cte_name)
    SELECT recursive_data
    FROM cte_name
    JOIN other_tables
    WHERE termination_condition
)
SELECT * FROM cte_name;
```

### Example: Employee Hierarchy Concept

```sql
-- Conceptual example for organization chart
WITH RECURSIVE emp_hierarchy AS (
    -- Anchor: Start with top-level employee (CEO)
    SELECT employee_id, name, manager_id, 1 AS level
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive: Find direct reports
    SELECT e.employee_id, e.name, e.manager_id, eh.level + 1
    FROM employees e
    JOIN emp_hierarchy eh ON e.manager_id = eh.employee_id
    WHERE eh.level < 10  -- Limit depth to prevent infinite recursion
)
SELECT 
    LPAD(' ', (level-1)*2, ' ') || name AS org_chart,
    level
FROM emp_hierarchy
ORDER BY level, employee_id;
```

**How it works:**
1. Anchor finds the CEO (no manager)
2. Recursive step finds employees managed by previous level
3. Level increments with each recursion until no more employees found

**Common use cases:**
- Organization charts (employee → manager)
- Product hierarchies (component → parent assembly)
- Category trees (subcategory → parent category)

## CTE Best Practices

### 1. Use Descriptive Names

```sql
-- Bad
WITH t1 AS (...), t2 AS (...), t3 AS (...)

-- Good
WITH 
    high_value_customers AS (...),
    customer_order_stats AS (...),
    customer_segments AS (...)
```

### 2. Add Comments for Complex Logic

```sql
WITH customer_metrics AS (
    -- Calculate lifetime value and recency for each customer
    -- Excludes cancelled orders and includes only active customers
    SELECT ...
)
```

### 3. Break Down Complex Queries

```sql
-- Instead of one massive query, use CTEs to break it into logical steps
WITH
    step1_filter AS (...),      -- Filter raw data
    step2_aggregate AS (...),   -- Aggregate to customer level
    step3_enrich AS (...),      -- Add calculated fields
    step4_rank AS (...)         -- Rank and categorize
SELECT * FROM step4_rank;
```

### 4. Avoid Deep Recursion

Set a recursion limit for safety:

```sql
WITH RECURSIVE hierarchy AS (
    SELECT *, 1 AS level FROM ...
    UNION ALL
    SELECT *, level + 1 FROM hierarchy WHERE level < 10  -- Limit depth
)
```

## Performance Considerations

### CTEs are Evaluated Once

Oracle materializes CTEs, so they're evaluated once even if referenced multiple times:

```sql
WITH expensive_calculation AS (
    SELECT customer_id, complex_calculation(data) AS result
    FROM large_table
)
SELECT a.* FROM expensive_calculation a
UNION ALL
SELECT b.* FROM expensive_calculation b WHERE b.result > 100;
-- expensive_calculation runs only once!
```

### Inline vs CTE

For simple one-time-use queries, inline subqueries might be optimized better:

```sql
-- CTE (good for readability, reuse)
WITH recent_orders AS (
    SELECT * FROM orders WHERE order_date > SYSDATE - 7
)
SELECT * FROM recent_orders;

-- Inline (might optimize better for single use)
SELECT * FROM orders WHERE order_date > SYSDATE - 7;
```

## Summary

**Key takeaways:**

1. **CTEs** - Named temporary result sets using WITH clause
2. **Readability** - Break complex queries into logical, named steps
3. **Multiple CTEs** - Chain multiple CTEs for step-by-step transformations
4. **Reusability** - Reference the same CTE multiple times in a query
5. **Recursive CTEs** - Process hierarchical data like org charts
6. **vs Subqueries** - CTEs are more readable and reusable than nested subqueries
7. **Best Practice** - Use descriptive names and comments for complex logic

CTEs are essential for writing maintainable SQL that handles complex business logic and hierarchical data structures.

