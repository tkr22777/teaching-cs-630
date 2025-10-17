# Indexes in PostgreSQL

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
    product_id SERIAL PRIMARY KEY,  -- Automatically indexed
    product_name VARCHAR(200),
    category VARCHAR(50),
    price NUMERIC(10, 2),
    stock_quantity INTEGER,
    supplier_id INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO products (product_name, category, price, stock_quantity, supplier_id) 
SELECT 
    'Product ' || generate_series,
    CASE (generate_series % 4)
        WHEN 0 THEN 'Electronics'
        WHEN 1 THEN 'Clothing'
        WHEN 2 THEN 'Books'
        ELSE 'Home & Garden'
    END,
    (random() * 1000)::NUMERIC(10,2),
    (random() * 100)::INTEGER,
    (random() * 10 + 1)::INTEGER
FROM generate_series(1, 10000);
```

</details>

**Products Table** (10,000 rows):
| product_id | product_name | category | price | stock_quantity | supplier_id |
|------------|--------------|----------|-------|----------------|-------------|
| 1 | Product 1 | Clothing | 245.67 | 45 | 3 |
| 2 | Product 2 | Books | 89.23 | 78 | 7 |
| ... | ... | ... | ... | ... | ... |
| 10000 | Product 10000 | Home & Garden | 567.89 | 23 | 5 |

## Types of Indexes in PostgreSQL

### 1. B-Tree Index (Default)

Most common index type, good for:
- Equality comparisons (=)
- Range queries (<, >, <=, >=, BETWEEN)
- Sorting (ORDER BY)
- Pattern matching (LIKE 'text%')

**Syntax:**
```sql
CREATE INDEX index_name ON table_name (column_name);
```

### 2. Hash Index

Good for simple equality comparisons only:
- Equality (=)
- Not good for ranges or sorting

**Syntax:**
```sql
CREATE INDEX index_name ON table_name USING HASH (column_name);
```

### 3. GIN (Generalized Inverted Index)

Good for:
- Full-text search
- Array operations
- JSONB data

### 4. GiST (Generalized Search Tree)

Good for:
- Geometric data
- Full-text search

## Creating Indexes

### Example 1: Single Column Index

**SQL Statement:**
```sql
CREATE INDEX idx_products_category ON products (category);
```

**Verification:**
```sql
SELECT indexname, indexdef
FROM pg_indexes
WHERE tablename = 'products';
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
ALTER TABLE customers ADD COLUMN email VARCHAR(100);

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

### Example 6: Measuring Query Performance

**Without Index:**
```sql
-- Drop the index if it exists
DROP INDEX IF EXISTS idx_products_supplier;

-- Explain query
EXPLAIN ANALYZE
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
EXPLAIN ANALYZE
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
SELECT schemaname,
       tablename,
       indexname,
       idx_scan AS number_of_scans,
       idx_tup_read AS tuples_read,
       idx_tup_fetch AS tuples_fetched
FROM pg_stat_user_indexes
WHERE tablename = 'products'
ORDER BY idx_scan DESC;
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
SELECT tablename,
       indexname,
       indexdef
FROM pg_indexes
WHERE schemaname = 'public'
ORDER BY tablename, indexname;
```

### Example 9: Checking Index Size

**SQL Statement:**
```sql
SELECT indexname,
       pg_size_pretty(pg_relation_size(indexname::regclass)) AS index_size
FROM pg_indexes
WHERE tablename = 'products';
```

**Result:**
| indexname | index_size |
|-----------|------------|
| products_pkey | 216 kB |
| idx_products_category | 184 kB |
| idx_products_supplier | 184 kB |

### Example 10: Finding Unused Indexes

**SQL Statement:**
```sql
SELECT schemaname,
       tablename,
       indexname,
       idx_scan,
       pg_size_pretty(pg_relation_size(indexname::regclass)) AS index_size
FROM pg_stat_user_indexes
WHERE idx_scan = 0
  AND indexname NOT LIKE '%_pkey'  -- Exclude primary keys
ORDER BY pg_relation_size(indexname::regclass) DESC;
```

**Use Case:** Identify indexes that are never used and can be dropped.

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

## Concurrent Index Operations

PostgreSQL allows creating/dropping indexes without blocking writes.

### Example 14: Create Index Concurrently

**SQL Statement:**
```sql
-- Doesn't lock the table
CREATE INDEX CONCURRENTLY idx_products_name ON products (product_name);
```

**Benefit:** Table remains available for writes during index creation.

**Use Case:** Production databases where downtime must be minimized.

### Example 15: Drop Index Concurrently

**SQL Statement:**
```sql
DROP INDEX CONCURRENTLY idx_products_old;
```

## Reindexing

Over time, indexes can become bloated. Reindexing rebuilds them.

### Example 16: Reindex Single Index

**SQL Statement:**
```sql
REINDEX INDEX idx_products_category;
```

### Example 17: Reindex Entire Table

**SQL Statement:**
```sql
REINDEX TABLE products;
```

### Example 18: Reindex Concurrently (PostgreSQL 12+)

**SQL Statement:**
```sql
REINDEX INDEX CONCURRENTLY idx_products_category;
```

## Advanced Index Techniques

### Example 19: Covering Index

Include additional columns in index to avoid table lookups:

**SQL Statement:**
```sql
CREATE INDEX idx_products_category_covering 
ON products (category) 
INCLUDE (product_name, price);
```

**Benefit:** Query can be answered entirely from index:
```sql
-- Faster: all columns in index
SELECT product_name, price
FROM products
WHERE category = 'Electronics';
```

### Example 20: Index with NULL Handling

**SQL Statement:**
```sql
-- Index products with NULL categories
CREATE INDEX idx_products_category_nulls 
ON products (category) 
WHERE category IS NULL;
```

## Best Practices

### 1. Name Indexes Descriptively

```sql
-- Good: Clear naming convention
CREATE INDEX idx_orders_customer_id ON orders (customer_id);
CREATE INDEX idx_products_category_price ON products (category, price);

-- Avoid: Generic names
CREATE INDEX index1 ON orders (customer_id);
```

### 2. Monitor Index Usage

```sql
-- Regularly check which indexes are used
SELECT indexname, idx_scan
FROM pg_stat_user_indexes
WHERE schemaname = 'public'
ORDER BY idx_scan DESC;
```

### 3. Don't Over-Index

```sql
-- Avoid: Too many indexes on same column combinations
CREATE INDEX idx1 ON products (category);
CREATE INDEX idx2 ON products (category, price);  -- idx1 is redundant
```

## Common Mistakes

### Mistake 1: Creating Redundant Indexes

**Problem:**
```sql
CREATE INDEX idx_a ON table (col1);
CREATE INDEX idx_b ON table (col1, col2);  -- idx_a is now redundant
```

**Solution:** Drop idx_a, idx_b can handle queries on just col1.

### Mistake 2: Indexing Low-Cardinality Columns

**Problem:**
```sql
-- Boolean column - only 2 possible values
CREATE INDEX idx_users_active ON users (is_active);  -- Low benefit
```

**Solution:** Only index if queries are very selective or use partial index:
```sql
CREATE INDEX idx_users_active ON users (is_active) WHERE is_active = true;
```

