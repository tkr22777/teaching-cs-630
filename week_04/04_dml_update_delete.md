# DML: UPDATE and DELETE Statements

## Overview

UPDATE and DELETE statements modify existing data in tables. These are powerful commands that can affect multiple rows, so use them carefully.

## Sample Data

<details>
<summary>Click to expand: Database setup script</summary>

```sql
CREATE TABLE inventory (
    product_id INTEGER PRIMARY KEY,
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

</details>

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

###

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

###

<details>
<summary>PostgreSQL: RETURNING</summary>

```sql
-- Return deleted rows
DELETE FROM inventory
WHERE price < 10
RETURNING product_id, product_name, price;
```

</details>

###

##

