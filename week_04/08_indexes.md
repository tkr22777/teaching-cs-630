# Indexes using standard SQL

## Overview

Indexes are database objects that improve query performance by allowing faster data retrieval. Think of an index like a book's index - it helps you find information quickly without reading every page.

## How Indexes Work

**Without Index:**
- Database performs a "sequential scan" - reads every row
- Slow for large tables
- Time complexity: O(n)

**With Index:**
- Database uses index to find data quickly
- Much faster for large tables
- Time complexity: O(log n) for B-tree indexes

## Sample Data

<details>
<summary>Click to expand: Database setup script</summary>

```sql
CREATE TABLE products (
    product_id INTEGER PRIMARY KEY,  -- Automatically indexed
    product_name VARCHAR2(200),
    category VARCHAR2(50),
    price NUMBER(10, 2),
    stock_quantity INTEGER,
    supplier_id INTEGER,
    created_at TIMESTAMP DEFAULT SYSTIMESTAMP
);

-- Generate sample data using Oracle's CONNECT BY
INSERT INTO products (product_id, product_name, category, price, stock_quantity, supplier_id) 
SELECT 
    LEVEL,
    'Product ' || LEVEL,
    CASE MOD(LEVEL, 4)
        WHEN 0 THEN 'Electronics'
        WHEN 1 THEN 'Clothing'
        WHEN 2 THEN 'Books'
        ELSE 'Home & Garden'
    END,
    ROUND(DBMS_RANDOM.VALUE(1, 1000), 2),
    FLOOR(DBMS_RANDOM.VALUE(1, 100)),
    FLOOR(DBMS_RANDOM.VALUE(1, 11))
FROM dual
CONNECT BY LEVEL <= 10000;
```

</details>

**Products Table** (10,000 rows):
| product_id | product_name | category | price | stock_quantity | supplier_id |
|------------|--------------|----------|-------|----------------|-------------|
| 1 | Product 1 | Clothing | 245.67 | 45 | 3 |
| 2 | Product 2 | Books | 89.23 | 78 | 7 |
| ... | ... | ... | ... | ... | ... |
| 10000 | Product 10000 | Home & Garden | 567.89 | 23 | 5 |

## Types of Indexes (brief)

- B-Tree (default and most common)
- Unique indexes enforce uniqueness
- Composite indexes cover multiple columns

## Creating Indexes

### Example 1: Single Column Index

**SQL Statement:**
```sql
CREATE INDEX idx_products_category ON products (category);
```

**Verification:**
```sql
-- Oracle: Query index information from USER_INDEXES
SELECT index_name, index_type, uniqueness
FROM user_indexes
WHERE table_name = 'PRODUCTS';
```

**Result:**
| indexname | indexdef |
|-----------|----------|
| products_pkey | CREATE UNIQUE INDEX products_pkey ON products USING btree (product_id) |
| idx_products_category | CREATE INDEX idx_products_category ON products USING btree (category) |

### Example 2: Multi-Column (Composite) Index

**SQL Statement:**
```sql
CREATE INDEX idx_products_category_price 
ON products (category, price);
```

**Use Case:** Efficient for queries filtering by category AND price:
```sql
-- This query benefits from the composite index
SELECT product_name, price
FROM products
WHERE category = 'Electronics' AND price < 500;
```

**Performance Benefit:**
| Query Type | Without Index | With Index |
|------------|---------------|------------|
| Sequential Scan | ~50ms | ~2ms |

### Example 3: Unique Index

**SQL Statement:**
```sql
-- Ensure email addresses are unique
ALTER TABLE customers ADD email VARCHAR2(100);

CREATE UNIQUE INDEX idx_customers_email_unique 
ON customers (email);
```

**Effect:** PostgreSQL will reject any INSERT or UPDATE that creates duplicate emails.

**Example:**
```sql
INSERT INTO customers (email) VALUES ('john@example.com');  -- Success
INSERT INTO customers (email) VALUES ('john@example.com');  -- Error: duplicate key
```

### Example 4: Partial Index

Index only a subset of rows:

**SQL Statement:**
```sql
-- Index only active products
CREATE INDEX idx_products_active 
ON products (product_name) 
WHERE stock_quantity > 0;
```

**Benefit:** Smaller index, faster queries for active products:
```sql
SELECT product_name, stock_quantity
FROM products
WHERE stock_quantity > 0 AND product_name LIKE 'Product 1%';
```

### Example 5: Expression Index

Index based on an expression:

**SQL Statement:**
```sql
-- Index for case-insensitive searches
CREATE INDEX idx_products_name_lower 
ON products (LOWER(product_name));
```

**Use Case:**
```sql
-- This query uses the expression index
SELECT product_name, price
FROM products
WHERE LOWER(product_name) = 'product 100';
```

## Index Performance Impact

### Example 6: Measuring Query Performance (conceptual)

**Without Index:**
```sql
-- Drop the index if it exists
DROP INDEX IF EXISTS idx_products_supplier;

-- Explain query
SELECT product_name, price
FROM products
WHERE supplier_id = 5;
```

**Result:**
```
Seq Scan on products  (cost=0.00..180.00 rows=1000 width=...) (actual time=0.05..15.2 rows=1023 loops=1)
  Filter: (supplier_id = 5)
Planning Time: 0.1 ms
Execution Time: 15.5 ms
```

**With Index:**
```sql
-- Create index
CREATE INDEX idx_products_supplier ON products (supplier_id);

-- Same query
SELECT product_name, price
FROM products
WHERE supplier_id = 5;
```

**Result:**
```
Index Scan using idx_products_supplier on products (cost=0.29..45.2 rows=1000 width=...) (actual time=0.01..1.8 rows=1023 loops=1)
  Index Cond: (supplier_id = 5)
Planning Time: 0.2 ms
Execution Time: 2.1 ms
```

**Performance Improvement:** ~7x faster!

### Example 7: Index Usage Statistics

**SQL Statement:**
```sql
-- Oracle: Monitor index usage statistics
-- Note: Requires ALTER INDEX index_name MONITORING USAGE to be enabled first
SELECT index_name,
       table_name,
       monitoring,
       used,
       start_monitoring
FROM v$object_usage
WHERE table_name = 'PRODUCTS'
ORDER BY index_name;
```

**Result:**
| schemaname | tablename | indexname | number_of_scans | tuples_read | tuples_fetched |
|------------|-----------|-----------|-----------------|-------------|----------------|
| public | products | idx_products_category | 156 | 2340 | 2340 |
| public | products | idx_products_supplier | 89 | 1245 | 1245 |
| public | products | products_pkey | 45 | 45 | 45 |

## When to Create Indexes

### ✅ Create Indexes For:

1. **Columns used in WHERE clauses**
```sql
CREATE INDEX idx_products_price ON products (price);

-- Benefits this query:
SELECT * FROM products WHERE price > 100;
```

2. **Foreign key columns**
```sql
CREATE INDEX idx_orders_customer_id ON orders (customer_id);

-- Benefits joins:
SELECT * FROM orders o
JOIN customers c ON o.customer_id = c.customer_id;
```

3. **Columns used in ORDER BY**
```sql
CREATE INDEX idx_products_created_at ON products (created_at);

-- Benefits this query:
SELECT * FROM products ORDER BY created_at DESC LIMIT 10;
```

4. **Columns with high selectivity**
```sql
-- Good: email (unique values)
CREATE INDEX idx_users_email ON users (email);

-- Less beneficial: gender (only 2-3 values)
-- CREATE INDEX idx_users_gender ON users (gender);  -- Probably not worth it
```

### ❌ Avoid Indexes For:

1. **Small tables** (< 1000 rows)
   - Sequential scan is already fast

2. **Columns frequently updated**
   - Index maintenance overhead

3. **Low selectivity columns**
   - Boolean fields
   - Status fields with few values

4. **Tables with frequent INSERTs**
   - Each INSERT updates all indexes

## Index Maintenance

### Example 8: Viewing All Indexes

**SQL Statement:**
```sql
-- Oracle: List all indexes in current schema
SELECT table_name,
       index_name,
       index_type,
       uniqueness
FROM user_indexes
ORDER BY table_name, index_name;
```

### Example 9: Checking Index Size

**SQL Statement:**
```sql
-- Oracle: Query index sizes from USER_SEGMENTS
SELECT segment_name AS index_name,
       ROUND(bytes / 1024 / 1024, 2) AS size_mb
FROM user_segments
WHERE segment_type = 'INDEX'
  AND segment_name LIKE '%PRODUCTS%'
ORDER BY bytes DESC;
```

**Result:**
| index_name | size_mb |
|-----------|---------|
| PRODUCTS_PK | 0.21 |
| IDX_PRODUCTS_CATEGORY | 0.18 |
| IDX_PRODUCTS_SUPPLIER | 0.18 |

### Example 10: Monitoring Index Usage

**SQL Statement:**
```sql
-- Oracle: Monitor index usage via V$OBJECT_USAGE (requires monitoring to be enabled)
-- First enable monitoring: ALTER INDEX index_name MONITORING USAGE;
SELECT index_name,
       table_name,
       monitoring,
       used,
       start_monitoring,
       end_monitoring
FROM v$object_usage
WHERE table_name = 'PRODUCTS';
```

**Use Case:** Identify indexes that are not being used and can potentially be dropped.

## Dropping Indexes

### Example 11: Drop Single Index

**SQL Statement:**
```sql
DROP INDEX idx_products_category;
```

### Example 12: Drop Index If Exists

**SQL Statement:**
```sql
DROP INDEX IF EXISTS idx_products_old_index;
```

### Example 13: Drop Multiple Indexes

**SQL Statement:**
```sql
DROP INDEX IF EXISTS idx_products_old1, 
                     idx_products_old2,
                     idx_products_old3;
```

<details>
<summary>Advanced: Concurrent Operations and Reindexing</summary>

## Concurrent Index Operations (PostgreSQL primary)

PostgreSQL supports creating/dropping indexes without blocking writes.

```sql
-- Doesn't lock the table
CREATE INDEX CONCURRENTLY idx_products_name ON products (product_name);
DROP INDEX CONCURRENTLY idx_products_old;
```

## Reindexing

Over time, indexes can become fragmented. Reindexing rebuilds them for better performance.

```sql
REINDEX INDEX idx_products_category;
REINDEX TABLE products;
REINDEX INDEX CONCURRENTLY idx_products_category;  -- PostgreSQL 12+
```

MySQL note: emulate concurrent behavior via online DDL (depends on storage engine and version).

</details>

##

