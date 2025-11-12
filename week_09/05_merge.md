# SQL MERGE Statement

## Overview

The **MERGE** statement (also called "upsert") allows you to INSERT, UPDATE, or DELETE data in a single statement based on whether a matching row exists. It's essential for data synchronization, ETL processes, and handling scenarios where you need to "update if exists, insert if not."

## Key Terms

**MERGE**: SQL statement that combines INSERT, UPDATE, and optionally DELETE operations.

**Upsert**: Portmanteau of "update" and "insert" - update existing rows or insert new ones.

**Target Table**: The table being modified by MERGE.

**Source**: The data source (table, view, or query) providing the new data.

**Match Condition**: ON clause that determines if a row in target matches a row in source.

**WHEN MATCHED**: Clause specifying what to do when a match is found (usually UPDATE or DELETE).

**WHEN NOT MATCHED**: Clause specifying what to do when no match is found (usually INSERT).

**Data Synchronization**: Process of keeping two datasets in sync.

**ETL (Extract, Transform, Load)**: Data integration process often using MERGE.

## Sample Database Schema

This module uses the e-commerce system.

**Note:** The complete database setup script is available in `00_initialization.md` in this directory.

## Why Use MERGE?

### Problem: Update or Insert?

Without MERGE, you need multiple steps:

```sql
-- Approach 1: Try UPDATE, then INSERT if no rows affected
UPDATE products SET stock_quantity = 100 WHERE product_id = 2015;

IF SQL%ROWCOUNT = 0 THEN
    INSERT INTO products (product_id, product_name, category, price, stock_quantity)
    VALUES (2015, 'New Product', 'Electronics', 99.99, 100);
END IF;

-- Approach 2: Check existence first
SELECT COUNT(*) INTO v_exists FROM products WHERE product_id = 2015;

IF v_exists = 0 THEN
    INSERT INTO products ...
ELSE
    UPDATE products ...
END IF;
```

**Problems:**
- Multiple round trips to database
- Race conditions in concurrent environments
- More code to write and maintain

### Solution: MERGE Statement

```sql
MERGE INTO products p
USING (SELECT 2015 AS product_id, 'New Product' AS product_name,
              'Electronics' AS category, 99.99 AS price, 100 AS stock_quantity
       FROM DUAL) src
ON (p.product_id = src.product_id)
WHEN MATCHED THEN
    UPDATE SET p.stock_quantity = src.stock_quantity
WHEN NOT MATCHED THEN
    INSERT (product_id, product_name, category, price, stock_quantity)
    VALUES (src.product_id, src.product_name, src.category, src.price, src.stock_quantity);
```

**Benefits:**
- Single atomic operation
- No race conditions
- Cleaner, more concise code
- Better performance

## Basic MERGE Syntax

```sql
MERGE INTO target_table t
USING source_table s
ON (t.key_column = s.key_column)
WHEN MATCHED THEN
    UPDATE SET t.column1 = s.column1, t.column2 = s.column2
WHEN NOT MATCHED THEN
    INSERT (column1, column2, ...)
    VALUES (s.column1, s.column2, ...);
```

## Simple MERGE Example

Update existing products or insert new ones:

```sql
-- Create a staging table with product updates
CREATE TABLE product_staging (
    product_id INTEGER,
    product_name VARCHAR2(100),
    price NUMBER(10, 2),
    stock_quantity INTEGER
);

-- Insert some test data
INSERT INTO product_staging VALUES (2001, 'Laptop Pro 15', 1199.99, 50);  -- Exists, new price
INSERT INTO product_staging VALUES (2002, 'Wireless Mouse', 29.99, 200);  -- Exists, new stock
INSERT INTO product_staging VALUES (2015, 'Tablet Pro', 599.99, 30);      -- New product

-- MERGE the changes
MERGE INTO products p
USING product_staging ps
ON (p.product_id = ps.product_id)
WHEN MATCHED THEN
    UPDATE SET 
        p.product_name = ps.product_name,
        p.price = ps.price,
        p.stock_quantity = ps.stock_quantity,
        p.last_updated = SYSDATE
WHEN NOT MATCHED THEN
    INSERT (product_id, product_name, category, price, stock_quantity)
    VALUES (ps.product_id, ps.product_name, 'Electronics', ps.price, ps.stock_quantity);

DBMS_OUTPUT.PUT_LINE(SQL%ROWCOUNT || ' rows merged');
```

**Output:**
```
3 rows merged
```

**Result:**
- Product 2001: Price updated from $1299.99 to $1199.99
- Product 2002: Stock updated from 150 to 200
- Product 2015: New product inserted

## MERGE with Inline Source

You don't need a physical table - use an inline query:

```sql
MERGE INTO customers c
USING (
    SELECT 1008 AS customer_id, 'John' AS first_name, 'Doe' AS last_name,
           'john.doe@email.com' AS email, 5000 AS credit_limit
    FROM DUAL
) src
ON (c.customer_id = src.customer_id)
WHEN MATCHED THEN
    UPDATE SET c.credit_limit = src.credit_limit
WHEN NOT MATCHED THEN
    INSERT (customer_id, first_name, last_name, email, credit_limit, status)
    VALUES (src.customer_id, src.first_name, src.last_name, src.email, src.credit_limit, 'Active');
```

## Conditional MERGE

Add WHERE clauses to control when updates/inserts happen:

```sql
MERGE INTO products p
USING product_staging ps
ON (p.product_id = ps.product_id)
WHEN MATCHED THEN
    UPDATE SET 
        p.price = ps.price,
        p.stock_quantity = ps.stock_quantity
    WHERE p.price != ps.price OR p.stock_quantity != ps.stock_quantity  -- Only update if changed
WHEN NOT MATCHED THEN
    INSERT (product_id, product_name, category, price, stock_quantity)
    VALUES (ps.product_id, ps.product_name, 'Electronics', ps.price, ps.stock_quantity)
    WHERE ps.price > 0 AND ps.stock_quantity > 0;  -- Only insert valid data
```

## MERGE with DELETE

Remove matched rows that meet certain conditions:

```sql
MERGE INTO products p
USING product_staging ps
ON (p.product_id = ps.product_id)
WHEN MATCHED THEN
    UPDATE SET p.stock_quantity = ps.stock_quantity
    DELETE WHERE ps.stock_quantity = 0  -- Remove discontinued products
WHEN NOT MATCHED THEN
    INSERT (product_id, product_name, category, price, stock_quantity)
    VALUES (ps.product_id, ps.product_name, 'Electronics', ps.price, ps.stock_quantity);
```

**Note:** DELETE clause must come after UPDATE SET clause.

## Practical Example: Inventory Synchronization

Synchronize inventory from external system:

```sql
-- Create external inventory feed table
CREATE TABLE inventory_feed (
    product_id INTEGER,
    quantity_received INTEGER,
    feed_date DATE
);

-- Simulate receiving inventory data
INSERT INTO inventory_feed VALUES (2001, 10, SYSDATE);
INSERT INTO inventory_feed VALUES (2002, 50, SYSDATE);
INSERT INTO inventory_feed VALUES (2003, 100, SYSDATE);
INSERT INTO inventory_feed VALUES (2016, 25, SYSDATE);  -- New product

-- Merge inventory updates
MERGE INTO products p
USING (
    SELECT 
        if.product_id,
        if.quantity_received,
        if.feed_date,
        'Product-' || if.product_id AS default_name
    FROM inventory_feed if
    WHERE if.feed_date = TRUNC(SYSDATE)
) src
ON (p.product_id = src.product_id)
WHEN MATCHED THEN
    UPDATE SET 
        p.stock_quantity = p.stock_quantity + src.quantity_received,
        p.last_updated = src.feed_date
WHEN NOT MATCHED THEN
    INSERT (product_id, product_name, category, price, stock_quantity, last_updated)
    VALUES (src.product_id, src.default_name, 'Electronics', 0, src.quantity_received, src.feed_date);

-- Verify results
SELECT product_id, product_name, stock_quantity, last_updated
FROM products
WHERE product_id IN (2001, 2002, 2003, 2016)
ORDER BY product_id;
```

**Output:**
```
PRODUCT_ID | PRODUCT_NAME    | STOCK_QUANTITY | LAST_UPDATED
-----------|-----------------|----------------|-------------
2001       | Laptop Pro 15   | 55             | 2024-11-12
2002       | Wireless Mouse  | 200            | 2024-11-12
2003       | USB-C Cable     | 300            | 2024-11-12
2016       | Product-2016    | 25             | 2024-11-12
```

**Explanation:**
- Existing products: Stock increased by quantity_received
- New product (2016): Inserted with received quantity

## MERGE Performance Tips

### 1. Index Match Columns

```sql
-- Create index on join column
CREATE INDEX idx_products_id ON products(product_id);
CREATE INDEX idx_staging_id ON product_staging(product_id);
```

### 2. Minimize Source Rows

```sql
-- Bad: Merge all historical data
MERGE INTO products p
USING all_product_updates u ON ...

-- Good: Merge only recent updates
MERGE INTO products p
USING (SELECT * FROM all_product_updates WHERE update_date = TRUNC(SYSDATE)) u
ON ...
```

### 3. Use APPEND Hint for Large Inserts

```sql
MERGE /*+ APPEND */ INTO products p
USING large_source s ON ...
```

### 4. Batch Large Operations

```sql
-- Instead of merging 1 million rows at once, batch them
FOR batch IN (SELECT DISTINCT batch_id FROM staging) LOOP
    MERGE INTO target
    USING (SELECT * FROM staging WHERE batch_id = batch.batch_id)
    ON ...
    COMMIT;
END LOOP;
```

## Common Patterns

### Pattern 1: Slowly Changing Dimension (Type 1)

Overwrite historical data:

```sql
MERGE INTO dim_customer dc
USING stg_customer sc ON (dc.customer_id = sc.customer_id)
WHEN MATCHED THEN
    UPDATE SET 
        dc.name = sc.name,
        dc.address = sc.address,
        dc.updated_date = SYSDATE
WHEN NOT MATCHED THEN
    INSERT VALUES (sc.customer_id, sc.name, sc.address, SYSDATE);
```

### Pattern 2: Incremental Load

Load only new/changed records:

```sql
MERGE INTO fact_orders fo
USING (
    SELECT * FROM stg_orders
    WHERE load_date = TRUNC(SYSDATE)
) so
ON (fo.order_id = so.order_id)
WHEN MATCHED THEN
    UPDATE SET fo.status = so.status
WHEN NOT MATCHED THEN
    INSERT VALUES (...);
```

### Pattern 3: Deduplication

Keep only the latest version:

```sql
MERGE INTO products_clean pc
USING (
    SELECT product_id, product_name, price,
           ROW_NUMBER() OVER (PARTITION BY product_id ORDER BY last_updated DESC) AS rn
    FROM products_raw
) pr
ON (pc.product_id = pr.product_id)
WHEN MATCHED AND pr.rn = 1 THEN
    UPDATE SET pc.product_name = pr.product_name, pc.price = pr.price
WHEN NOT MATCHED AND pr.rn = 1 THEN
    INSERT VALUES (pr.product_id, pr.product_name, pr.price);
```

## MERGE Restrictions

1. **Cannot update join column:**
```sql
-- ERROR: Cannot update product_id in ON clause
MERGE INTO products p
USING staging s ON (p.product_id = s.product_id)
WHEN MATCHED THEN
    UPDATE SET p.product_id = s.new_product_id;  -- Not allowed!
```

2. **Cannot insert duplicate keys:**
```sql
-- If source has duplicates, MERGE will fail
-- Solution: Deduplicate source first
```

3. **Multiple WHEN MATCHED not allowed:**
```sql
-- ERROR: Only one WHEN MATCHED clause allowed
MERGE INTO products p USING staging s ON (...)
WHEN MATCHED THEN UPDATE SET p.price = s.price
WHEN MATCHED THEN DELETE WHERE s.discontinued = 'Y';  -- Not allowed!

-- Solution: Combine into one WHEN MATCHED with DELETE
WHEN MATCHED THEN
    UPDATE SET p.price = s.price
    DELETE WHERE s.discontinued = 'Y';
```

## Summary

**Key takeaways:**

1. **MERGE** - Combines INSERT, UPDATE, and DELETE in a single atomic operation
2. **Upsert** - Update if exists, insert if not
3. **ON Clause** - Defines the match condition between source and target
4. **WHEN MATCHED** - Handles existing rows (UPDATE or DELETE)
5. **WHEN NOT MATCHED** - Handles new rows (INSERT)
6. **Use Cases** - Data synchronization, ETL processes, inventory management
7. **Performance** - Index match columns and batch large operations

MERGE is essential for data integration scenarios where you need to efficiently synchronize data between systems or maintain data warehouses.

