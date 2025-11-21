# Indexes using standard SQL

## Overview

Indexes are database objects that improve query performance by allowing faster data retrieval. Think of an index like a book's index - it helps you find information quickly without reading every page.

## How Indexes Work

**Without Index:**
- Database performs a **sequential scan** - reads every row one by one
- Slow for large tables (reads all 10,000 rows)

**With Index:**
- Database uses index to find data quickly
- Much faster for large tables (reads about 10-20 rows using **B-tree indexes**, the default index type)

## Sample Data

For index examples, imagine we have a large products table with 10,000 rows containing product information like names, categories, prices, and supplier IDs. Indexes help speed up queries on tables this size.

---

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

This creates an index on the category column, making queries that filter by category much faster.

### Example 2: Multi-Column (Composite) Index

**SQL Statement:**
```sql
CREATE INDEX idx_products_category_price 
ON products (category, price);
```

**Use Case:**
```sql
-- This query benefits from the composite index
SELECT product_name, price
FROM products
WHERE category = 'Electronics' AND price < 500;
```

The database can use the index to quickly find rows matching both conditions.

---

### Example 3: Unique Index

**SQL Statement:**
```sql
CREATE UNIQUE INDEX idx_users_email_unique 
ON users (email);
```

**Effect:** The database will reject any INSERT or UPDATE that creates duplicate email values, ensuring data integrity while also speeding up lookups.

## Index Performance Impact

Creating an index on frequently queried columns (like `supplier_id`, `category`, or columns in WHERE clauses) can dramatically improve query performance - often 5-10x faster for large tables. However, indexes add overhead to INSERT, UPDATE, and DELETE operations.

---

## When to Create Indexes

**Create indexes on:**
- Columns used in WHERE clauses
- Foreign key columns (for joins)
- Columns used in ORDER BY
- Columns with many unique values (**high selectivity**)

**Avoid indexes on:**
- Small tables (< 1000 rows)
- Columns frequently updated
- Columns with few unique values (low selectivity like boolean fields)

---

## Index Maintenance

### Dropping Indexes

When an index is no longer needed or not being used, you can remove it to save storage space and reduce INSERT/UPDATE overhead.

**SQL Statement:**
```sql
DROP INDEX idx_products_category;
```

