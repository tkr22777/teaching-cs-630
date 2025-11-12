# SQL Window Functions

## Overview

**Window functions** perform calculations across a set of rows related to the current row, without collapsing the result set like GROUP BY does. They are essential for analytics, rankings, running totals, and comparing rows within partitions.

## Key Terms

**Window Function**: Function that performs calculations across a "window" of rows related to the current row.

**OVER Clause**: Defines the window (partition and ordering) for the function.

**PARTITION BY**: Divides rows into partitions (groups) for the window function.

**ORDER BY** (in OVER): Defines the order of rows within each partition.

**Frame**: Subset of rows within a partition (e.g., ROWS BETWEEN).

**ROW_NUMBER()**: Assigns unique sequential numbers to rows.

**RANK()**: Assigns ranks with gaps for ties.

**DENSE_RANK()**: Assigns ranks without gaps for ties.

**LEAD()**: Access values from following rows.

**LAG()**: Access values from preceding rows.

**Running Total**: Cumulative sum calculated row by row.

## Sample Database Schema

This module uses the e-commerce system.

**Note:** The complete database setup script is available in `00_initialization.md` in this directory.

## Why Window Functions Matter

Window functions solve common business problems:

- **Rankings**: "Show top 3 products per category"
- **Comparisons**: "Compare each month's sales to previous month"
- **Running Totals**: "Calculate cumulative revenue by date"
- **Moving Averages**: "Calculate 7-day average"
- **Percentiles**: "Find 90th percentile of customer spending"

Without window functions, these require complex self-joins or subqueries.

## Basic Syntax

```sql
function_name() OVER (
    [PARTITION BY column1, column2, ...]
    [ORDER BY column3, column4, ...]
    [ROWS BETWEEN ... AND ...]
)
```

## ROW_NUMBER()

Assigns unique sequential numbers to rows:

```sql
SELECT 
    product_id,
    product_name,
    category,
    price,
    ROW_NUMBER() OVER (ORDER BY price DESC) AS overall_rank,
    ROW_NUMBER() OVER (PARTITION BY category ORDER BY price DESC) AS category_rank
FROM products
ORDER BY category, category_rank;
```

**Output:**
```
PRODUCT_ID | PRODUCT_NAME         | CATEGORY    | PRICE   | OVERALL_RANK | CATEGORY_RANK
-----------|----------------------|-------------|---------|--------------|---------------
2001       | Laptop Pro 15        | Electronics | 1299.99 | 1            | 1
2006       | Monitor 27"          | Electronics | 349.99  | 3            | 2
2010       | Headphones Wireless  | Electronics | 159.99  | 5            | 3
2007       | Keyboard Mechanical  | Electronics | 89.99   | 7            | 4
2009       | Webcam HD            | Electronics | 79.99   | 8            | 5
2002       | Wireless Mouse       | Electronics | 29.99   | 10           | 6
2003       | USB-C Cable          | Electronics | 12.99   | 11           | 7
2005       | Standing Desk        | Furniture   | 399.99  | 2            | 1
2004       | Office Chair         | Furniture   | 249.99  | 4            | 2
2008       | Desk Lamp            | Furniture   | 39.99   | 9            | 3
```

**Explanation:**
- `overall_rank`: Ranks all products by price
- `category_rank`: Ranks products within each category

## RANK() vs DENSE_RANK()

Both handle ties differently:

```sql
SELECT 
    product_name,
    category,
    price,
    RANK() OVER (PARTITION BY category ORDER BY price DESC) AS rank_with_gaps,
    DENSE_RANK() OVER (PARTITION BY category ORDER BY price DESC) AS rank_no_gaps
FROM products
WHERE category = 'Electronics'
ORDER BY rank_with_gaps;
```

**Output:**
```
PRODUCT_NAME         | CATEGORY    | PRICE   | RANK_WITH_GAPS | RANK_NO_GAPS
---------------------|-------------|---------|----------------|-------------
Laptop Pro 15        | Electronics | 1299.99 | 1              | 1
Monitor 27"          | Electronics | 349.99  | 2              | 2
Headphones Wireless  | Electronics | 159.99  | 3              | 3
Keyboard Mechanical  | Electronics | 89.99   | 4              | 4
Webcam HD            | Electronics | 79.99   | 5              | 5
Wireless Mouse       | Electronics | 29.99   | 6              | 6
USB-C Cable          | Electronics | 12.99   | 7              | 7
```

**With Ties Example:**
```sql
-- Simulate tied prices
SELECT 
    product_name,
    ROUND(price/100)*100 AS price_bucket,
    RANK() OVER (ORDER BY ROUND(price/100)*100 DESC) AS rank_with_gaps,
    DENSE_RANK() OVER (ORDER BY ROUND(price/100)*100 DESC) AS rank_no_gaps
FROM products
WHERE category = 'Electronics';
```

**Output:**
```
PRODUCT_NAME         | PRICE_BUCKET | RANK_WITH_GAPS | RANK_NO_GAPS
---------------------|--------------|----------------|-------------
Laptop Pro 15        | 1300         | 1              | 1
Monitor 27"          | 300          | 2              | 2
Headphones Wireless  | 200          | 3              | 3
Keyboard Mechanical  | 100          | 4              | 4
Webcam HD            | 100          | 4              | 4
Wireless Mouse       | 0            | 6              | 5
USB-C Cable          | 0            | 6              | 5
```

**Explanation:**
- **RANK()**: Tied products get same rank (4, 4), next rank skips to 6
- **DENSE_RANK()**: Tied products get same rank (4, 4), next rank is 5 (no gap)

## Top N Per Group

Find top 3 most expensive products per category:

```sql
WITH ranked_products AS (
    SELECT 
        product_id,
        product_name,
        category,
        price,
        RANK() OVER (PARTITION BY category ORDER BY price DESC) AS price_rank
    FROM products
)
SELECT product_id, product_name, category, price, price_rank
FROM ranked_products
WHERE price_rank <= 3
ORDER BY category, price_rank;
```

**Output:**
```
PRODUCT_ID | PRODUCT_NAME         | CATEGORY    | PRICE   | PRICE_RANK
-----------|----------------------|-------------|---------|------------
2001       | Laptop Pro 15        | Electronics | 1299.99 | 1
2006       | Monitor 27"          | Electronics | 349.99  | 2
2010       | Headphones Wireless  | Electronics | 159.99  | 3
2005       | Standing Desk        | Furniture   | 399.99  | 1
2004       | Office Chair         | Furniture   | 249.99  | 2
2008       | Desk Lamp            | Furniture   | 39.99   | 3
```

## LAG() and LEAD()

Access values from previous or next rows:

```sql
SELECT 
    order_id,
    customer_id,
    order_date,
    total_amount,
    LAG(total_amount) OVER (PARTITION BY customer_id ORDER BY order_date) AS previous_order_amount,
    LEAD(total_amount) OVER (PARTITION BY customer_id ORDER BY order_date) AS next_order_amount,
    total_amount - LAG(total_amount) OVER (PARTITION BY customer_id ORDER BY order_date) AS change_from_previous
FROM orders
WHERE customer_id IN (1001, 1003, 1005)
ORDER BY customer_id, order_date;
```

**Output:**
```
ORDER_ID | CUSTOMER_ID | ORDER_DATE | TOTAL_AMOUNT | PREVIOUS | NEXT   | CHANGE
---------|-------------|------------|--------------|----------|--------|--------
3001     | 1001        | 2024-01-15 | 1329.98      | NULL     | 289.98 | NULL
3002     | 1001        | 2024-02-20 | 289.98       | 1329.98  | NULL   | -1040.00
3004     | 1003        | 2024-03-01 | 1659.96      | NULL     | 399.99 | NULL
3005     | 1003        | 2024-03-15 | 399.99       | 1659.96  | NULL   | -1259.97
3008     | 1005        | 2024-04-10 | 1539.97      | NULL     | 249.99 | NULL
3009     | 1005        | 2024-04-25 | 249.99       | 1539.97  | NULL   | -1289.98
```

**Explanation:**
- `LAG()`: Gets value from previous row (NULL for first row)
- `LEAD()`: Gets value from next row (NULL for last row)
- Useful for comparing periods, calculating growth rates

## Running Totals (Cumulative Sum)

Calculate cumulative revenue per customer:

```sql
SELECT 
    order_id,
    customer_id,
    order_date,
    total_amount,
    SUM(total_amount) OVER (
        PARTITION BY customer_id 
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS cumulative_total
FROM orders
WHERE customer_id IN (1001, 1003, 1005)
ORDER BY customer_id, order_date;
```

**Output:**
```
ORDER_ID | CUSTOMER_ID | ORDER_DATE | TOTAL_AMOUNT | CUMULATIVE_TOTAL
---------|-------------|------------|--------------|------------------
3001     | 1001        | 2024-01-15 | 1329.98      | 1329.98
3002     | 1001        | 2024-02-20 | 289.98       | 1619.96
3004     | 1003        | 2024-03-01 | 1659.96      | 1659.96
3005     | 1003        | 2024-03-15 | 399.99       | 2059.95
3008     | 1005        | 2024-04-10 | 1539.97      | 1539.97
3009     | 1005        | 2024-04-25 | 249.99       | 1789.96
```

**Explanation:** Each row shows the cumulative sum of all orders up to and including that order for each customer.

## Moving Average

Calculate 3-order moving average:

```sql
SELECT 
    order_id,
    customer_id,
    order_date,
    total_amount,
    ROUND(AVG(total_amount) OVER (
        PARTITION BY customer_id 
        ORDER BY order_date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ), 2) AS moving_avg_3orders
FROM orders
WHERE customer_id IN (1003, 1005)
ORDER BY customer_id, order_date;
```

**Output:**
```
ORDER_ID | CUSTOMER_ID | ORDER_DATE | TOTAL_AMOUNT | MOVING_AVG_3ORDERS
---------|-------------|------------|--------------|--------------------
3004     | 1003        | 2024-03-01 | 1659.96      | 1659.96
3005     | 1003        | 2024-03-15 | 399.99       | 1029.98
3008     | 1005        | 2024-04-10 | 1539.97      | 1539.97
3009     | 1005        | 2024-04-25 | 249.99       | 894.98
```

**Explanation:** The moving average considers current row and 2 preceding rows (or fewer if near start).

## Practical Example: Customer Purchase Analysis

Comprehensive analysis using multiple window functions:

```sql
WITH customer_orders AS (
    SELECT 
        c.customer_id,
        c.first_name || ' ' || c.last_name AS customer_name,
        o.order_id,
        o.order_date,
        o.total_amount,
        -- Row number for each order
        ROW_NUMBER() OVER (PARTITION BY c.customer_id ORDER BY o.order_date) AS order_number,
        -- Days since previous order
        o.order_date - LAG(o.order_date) OVER (PARTITION BY c.customer_id ORDER BY o.order_date) AS days_since_last_order,
        -- Running total spend
        SUM(o.total_amount) OVER (
            PARTITION BY c.customer_id 
            ORDER BY o.order_date
            ROWS UNBOUNDED PRECEDING
        ) AS lifetime_value,
        -- Average order value so far
        ROUND(AVG(o.total_amount) OVER (
            PARTITION BY c.customer_id 
            ORDER BY o.order_date
            ROWS UNBOUNDED PRECEDING
        ), 2) AS avg_order_value
    FROM customers c
    JOIN orders o ON c.customer_id = o.customer_id
    WHERE o.status != 'Cancelled'
)
SELECT *
FROM customer_orders
WHERE customer_id IN (1001, 1003, 1005)
ORDER BY customer_id, order_date;
```

**Output:**
```
CUSTOMER_ID | CUSTOMER_NAME  | ORDER_ID | ORDER_DATE | TOTAL_AMOUNT | ORDER_NUMBER | DAYS_SINCE_LAST | LIFETIME_VALUE | AVG_ORDER_VALUE
------------|----------------|----------|------------|--------------|--------------|-----------------|----------------|----------------
1001        | Alice Johnson  | 3001     | 2024-01-15 | 1329.98      | 1            | NULL            | 1329.98        | 1329.98
1001        | Alice Johnson  | 3002     | 2024-02-20 | 289.98       | 2            | 36              | 1619.96        | 809.98
1003        | Carol Williams | 3004     | 2024-03-01 | 1659.96      | 1            | NULL            | 1659.96        | 1659.96
1003        | Carol Williams | 3005     | 2024-03-15 | 399.99       | 2            | 14              | 2059.95        | 1029.98
1005        | Emma Davis     | 3008     | 2024-04-10 | 1539.97      | 1            | NULL            | 1539.97        | 1539.97
1005        | Emma Davis     | 3009     | 2024-04-25 | 249.99       | 2            | 15              | 1789.96        | 894.98
```

**Insights:**
- Alice's 2nd order was much smaller and came 36 days after first
- Carol's lifetime value is highest at $2,059.95
- Emma's average order value dropped significantly on 2nd order

## Percent of Total

Calculate each order's percentage of customer's total spend:

```sql
SELECT 
    customer_id,
    order_id,
    total_amount,
    SUM(total_amount) OVER (PARTITION BY customer_id) AS customer_total,
    ROUND(total_amount * 100.0 / SUM(total_amount) OVER (PARTITION BY customer_id), 2) AS pct_of_customer_total
FROM orders
WHERE customer_id IN (1001, 1003, 1005)
  AND status != 'Cancelled'
ORDER BY customer_id, order_id;
```

**Output:**
```
CUSTOMER_ID | ORDER_ID | TOTAL_AMOUNT | CUSTOMER_TOTAL | PCT_OF_CUSTOMER_TOTAL
------------|----------|--------------|----------------|----------------------
1001        | 3001     | 1329.98      | 1619.96        | 82.09
1001        | 3002     | 289.98       | 1619.96        | 17.91
1003        | 3004     | 1659.96      | 2059.95        | 80.58
1003        | 3005     | 399.99       | 2059.95        | 19.42
1005        | 3008     | 1539.97      | 1789.96        | 86.03
1005        | 3009     | 249.99       | 1789.96        | 13.97
```

## Frame Specifications

Control which rows are included in the window:

```sql
-- Running total (all previous rows)
SUM(amount) OVER (ORDER BY date ROWS UNBOUNDED PRECEDING)

-- Moving average (last 3 rows including current)
AVG(amount) OVER (ORDER BY date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW)

-- Centered average (1 before, current, 1 after)
AVG(amount) OVER (ORDER BY date ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING)

-- All rows in partition
SUM(amount) OVER (PARTITION BY category)
```

## Common Patterns

### Pattern 1: Ranking with Row Number
```sql
-- Find 5th most expensive product
WITH ranked AS (
    SELECT product_name, price,
           ROW_NUMBER() OVER (ORDER BY price DESC) AS rn
    FROM products
)
SELECT product_name, price
FROM ranked
WHERE rn = 5;
```

### Pattern 2: Identify First/Last in Group
```sql
-- Find each customer's first and last order
WITH order_flags AS (
    SELECT 
        customer_id,
        order_id,
        order_date,
        ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date ASC) AS is_first,
        ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date DESC) AS is_last
    FROM orders
)
SELECT customer_id, order_id, order_date
FROM order_flags
WHERE is_first = 1 OR is_last = 1;
```

### Pattern 3: Period-over-Period Comparison
```sql
-- Compare each month to previous month
WITH monthly_sales AS (
    SELECT 
        TO_CHAR(order_date, 'YYYY-MM') AS month,
        SUM(total_amount) AS total_sales
    FROM orders
    WHERE status != 'Cancelled'
    GROUP BY TO_CHAR(order_date, 'YYYY-MM')
)
SELECT 
    month,
    total_sales,
    LAG(total_sales) OVER (ORDER BY month) AS previous_month_sales,
    total_sales - LAG(total_sales) OVER (ORDER BY month) AS growth,
    ROUND((total_sales - LAG(total_sales) OVER (ORDER BY month)) * 100.0 / 
          LAG(total_sales) OVER (ORDER BY month), 2) AS growth_pct
FROM monthly_sales;
```

## Summary

**Key takeaways:**

1. **Window Functions** - Perform calculations across related rows without collapsing results
2. **OVER Clause** - Defines the window with PARTITION BY and ORDER BY
3. **ROW_NUMBER()** - Sequential numbering, useful for pagination and deduplication
4. **RANK() vs DENSE_RANK()** - Handle ties differently (with/without gaps)
5. **LAG() and LEAD()** - Access previous/next row values for comparisons
6. **Running Totals** - Cumulative calculations using frame specifications
7. **No GROUP BY** - Results include all rows while performing aggregations

Window functions are essential for analytics, reporting, and solving complex business problems that would otherwise require complicated self-joins or subqueries.

