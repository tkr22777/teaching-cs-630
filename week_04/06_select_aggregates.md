# SELECT with Aggregate Functions

## Overview

When you need summary information - totals, averages, counts - you'll use aggregate functions. Instead of returning every row, these functions calculate a single result from multiple rows. This is essential for reports, analytics, and understanding your data.

## Sample Data

<details>
<summary>Click to expand: Database setup script</summary>

```sql
CREATE TABLE sales (
    sale_id INTEGER PRIMARY KEY,
    product_name VARCHAR2(100),
    category VARCHAR2(50),
    quantity INTEGER,
    unit_price NUMBER(10, 2),
    sale_date DATE,
    region VARCHAR2(50)
);

INSERT INTO sales (sale_id, product_name, category, quantity, unit_price, sale_date, region) VALUES (1, 'Laptop', 'Electronics', 5, 999.99, DATE '2024-10-01', 'North');
INSERT INTO sales (sale_id, product_name, category, quantity, unit_price, sale_date, region) VALUES (2, 'Mouse', 'Electronics', 20, 25.50, DATE '2024-10-01', 'North');
INSERT INTO sales (sale_id, product_name, category, quantity, unit_price, sale_date, region) VALUES (3, 'Desk', 'Furniture', 3, 450.00, DATE '2024-10-02', 'South');
INSERT INTO sales (sale_id, product_name, category, quantity, unit_price, sale_date, region) VALUES (4, 'Chair', 'Furniture', 10, 199.99, DATE '2024-10-02', 'South');
INSERT INTO sales (sale_id, product_name, category, quantity, unit_price, sale_date, region) VALUES (5, 'Laptop', 'Electronics', 3, 999.99, DATE '2024-10-03', 'East');
INSERT INTO sales (sale_id, product_name, category, quantity, unit_price, sale_date, region) VALUES (6, 'Monitor', 'Electronics', 8, 299.99, DATE '2024-10-03', 'East');
INSERT INTO sales (sale_id, product_name, category, quantity, unit_price, sale_date, region) VALUES (7, 'Desk', 'Furniture', 5, 450.00, DATE '2024-10-04', 'West');
INSERT INTO sales (sale_id, product_name, category, quantity, unit_price, sale_date, region) VALUES (8, 'Laptop', 'Electronics', 2, 999.99, DATE '2024-10-05', 'North');
INSERT INTO sales (sale_id, product_name, category, quantity, unit_price, sale_date, region) VALUES (9, 'Mouse', 'Electronics', 15, 25.50, DATE '2024-10-05', 'West');
INSERT INTO sales (sale_id, product_name, category, quantity, unit_price, sale_date, region) VALUES (10, 'Chair', 'Furniture', 12, 199.99, DATE '2024-10-06', 'East');
```

</details>

**Sales Table:**
| sale_id | product_name | category | quantity | unit_price | sale_date | region |
|---------|--------------|----------|----------|------------|-----------|--------|
| 1 | Laptop | Electronics | 5 | 999.99 | 2024-10-01 | North |
| 2 | Mouse | Electronics | 20 | 25.50 | 2024-10-01 | North |
| 3 | Desk | Furniture | 3 | 450.00 | 2024-10-02 | South |
| 4 | Chair | Furniture | 10 | 199.99 | 2024-10-02 | South |
| 5 | Laptop | Electronics | 3 | 999.99 | 2024-10-03 | East |
| 6 | Monitor | Electronics | 8 | 299.99 | 2024-10-03 | East |
| 7 | Desk | Furniture | 5 | 450.00 | 2024-10-04 | West |
| 8 | Laptop | Electronics | 2 | 999.99 | 2024-10-05 | North |
| 9 | Mouse | Electronics | 15 | 25.50 | 2024-10-05 | West |
| 10 | Chair | Furniture | 12 | 199.99 | 2024-10-06 | East |

## Aggregate Functions

**Aggregate functions** calculate a single result from multiple rows: `COUNT()` counts rows, `SUM()` adds values, `AVG()` calculates average, `MAX()` finds largest, `MIN()` finds smallest.

**Example:**
```sql
SELECT COUNT(*) AS total_sales,
       SUM(quantity) AS total_quantity,
       AVG(unit_price) AS avg_price,
       MAX(sale_date) AS last_sale,
       MIN(sale_date) AS first_sale
FROM sales;
```

These functions ignore NULL values (except COUNT(*) which counts all rows). For example, a SUM of 5 rows with 2 NULLs only adds 3 values.

---

## GROUP BY Clause

**GROUP BY** groups rows that have the same values in specified columns, allowing you to calculate aggregates for each group.

### Example 11: Sales by Category

**SQL Statement:**
```sql
SELECT category,
       COUNT(*) AS number_of_sales,
       SUM(quantity) AS total_quantity,
       SUM(quantity * unit_price) AS total_revenue
FROM sales
GROUP BY category
ORDER BY total_revenue DESC;
```

**Result:**
| category | number_of_sales | total_quantity | total_revenue |
|----------|-----------------|----------------|---------------|
| Electronics | 6 | 53 | 12299.18 |
| Furniture | 4 | 30 | 6599.93 |

Notice how GROUP BY splits the data into separate groups, and each aggregate function calculates for its group.

---

## HAVING Clause

**HAVING** filters groups after GROUP BY (WHERE filters rows before grouping).

### Example 16: Filter Groups

**SQL Statement:**
```sql
SELECT category,
       COUNT(*) AS sales_count,
       SUM(quantity * unit_price) AS total_revenue
FROM sales
GROUP BY category
HAVING SUM(quantity * unit_price) > 7000
ORDER BY total_revenue DESC;
```

**Result:**
| category | sales_count | total_revenue |
|----------|-------------|---------------|
| Electronics | 6 | 13292.32 |

##

<details>
<summary>Advanced Statistical Aggregates</summary>

## STDDEV and VARIANCE Functions

Standard SQL includes statistical aggregate functions for analyzing data distribution.

**SQL Statement:**
```sql
SELECT category,
       ROUND(AVG(quantity), 2) AS avg_quantity,
       ROUND(STDDEV(quantity), 2) AS stddev_quantity,
       ROUND(VARIANCE(quantity), 2) AS variance_quantity
FROM sales
GROUP BY category;
```

**Result:**
| category | avg_quantity | stddev_quantity | variance_quantity |
|----------|--------------|-----------------|-------------------|
| Electronics | 8.83 | 6.65 | 44.17 |
| Furniture | 7.50 | 3.70 | 13.67 |

*Note: Not all databases support these functions. Check your database documentation.*

</details>

