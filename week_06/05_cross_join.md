# CROSS JOIN

## Overview

A **CROSS JOIN** produces the Cartesian product of two tables, meaning every row from the first table is combined with every row from the second table. If table A has M rows and table B has N rows, the result will have M × N rows.

CROSS JOIN is unique because it doesn't require a join condition - it simply combines all possible pairs.

## Syntax

**Explicit syntax (preferred):**
```sql
SELECT columns
FROM table1
CROSS JOIN table2;
```

**Best Practice:** Use explicit `CROSS JOIN` syntax for clarity.

## How CROSS JOIN Works

**Visual Representation:**

```
Table A (3 rows)    Table B (2 rows)    Result (A CROSS JOIN B)
┌────┬────┐          ┌────┬────┐        ┌────┬────┬────┬────┐
│ ID │Val │          │ ID │Val │        │A_ID│A_Val│B_ID│B_Val│
├────┼────┤          ├────┼────┤        ├────┼────┼────┼────┤
│  1 │ A1 │─────┬───►│  X │ BX │───────►│  1 │ A1 │ X  │ BX │
│  2 │ A2 │─────┤ └─►│  Y │ BY │───┬───►│  1 │ A1 │ Y  │ BY │
│  3 │ A3 │─────┤                 ├───►│  2 │ A2 │ X  │ BX │
└────┴────┘     │                 ├───►│  2 │ A2 │ Y  │ BY │
                │                 ├───►│  3 │ A3 │ X  │ BX │
                └─────────────────┴───►│  3 │ A3 │ Y  │ BY │
                                       └────┴────┴────┴────┘
                                       6 rows (3 × 2 = 6)
```

**Key Point:** Every row from table A is paired with every row from table B.

## Basic CROSS JOIN Example

### Example 1: Simple Cartesian Product

**Query:** Generate all possible student-course combinations.

```sql
SELECT s.student_id,
       s.first_name,
       s.last_name,
       c.course_id,
       c.course_name
FROM students s
CROSS JOIN courses c
ORDER BY s.student_id, c.course_id
LIMIT 10;
```

**Result (First 10 rows of 30 total):**
| student_id | first_name | last_name | course_id | course_name |
|------------|------------|-----------|-----------|-------------|
| 1 | John | Smith | CS101 | Introduction to Programming |
| 1 | John | Smith | CS201 | Data Structures |
| 1 | John | Smith | CS301 | Database Systems |
| 1 | John | Smith | ENG101 | English Composition |
| 1 | John | Smith | MATH101 | Calculus I |
| 1 | John | Smith | PHYS101 | Physics I |
| 2 | Jane | Doe | CS101 | Introduction to Programming |
| 2 | Jane | Doe | CS201 | Data Structures |
| 2 | Jane | Doe | CS301 | Database Systems |
| 2 | Jane | Doe | ENG101 | English Composition |

**Explanation:**
- 5 students × 6 courses = 30 total rows
- Each student paired with each course
- No relationship required - all combinations generated

###

**Query:** How many possible student-course pairings exist?

```sql
SELECT COUNT(*) AS total_combinations,
       (SELECT COUNT(*) FROM students) AS student_count,
       (SELECT COUNT(*) FROM courses) AS course_count
FROM students
CROSS JOIN courses;
```

**Result:**
| total_combinations | student_count | course_count |
|--------------------|---------------|--------------|
| 30 | 5 | 6 |

**Explanation:** 5 × 6 = 30 possible combinations

## CROSS JOIN with WHERE Clause (filtering the product)

### Example 2: Filter Combinations

**Query:** Generate all CS student and CS course combinations.

```sql
SELECT s.first_name,
       s.last_name,
       s.major,
       c.course_id,
       c.course_name,
       c.department
FROM students s
CROSS JOIN courses c
WHERE s.major = 'Computer Science'
  AND c.department = 'Computer Science'
ORDER BY s.last_name, c.course_id;
```

**Result:**
| first_name | last_name | major | course_id | course_name | department |
|------------|-----------|-------|-----------|-------------|------------|
| John | Smith | Computer Science | CS101 | Introduction to Programming | Computer Science |
| John | Smith | Computer Science | CS201 | Data Structures | Computer Science |
| John | Smith | Computer Science | CS301 | Database Systems | Computer Science |
| Bob | Wilson | Computer Science | CS101 | Introduction to Programming | Computer Science |
| Bob | Wilson | Computer Science | CS201 | Data Structures | Computer Science |
| Bob | Wilson | Computer Science | CS301 | Database Systems | Computer Science |

**Explanation:**
- 2 CS students × 3 CS courses = 6 rows
- WHERE clause filters after Cartesian product is generated
- Useful for seeing all possible valid pairings

### Example 3: Finding Potential Enrollments

**Query:** Show CS students and courses they're NOT currently enrolled in.

```sql
SELECT s.student_id,
       s.first_name || ' ' || s.last_name AS student_name,
       c.course_id,
       c.course_name
FROM students s
CROSS JOIN courses c
WHERE s.major = 'Computer Science'
  AND c.department = 'Computer Science'
  AND NOT EXISTS (
      SELECT 1
      FROM enrollments e
      WHERE e.student_id = s.student_id
        AND e.course_id = c.course_id
  )
ORDER BY s.student_id, c.course_id;
```

**Result:**
| student_id | student_name | course_id | course_name |
|------------|--------------|-----------|-------------|
| 3 | Bob Wilson | CS301 | Database Systems |

**Explanation:**
- Generates all possible CS student-course pairs
- Filters out existing enrollments
- Shows courses Bob Wilson could still enroll in
- John Smith is enrolled in all 3 CS courses (doesn't appear)

## Practical Use Case

### Use Case: Generating Date/Time Combinations

**Setup:**
```sql
CREATE TABLE dates (date_value DATE);
CREATE TABLE time_slots (slot_time TIMESTAMP, slot_name VARCHAR2(50));

INSERT INTO dates VALUES 
    ('2024-10-21'), ('2024-10-22'), ('2024-10-23');

INSERT INTO time_slots VALUES 
    ('09:00', 'Morning'),
    ('14:00', 'Afternoon'),
    ('19:00', 'Evening');
```

**Query:** Generate all possible appointment slots.

```sql
SELECT d.date_value,
       t.slot_time,
       t.slot_name,
       d.date_value + t.slot_time AS appointment_datetime
FROM dates d
CROSS JOIN time_slots t
ORDER BY d.date_value, t.slot_time;
```

**Result:**
| date_value | slot_time | slot_name | appointment_datetime |
|------------|-----------|-----------|----------------------|
| 2024-10-21 | 09:00:00 | Morning | 2024-10-21 09:00:00 |
| 2024-10-21 | 14:00:00 | Afternoon | 2024-10-21 14:00:00 |
| 2024-10-21 | 19:00:00 | Evening | 2024-10-21 19:00:00 |
| 2024-10-22 | 09:00:00 | Morning | 2024-10-22 09:00:00 |
| 2024-10-22 | 14:00:00 | Afternoon | 2024-10-22 14:00:00 |
| 2024-10-22 | 19:00:00 | Evening | 2024-10-22 19:00:00 |
| 2024-10-23 | 09:00:00 | Morning | 2024-10-23 09:00:00 |
| 2024-10-23 | 14:00:00 | Afternoon | 2024-10-23 14:00:00 |
| 2024-10-23 | 19:00:00 | Evening | 2024-10-23 19:00:00 |

**Explanation:** Creates 9 appointment slots (3 days × 3 time slots)

###

**Setup:**
```sql
CREATE TABLE products (product_id INTEGER PRIMARY KEY, product_name VARCHAR2(50));
CREATE TABLE sizes (size_code VARCHAR2(5), size_name VARCHAR2(20));
CREATE TABLE colors (color_code VARCHAR2(10), color_name VARCHAR2(20));

INSERT INTO products (product_name) VALUES ('T-Shirt'), ('Hoodie');
INSERT INTO sizes VALUES ('S', 'Small'), ('M', 'Medium'), ('L', 'Large');
INSERT INTO colors VALUES ('RED', 'Red'), ('BLUE', 'Blue'), ('BLACK', 'Black');
```

**Query:** Generate all product variants.

```sql
SELECT p.product_name,
       s.size_name,
       c.color_name,
       p.product_name || ' - ' || s.size_name || ' - ' || c.color_name AS sku_description
FROM products p
CROSS JOIN sizes s
CROSS JOIN colors c
ORDER BY p.product_name, s.size_code, c.color_code;
```

**Result (18 total rows):**
| product_name | size_name | color_name | sku_description |
|--------------|-----------|------------|-----------------|
| Hoodie | Small | Black | Hoodie - Small - Black |
| Hoodie | Small | Blue | Hoodie - Small - Blue |
| Hoodie | Small | Red | Hoodie - Small - Red |
| Hoodie | Medium | Black | Hoodie - Medium - Black |
| ... | ... | ... | ... |
| T-Shirt | Large | Red | T-Shirt - Large - Red |

**Explanation:** 2 products × 3 sizes × 3 colors = 18 variants

###

**Query:** Create test enrollment data for all possible student-course pairs.

```sql
CREATE TABLE test_enrollments AS
SELECT s.student_id,
       c.course_id,
       'Spring 2025' AS semester,
       NULL AS grade
FROM students s
CROSS JOIN courses c
WHERE s.major = c.department;

-- Check results
SELECT * FROM test_enrollments
ORDER BY student_id, course_id
LIMIT 5;
```

**Result:**
| student_id | course_id | semester | grade |
|------------|-----------|----------|-------|
| 1 | CS101 | Spring 2025 | NULL |
| 1 | CS201 | Spring 2025 | NULL |
| 1 | CS301 | Spring 2025 | NULL |
| 2 | MATH101 | Spring 2025 | NULL |
| 3 | CS101 | Spring 2025 | NULL |

**Explanation:** Creates test enrollments matching students with their department courses

###

**Setup:**
```sql
CREATE TABLE products_simple (product VARCHAR2(50), base_price NUMBER(10,2));
CREATE TABLE discount_tiers (tier VARCHAR2(20), discount_pct NUMBER(5,2));

INSERT INTO products_simple VALUES 
    ('Laptop', 999.99),
    ('Mouse', 25.50),
    ('Keyboard', 75.00);

INSERT INTO discount_tiers VALUES 
    ('Regular', 0),
    ('Student', 10),
    ('Corporate', 15);
```

**Query:** Generate price matrix for all products and discount tiers.

```sql
SELECT p.product,
       p.base_price,
       d.tier,
       d.discount_pct,
       ROUND(p.base_price * (1 - d.discount_pct / 100), 2) AS final_price,
       ROUND(p.base_price * d.discount_pct / 100, 2) AS savings
FROM products_simple p
CROSS JOIN discount_tiers d
ORDER BY p.product, d.discount_pct;
```

**Result:**
| product | base_price | tier | discount_pct | final_price | savings |
|---------|------------|------|--------------|-------------|---------|
| Keyboard | 75.00 | Regular | 0 | 75.00 | 0.00 |
| Keyboard | 75.00 | Student | 10 | 67.50 | 7.50 |
| Keyboard | 75.00 | Corporate | 15 | 63.75 | 11.25 |
| Laptop | 999.99 | Regular | 0 | 999.99 | 0.00 |
| Laptop | 999.99 | Student | 10 | 899.99 | 100.00 |
| Laptop | 999.99 | Corporate | 15 | 849.99 | 150.00 |
| Mouse | 25.50 | Regular | 0 | 25.50 | 0.00 |
| Mouse | 25.50 | Student | 10 | 22.95 | 2.55 |
| Mouse | 25.50 | Corporate | 15 | 21.68 | 3.83 |

**Explanation:** Creates pricing table for all product/discount combinations

## When to Use CROSS JOIN

### ✅ Use CROSS JOIN When:

1. **Generating all combinations**
   - Product variants (size × color)
   - Schedule slots (dates × times)
   - Pricing tiers

2. **Creating test data**
   - All possible pairings for testing
   - Generating sample datasets

3. **Calendar/scheduling tables**
   - Days × time slots
   - Employees × shifts

4. **Matrix/grid generation**
   - Price matrices
   - Comparison tables

5. **Finding missing combinations**
   - All possible vs. actual (using NOT EXISTS)

### ❌ Avoid CROSS JOIN When:

1. **Tables have a relationship**
   - Use appropriate JOIN (INNER, LEFT, etc.)
   - CROSS JOIN ignores relationships

2. **Large tables**
   - 10,000 rows × 10,000 rows = 100,000,000 rows!
   - Can cause performance issues or crashes

3. **You forget the join condition**
   - Accidental Cartesian product is a common mistake

