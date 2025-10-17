# DML: UPDATE and DELETE Statements

## Overview

UPDATE and DELETE statements modify existing data in tables. These are powerful commands that can affect multiple rows, so use them carefully.

## Sample Data

Let's create and populate a sample table:

```sql
CREATE TABLE inventory (
    product_id SERIAL PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    category VARCHAR(50),
    price NUMERIC(10, 2),
    stock_quantity INTEGER DEFAULT 0,
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO inventory (product_name, category, price, stock_quantity) VALUES
('Laptop', 'Electronics', 999.99, 15),
('Mouse', 'Electronics', 25.50, 100),
('Keyboard', 'Electronics', 75.00, 50),
('Monitor', 'Electronics', 299.99, 30),
('Desk Chair', 'Furniture', 199.99, 25),
('Standing Desk', 'Furniture', 450.00, 10),
('Notebook', 'Office Supplies', 5.99, 200),
('Pen Set', 'Office Supplies', 12.99, 150);
```

**Initial Inventory Table:**
| product_id | product_name | category | price | stock_quantity | last_updated |
|------------|--------------|----------|--------|----------------|--------------|
| 1 | Laptop | Electronics | 999.99 | 15 | 2024-10-17 10:00:00 |
| 2 | Mouse | Electronics | 25.50 | 100 | 2024-10-17 10:00:00 |
| 3 | Keyboard | Electronics | 75.00 | 50 | 2024-10-17 10:00:00 |
| 4 | Monitor | Electronics | 299.99 | 30 | 2024-10-17 10:00:00 |
| 5 | Desk Chair | Furniture | 199.99 | 25 | 2024-10-17 10:00:00 |
| 6 | Standing Desk | Furniture | 450.00 | 10 | 2024-10-17 10:00:00 |
| 7 | Notebook | Office Supplies | 5.99 | 200 | 2024-10-17 10:00:00 |
| 8 | Pen Set | Office Supplies | 12.99 | 150 | 2024-10-17 10:00:00 |

## UPDATE Statement

### Basic Syntax

```sql
UPDATE table_name
SET column1 = value1, column2 = value2, ...
WHERE condition;
```

### Example 1: Update Single Row

**SQL Statement:**
```sql
UPDATE inventory
SET price = 899.99
WHERE product_id = 1;
```

**Result:**
| product_id | product_name | category | price | stock_quantity |
|------------|--------------|----------|--------|----------------|
| 1 | Laptop | Electronics | 899.99 | 15 |
| 2 | Mouse | Electronics | 25.50 | 100 |
| ... | ... | ... | ... | ... |

**Verification Query:**
```sql
SELECT product_id, product_name, price 
FROM inventory 
WHERE product_id = 1;
```

### Example 2: Update Multiple Columns

**SQL Statement:**
```sql
UPDATE inventory
SET price = 79.99,
    stock_quantity = 75
WHERE product_name = 'Keyboard';
```

**Result:**
| product_id | product_name | category | price | stock_quantity |
|------------|--------------|----------|--------|----------------|
| 3 | Keyboard | Electronics | 79.99 | 75 |

### Example 3: Update Multiple Rows

**SQL Statement:**
```sql
-- Give 10% discount on all Electronics
UPDATE inventory
SET price = price * 0.90
WHERE category = 'Electronics';
```

**Result:**
| product_id | product_name | category | price | stock_quantity |
|------------|--------------|----------|--------|----------------|
| 1 | Laptop | Electronics | 809.99 | 15 |
| 2 | Mouse | Electronics | 22.95 | 100 |
| 3 | Keyboard | Electronics | 71.99 | 75 |
| 4 | Monitor | Electronics | 269.99 | 30 |
| 5 | Desk Chair | Furniture | 199.99 | 25 |
| ... | ... | ... | ... | ... |

### Example 4: Update with Calculation

**SQL Statement:**
```sql
-- Increase stock by 20 units for low-stock items
UPDATE inventory
SET stock_quantity = stock_quantity + 20
WHERE stock_quantity < 30;
```

**Before:**
| product_id | product_name | stock_quantity |
|------------|--------------|----------------|
| 1 | Laptop | 15 |
| 6 | Standing Desk | 10 |

**After:**
| product_id | product_name | stock_quantity |
|------------|--------------|----------------|
| 1 | Laptop | 35 |
| 6 | Standing Desk | 30 |

### Example 5: Update with RETURNING

**SQL Statement:**
```sql
UPDATE inventory
SET price = 299.99,
    stock_quantity = stock_quantity - 5
WHERE product_name = 'Monitor'
RETURNING product_id, product_name, price, stock_quantity;
```

**Query Result:**
| product_id | product_name | price | stock_quantity |
|------------|--------------|--------|----------------|
| 4 | Monitor | 299.99 | 25 |

### Example 6: Update Using FROM (PostgreSQL Specific)

Setup additional table:
```sql
CREATE TABLE price_adjustments (
    category VARCHAR(50),
    adjustment_factor NUMERIC(3, 2)
);

INSERT INTO price_adjustments VALUES
('Electronics', 1.10),  -- 10% increase
('Furniture', 0.95),    -- 5% decrease
('Office Supplies', 1.05);  -- 5% increase
```

**SQL Statement:**
```sql
UPDATE inventory
SET price = inventory.price * pa.adjustment_factor
FROM price_adjustments pa
WHERE inventory.category = pa.category;
```

**Result (Price Changes):**
| product_id | product_name | category | old_price | new_price |
|------------|--------------|----------|-----------|-----------|
| 1 | Laptop | Electronics | 809.99 | 890.99 |
| 2 | Mouse | Electronics | 22.95 | 25.25 |
| 5 | Desk Chair | Furniture | 199.99 | 189.99 |
| ... | ... | ... | ... | ... |

### Example 7: Conditional Update with CASE

**SQL Statement:**
```sql
UPDATE inventory
SET price = CASE 
    WHEN stock_quantity > 100 THEN price * 0.95  -- 5% discount for high stock
    WHEN stock_quantity < 20 THEN price * 1.10   -- 10% markup for low stock
    ELSE price  -- No change
END;
```

**Result:**
| product_id | product_name | stock_quantity | price_change |
|------------|--------------|----------------|--------------|
| 2 | Mouse | 100 | Discounted 5% |
| 7 | Notebook | 200 | Discounted 5% |
| 1 | Laptop | 35 | No change |
| 6 | Standing Desk | 30 | Markup 10% |

## DELETE Statement

### Basic Syntax

```sql
DELETE FROM table_name
WHERE condition;
```

### Example 8: Delete Single Row

**SQL Statement:**
```sql
DELETE FROM inventory
WHERE product_id = 8;
```

**Before:**
| product_id | product_name | category |
|------------|--------------|----------|
| 7 | Notebook | Office Supplies |
| 8 | Pen Set | Office Supplies |

**After:**
| product_id | product_name | category |
|------------|--------------|----------|
| 7 | Notebook | Office Supplies |

**Verification:**
```sql
SELECT COUNT(*) FROM inventory WHERE product_id = 8;
```

**Result:**
| count |
|-------|
| 0 |

### Example 9: Delete Multiple Rows

**SQL Statement:**
```sql
-- Remove all out-of-stock items
DELETE FROM inventory
WHERE stock_quantity = 0;
```

### Example 10: Delete with Multiple Conditions

**SQL Statement:**
```sql
-- Remove expensive furniture items
DELETE FROM inventory
WHERE category = 'Furniture' 
  AND price > 400;
```

**Before:**
| product_id | product_name | category | price |
|------------|--------------|----------|--------|
| 5 | Desk Chair | Furniture | 189.99 |
| 6 | Standing Desk | Furniture | 495.00 |

**After:**
| product_id | product_name | category | price |
|------------|--------------|----------|--------|
| 5 | Desk Chair | Furniture | 189.99 |

### Example 11: DELETE with RETURNING

**SQL Statement:**
```sql
DELETE FROM inventory
WHERE price < 10
RETURNING product_id, product_name, price;
```

**Query Result (Deleted Rows):**
| product_id | product_name | price |
|------------|--------------|-------|
| 7 | Notebook | 5.99 |

### Example 12: DELETE with Subquery

Setup orders table:
```sql
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    product_id INTEGER,
    quantity INTEGER,
    order_date DATE
);

INSERT INTO orders (product_id, quantity, order_date) VALUES
(1, 2, '2024-10-15'),
(2, 10, '2024-10-16');
```

**SQL Statement:**
```sql
-- Delete products that have never been ordered
DELETE FROM inventory
WHERE product_id NOT IN (
    SELECT DISTINCT product_id FROM orders
);
```

**Before DELETE:**
| product_id | product_name | has_orders |
|------------|--------------|------------|
| 1 | Laptop | Yes |
| 2 | Mouse | Yes |
| 3 | Keyboard | No |
| 4 | Monitor | No |

**After DELETE:**
| product_id | product_name |
|------------|--------------|
| 1 | Laptop |
| 2 | Mouse |

## UPDATE vs DELETE: Important Differences

| Aspect | UPDATE | DELETE |
|--------|--------|--------|
| Purpose | Modifies existing data | Removes rows |
| Columns | Can update specific columns | Removes entire row |
| WHERE clause | Optional (but recommended) | Optional (but recommended) |
| RETURNING | Supported | Supported |
| Rollback | Yes (within transaction) | Yes (within transaction) |

## Common Errors and Solutions

### Error 1: Forgetting WHERE Clause

**Problem:**
```sql
-- DANGER: Updates ALL rows!
UPDATE inventory
SET price = 0;
```

**Solution:**
```sql
-- Always use WHERE for specific updates
UPDATE inventory
SET price = 0
WHERE product_id = 999;
```

### Error 2: NULL Constraint Violation

**Problem:**
```sql
UPDATE inventory
SET product_name = NULL
WHERE product_id = 1;
-- ERROR: null value in column "product_name" violates not-null constraint
```

**Solution:**
```sql
UPDATE inventory
SET product_name = 'Unknown Product'
WHERE product_id = 1;
```

### Error 3: Check Constraint Violation

Assuming we have:
```sql
ALTER TABLE inventory
ADD CONSTRAINT check_positive_price CHECK (price > 0);
```

**Problem:**
```sql
UPDATE inventory
SET price = -10
WHERE product_id = 1;
-- ERROR: new row for relation "inventory" violates check constraint
```

**Solution:**
```sql
UPDATE inventory
SET price = 10  -- Use positive value
WHERE product_id = 1;
```

## Transaction Safety

### Example 13: Safe UPDATE with Transaction

```sql
BEGIN;

-- Update prices
UPDATE inventory
SET price = price * 1.15
WHERE category = 'Electronics';

-- Verify changes
SELECT product_name, price FROM inventory WHERE category = 'Electronics';

-- If correct:
COMMIT;

-- If wrong:
-- ROLLBACK;
```

### Example 14: Safe DELETE with Verification

```sql
BEGIN;

-- See what will be deleted
SELECT * FROM inventory WHERE stock_quantity = 0;

-- Delete if correct
DELETE FROM inventory WHERE stock_quantity = 0;

-- Verify deletion count
SELECT COUNT(*) FROM inventory;

COMMIT;
```

## Best Practices

### 1. Always Use WHERE Clause

```sql
-- Good: Specific update
UPDATE inventory SET price = 99.99 WHERE product_id = 1;

-- Dangerous: Updates ALL rows
UPDATE inventory SET price = 99.99;
```

### 2. Test with SELECT First

```sql
-- First, SELECT to see what will be affected
SELECT * FROM inventory WHERE category = 'Electronics';

-- Then UPDATE
UPDATE inventory SET price = price * 0.9 WHERE category = 'Electronics';
```

### 3. Use Transactions for Multiple Operations

```sql
BEGIN;
    UPDATE inventory SET stock_quantity = stock_quantity - 5 WHERE product_id = 1;
    DELETE FROM inventory WHERE stock_quantity <= 0;
COMMIT;
```

## Practical Examples

### Example 15: Bulk Price Update with Conditions

**SQL Statement:**
```sql
UPDATE inventory
SET price = CASE 
    WHEN category = 'Electronics' THEN price * 1.05
    WHEN category = 'Furniture' THEN price * 1.10
    WHEN category = 'Office Supplies' THEN price * 1.02
    ELSE price
END,
last_updated = CURRENT_TIMESTAMP
WHERE price IS NOT NULL;
```

### Example 16: Archive and Delete Old Records

```sql
-- Create archive table
CREATE TABLE inventory_archive AS SELECT * FROM inventory WHERE 1=0;

-- Move old records to archive
INSERT INTO inventory_archive
SELECT * FROM inventory WHERE stock_quantity = 0;

-- Delete from main table
DELETE FROM inventory
WHERE product_id IN (
    SELECT product_id FROM inventory_archive
);
```

### Example 17: Cascading Updates

```sql
-- Update related records across tables
UPDATE inventory
SET category = 'Tech'
WHERE category = 'Electronics';

-- This affects all related queries/reports automatically
```

