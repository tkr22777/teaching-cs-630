# Database Initialization - Week 10: Advanced SQL Topics

## Overview

This initialization script sets up the database used throughout all Week 10 Advanced SQL lessons. The database represents an e-commerce system with customers, orders, products, and order details.

## Why These Topics Matter

Understanding advanced SQL concepts is crucial for building production-ready applications:

- **Transactions**: Ensure data consistency when multiple operations must succeed or fail together (e.g., transferring money, processing orders)
- **Cursor-Based Pagination**: Scale to millions of rows efficiently without performance degradation
- **Window Functions**: Perform complex analytics without self-joins or subqueries
- **CTEs**: Write readable, maintainable queries for complex data transformations

These skills are essential for CS, DS, and BA professionals working with real-world data systems.

## Database Schema

The database consists of four interconnected tables:

- **Customers**: Customer information
- **Products**: Product catalog with inventory
- **Orders**: Customer order headers
- **Order_Details**: Individual items within each order (junction table)

## Setup Script

```sql
-- Clean up existing tables if they exist
BEGIN
   EXECUTE IMMEDIATE 'DROP TABLE order_details CASCADE CONSTRAINTS';
EXCEPTION WHEN OTHERS THEN NULL;
END;
/

BEGIN
   EXECUTE IMMEDIATE 'DROP TABLE orders CASCADE CONSTRAINTS';
EXCEPTION WHEN OTHERS THEN NULL;
END;
/

BEGIN
   EXECUTE IMMEDIATE 'DROP TABLE products CASCADE CONSTRAINTS';
EXCEPTION WHEN OTHERS THEN NULL;
END;
/

BEGIN
   EXECUTE IMMEDIATE 'DROP TABLE customers CASCADE CONSTRAINTS';
EXCEPTION WHEN OTHERS THEN NULL;
END;
/

-- Create Customers table
CREATE TABLE customers (
    customer_id INTEGER PRIMARY KEY,
    first_name VARCHAR2(50) NOT NULL,
    last_name VARCHAR2(50) NOT NULL,
    email VARCHAR2(100) UNIQUE NOT NULL,
    registration_date DATE DEFAULT SYSDATE,
    credit_limit NUMBER(10, 2),
    status VARCHAR2(20) DEFAULT 'Active'
);

-- Create Products table
CREATE TABLE products (
    product_id INTEGER PRIMARY KEY,
    product_name VARCHAR2(100) NOT NULL,
    category VARCHAR2(50),
    price NUMBER(10, 2) NOT NULL,
    stock_quantity INTEGER DEFAULT 0,
    last_updated DATE DEFAULT SYSDATE
);

-- Create Orders table
CREATE TABLE orders (
    order_id INTEGER PRIMARY KEY,
    customer_id INTEGER NOT NULL,
    order_date DATE DEFAULT SYSDATE,
    total_amount NUMBER(10, 2),
    status VARCHAR2(20) DEFAULT 'Pending',
    CONSTRAINT fk_ord_cust FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
    CONSTRAINT chk_ord_status CHECK (status IN ('Pending', 'Processing', 'Shipped', 'Delivered', 'Cancelled'))
);

-- Create Order_Details table
CREATE TABLE order_details (
    detail_id INTEGER PRIMARY KEY,
    order_id INTEGER NOT NULL,
    product_id INTEGER NOT NULL,
    quantity INTEGER NOT NULL,
    unit_price NUMBER(10, 2) NOT NULL,
    CONSTRAINT fk_det_ord FOREIGN KEY (order_id) REFERENCES orders(order_id),
    CONSTRAINT fk_det_prod FOREIGN KEY (product_id) REFERENCES products(product_id)
);

-- Insert Customers
INSERT INTO customers (customer_id, first_name, last_name, email, registration_date, credit_limit, status) VALUES
(1001, 'Alice', 'Johnson', 'alice.j@email.com', DATE '2023-01-15', 5000, 'Active');
INSERT INTO customers (customer_id, first_name, last_name, email, registration_date, credit_limit, status) VALUES
(1002, 'Bob', 'Smith', 'bob.s@email.com', DATE '2023-02-20', 3000, 'Active');
INSERT INTO customers (customer_id, first_name, last_name, email, registration_date, credit_limit, status) VALUES
(1003, 'Carol', 'Williams', 'carol.w@email.com', DATE '2023-03-10', 7500, 'Active');
INSERT INTO customers (customer_id, first_name, last_name, email, registration_date, credit_limit, status) VALUES
(1004, 'David', 'Brown', 'david.b@email.com', DATE '2023-04-05', 2000, 'Active');
INSERT INTO customers (customer_id, first_name, last_name, email, registration_date, credit_limit, status) VALUES
(1005, 'Emma', 'Davis', 'emma.d@email.com', DATE '2023-05-12', 4500, 'Active');
INSERT INTO customers (customer_id, first_name, last_name, email, registration_date, credit_limit, status) VALUES
(1006, 'Frank', 'Miller', 'frank.m@email.com', DATE '2023-06-18', 6000, 'Inactive');

-- Insert Products
INSERT INTO products (product_id, product_name, category, price, stock_quantity) VALUES
(2001, 'Laptop Pro 15', 'Electronics', 1299.99, 45);
INSERT INTO products (product_id, product_name, category, price, stock_quantity) VALUES
(2002, 'Wireless Mouse', 'Electronics', 29.99, 150);
INSERT INTO products (product_id, product_name, category, price, stock_quantity) VALUES
(2003, 'USB-C Cable', 'Electronics', 12.99, 200);
INSERT INTO products (product_id, product_name, category, price, stock_quantity) VALUES
(2004, 'Office Chair', 'Furniture', 249.99, 30);
INSERT INTO products (product_id, product_name, category, price, stock_quantity) VALUES
(2005, 'Standing Desk', 'Furniture', 399.99, 20);
INSERT INTO products (product_id, product_name, category, price, stock_quantity) VALUES
(2006, 'Monitor 27"', 'Electronics', 349.99, 55);
INSERT INTO products (product_id, product_name, category, price, stock_quantity) VALUES
(2007, 'Keyboard Mechanical', 'Electronics', 89.99, 80);
INSERT INTO products (product_id, product_name, category, price, stock_quantity) VALUES
(2008, 'Desk Lamp', 'Furniture', 39.99, 100);
INSERT INTO products (product_id, product_name, category, price, stock_quantity) VALUES
(2009, 'Webcam HD', 'Electronics', 79.99, 65);
INSERT INTO products (product_id, product_name, category, price, stock_quantity) VALUES
(2010, 'Headphones Wireless', 'Electronics', 159.99, 90);

-- Insert Orders
INSERT INTO orders (order_id, customer_id, order_date, total_amount, status) VALUES
(3001, 1001, DATE '2024-01-15', 1329.98, 'Delivered');
INSERT INTO orders (order_id, customer_id, order_date, total_amount, status) VALUES
(3002, 1001, DATE '2024-02-20', 289.98, 'Delivered');
INSERT INTO orders (order_id, customer_id, order_date, total_amount, status) VALUES
(3003, 1002, DATE '2024-02-22', 649.98, 'Shipped');
INSERT INTO orders (order_id, customer_id, order_date, total_amount, status) VALUES
(3004, 1003, DATE '2024-03-01', 1659.96, 'Processing');
INSERT INTO orders (order_id, customer_id, order_date, total_amount, status) VALUES
(3005, 1003, DATE '2024-03-15', 399.99, 'Delivered');
INSERT INTO orders (order_id, customer_id, order_date, total_amount, status) VALUES
(3006, 1004, DATE '2024-03-20', 119.98, 'Cancelled');
INSERT INTO orders (order_id, customer_id, order_date, total_amount, status) VALUES
(3007, 1004, DATE '2024-04-01', 449.98, 'Delivered');
INSERT INTO orders (order_id, customer_id, order_date, total_amount, status) VALUES
(3008, 1005, DATE '2024-04-10', 1539.97, 'Shipped');
INSERT INTO orders (order_id, customer_id, order_date, total_amount, status) VALUES
(3009, 1005, DATE '2024-04-25', 249.99, 'Pending');

-- Insert Order Details
INSERT INTO order_details (detail_id, order_id, product_id, quantity, unit_price) VALUES
(4001, 3001, 2001, 1, 1299.99);
INSERT INTO order_details (detail_id, order_id, product_id, quantity, unit_price) VALUES
(4002, 3001, 2002, 1, 29.99);
INSERT INTO order_details (detail_id, order_id, product_id, quantity, unit_price) VALUES
(4003, 3002, 2004, 1, 249.99);
INSERT INTO order_details (detail_id, order_id, product_id, quantity, unit_price) VALUES
(4004, 3002, 2008, 1, 39.99);
INSERT INTO order_details (detail_id, order_id, product_id, quantity, unit_price) VALUES
(4005, 3003, 2005, 1, 399.99);
INSERT INTO order_details (detail_id, order_id, product_id, quantity, unit_price) VALUES
(4006, 3003, 2004, 1, 249.99);
INSERT INTO order_details (detail_id, order_id, product_id, quantity, unit_price) VALUES
(4007, 3004, 2001, 1, 1299.99);
INSERT INTO order_details (detail_id, order_id, product_id, quantity, unit_price) VALUES
(4008, 3004, 2006, 1, 349.99);
INSERT INTO order_details (detail_id, order_id, product_id, quantity, unit_price) VALUES
(4009, 3004, 2010, 1, 159.99);
INSERT INTO order_details (detail_id, order_id, product_id, quantity, unit_price) VALUES
(4010, 3005, 2005, 1, 399.99);
INSERT INTO order_details (detail_id, order_id, product_id, quantity, unit_price) VALUES
(4011, 3006, 2002, 2, 29.99);
INSERT INTO order_details (detail_id, order_id, product_id, quantity, unit_price) VALUES
(4012, 3006, 2003, 5, 12.99);
INSERT INTO order_details (detail_id, order_id, product_id, quantity, unit_price) VALUES
(4013, 3007, 2005, 1, 399.99);
INSERT INTO order_details (detail_id, order_id, product_id, quantity, unit_price) VALUES
(4014, 3007, 2008, 1, 39.99);
INSERT INTO order_details (detail_id, order_id, product_id, quantity, unit_price) VALUES
(4015, 3008, 2001, 1, 1299.99);
INSERT INTO order_details (detail_id, order_id, product_id, quantity, unit_price) VALUES
(4016, 3008, 2007, 1, 89.99);
INSERT INTO order_details (detail_id, order_id, product_id, quantity, unit_price) VALUES
(4017, 3008, 2006, 1, 349.99);
INSERT INTO order_details (detail_id, order_id, product_id, quantity, unit_price) VALUES
(4018, 3009, 2004, 1, 249.99);

COMMIT;
```

## Summary

This script creates a complete e-commerce database with:
- 6 customers with varying credit limits and statuses
- 10 products across Electronics and Furniture categories
- 9 orders with different statuses (Delivered, Shipped, Processing, Pending, Cancelled)
- 18 order detail records linking orders to products

The data includes various scenarios useful for demonstrating advanced SQL concepts:
- Multiple orders per customer
- Multiple items per order
- Different order statuses for transaction examples
- Inventory tracking for stock management examples
- Date-based data for time-series analysis

