# DML: UPDATE and DELETE Statements

## Overview

**DML (Data Manipulation Language)** commands work with the data inside tables. UPDATE lets you modify existing data (like correcting errors or updating prices), while DELETE removes rows you no longer need. Both can affect multiple rows at once, so always use a WHERE clause carefully.

## Sample Data

<details>
<summary>Click to expand: Database setup script</summary>

```sql
CREATE TABLE inventory (
    product_id INTEGER PRIMARY KEY,
    product_name VARCHAR2(100) NOT NULL,
    category VARCHAR2(50),
    price NUMBER(10, 2),
    stock_quantity INTEGER DEFAULT 0,
    last_updated TIMESTAMP DEFAULT SYSTIMESTAMP
);

INSERT INTO inventory (product_id, product_name, category, price, stock_quantity) VALUES (1, 'Laptop', 'Electronics', 999.99, 15);
INSERT INTO inventory (product_id, product_name, category, price, stock_quantity) VALUES (2, 'Mouse', 'Electronics', 25.50, 100);
INSERT INTO inventory (product_id, product_name, category, price, stock_quantity) VALUES (3, 'Keyboard', 'Electronics', 75.00, 50);
INSERT INTO inventory (product_id, product_name, category, price, stock_quantity) VALUES (4, 'Monitor', 'Electronics', 299.99, 30);
INSERT INTO inventory (product_id, product_name, category, price, stock_quantity) VALUES (5, 'Desk Chair', 'Furniture', 199.99, 25);
INSERT INTO inventory (product_id, product_name, category, price, stock_quantity) VALUES (6, 'Standing Desk', 'Furniture', 450.00, 10);
INSERT INTO inventory (product_id, product_name, category, price, stock_quantity) VALUES (7, 'Notebook', 'Office Supplies', 5.99, 200);
INSERT INTO inventory (product_id, product_name, category, price, stock_quantity) VALUES (8, 'Pen Set', 'Office Supplies', 12.99, 150);
```

</details>

**Initial Inventory Table:**

| product_id | product_name  | category        | price  | stock_quantity | last_updated        |
| ---------- | ------------- | --------------- | ------ | -------------- | ------------------- |
| 1          | Laptop        | Electronics     | 999.99 | 15             | 2024-10-17 10:00:00 |
| 2          | Mouse         | Electronics     | 25.50  | 100            | 2024-10-17 10:00:00 |
| 3          | Keyboard      | Electronics     | 75.00  | 50             | 2024-10-17 10:00:00 |
| 4          | Monitor       | Electronics     | 299.99 | 30             | 2024-10-17 10:00:00 |
| 5          | Desk Chair    | Furniture       | 199.99 | 25             | 2024-10-17 10:00:00 |
| 6          | Standing Desk | Furniture       | 450.00 | 10             | 2024-10-17 10:00:00 |
| 7          | Notebook      | Office Supplies | 5.99   | 200            | 2024-10-17 10:00:00 |
| 8          | Pen Set       | Office Supplies | 12.99  | 150            | 2024-10-17 10:00:00 |

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

| product_id | product_name | category    | price  | stock_quantity |
| ---------- | ------------ | ----------- | ------ | -------------- |
| 1          | Laptop       | Electronics | 899.99 | 15             |

When your WHERE clause matches multiple rows, all matching rows get updated. This is useful for bulk changes like applying a discount to a category.

### Example 2: Update Multiple Rows

**SQL Statement:**

```sql
UPDATE inventory
SET price = price * 1.10
WHERE category = 'Electronics';
```

**Result:** All Electronics products have prices increased by 10%.

| product_id | product_name | category    | old_price | new_price |
| ---------- | ------------ | ----------- | --------- | --------- |
| 1          | Laptop       | Electronics | 999.99    | 1099.99   |
| 2          | Mouse        | Electronics | 25.50     | 28.05     |
| 3          | Keyboard     | Electronics | 75.00     | 82.50     |
| 4          | Monitor      | Electronics | 299.99    | 329.99    |

---

## DELETE Statement

### Basic Syntax

```sql
DELETE FROM table_name
WHERE condition;
```

### Example 1: Delete Single Row

**SQL Statement:**

```sql
DELETE FROM inventory
WHERE product_id = 8;
```

**Before:**

| product_id | product_name | category        |
| ---------- | ------------ | --------------- |
| 7          | Notebook     | Office Supplies |
| 8          | Pen Set      | Office Supplies |

**After:**

| product_id | product_name | category        |
| ---------- | ------------ | --------------- |
| 7          | Notebook     | Office Supplies |

Just like UPDATE, your WHERE clause determines how many rows get deleted. You can combine conditions with AND/OR just like in SELECT queries.

### Example 2: Delete All Rows (Be Careful!)

**SQL Statement:**

```sql
-- This deletes ALL rows from the table!
DELETE FROM inventory;
```

**Warning:** Without a WHERE clause, DELETE removes ALL rows. Use TRUNCATE instead for better performance when clearing entire tables.

## Summary

**UPDATE** modifies existing rows - use SET to specify new values and WHERE to target specific rows.
**DELETE** removes rows - always use WHERE to avoid deleting all data.
