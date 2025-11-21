# SELECT Basics: Retrieving Data

## Overview

SELECT is how you retrieve data from your database. Whether you need to display information to users, generate reports, or check what's in your tables, SELECT is your starting point. You'll use it more than any other SQL command.

## Sample Data

<details>
<summary>Click to expand: Database setup script</summary>

```sql
CREATE TABLE books (
    book_id INTEGER PRIMARY KEY,
    title VARCHAR2(200) NOT NULL,
    author VARCHAR2(100) NOT NULL,
    genre VARCHAR2(50),
    publication_year INTEGER,
    price NUMBER(10, 2),
    stock_quantity INTEGER,
    rating NUMBER(3, 2)
);

INSERT INTO books (book_id, title, author, genre, publication_year, price, stock_quantity, rating) VALUES (1, 'The Great Gatsby', 'F. Scott Fitzgerald', 'Fiction', 1925, 12.99, 45, 4.5);
INSERT INTO books (book_id, title, author, genre, publication_year, price, stock_quantity, rating) VALUES (2, 'To Kill a Mockingbird', 'Harper Lee', 'Fiction', 1960, 14.99, 38, 4.8);
INSERT INTO books (book_id, title, author, genre, publication_year, price, stock_quantity, rating) VALUES (3, '1984', 'George Orwell', 'Science Fiction', 1949, 13.99, 52, 4.7);
INSERT INTO books (book_id, title, author, genre, publication_year, price, stock_quantity, rating) VALUES (4, 'Pride and Prejudice', 'Jane Austen', 'Romance', 1813, 11.99, 30, 4.6);
INSERT INTO books (book_id, title, author, genre, publication_year, price, stock_quantity, rating) VALUES (5, 'The Hobbit', 'J.R.R. Tolkien', 'Fantasy', 1937, 15.99, 25, 4.9);
INSERT INTO books (book_id, title, author, genre, publication_year, price, stock_quantity, rating) VALUES (6, 'Harry Potter', 'J.K. Rowling', 'Fantasy', 1997, 19.99, 60, 4.8);
INSERT INTO books (book_id, title, author, genre, publication_year, price, stock_quantity, rating) VALUES (7, 'The Catcher in the Rye', 'J.D. Salinger', 'Fiction', 1951, 12.49, 20, 4.2);
INSERT INTO books (book_id, title, author, genre, publication_year, price, stock_quantity, rating) VALUES (8, 'Animal Farm', 'George Orwell', 'Fiction', 1945, 10.99, 35, 4.5);
INSERT INTO books (book_id, title, author, genre, publication_year, price, stock_quantity, rating) VALUES (9, 'Brave New World', 'Aldous Huxley', 'Science Fiction', 1932, 13.49, 28, 4.3);
INSERT INTO books (book_id, title, author, genre, publication_year, price, stock_quantity, rating) VALUES (10, 'The Lord of the Rings', 'J.R.R. Tolkien', 'Fantasy', 1954, 25.99, 15, 4.9);
```

</details>

**Books Table:**
| book_id | title | author | genre | publication_year | price | stock_quantity | rating |
|---------|-------|--------|-------|------------------|-------|----------------|--------|
| 1 | The Great Gatsby | F. Scott Fitzgerald | Fiction | 1925 | 12.99 | 45 | 4.5 |
| 2 | To Kill a Mockingbird | Harper Lee | Fiction | 1960 | 14.99 | 38 | 4.8 |
| 3 | 1984 | George Orwell | Science Fiction | 1949 | 13.99 | 52 | 4.7 |
| 4 | Pride and Prejudice | Jane Austen | Romance | 1813 | 11.99 | 30 | 4.6 |
| 5 | The Hobbit | J.R.R. Tolkien | Fantasy | 1937 | 15.99 | 25 | 4.9 |
| 6 | Harry Potter | J.K. Rowling | Fantasy | 1997 | 19.99 | 60 | 4.8 |
| 7 | The Catcher in the Rye | J.D. Salinger | Fiction | 1951 | 12.49 | 20 | 4.2 |
| 8 | Animal Farm | George Orwell | Fiction | 1945 | 10.99 | 35 | 4.5 |
| 9 | Brave New World | Aldous Huxley | Science Fiction | 1932 | 13.49 | 28 | 4.3 |
| 10 | The Lord of the Rings | J.R.R. Tolkien | Fantasy | 1954 | 25.99 | 15 | 4.9 |

## Basic SELECT Statement

Most of the time you don't need all columns from a table. Select only what you need to make queries faster and results clearer.

### Example 2: Select Specific Columns

**SQL Statement:**
```sql
SELECT title, author, price 
FROM books;
```

**Result:**
| title | author | price |
|-------|--------|-------|
| The Great Gatsby | F. Scott Fitzgerald | 12.99 |
| To Kill a Mockingbird | Harper Lee | 14.99 |
| 1984 | George Orwell | 13.99 |
| Pride and Prejudice | Jane Austen | 11.99 |
| The Hobbit | J.R.R. Tolkien | 15.99 |
| Harry Potter | J.K. Rowling | 19.99 |
| The Catcher in the Rye | J.D. Salinger | 12.49 |
| Animal Farm | George Orwell | 10.99 |
| Brave New World | Aldous Huxley | 13.49 |
| The Lord of the Rings | J.R.R. Tolkien | 25.99 |

### Example 3: Select with Calculated Columns

**SQL Statement:**
```sql
SELECT title, 
       price, 
       price * 0.9 AS discounted_price,
       stock_quantity * price AS inventory_value
FROM books;
```

**Result:**
| title | price | discounted_price | inventory_value |
|-------|-------|------------------|-----------------|
| The Great Gatsby | 12.99 | 11.69 | 584.55 |
| To Kill a Mockingbird | 14.99 | 13.49 | 569.62 |
| 1984 | 13.99 | 12.59 | 727.48 |
| Pride and Prejudice | 11.99 | 10.79 | 359.70 |
| The Hobbit | 15.99 | 14.39 | 399.75 |
| ... | ... | ... | ... |

---

## WHERE Clause

The **WHERE clause** filters rows based on conditions - it lets you specify which rows you want to retrieve instead of getting all rows.

### Example 4: Simple WHERE Condition

**SQL Statement:**
```sql
SELECT title, author, publication_year
FROM books
WHERE publication_year > 1950;
```

**Result:**
| title | author | publication_year |
|-------|--------|------------------|
| To Kill a Mockingbird | Harper Lee | 1960 |
| The Catcher in the Rye | J.D. Salinger | 1951 |
| The Lord of the Rings | J.R.R. Tolkien | 1954 |
| Harry Potter | J.K. Rowling | 1997 |

### Example 5: Multiple WHERE Conditions (AND)

**SQL Statement:**
```sql
SELECT title, genre, price, rating
FROM books
WHERE genre = 'Fiction' AND rating > 4.5;
```

**Result:**
| title | genre | price | rating |
|-------|-------|-------|--------|
| To Kill a Mockingbird | Fiction | 14.99 | 4.8 |

### Example 8: IN Operator

**SQL Statement:**
```sql
SELECT title, author, genre
FROM books
WHERE author IN ('George Orwell', 'J.R.R. Tolkien');
```

**Result:**
| title | author | genre |
|-------|--------|-------|
| 1984 | George Orwell | Science Fiction |
| The Hobbit | J.R.R. Tolkien | Fantasy |
| Animal Farm | George Orwell | Fiction |
| The Lord of the Rings | J.R.R. Tolkien | Fantasy |

---

## ORDER BY Clause

The **ORDER BY clause** sorts the result set. Use `ASC` for ascending order (smallest to largest) or `DESC` for descending order (largest to smallest).

### Example 15: Sort by Multiple Columns

**SQL Statement:**
```sql
SELECT title, genre, rating
FROM books
ORDER BY genre ASC, rating DESC;
```

**Result:**
| title | genre | rating |
|-------|-------|--------|
| The Hobbit | Fantasy | 4.9 |
| The Lord of the Rings | Fantasy | 4.9 |
| Harry Potter | Fantasy | 4.8 |
| To Kill a Mockingbird | Fiction | 4.8 |
| The Great Gatsby | Fiction | 4.5 |
| Animal Farm | Fiction | 4.5 |
| The Catcher in the Rye | Fiction | 4.2 |
| Pride and Prejudice | Romance | 4.6 |
| 1984 | Science Fiction | 4.7 |
| Brave New World | Science Fiction | 4.3 |

---

## DISTINCT Keyword

The **DISTINCT keyword** removes duplicate values from results - you'll only see each unique value once.

### Example 17: Select Distinct Values

**SQL Statement:**
```sql
SELECT DISTINCT genre
FROM books
ORDER BY genre;
```

**Result:**
| genre |
|-------|
| Fantasy |
| Fiction |
| Romance |
| Science Fiction |

---

### Example 20: Pagination with OFFSET and FETCH

**Pagination** means breaking large result sets into smaller "pages" for easier viewing (like showing 10 search results per page). Use `OFFSET` to skip rows and `FETCH FIRST` to limit how many rows to return.

**SQL Statement:**
```sql
-- Page 1 (first 3 books)
SELECT title, price
FROM books
ORDER BY title
OFFSET 0 ROWS FETCH FIRST 3 ROWS ONLY;
```

**Result (Page 1):**
| title | price |
|-------|-------|
| 1984 | 13.99 |
| Animal Farm | 10.99 |
| Brave New World | 13.49 |

**Page 2 (next 3 books):**
```sql
SELECT title, price
FROM books
ORDER BY title
OFFSET 3 ROWS FETCH FIRST 3 ROWS ONLY;
```

## Combining WHERE, ORDER BY, and FETCH

### Example 21: Complex Query

**SQL Statement:**
```sql
SELECT title, author, genre, price, rating
FROM books
WHERE genre IN ('Fiction', 'Science Fiction')
  AND rating >= 4.5
  AND price < 15.00
ORDER BY rating DESC, price ASC
FETCH FIRST 3 ROWS ONLY;
```

**Result:**
| title | author | genre | price | rating |
|-------|--------|-------|-------|--------|
| To Kill a Mockingbird | Harper Lee | Fiction | 14.99 | 4.8 |
| 1984 | George Orwell | Science Fiction | 13.99 | 4.7 |
| The Great Gatsby | F. Scott Fitzgerald | Fiction | 12.99 | 4.5 |

