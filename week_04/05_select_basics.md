# SELECT Basics: Retrieving Data

## Overview

The SELECT statement is the most commonly used SQL command for retrieving data from database tables. This guide covers basic SELECT operations using standard SQL.

## Sample Data

<details>
<summary>Click to expand: Database setup script</summary>

```sql
CREATE TABLE books (
    book_id INTEGER PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    author VARCHAR(100) NOT NULL,
    genre VARCHAR(50),
    publication_year INTEGER,
    price NUMERIC(10, 2),
    stock_quantity INTEGER,
    rating NUMERIC(3, 2)
);

INSERT INTO books (title, author, genre, publication_year, price, stock_quantity, rating) VALUES
('The Great Gatsby', 'F. Scott Fitzgerald', 'Fiction', 1925, 12.99, 45, 4.5),
('To Kill a Mockingbird', 'Harper Lee', 'Fiction', 1960, 14.99, 38, 4.8),
('1984', 'George Orwell', 'Science Fiction', 1949, 13.99, 52, 4.7),
('Pride and Prejudice', 'Jane Austen', 'Romance', 1813, 11.99, 30, 4.6),
('The Hobbit', 'J.R.R. Tolkien', 'Fantasy', 1937, 15.99, 25, 4.9),
('Harry Potter', 'J.K. Rowling', 'Fantasy', 1997, 19.99, 60, 4.8),
('The Catcher in the Rye', 'J.D. Salinger', 'Fiction', 1951, 12.49, 20, 4.2),
('Animal Farm', 'George Orwell', 'Fiction', 1945, 10.99, 35, 4.5),
('Brave New World', 'Aldous Huxley', 'Science Fiction', 1932, 13.49, 28, 4.3),
('The Lord of the Rings', 'J.R.R. Tolkien', 'Fantasy', 1954, 25.99, 15, 4.9);
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

### Example 1: Select All Columns

**SQL Statement:**
```sql
SELECT * FROM books;
```

**Result:** Returns all 10 rows with all columns (shown above).

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

## WHERE Clause

The WHERE clause filters rows based on conditions.

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

### Example 6: Multiple WHERE Conditions (OR)

**SQL Statement:**
```sql
SELECT title, genre, price
FROM books
WHERE genre = 'Fantasy' OR genre = 'Science Fiction';
```

**Result:**
| title | genre | price |
|-------|-------|-------|
| 1984 | Science Fiction | 13.99 |
| The Hobbit | Fantasy | 15.99 |
| Harry Potter | Fantasy | 19.99 |
| Brave New World | Science Fiction | 13.49 |
| The Lord of the Rings | Fantasy | 25.99 |

### Example 7: BETWEEN Operator

**SQL Statement:**
```sql
SELECT title, publication_year, price
FROM books
WHERE publication_year BETWEEN 1920 AND 1950;
```

**Result:**
| title | publication_year | price |
|-------|------------------|-------|
| The Great Gatsby | 1925 | 12.99 |
| The Hobbit | 1937 | 15.99 |
| 1984 | 1949 | 13.99 |
| Animal Farm | 1945 | 10.99 |
| Brave New World | 1932 | 13.49 |

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

### Example 9: LIKE Operator for Pattern Matching

**SQL Statement:**
```sql
-- Books with "The" in the title
SELECT title, author
FROM books
WHERE title LIKE 'The%';
```

**Result:**
| title | author |
|-------|--------|
| The Great Gatsby | F. Scott Fitzgerald |
| The Hobbit | J.R.R. Tolkien |
| The Catcher in the Rye | J.D. Salinger |
| The Lord of the Rings | J.R.R. Tolkien |

**LIKE Pattern Examples:**
```sql
-- Starts with "The"
WHERE title LIKE 'The%'

-- Ends with "Farm"
WHERE title LIKE '%Farm'

-- Contains "and"
WHERE title LIKE '%and%'

-- Second character is 'h'
WHERE title LIKE '_h%'
```

### Example 10: Case-Insensitive Pattern Matching (ILIKE)

**SQL Statement:**
```sql
SELECT title, author
FROM books
WHERE UPPER(title) LIKE UPPER('%harry%');  -- Case-insensitive using standard SQL
```

**Result:**
| title | author |
|-------|--------|
| Harry Potter | J.K. Rowling |

### Example 11: IS NULL and IS NOT NULL

**Setup:**
```sql
-- Add a book with missing information
INSERT INTO books (title, author, genre, price) 
VALUES ('Unknown Book', 'Unknown Author', NULL, 9.99);
```

**SQL Statement:**
```sql
SELECT title, author, genre
FROM books
WHERE genre IS NULL;
```

**Result:**
| title | author | genre |
|-------|--------|-------|
| Unknown Book | Unknown Author | NULL |

**SQL Statement:**
```sql
SELECT title, genre
FROM books
WHERE genre IS NOT NULL
  AND stock_quantity > 40;
```

**Result:**
| title | genre |
|-------|-------|
| The Great Gatsby | Fiction |
| 1984 | Science Fiction |
| Harry Potter | Fantasy |

### Example 12: Comparison Operators

**SQL Statement:**
```sql
SELECT title, price, stock_quantity
FROM books
WHERE price >= 15.00 AND stock_quantity < 30;
```

**Result:**
| title | price | stock_quantity |
|-------|-------|----------------|
| The Hobbit | 15.99 | 25 |
| The Lord of the Rings | 25.99 | 15 |

**All Comparison Operators:**
- `=` Equal to
- `<>` or `!=` Not equal to
- `>` Greater than
- `<` Less than
- `>=` Greater than or equal
- `<=` Less than or equal

## ORDER BY Clause

The ORDER BY clause sorts the result set.

### Example 13: Sort Ascending (Default)

**SQL Statement:**
```sql
SELECT title, price
FROM books
ORDER BY price;
```

**Result:**
| title | price |
|-------|-------|
| Animal Farm | 10.99 |
| Pride and Prejudice | 11.99 |
| The Catcher in the Rye | 12.49 |
| The Great Gatsby | 12.99 |
| Brave New World | 13.49 |
| 1984 | 13.99 |
| To Kill a Mockingbird | 14.99 |
| The Hobbit | 15.99 |
| Harry Potter | 19.99 |
| The Lord of the Rings | 25.99 |

### Example 14: Sort Descending

**SQL Statement:**
```sql
SELECT title, rating
FROM books
ORDER BY rating DESC;
```

**Result:**
| title | rating |
|-------|--------|
| The Hobbit | 4.9 |
| The Lord of the Rings | 4.9 |
| To Kill a Mockingbird | 4.8 |
| Harry Potter | 4.8 |
| 1984 | 4.7 |
| Pride and Prejudice | 4.6 |
| The Great Gatsby | 4.5 |
| Animal Farm | 4.5 |
| Brave New World | 4.3 |
| The Catcher in the Rye | 4.2 |

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

### Example 16: Order by Calculated Column

**SQL Statement:**
```sql
SELECT title, price, stock_quantity,
       price * stock_quantity AS total_value
FROM books
ORDER BY total_value DESC;
```

**Result:**
| title | price | stock_quantity | total_value |
|-------|-------|----------------|-------------|
| Harry Potter | 19.99 | 60 | 1199.40 |
| 1984 | 13.99 | 52 | 727.48 |
| The Great Gatsby | 12.99 | 45 | 584.55 |
| To Kill a Mockingbird | 14.99 | 38 | 569.62 |
| ... | ... | ... | ... |

## DISTINCT Keyword

DISTINCT removes duplicate values from results.

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

### Example 18: DISTINCT on Multiple Columns

**SQL Statement:**
```sql
SELECT DISTINCT author, genre
FROM books
ORDER BY author, genre;
```

**Result:**
| author | genre |
|--------|-------|
| Aldous Huxley | Science Fiction |
| F. Scott Fitzgerald | Fiction |
| George Orwell | Fiction |
| George Orwell | Science Fiction |
| Harper Lee | Fiction |
| J.D. Salinger | Fiction |
| J.K. Rowling | Fantasy |
| J.R.R. Tolkien | Fantasy |
| Jane Austen | Romance |

## LIMIT and OFFSET

Control the number of rows returned.

### Example 19: LIMIT

**SQL Statement:**
```sql
SELECT title, price
FROM books
ORDER BY price DESC
LIMIT 5;
```

**Result:**
| title | price |
|-------|-------|
| The Lord of the Rings | 25.99 |
| Harry Potter | 19.99 |
| The Hobbit | 15.99 |
| To Kill a Mockingbird | 14.99 |
| 1984 | 13.99 |

### Example 20: LIMIT with OFFSET (Pagination)

**SQL Statement:**
```sql
-- Page 1 (first 3 books)
SELECT title, price
FROM books
ORDER BY title
LIMIT 3 OFFSET 0;
```

**Result (Page 1):**
| title | price |
|-------|-------|
| 1984 | 13.99 |
| Animal Farm | 10.99 |
| Brave New World | 13.49 |

**SQL Statement:**
```sql
-- Page 2 (next 3 books)
SELECT title, price
FROM books
ORDER BY title
LIMIT 3 OFFSET 3;
```

**Result (Page 2):**
| title | price |
|-------|-------|
| Harry Potter | 19.99 |
| Pride and Prejudice | 11.99 |
| The Catcher in the Rye | 12.49 |

## Combining WHERE, ORDER BY, and LIMIT

### Example 21: Complex Query

**SQL Statement:**
```sql
SELECT title, author, genre, price, rating
FROM books
WHERE genre IN ('Fiction', 'Science Fiction')
  AND rating >= 4.5
  AND price < 15.00
ORDER BY rating DESC, price ASC
LIMIT 3;
```

**Result:**
| title | author | genre | price | rating |
|-------|--------|-------|-------|--------|
| To Kill a Mockingbird | Harper Lee | Fiction | 14.99 | 4.8 |
| 1984 | George Orwell | Science Fiction | 13.99 | 4.7 |
| The Great Gatsby | F. Scott Fitzgerald | Fiction | 12.99 | 4.5 |

