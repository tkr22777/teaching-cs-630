# SELECT with Aggregate Functions

## Overview

Aggregate functions perform calculations on multiple rows and return a single result. This guide covers aggregate functions, GROUP BY, and HAVING clauses using standard SQL.

## Sample Data

<details>
<summary>Click to expand: Database setup script</summary>

```sql
CREATE TABLE sales (
    sale_id INTEGER PRIMARY KEY,
    product_name VARCHAR(100),
    category VARCHAR(50),
    quantity INTEGER,
    unit_price NUMERIC(10, 2),
    sale_date DATE,
    region VARCHAR(50)
);

INSERT INTO sales (product_name, category, quantity, unit_price, sale_date, region) VALUES
('Laptop', 'Electronics', 5, 999.99, '2024-10-01', 'North'),
('Mouse', 'Electronics', 20, 25.50, '2024-10-01', 'North'),
('Desk', 'Furniture', 3, 450.00, '2024-10-02', 'South'),
('Chair', 'Furniture', 10, 199.99, '2024-10-02', 'South'),
('Laptop', 'Electronics', 3, 999.99, '2024-10-03', 'East'),
('Monitor', 'Electronics', 8, 299.99, '2024-10-03', 'East'),
('Desk', 'Furniture', 5, 450.00, '2024-10-04', 'West'),
('Laptop', 'Electronics', 2, 999.99, '2024-10-05', 'North'),
('Mouse', 'Electronics', 15, 25.50, '2024-10-05', 'West'),
('Chair', 'Furniture', 12, 199.99, '2024-10-06', 'East');
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

### COUNT()

Counts the number of rows.

**Example 1: Count All Rows**
```sql
SELECT COUNT(*) AS total_sales
FROM sales;
```

**Result:**
| total_sales |
|-------------|
| 10 |

**Example 2: Count Non-NULL Values**
```sql
SELECT COUNT(product_name) AS products_sold,
       COUNT(DISTINCT product_name) AS unique_products
FROM sales;
```

**Result:**
| products_sold | unique_products |
|---------------|-----------------|
| 10 | 5 |

**Example 3: Count with WHERE**
```sql
SELECT COUNT(*) AS electronics_sales
FROM sales
WHERE category = 'Electronics';
```

**Result:**
| electronics_sales |
|-------------------|
| 6 |

### SUM()

Calculates the total sum of a numeric column.

**Example 4: Total Quantity Sold**
```sql
SELECT SUM(quantity) AS total_quantity
FROM sales;
```

**Result:**
| total_quantity |
|----------------|
| 83 |

**Example 5: Total Revenue**
```sql
SELECT SUM(quantity * unit_price) AS total_revenue
FROM sales;
```

**Result:**
| total_revenue |
|---------------|
| 14899.11 |

**Example 6: Sum by Category**
```sql
SELECT category,
       SUM(quantity) AS total_quantity,
       SUM(quantity * unit_price) AS total_revenue
FROM sales
GROUP BY category;
```

**Result:**
| category | total_quantity | total_revenue |
|----------|----------------|---------------|
| Electronics | 53 | 12299.18 |
| Furniture | 30 | 6599.93 |

### AVG()

Calculates the average value.

**Example 7: Average Quantity**
```sql
SELECT AVG(quantity) AS avg_quantity,
       AVG(unit_price) AS avg_price
FROM sales;
```

**Result:**
| avg_quantity | avg_price |
|--------------|-----------|
| 8.30 | 385.09 |

**Example 8: Average by Category**
```sql
SELECT category,
       AVG(quantity) AS avg_quantity,
       ROUND(AVG(unit_price), 2) AS avg_price
FROM sales
GROUP BY category;
```

**Result:**
| category | avg_quantity | avg_price |
|----------|--------------|-----------|
| Electronics | 8.83 | 558.49 |
| Furniture | 7.50 | 325.00 |

### MAX() and MIN()

Find maximum and minimum values.

**Example 9: Price Range**
```sql
SELECT MIN(unit_price) AS lowest_price,
       MAX(unit_price) AS highest_price,
       MAX(unit_price) - MIN(unit_price) AS price_range
FROM sales;
```

**Result:**
| lowest_price | highest_price | price_range |
|--------------|---------------|-------------|
| 25.50 | 999.99 | 974.49 |

**Example 10: Date Range**
```sql
SELECT MIN(sale_date) AS first_sale,
       MAX(sale_date) AS last_sale
FROM sales;
```

**Result:**
| first_sale | last_sale |
|------------|-----------|
| 2024-10-01 | 2024-10-06 |

## GROUP BY Clause

GROUP BY groups rows that have the same values in specified columns.

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

### Example 12: Sales by Region

**SQL Statement:**
```sql
SELECT region,
       COUNT(*) AS number_of_sales,
       SUM(quantity * unit_price) AS revenue,
       ROUND(AVG(quantity * unit_price), 2) AS avg_sale_value
FROM sales
GROUP BY region
ORDER BY revenue DESC;
```

**Result:**
| region | number_of_sales | revenue | avg_sale_value |
|--------|-----------------|---------|----------------|
| North | 3 | 5509.93 | 1836.64 |
| East | 3 | 5399.85 | 1799.95 |
| West | 2 | 2632.50 | 1316.25 |
| South | 2 | 3356.80 | 1678.40 |

### Example 13: Sales by Product

**SQL Statement:**
```sql
SELECT product_name,
       COUNT(*) AS times_sold,
       SUM(quantity) AS total_quantity,
       SUM(quantity * unit_price) AS total_revenue
FROM sales
GROUP BY product_name
ORDER BY total_revenue DESC;
```

**Result:**
| product_name | times_sold | total_quantity | total_revenue |
|--------------|------------|----------------|---------------|
| Laptop | 3 | 10 | 9999.90 |
| Desk | 2 | 8 | 3600.00 |
| Monitor | 1 | 8 | 2399.92 |
| Chair | 2 | 22 | 4399.78 |
| Mouse | 2 | 35 | 892.50 |

### Example 14: Multiple Grouping Columns

**SQL Statement:**
```sql
SELECT category, 
       region,
       COUNT(*) AS sales_count,
       SUM(quantity * unit_price) AS revenue
FROM sales
GROUP BY category, region
ORDER BY category, revenue DESC;
```

**Result:**
| category | region | sales_count | revenue |
|----------|--------|-------------|---------|
| Electronics | North | 2 | 5509.93 |
| Electronics | East | 2 | 5399.85 |
| Electronics | West | 1 | 382.50 |
| Electronics | South | 0 | 0.00 |
| Furniture | East | 1 | 2399.88 |
| Furniture | South | 1 | 2349.90 |
| Furniture | West | 1 | 2250.00 |
| Furniture | North | 0 | 0.00 |

### Example 15: Group by Date Components

**SQL Statement:**
```sql
SELECT EXTRACT(YEAR FROM sale_date) AS year,
       EXTRACT(MONTH FROM sale_date) AS month,
       COUNT(*) AS sales_count,
       SUM(quantity * unit_price) AS monthly_revenue
FROM sales
GROUP BY EXTRACT(YEAR FROM sale_date), EXTRACT(MONTH FROM sale_date)
ORDER BY year, month;
```

**Result:**
| year | month | sales_count | monthly_revenue |
|------|-------|-------------|-----------------|
| 2024 | 10 | 10 | 14899.11 |

## HAVING Clause

HAVING filters groups after GROUP BY (WHERE filters before grouping).

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
| Electronics | 6 | 12299.18 |

### Example 17: HAVING with Multiple Conditions

**SQL Statement:**
```sql
SELECT region,
       COUNT(*) AS number_of_sales,
       SUM(quantity * unit_price) AS revenue
FROM sales
GROUP BY region
HAVING COUNT(*) >= 2 
   AND SUM(quantity * unit_price) > 3000
ORDER BY revenue DESC;
```

**Result:**
| region | number_of_sales | revenue |
|--------|-----------------|---------|
| North | 3 | 5509.93 |
| East | 3 | 5399.85 |
| South | 2 | 3356.80 |

### Example 18: WHERE vs HAVING

**SQL Statement:**
```sql
-- Filter products first (WHERE), then group (HAVING)
SELECT category,
       COUNT(*) AS sales_count,
       AVG(quantity) AS avg_quantity
FROM sales
WHERE quantity > 5  -- Filter rows before grouping
GROUP BY category
HAVING COUNT(*) >= 2  -- Filter groups after grouping
ORDER BY avg_quantity DESC;
```

**Result:**
| category | sales_count | avg_quantity |
|----------|-------------|--------------|
| Electronics | 2 | 11.50 |
| Furniture | 2 | 11.00 |

## Complex Aggregate Queries

### Example 19: Aggregates with Calculations

**SQL Statement:**
```sql
SELECT category,
       COUNT(*) AS sales_count,
       SUM(quantity) AS total_units,
       SUM(quantity * unit_price) AS total_revenue,
       ROUND(AVG(quantity * unit_price), 2) AS avg_sale_value,
       ROUND(SUM(quantity * unit_price) / SUM(quantity), 2) AS revenue_per_unit
FROM sales
GROUP BY category
ORDER BY total_revenue DESC;
```

**Result:**
| category | sales_count | total_units | total_revenue | avg_sale_value | revenue_per_unit |
|----------|-------------|-------------|---------------|----------------|------------------|
| Electronics | 6 | 53 | 12299.18 | 2049.86 | 232.06 |
| Furniture | 4 | 30 | 6599.93 | 1649.98 | 220.00 |

### Example 20: Conditional Aggregation

**SQL Statement:**
```sql
SELECT category,
       COUNT(*) AS total_sales,
       COUNT(CASE WHEN quantity > 10 THEN 1 END) AS large_sales,
       COUNT(CASE WHEN quantity <= 10 THEN 1 END) AS small_sales,
       SUM(CASE WHEN region = 'North' THEN quantity * unit_price ELSE 0 END) AS north_revenue,
       SUM(CASE WHEN region != 'North' THEN quantity * unit_price ELSE 0 END) AS other_revenue
FROM sales
GROUP BY category;
```

**Result:**
| category | total_sales | large_sales | small_sales | north_revenue | other_revenue |
|----------|-------------|-------------|-------------|---------------|---------------|
| Electronics | 6 | 2 | 4 | 5509.93 | 6789.25 |
| Furniture | 4 | 1 | 3 | 0.00 | 6599.93 |

### Example 21: Top N Per Group

**SQL Statement:**
```sql
-- Find top 2 products by revenue in each category
WITH ranked_products AS (
    SELECT category,
           product_name,
           SUM(quantity * unit_price) AS revenue,
           ROW_NUMBER() OVER (PARTITION BY category ORDER BY SUM(quantity * unit_price) DESC) AS rank
    FROM sales
    GROUP BY category, product_name
)
SELECT category, product_name, revenue
FROM ranked_products
WHERE rank <= 2
ORDER BY category, rank;
```

**Result:**
| category | product_name | revenue |
|----------|--------------|---------|
| Electronics | Laptop | 9999.90 |
| Electronics | Monitor | 2399.92 |
| Furniture | Chair | 4399.78 |
| Furniture | Desk | 3600.00 |

## Statistical Aggregates (PostgreSQL)

### Example 22: Standard Deviation and Variance

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

