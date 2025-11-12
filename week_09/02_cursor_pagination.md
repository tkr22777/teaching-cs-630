# Cursor-Based Pagination

## Overview

**Pagination** divides large result sets into smaller pages for better performance and user experience. While OFFSET-based pagination is simple, it becomes extremely slow with large datasets. **Cursor-based pagination** (also called keyset pagination) provides consistent performance regardless of dataset size.

## Key Terms

**Pagination**: Dividing query results into discrete pages.

**OFFSET Pagination**: Uses OFFSET and LIMIT/FETCH to skip rows (e.g., `OFFSET 1000 LIMIT 10`).

**Cursor Pagination**: Uses indexed columns (cursor keys) to find the next set of results.

**Keyset**: The values used to identify where to continue pagination (e.g., last seen ID, timestamp).

**Seek Method**: Finding rows based on WHERE conditions rather than skipping rows.

**Index Scan**: Efficient lookup using database indexes.

**Table Scan**: Slower operation scanning all rows sequentially.

## Sample Database Schema

This module uses the e-commerce system.

**Note:** The complete database setup script is available in `00_initialization.md` in this directory.

## The OFFSET Problem

### How OFFSET Works

```sql
-- Page 1: First 5 products
SELECT product_id, product_name, price
FROM products
ORDER BY product_id
OFFSET 0 ROWS FETCH NEXT 5 ROWS ONLY;

-- Page 2: Next 5 products
SELECT product_id, product_name, price
FROM products
ORDER BY product_id
OFFSET 5 ROWS FETCH NEXT 5 ROWS ONLY;

-- Page 100: Products 495-500
SELECT product_id, product_name, price
FROM products
ORDER BY product_id
OFFSET 495 ROWS FETCH NEXT 5 ROWS ONLY;
```

**Page 1 Output:**
```
PRODUCT_ID | PRODUCT_NAME        | PRICE
-----------|---------------------|-------
2001       | Laptop Pro 15       | 1299.99
2002       | Wireless Mouse      | 29.99
2003       | USB-C Cable         | 12.99
2004       | Office Chair        | 249.99
2005       | Standing Desk       | 399.99
```

**Page 2 Output:**
```
PRODUCT_ID | PRODUCT_NAME           | PRICE
-----------|------------------------|-------
2006       | Monitor 27"            | 349.99
2007       | Keyboard Mechanical    | 89.99
2008       | Desk Lamp              | 39.99
2009       | Webcam HD              | 79.99
2010       | Headphones Wireless    | 159.99
```

### The Performance Problem

**With OFFSET, the database must:**
1. Scan and fetch all rows from 0 to OFFSET + LIMIT
2. Discard the first OFFSET rows
3. Return only the LIMIT rows

**Example:** For page 100 with OFFSET 495:
- Database scans rows 1-500
- Discards rows 1-495
- Returns rows 496-500

**Performance Impact:**
- Page 1: Scan 5 rows ✓ Fast
- Page 100: Scan 500 rows ✗ Slow
- Page 10,000: Scan 50,000 rows ✗✗ Very Slow
- Page 100,000: Scan 500,000 rows ✗✗✗ Extremely Slow

## Cursor-Based Pagination (The Solution)

Instead of counting rows to skip, use the last seen value to find the next page.

### Basic Cursor Pagination

**Page 1: Get first 5 products**
```sql
SELECT product_id, product_name, price
FROM products
ORDER BY product_id
FETCH FIRST 5 ROWS ONLY;
```

**Output:**
```
PRODUCT_ID | PRODUCT_NAME        | PRICE
-----------|---------------------|-------
2001       | Laptop Pro 15       | 1299.99
2002       | Wireless Mouse      | 29.99
2003       | USB-C Cable         | 12.99
2004       | Office Chair        | 249.99
2005       | Standing Desk       | 399.99

Last cursor: 2005
```

**Page 2: Get next 5 products after cursor 2005**
```sql
SELECT product_id, product_name, price
FROM products
WHERE product_id > 2005  -- Start after last cursor
ORDER BY product_id
FETCH FIRST 5 ROWS ONLY;
```

**Output:**
```
PRODUCT_ID | PRODUCT_NAME           | PRICE
-----------|------------------------|-------
2006       | Monitor 27"            | 349.99
2007       | Keyboard Mechanical    | 89.99
2008       | Desk Lamp              | 39.99
2009       | Webcam HD              | 79.99
2010       | Headphones Wireless    | 159.99

Last cursor: 2010
```

**Page 3: Get next 5 products after cursor 2010**
```sql
SELECT product_id, product_name, price
FROM products
WHERE product_id > 2010
ORDER BY product_id
FETCH FIRST 5 ROWS ONLY;
```

**Output:**
```
No rows returned (end of results)
```

### Performance Comparison

| Pagination Type | Page 1 | Page 100 | Page 10,000 | Page 100,000 |
|-----------------|--------|----------|-------------|--------------|
| **OFFSET** | Fast | Slow | Very Slow | Extremely Slow |
| **Cursor** | Fast | Fast | Fast | Fast |

**Why Cursor is Faster:**
- Uses indexed WHERE clause: `WHERE product_id > 2010`
- Database uses index to jump directly to the starting row
- Always scans only `LIMIT` rows, not `OFFSET + LIMIT`
- Consistent performance regardless of page number

## Cursor Pagination with Multiple Sort Columns

When sorting by multiple columns, use all of them in the cursor:

### Example: Sort by price, then product_id

**Page 1:**
```sql
SELECT product_id, product_name, price
FROM products
ORDER BY price DESC, product_id
FETCH FIRST 5 ROWS ONLY;
```

**Output:**
```
PRODUCT_ID | PRODUCT_NAME        | PRICE
-----------|---------------------|--------
2001       | Laptop Pro 15       | 1299.99
2005       | Standing Desk       | 399.99
2006       | Monitor 27"         | 349.99
2004       | Office Chair        | 249.99
2010       | Headphones Wireless | 159.99

Last cursor: price=159.99, id=2010
```

**Page 2:**
```sql
SELECT product_id, product_name, price
FROM products
WHERE price < 159.99 OR (price = 159.99 AND product_id > 2010)
ORDER BY price DESC, product_id
FETCH FIRST 5 ROWS ONLY;
```

**Output:**
```
PRODUCT_ID | PRODUCT_NAME        | PRICE
-----------|---------------------|-------
2007       | Keyboard Mechanical | 89.99
2009       | Webcam HD           | 79.99
2008       | Desk Lamp           | 39.99
2002       | Wireless Mouse      | 29.99
2003       | USB-C Cable         | 12.99

Last cursor: price=12.99, id=2003
```

**Explanation:** The WHERE clause handles ties in price by also checking product_id.

## Practical Example: Order History Pagination

Paginating through a customer's order history:

```sql
-- Page 1: Get first 5 orders for customer 1001
SELECT order_id, order_date, total_amount, status
FROM orders
WHERE customer_id = 1001
ORDER BY order_date DESC, order_id DESC
FETCH FIRST 5 ROWS ONLY;
```

**Output:**
```
ORDER_ID | ORDER_DATE | TOTAL_AMOUNT | STATUS
---------|------------|--------------|----------
3002     | 2024-02-20 | 289.98       | Delivered
3001     | 2024-01-15 | 1329.98      | Delivered

Last cursor: order_date=2024-01-15, order_id=3001
```

**Page 2: Get next 5 orders**
```sql
SELECT order_id, order_date, total_amount, status
FROM orders
WHERE customer_id = 1001
  AND (order_date < DATE '2024-01-15' 
       OR (order_date = DATE '2024-01-15' AND order_id < 3001))
ORDER BY order_date DESC, order_id DESC
FETCH FIRST 5 ROWS ONLY;
```

**Output:**
```
No rows returned (customer has only 2 orders)
```

## Bidirectional Pagination

Cursor pagination supports both forward and backward navigation:

### Forward (Next Page)

```sql
SELECT product_id, product_name, price
FROM products
WHERE product_id > :last_cursor
ORDER BY product_id ASC
FETCH FIRST 5 ROWS ONLY;
```

### Backward (Previous Page)

```sql
SELECT product_id, product_name, price
FROM products
WHERE product_id < :first_cursor
ORDER BY product_id DESC  -- Note: DESC for backward
FETCH FIRST 5 ROWS ONLY;
```

Then reverse the results in application code to display in correct order.

## Cursor Pagination with Filters

Cursor pagination works with WHERE filters:

```sql
-- Page 1: Electronics under $100
SELECT product_id, product_name, price, category
FROM products
WHERE category = 'Electronics'
  AND price < 100
ORDER BY product_id
FETCH FIRST 5 ROWS ONLY;
```

**Output:**
```
PRODUCT_ID | PRODUCT_NAME        | PRICE | CATEGORY
-----------|---------------------|-------|------------
2002       | Wireless Mouse      | 29.99 | Electronics
2003       | USB-C Cable         | 12.99 | Electronics
2007       | Keyboard Mechanical | 89.99 | Electronics
2009       | Webcam HD           | 79.99 | Electronics

Last cursor: 2009
```

**Page 2:**
```sql
SELECT product_id, product_name, price, category
FROM products
WHERE category = 'Electronics'
  AND price < 100
  AND product_id > 2009  -- Continue after cursor
ORDER BY product_id
FETCH FIRST 5 ROWS ONLY;
```

## Practical Example: Building an API

Application code pattern for cursor pagination:

```sql
-- Function to get paginated products
CREATE OR REPLACE FUNCTION get_products_page(
    p_cursor IN NUMBER DEFAULT NULL,
    p_page_size IN NUMBER DEFAULT 10
) RETURN SYS_REFCURSOR IS
    v_result SYS_REFCURSOR;
BEGIN
    IF p_cursor IS NULL THEN
        -- First page
        OPEN v_result FOR
            SELECT product_id, product_name, price
            FROM products
            ORDER BY product_id
            FETCH FIRST p_page_size ROWS ONLY;
    ELSE
        -- Subsequent pages
        OPEN v_result FOR
            SELECT product_id, product_name, price
            FROM products
            WHERE product_id > p_cursor
            ORDER BY product_id
            FETCH FIRST p_page_size ROWS ONLY;
    END IF;
    
    RETURN v_result;
END;
/

-- Usage: Get first page
DECLARE
    v_cursor SYS_REFCURSOR;
    v_product_id NUMBER;
    v_product_name VARCHAR2(100);
    v_price NUMBER;
BEGIN
    v_cursor := get_products_page(NULL, 3);
    
    DBMS_OUTPUT.PUT_LINE('=== PAGE 1 ===');
    LOOP
        FETCH v_cursor INTO v_product_id, v_product_name, v_price;
        EXIT WHEN v_cursor%NOTFOUND;
        
        DBMS_OUTPUT.PUT_LINE(v_product_id || ': ' || v_product_name || ' - $' || v_price);
    END LOOP;
    CLOSE v_cursor;
    
    -- Get next page using last cursor (2003)
    v_cursor := get_products_page(2003, 3);
    
    DBMS_OUTPUT.PUT_LINE('=== PAGE 2 ===');
    LOOP
        FETCH v_cursor INTO v_product_id, v_product_name, v_price;
        EXIT WHEN v_cursor%NOTFOUND;
        
        DBMS_OUTPUT.PUT_LINE(v_product_id || ': ' || v_product_name || ' - $' || v_price);
    END LOOP;
    CLOSE v_cursor;
END;
/
```

**Output:**
```
=== PAGE 1 ===
2001: Laptop Pro 15 - $1299.99
2002: Wireless Mouse - $29.99
2003: USB-C Cable - $12.99

=== PAGE 2 ===
2004: Office Chair - $249.99
2005: Standing Desk - $399.99
2006: Monitor 27" - $349.99
```

## When to Use Each Method

### Use OFFSET When:
- Small datasets (< 1,000 rows)
- Random page access is required (e.g., "jump to page 50")
- Simplicity is more important than performance
- Dataset rarely changes during pagination

### Use Cursor Pagination When:
- Large datasets (> 10,000 rows)
- Performance is critical
- Sequential navigation (next/previous only)
- Mobile apps or infinite scroll
- Dataset may change during pagination

## Implementation Checklist

✅ **Create indexes** on cursor columns:
```sql
CREATE INDEX idx_products_id ON products(product_id);
CREATE INDEX idx_orders_cust_date ON orders(customer_id, order_date DESC, order_id DESC);
```

✅ **Use unique, immutable columns** for cursors (IDs, timestamps)

✅ **Handle tie-breaking** when sorting by non-unique columns

✅ **Return cursor info** in API responses:
```json
{
  "data": [...],
  "cursor": "2010",
  "has_more": true
}
```

✅ **Test edge cases**: first page, last page, empty results

## Summary

**Key takeaways:**

1. **OFFSET Problem** - Performance degrades linearly with page number
2. **Cursor Solution** - Consistent performance using indexed WHERE clauses
3. **Basic Pattern** - `WHERE id > last_cursor` for next page
4. **Multi-Column Cursors** - Use all sort columns in WHERE clause
5. **Bidirectional** - Support forward and backward navigation
6. **Indexes Required** - Cursor columns must be indexed for performance
7. **Best for Scale** - Essential for applications handling large datasets

Cursor-based pagination is the industry standard for high-performance APIs and applications dealing with large result sets.

