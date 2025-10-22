# SELECT with Aggregate Functions

## Overview

Aggregate functions perform calculations on multiple rows and return a single result. This guide covers aggregate functions, GROUP BY, and HAVING clauses using standard SQL.

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

### COUNT()

```sql
SELECT COUNT(*) AS total_sales
FROM sales;
```

### SUM()

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
| Electronics | 53 | 13292.32 |
| Furniture | 30 | 7999.78 |

### AVG()

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

###

**Result:**
| region | number_of_sales | revenue | avg_sale_value |
|--------|-----------------|---------|----------------|
| North | 3 | 7509.93 | 2503.31 |
| East | 3 | 7799.77 | 2599.92 |
| West | 2 | 2632.50 | 1316.25 |
| South | 2 | 3349.90 | 1674.95 |

###

**Result:**
| product_name | times_sold | total_quantity | total_revenue |
|--------------|------------|----------------|---------------|
| Laptop | 3 | 10 | 9999.90 |
| Desk | 2 | 8 | 3600.00 |
| Monitor | 1 | 8 | 2399.92 |
| Chair | 2 | 22 | 4399.78 |
| Mouse | 2 | 35 | 892.50 |

###

**Result:**
| category | region | sales_count | revenue |
|----------|--------|-------------|---------|
| Electronics | North | 3 | 7509.93 |
| Electronics | East | 2 | 5399.92 |
| Electronics | West | 1 | 382.50 |
| Electronics | South | 0 | 0.00 |
| Furniture | East | 1 | 2399.88 |
| Furniture | South | 2 | 3349.90 |
| Furniture | West | 1 | 2250.00 |
| Furniture | North | 0 | 0.00 |

###

**Result:**
| year | month | sales_count | monthly_revenue |
|------|-------|-------------|-----------------|
| 2024 | 10 | 10 | 21292.10 |

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
| Electronics | 6 | 13292.32 |

###

**Result:**
| region | number_of_sales | revenue |
|--------|-----------------|---------|
| North | 3 | 5509.93 |
| East | 3 | 5399.85 |
| South | 2 | 3356.80 |

###

**Result:**
| category | sales_count | avg_quantity |
|----------|-------------|--------------|
| Electronics | 2 | 11.50 |
| Furniture | 2 | 11.00 |

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

