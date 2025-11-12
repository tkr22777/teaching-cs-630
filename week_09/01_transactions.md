# SQL Transactions and Transaction Control

## Overview

A **transaction** is a logical unit of work containing one or more SQL statements. Transactions ensure data consistency and integrity by guaranteeing that either all operations succeed (commit) or all fail (rollback).

## Key Terms

**Transaction**: A sequence of SQL operations treated as a single unit of work.

**COMMIT**: Saves all changes made in the current transaction permanently to the database.

**ROLLBACK**: Undoes all changes made in the current transaction.

**SAVEPOINT**: Creates a marker within a transaction to which you can rollback.

**ACID Properties**: Atomicity, Consistency, Isolation, Durability - the four key properties of transactions.

**Atomicity**: All operations in a transaction succeed or all fail (all-or-nothing).

**Consistency**: Transactions bring the database from one valid state to another.

**Isolation**: Concurrent transactions don't interfere with each other.

**Durability**: Once committed, changes are permanent even if system fails.

**Autocommit**: Automatic commit after each SQL statement (default in many SQL tools).

## Sample Database Schema

This module uses the e-commerce system.

**Note:** The complete database setup script is available in `00_initialization.md` in this directory.

## Why Transactions Matter

Consider these real-world scenarios where transactions are critical:

**Banking**: Transfer $500 from Account A to Account B
- Debit $500 from Account A
- Credit $500 to Account B
- Both must succeed or both must fail

**E-Commerce**: Process an order
- Create order record
- Reduce product inventory
- Update customer credit limit
- All must succeed or nothing should change

**Data Import**: Load 10,000 records
- If record 9,999 has an error, rollback all 10,000
- Maintain database consistency

## Basic Transaction Control

### COMMIT

COMMIT makes all changes permanent:

```sql
-- Update product inventory
UPDATE products
SET stock_quantity = stock_quantity - 5
WHERE product_id = 2002;

-- Verify the change
SELECT product_id, product_name, stock_quantity
FROM products
WHERE product_id = 2002;

-- Make it permanent
COMMIT;
```

**Output:**
```
1 row updated.

PRODUCT_ID | PRODUCT_NAME     | STOCK_QUANTITY
-----------|------------------|---------------
2002       | Wireless Mouse   | 145

Commit complete.
```

**Explanation:** After COMMIT, the inventory reduction is saved permanently. Even if the session disconnects, the change persists.

### ROLLBACK

ROLLBACK undoes all changes since the last COMMIT:

```sql
-- Start making changes
UPDATE products
SET stock_quantity = stock_quantity - 10
WHERE product_id = 2003;

SELECT product_id, product_name, stock_quantity
FROM products
WHERE product_id = 2003;

-- Oops, wrong quantity! Undo it
ROLLBACK;

-- Verify rollback
SELECT product_id, product_name, stock_quantity
FROM products
WHERE product_id = 2003;
```

**Output:**
```
1 row updated.

PRODUCT_ID | PRODUCT_NAME  | STOCK_QUANTITY
-----------|---------------|---------------
2003       | USB-C Cable   | 190

Rollback complete.

PRODUCT_ID | PRODUCT_NAME  | STOCK_QUANTITY
-----------|---------------|---------------
2003       | USB-C Cable   | 200
```

**Explanation:** ROLLBACK restored the stock_quantity to its original value of 200.

## SAVEPOINT

SAVEPOINT creates markers within a transaction for partial rollback:

### Syntax

```sql
SAVEPOINT savepoint_name;
ROLLBACK TO savepoint_name;
```

### Example: Multiple Savepoints

```sql
-- Initial values
SELECT customer_id, first_name, credit_limit
FROM customers
WHERE customer_id IN (1001, 1002, 1003);

-- Transaction start
UPDATE customers SET credit_limit = credit_limit + 500 WHERE customer_id = 1001;
SAVEPOINT after_customer_1;

UPDATE customers SET credit_limit = credit_limit + 1000 WHERE customer_id = 1002;
SAVEPOINT after_customer_2;

UPDATE customers SET credit_limit = credit_limit + 1500 WHERE customer_id = 1003;
SAVEPOINT after_customer_3;

-- Check current state
SELECT customer_id, first_name, credit_limit
FROM customers
WHERE customer_id IN (1001, 1002, 1003);

-- Rollback to after_customer_2 (undo only customer 1003)
ROLLBACK TO after_customer_2;

-- Check after partial rollback
SELECT customer_id, first_name, credit_limit
FROM customers
WHERE customer_id IN (1001, 1002, 1003);

-- Final rollback to undo everything
ROLLBACK;
```

**Output:**
```
Initial State:
CUSTOMER_ID | FIRST_NAME | CREDIT_LIMIT
------------|------------|-------------
1001        | Alice      | 5000
1002        | Bob        | 3000
1003        | Carol      | 7500

After all updates:
CUSTOMER_ID | FIRST_NAME | CREDIT_LIMIT
------------|------------|-------------
1001        | Alice      | 5500
1002        | Bob        | 4000
1003        | Carol      | 9000

After ROLLBACK TO after_customer_2:
CUSTOMER_ID | FIRST_NAME | CREDIT_LIMIT
------------|------------|-------------
1001        | Alice      | 5500
1002        | Bob        | 4000
1003        | Carol      | 7500

After final ROLLBACK:
CUSTOMER_ID | FIRST_NAME | CREDIT_LIMIT
------------|------------|-------------
1001        | Alice      | 5000
1002        | Bob        | 3000
1003        | Carol      | 7500
```

**Explanation:** ROLLBACK TO allows you to undo part of a transaction while keeping earlier changes.

## Practical Example: Order Processing

A complete order processing transaction:

```sql
-- Enable output
SET SERVEROUTPUT ON;

DECLARE
    v_order_id orders.order_id%TYPE := 3010;
    v_customer_id customers.customer_id%TYPE := 1002;
    v_product_id products.product_id%TYPE := 2001;
    v_quantity INTEGER := 2;
    v_unit_price products.price%TYPE;
    v_total_amount NUMBER;
    v_current_stock products.stock_quantity%TYPE;
    v_credit_limit customers.credit_limit%TYPE;
BEGIN
    -- Step 1: Get product info
    SELECT price, stock_quantity
    INTO v_unit_price, v_current_stock
    FROM products
    WHERE product_id = v_product_id;
    
    v_total_amount := v_unit_price * v_quantity;
    
    DBMS_OUTPUT.PUT_LINE('=== ORDER PROCESSING ===');
    DBMS_OUTPUT.PUT_LINE('Product Price: $' || v_unit_price);
    DBMS_OUTPUT.PUT_LINE('Quantity: ' || v_quantity);
    DBMS_OUTPUT.PUT_LINE('Total: $' || v_total_amount);
    DBMS_OUTPUT.PUT_LINE('Current Stock: ' || v_current_stock);
    
    -- Step 2: Validate stock
    IF v_current_stock < v_quantity THEN
        RAISE_APPLICATION_ERROR(-20001, 'Insufficient stock');
    END IF;
    
    -- Step 3: Validate customer credit
    SELECT credit_limit INTO v_credit_limit
    FROM customers
    WHERE customer_id = v_customer_id;
    
    IF v_credit_limit < v_total_amount THEN
        RAISE_APPLICATION_ERROR(-20002, 'Exceeds credit limit');
    END IF;
    
    -- Step 4: Create order (savepoint before major changes)
    SAVEPOINT before_order_create;
    
    INSERT INTO orders (order_id, customer_id, order_date, total_amount, status)
    VALUES (v_order_id, v_customer_id, SYSDATE, v_total_amount, 'Processing');
    
    DBMS_OUTPUT.PUT_LINE('Order created: ' || v_order_id);
    
    -- Step 5: Add order details
    INSERT INTO order_details (detail_id, order_id, product_id, quantity, unit_price)
    VALUES (4019, v_order_id, v_product_id, v_quantity, v_unit_price);
    
    -- Step 6: Update product inventory
    UPDATE products
    SET stock_quantity = stock_quantity - v_quantity
    WHERE product_id = v_product_id;
    
    DBMS_OUTPUT.PUT_LINE('Inventory updated: -' || v_quantity || ' units');
    
    -- Step 7: Update customer credit
    UPDATE customers
    SET credit_limit = credit_limit - v_total_amount
    WHERE customer_id = v_customer_id;
    
    DBMS_OUTPUT.PUT_LINE('Credit limit reduced: $' || v_total_amount);
    
    -- Success! Commit the transaction
    COMMIT;
    DBMS_OUTPUT.PUT_LINE('Transaction COMMITTED successfully');
    
EXCEPTION
    WHEN OTHERS THEN
        -- Error occurred, rollback everything
        ROLLBACK;
        DBMS_OUTPUT.PUT_LINE('Transaction ROLLED BACK: ' || SQLERRM);
END;
/
```

**Output (Success Case):**
```
=== ORDER PROCESSING ===
Product Price: $1299.99
Quantity: 2
Total: $2599.98
Current Stock: 45
Order created: 3010
Inventory updated: -2 units
Credit limit reduced: $2599.98
Transaction COMMITTED successfully
```

**Output (Error Case - Insufficient Stock):**
```
=== ORDER PROCESSING ===
Product Price: $1299.99
Quantity: 100
Total: $129999
Current Stock: 45
Transaction ROLLED BACK: ORA-20001: Insufficient stock
```

**Explanation:** The transaction ensures that either all changes succeed (order created, inventory reduced, credit updated) or nothing changes (rollback on error).

## Transaction Best Practices

### 1. Keep Transactions Short

**Bad:**
```sql
BEGIN
    -- Long-running operation
    UPDATE products SET price = price * 1.10;  -- Updates 1 million rows
    
    -- User interaction (waits for confirmation)
    -- Other operations...
    
    COMMIT;  -- Holds locks for too long
END;
```

**Good:**
```sql
BEGIN
    -- Quick, focused transaction
    UPDATE products 
    SET price = price * 1.10 
    WHERE category = 'Electronics';  -- Updates 1000 rows
    
    COMMIT;
END;
```

### 2. Handle Errors Gracefully

```sql
DECLARE
    v_error_msg VARCHAR2(200);
BEGIN
    -- Transaction operations
    SAVEPOINT before_critical_update;
    
    UPDATE orders SET status = 'Shipped' WHERE order_id = 3004;
    
    IF SQL%ROWCOUNT = 0 THEN
        RAISE_APPLICATION_ERROR(-20003, 'Order not found');
    END IF;
    
    COMMIT;
    
EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK TO before_critical_update;
        v_error_msg := SQLERRM;
        DBMS_OUTPUT.PUT_LINE('Error: ' || v_error_msg);
        -- Log error to error table
        ROLLBACK;  -- Rollback everything
END;
/
```

### 3. Use Savepoints for Complex Transactions

```sql
BEGIN
    -- Part 1: Customer updates
    UPDATE customers SET status = 'VIP' WHERE customer_id = 1001;
    SAVEPOINT after_customer_update;
    
    -- Part 2: Order updates (might fail)
    UPDATE orders SET status = 'Priority' WHERE customer_id = 1001;
    SAVEPOINT after_order_update;
    
    -- Part 3: Notification (external system - might fail)
    -- send_notification_to_external_system();
    
    COMMIT;
EXCEPTION
    WHEN OTHERS THEN
        -- Rollback to last successful savepoint
        ROLLBACK TO after_customer_update;
        COMMIT;  -- Keep partial work
END;
/
```

## Implicit vs Explicit Transactions

### Implicit Commit

Some DDL statements automatically commit:

```sql
UPDATE products SET price = price * 1.10 WHERE product_id = 2001;
-- No COMMIT yet

CREATE TABLE temp_table (id INTEGER);  -- Implicit COMMIT!

-- Previous UPDATE is now committed
```

**DDL operations that trigger implicit commit:**
- CREATE, ALTER, DROP (table, index, sequence)
- GRANT, REVOKE
- TRUNCATE

### Explicit Control

Use explicit COMMIT/ROLLBACK for full control:

```sql
BEGIN
    UPDATE products SET price = price * 1.10 WHERE category = 'Electronics';
    
    IF SQL%ROWCOUNT > 100 THEN
        DBMS_OUTPUT.PUT_LINE('Too many rows affected');
        ROLLBACK;
    ELSE
        COMMIT;
    END IF;
END;
/
```

## Summary

**Key takeaways:**

1. **Transactions** - Group multiple SQL operations into an all-or-nothing unit
2. **COMMIT** - Makes changes permanent
3. **ROLLBACK** - Undoes changes back to last COMMIT
4. **SAVEPOINT** - Creates markers for partial rollback
5. **ACID Properties** - Atomicity, Consistency, Isolation, Durability ensure data integrity
6. **Error Handling** - Always handle exceptions and rollback on errors
7. **Best Practices** - Keep transactions short, use savepoints for complex operations

Understanding transactions is essential for building reliable applications that maintain data consistency even when errors occur.

