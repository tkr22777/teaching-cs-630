# SQL Assignment: E-commerce Platform Database

**Database System:** Oracle SQL
**Due Date:** November 19, 2025
**Total Problems:** 38 (Part A: 26 problems, Part B: 12 problems)

## Setup Instructions

Run the SQL scripts below in your Oracle database. **Important:** Suppliers table is NOT included - you'll create it in Problem 1.

<details>
<summary><b>Step 1: Drop Existing Tables (Click to expand)</b></summary>

```sql
-- Clean up existing tables if they exist
BEGIN
   EXECUTE IMMEDIATE 'DROP TABLE Reviews CASCADE CONSTRAINTS';
EXCEPTION
   WHEN OTHERS THEN NULL;
END;
/

BEGIN
   EXECUTE IMMEDIATE 'DROP TABLE OrderItems CASCADE CONSTRAINTS';
EXCEPTION
   WHEN OTHERS THEN NULL;
END;
/

BEGIN
   EXECUTE IMMEDIATE 'DROP TABLE Orders CASCADE CONSTRAINTS';
EXCEPTION
   WHEN OTHERS THEN NULL;
END;
/

BEGIN
   EXECUTE IMMEDIATE 'DROP TABLE Products CASCADE CONSTRAINTS';
EXCEPTION
   WHEN OTHERS THEN NULL;
END;
/

BEGIN
   EXECUTE IMMEDIATE 'DROP TABLE Categories CASCADE CONSTRAINTS';
EXCEPTION
   WHEN OTHERS THEN NULL;
END;
/

BEGIN
   EXECUTE IMMEDIATE 'DROP TABLE Customers CASCADE CONSTRAINTS';
EXCEPTION
   WHEN OTHERS THEN NULL;
END;
/
```

</details>

<details>
<summary><b>Step 2: Create Tables (Click to expand)</b></summary>

```sql
-- Customers Table
CREATE TABLE Customers (
    customer_id NUMBER PRIMARY KEY,
    first_name VARCHAR2(50) NOT NULL,
    last_name VARCHAR2(50) NOT NULL,
    email VARCHAR2(100) UNIQUE NOT NULL,
    phone VARCHAR2(20),
    registration_date DATE DEFAULT SYSDATE,
    account_status VARCHAR2(20) DEFAULT 'active'
);

-- Categories Table
CREATE TABLE Categories (
    category_id NUMBER PRIMARY KEY,
    category_name VARCHAR2(50) NOT NULL,
    description VARCHAR2(200)
);

-- Products Table
CREATE TABLE Products (
    product_id NUMBER PRIMARY KEY,
    product_name VARCHAR2(100) NOT NULL,
    category_id NUMBER NOT NULL,
    price NUMBER(10,2) NOT NULL,
    stock_quantity NUMBER DEFAULT 0,
    date_added DATE DEFAULT SYSDATE,
    CONSTRAINT fk_product_category FOREIGN KEY (category_id) 
        REFERENCES Categories(category_id)
);

-- Orders Table
CREATE TABLE Orders (
    order_id NUMBER PRIMARY KEY,
    customer_id NUMBER NOT NULL,
    order_date DATE DEFAULT SYSDATE,
    total_amount NUMBER(10,2),
    order_status VARCHAR2(20) DEFAULT 'pending',
    CONSTRAINT fk_order_customer FOREIGN KEY (customer_id) 
        REFERENCES Customers(customer_id)
);

-- OrderItems Table (Junction)
CREATE TABLE OrderItems (
    order_item_id NUMBER PRIMARY KEY,
    order_id NUMBER NOT NULL,
    product_id NUMBER NOT NULL,
    quantity NUMBER NOT NULL,
    unit_price NUMBER(10,2) NOT NULL,
    CONSTRAINT fk_orderitem_order FOREIGN KEY (order_id) 
        REFERENCES Orders(order_id),
    CONSTRAINT fk_orderitem_product FOREIGN KEY (product_id) 
        REFERENCES Products(product_id)
);

-- Reviews Table
CREATE TABLE Reviews (
    review_id NUMBER PRIMARY KEY,
    product_id NUMBER NOT NULL,
    customer_id NUMBER NOT NULL,
    rating NUMBER CHECK (rating BETWEEN 1 AND 5),
    review_date DATE DEFAULT SYSDATE,
    CONSTRAINT fk_review_product FOREIGN KEY (product_id) 
        REFERENCES Products(product_id),
    CONSTRAINT fk_review_customer FOREIGN KEY (customer_id) 
        REFERENCES Customers(customer_id)
);
```

</details>

<details>
<summary><b>Step 3: Insert Sample Data (Click to expand)</b></summary>

```sql
-- Insert Customers
INSERT INTO Customers VALUES (1, 'John', 'Smith', 'john.smith@email.com', '555-0101', DATE '2024-01-15', 'active');
INSERT INTO Customers VALUES (2, 'Sarah', 'Johnson', 'sarah.j@email.com', '555-0102', DATE '2024-03-20', 'active');
INSERT INTO Customers VALUES (3, 'Michael', 'Williams', 'mwilliams@email.com', '555-0103', DATE '2024-05-10', 'active');
INSERT INTO Customers VALUES (4, 'Emily', 'Brown', 'emily.brown@email.com', '555-0104', DATE '2024-06-25', 'active');
INSERT INTO Customers VALUES (5, 'David', 'Jones', 'djones@email.com', '555-0105', DATE '2024-08-05', 'active');
INSERT INTO Customers VALUES (6, 'Jennifer', 'Davis', 'jdavis@email.com', '555-0106', DATE '2024-09-12', 'inactive');
INSERT INTO Customers VALUES (7, 'Robert', 'Miller', 'rmiller@email.com', '555-0107', DATE '2024-10-01', 'active');
INSERT INTO Customers VALUES (8, 'Lisa', 'Wilson', 'lwilson@email.com', '555-0108', DATE '2024-10-15', 'active');

-- Insert Categories
INSERT INTO Categories VALUES (1, 'Electronics', 'Electronic devices and accessories');
INSERT INTO Categories VALUES (2, 'Clothing', 'Apparel and fashion items');
INSERT INTO Categories VALUES (3, 'Books', 'Physical and digital books');
INSERT INTO Categories VALUES (4, 'Home & Garden', 'Home improvement and garden supplies');
INSERT INTO Categories VALUES (5, 'Sports', 'Sports equipment and fitness items');

-- Insert Products
INSERT INTO Products VALUES (101, 'Wireless Mouse', 1, 25.99, 50, DATE '2024-01-10');
INSERT INTO Products VALUES (102, 'USB-C Cable', 1, 12.50, 100, DATE '2024-01-10');
INSERT INTO Products VALUES (103, 'Bluetooth Headphones', 1, 89.99, 30, DATE '2024-02-15');
INSERT INTO Products VALUES (104, 'Laptop Stand', 1, 45.00, 25, DATE '2024-03-01');
INSERT INTO Products VALUES (105, 'Webcam HD', 1, 129.99, 15, DATE '2024-04-20');
INSERT INTO Products VALUES (201, 'Cotton T-Shirt', 2, 19.99, 75, DATE '2024-01-05');
INSERT INTO Products VALUES (202, 'Denim Jeans', 2, 59.99, 40, DATE '2024-01-05');
INSERT INTO Products VALUES (203, 'Winter Jacket', 2, 149.99, 20, DATE '2024-02-10');
INSERT INTO Products VALUES (204, 'Running Shoes', 2, 89.99, 30, DATE '2024-03-15');
INSERT INTO Products VALUES (301, 'Database Design Book', 3, 45.00, 35, DATE '2024-01-20');
INSERT INTO Products VALUES (302, 'SQL Mastery Guide', 3, 39.99, 40, DATE '2024-01-20');
INSERT INTO Products VALUES (303, 'Python Programming', 3, 54.99, 25, DATE '2024-02-25');
INSERT INTO Products VALUES (401, 'Garden Tools Set', 4, 79.99, 20, DATE '2024-03-05');
INSERT INTO Products VALUES (402, 'LED Desk Lamp', 4, 34.99, 45, DATE '2024-03-05');
INSERT INTO Products VALUES (403, 'Kitchen Mixer', 4, 199.99, 10, DATE '2024-04-10');
INSERT INTO Products VALUES (501, 'Yoga Mat', 5, 29.99, 50, DATE '2024-02-01');
INSERT INTO Products VALUES (502, 'Dumbbell Set', 5, 89.99, 15, DATE '2024-02-01');
INSERT INTO Products VALUES (503, 'Tennis Racket', 5, 119.99, 12, DATE '2024-03-20');

-- Insert Orders
INSERT INTO Orders VALUES (1001, 1, DATE '2024-05-15', 115.98, 'completed');
INSERT INTO Orders VALUES (1002, 2, DATE '2024-05-20', 59.99, 'completed');
INSERT INTO Orders VALUES (1003, 1, DATE '2024-06-10', 89.99, 'completed');
INSERT INTO Orders VALUES (1004, 3, DATE '2024-06-25', 179.98, 'completed');
INSERT INTO Orders VALUES (1005, 4, DATE '2024-07-05', 45.00, 'completed');
INSERT INTO Orders VALUES (1006, 2, DATE '2024-07-20', 249.97, 'completed');
INSERT INTO Orders VALUES (1007, 5, DATE '2024-08-10', 129.99, 'completed');
INSERT INTO Orders VALUES (1008, 3, DATE '2024-08-25', 89.98, 'pending');
INSERT INTO Orders VALUES (1009, 1, DATE '2024-09-05', 199.99, 'completed');
INSERT INTO Orders VALUES (1010, 7, DATE '2024-09-15', 154.98, 'completed');
INSERT INTO Orders VALUES (1011, 8, DATE '2024-10-01', 79.99, 'pending');
INSERT INTO Orders VALUES (1012, 4, DATE '2024-10-20', 89.99, 'completed');
INSERT INTO Orders VALUES (1013, 2, DATE '2024-10-25', 34.99, 'cancelled');

-- Insert OrderItems
INSERT INTO OrderItems VALUES (1, 1001, 103, 1, 89.99);
INSERT INTO OrderItems VALUES (2, 1001, 102, 2, 12.99);
INSERT INTO OrderItems VALUES (3, 1002, 202, 1, 59.99);
INSERT INTO OrderItems VALUES (4, 1003, 204, 1, 89.99);
INSERT INTO OrderItems VALUES (5, 1004, 203, 1, 149.99);
INSERT INTO OrderItems VALUES (6, 1004, 501, 1, 29.99);
INSERT INTO OrderItems VALUES (7, 1005, 301, 1, 45.00);
INSERT INTO OrderItems VALUES (8, 1006, 403, 1, 199.99);
INSERT INTO OrderItems VALUES (9, 1006, 302, 1, 39.99);
INSERT INTO OrderItems VALUES (10, 1006, 101, 1, 9.99);
INSERT INTO OrderItems VALUES (11, 1007, 105, 1, 129.99);
INSERT INTO OrderItems VALUES (12, 1008, 501, 2, 29.99);
INSERT INTO OrderItems VALUES (13, 1008, 502, 1, 29.99);
INSERT INTO OrderItems VALUES (14, 1009, 403, 1, 199.99);
INSERT INTO OrderItems VALUES (15, 1010, 503, 1, 119.99);
INSERT INTO OrderItems VALUES (16, 1010, 402, 1, 34.99);
INSERT INTO OrderItems VALUES (17, 1011, 401, 1, 79.99);
INSERT INTO OrderItems VALUES (18, 1012, 204, 1, 89.99);
INSERT INTO OrderItems VALUES (19, 1013, 402, 1, 34.99);

-- Insert Reviews
INSERT INTO Reviews VALUES (1, 103, 1, 5, DATE '2024-05-20');
INSERT INTO Reviews VALUES (2, 103, 2, 4, DATE '2024-06-01');
INSERT INTO Reviews VALUES (3, 202, 2, 5, DATE '2024-05-25');
INSERT INTO Reviews VALUES (4, 204, 1, 4, DATE '2024-06-15');
INSERT INTO Reviews VALUES (5, 203, 3, 5, DATE '2024-07-01');
INSERT INTO Reviews VALUES (6, 301, 4, 5, DATE '2024-07-10');
INSERT INTO Reviews VALUES (7, 403, 2, 4, DATE '2024-07-25');
INSERT INTO Reviews VALUES (8, 105, 5, 5, DATE '2024-08-15');
INSERT INTO Reviews VALUES (9, 501, 3, 4, DATE '2024-08-30');
INSERT INTO Reviews VALUES (10, 103, 3, 5, DATE '2024-09-05');
INSERT INTO Reviews VALUES (11, 503, 7, 5, DATE '2024-09-20');
INSERT INTO Reviews VALUES (12, 204, 4, 4, DATE '2024-10-25');
INSERT INTO Reviews VALUES (13, 302, 2, 5, DATE '2024-08-01');
INSERT INTO Reviews VALUES (14, 501, 8, 3, DATE '2024-10-10');

COMMIT;
```

</details>

## Submission Format

- Google Slides (read permissions), one problem per slide
- Include screenshots of query results where specified
- Submit Google Slides link with public read permissions

## Problem Types

- **Query Writing (29 problems):** Write SQL queries based on problem description
- **Query Analysis (9 problems):** Run provided SQL, screenshot results/errors, and explain what's happening (includes intentional errors for learning)

**Total: 38 Problems**

- Part A: 26 problems - Fundamentals including JOINs (covers DDL, DML, SELECT, JOINs, Aggregates with JOINs)
- Part B: 12 problems - Advanced Topics (covers Subqueries, Functions, Set Operators)

---

## Database Schema Overview

**E-Commerce Platform Entities:**

1. **Customers** - Customer account information
2. **Categories** - Product categories (electronics, clothing, etc.)
3. **Products** - Product catalog with pricing and inventory
4. **Orders** - Customer orders with order date and status
5. **OrderItems** - Individual items within each order (junction table)
6. **Reviews** - Customer product reviews and ratings
7. **Suppliers** - Product suppliers/vendors (you will create this table)

---

## Part A: SQL Fundamentals with JOINs

**Due: November 19, 2025**
**Topics:** DDL, DML, SELECT, JOINs, Aggregates with JOINs, Indexes, Views

**Note:** This part introduces JOINs early so you can practice aggregates and filtering in realistic multi-table scenarios.

### Section 1: Table Creation & Data Manipulation

### Problem 1: CREATE TABLE
Create a **Suppliers** table with columns: `supplier_id` (PK, `NUMBER`), `supplier_name` (`VARCHAR2(100)`), `contact_email` (`VARCHAR2(100)`), `phone` (`VARCHAR2(20)`), and `country` (`VARCHAR2(50)`). Choose appropriate constraints (e.g., `NOT NULL`).

### Problem 2: ALTER TABLE
Add a `supplier_id` column to the `Products` table and make it a foreign key referencing `Suppliers(supplier_id)`. Ensure the data type matches the primary key in `Suppliers`.

### Problem 3: INSERT, UPDATE, DELETE
Write three queries:
1. Insert 2 new products into Products
2. Update stock_quantity for all products in category_id = 1 by adding 10
3. Delete all orders with status 'cancelled'

### Problem 4: Query Analysis - INSERT Error
Run this query, screenshot the error, and explain what went wrong and how to fix it:

```sql
INSERT INTO Products (product_id, product_name, price, stock_quantity)
VALUES (601, 'Wireless Keyboard', 45.99, 20);
```

### Problem 5: Query Analysis - Column Name Error
Run this query, screenshot the error, and explain what's wrong:

```sql
SELECT first_name, last_name, registration_dt
FROM Customers
WHERE account_status = 'active';
```

### Problem 6: SELECT with WHERE
Find products priced between $20 and $100 that have stock_quantity > 0, ordered by price.

### Section 2: JOIN Fundamentals

### Problem 7: INNER JOIN - Basic
List all products with their category names. Display product_name, category_name, and price. Use INNER JOIN between Products and Categories.

### Problem 8: INNER JOIN - 3 Tables
List all products that have been ordered and the date of the order. Display `product_name` and `order_date`. This will require joining `Products`, `OrderItems`, and `Orders`.

### Problem 9: LEFT JOIN - With NULLs
Show all customers and their order count, including customers with no orders. Use LEFT JOIN between Customers and Orders with COUNT and GROUP BY.

### Problem 10: LEFT JOIN - Product Reviews
Show all products and their total review count, including products with no reviews. Use LEFT JOIN between Products and Reviews with COUNT and GROUP BY.

### Problem 11: RIGHT JOIN
Show all products and their total quantity ordered, including products with no orders. Use RIGHT JOIN between OrderItems and Products with SUM and GROUP BY.

### Problem 12: SELF JOIN
Find pairs of products in the same category with similar prices (within $10 of each other). Avoid duplicate pairs using a WHERE condition (e.g., p1.product_id < p2.product_id).

### Problem 13: Multi-Table JOIN (4 tables)
Display complete order information: order_id, customer full name, product_name, quantity, unit_price. Join Orders, Customers, OrderItems, and Products.

### Section 3: Aggregates with JOINs

### Problem 14: COUNT with INNER JOIN
Count how many orders each customer has placed. Show customer's first name, last name, and order count. Use INNER JOIN between Customers and Orders with GROUP BY.

### Problem 15: SUM with JOIN and WHERE
Calculate total revenue (sum of total_amount) for each customer who has orders with `order_status = 'completed'`. Show the customer's full name (first and last) and total revenue. Use JOIN and WHERE.

### Problem 16: AVG with JOIN and HAVING
Find the average rating for each product that has been reviewed, showing only products with average rating > 3.5. Display product_name and average rating. Join Products and Reviews.

### Problem 17: MIN/MAX with JOIN
Find the cheapest and most expensive product ordered in each order. Display order_id, MIN(unit_price) as "Cheapest_Item", MAX(unit_price) as "Most_Expensive_Item". Join Orders and OrderItems.

### Problem 18: GROUP BY with HAVING and JOIN
Show categories that have more than 2 products with stock_quantity > 10. Display category_name and product count. Join Categories and Products.

### Problem 19: Date Range with JOIN
Find all orders placed between June 1, 2024 and September 1, 2024. Display order_id, customer full name, order_date, and total_amount. Join Orders and Customers with WHERE on date range.

### Section 4: Indexes & Views

### Problem 20: CREATE INDEX
Create an index on Orders.order_date. Explain when this index helps query performance and give an example query that would benefit.

### Problem 21: CREATE VIEW with JOIN
Create a view called **CustomerOrderSummary** showing customer_id, full name (first + last), total_orders (COUNT), and total_spent (SUM of total_amount). Join Customers and Orders with GROUP BY.

### Section 5: Query Analysis

### Problem 22: Query Analysis - Foreign Key Error
Run this query, screenshot the error, and explain what happens:

```sql
CREATE TABLE TestTable (
    id NUMBER PRIMARY KEY,
    category_id NUMBER,
    FOREIGN KEY (category_id) REFERENCES NonExistentTable(id)
);
```

### Problem 23: Query Analysis - Aggregate with JOIN
Run this query and explain the results (what does each column show?):

```sql
SELECT c.category_name, COUNT(p.product_id), AVG(p.price), MAX(p.stock_quantity)
FROM Categories c
LEFT JOIN Products p ON c.category_id = p.category_id
GROUP BY c.category_name;
```

### Problem 24: Query Analysis - Missing GROUP BY
Run this query, screenshot the error, and explain what's wrong:

```sql
SELECT category_id, product_name, COUNT(*)
FROM Products;
```

### Problem 25: Query Analysis - TRUNCATE vs DELETE
Run these queries and explain the difference:

```sql
-- Query A
DELETE FROM Reviews WHERE rating < 3;

-- Query B (WARNING: Run only after backing up your data)
TRUNCATE TABLE Reviews;
```
What's the difference? When would you use each?

### Problem 26: Query Analysis - JOIN Without ON
Run this query, screenshot the error or result, and explain what happens:

```sql
SELECT c.first_name, o.order_id
FROM Customers c, Orders o;
```
Is this a valid query? What type of result does it produce?

---

## Part B: Advanced Topics

**Due: November 19, 2025**
**Topics:** Subqueries, Functions, Set Operators, Complex Analytics

### Section 6: Subqueries

### Problem 27: Subquery with IN
List products that have been reviewed. Use IN with a subquery to find product_ids from the Reviews table. Display product_id, product_name, and price.

### Problem 28: Subquery with NOT EXISTS
Find products that have never been ordered. Use NOT EXISTS with a subquery checking OrderItems. Display product_id, product_name, and stock_quantity.

### Problem 29: Subquery in WHERE - Comparison
Find customers who have placed orders with total_amount greater than the average order amount. Show customer first name, last name, and their individual order totals. Use a subquery to calculate the average.

### Problem 30: Correlated Subquery
Show products with price higher than the average price in their category. Display product_name, category_id, price, and the category's average price (use a correlated subquery in SELECT).

### Problem 31: Inline View (Subquery in FROM)
Create a query that uses an inline view: First calculate total revenue per customer (subquery in FROM), then show only customers with revenue > $200. Display customer_id and total_revenue.

### Section 7: Single-Row Functions

### Problem 32: String Functions
Display customer information with:
- Full name formatted as "LAST, First" (use `UPPER` for last, proper case for first). **Hint:** Oracle's `INITCAP` function converts to proper case.
- First 3 characters of email (`SUBSTR`)
- Email length (`LENGTH`)

Note: Oracle's `CONCAT` takes only 2 arguments, use `||` for multiple concatenations.

### Problem 33: Date Functions
Display orders with:
- order_date formatted as 'Month DD, YYYY' (TO_CHAR with format 'Month DD, YYYY')
- Days since order (SYSDATE - order_date)
- Months since order (MONTHS_BETWEEN)

Filter to show only orders from June 2024 onwards (order_date >= DATE '2024-06-01').

### Problem 34: Numeric Functions and CASE
Display products with:
- Original price
- Price rounded to nearest dollar (ROUND)
- Price truncated to integer (TRUNC)
- Price category using CASE: 'Premium' (>$100), 'Standard' ($50-$100), 'Budget' (<$50)

### Section 8: Set Operators

### Problem 35: UNION
Get all customer_ids who either placed orders OR wrote reviews (remove duplicates). Use UNION between Orders and Reviews.

### Problem 36: UNION ALL
List all product_ids from OrderItems UNION ALL product_ids from Reviews (keep all occurrences, including duplicates). Order the final result by product_id.

### Section 9: Query Analysis - Advanced

### Problem 37: Query Analysis - Subquery Error
Run this query, screenshot the error, and explain what happens:

```sql
SELECT product_name, 
       (SELECT AVG(rating) FROM Reviews)
FROM Products
WHERE product_id = (SELECT product_id FROM Reviews);
```
What are the two errors in this query?

### Problem 38: Query Analysis - Set Operators
Run these queries and explain the difference in results:

```sql
-- Query A
SELECT customer_id FROM Orders
UNION
SELECT customer_id FROM Reviews;

-- Query B
SELECT customer_id FROM Orders
UNION ALL
SELECT customer_id FROM Reviews;
```
How many rows does each return? When would you use each operator?
