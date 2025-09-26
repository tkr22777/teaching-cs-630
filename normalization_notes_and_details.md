# Database Normalization Study Guide

## Key Terms (Glossary)

### Basic Database Concepts
- **Entity**: A real-world object or concept that can be distinctly identified (e.g., Customer, Product, Order)
- **Attribute**: A property or characteristic of an entity (e.g., CustomerName, ProductPrice)
- **Relationship**: An association between two or more entities (e.g., Customer places Order)

### Attribute Types
- **Simple Attribute**: An attribute that cannot be divided into smaller parts (e.g., Age, Gender)
- **Composite Attribute**: An attribute that can be divided into smaller sub-parts (e.g., Address = Street + City + State + ZIP)
- **Multi-valued Attribute**: An attribute that can have multiple values for a single entity (e.g., PhoneNumbers, Skills)
- **Single-valued Attribute**: An attribute that has only one value for each entity instance (e.g., DateOfBirth)
- **Derived Attribute**: An attribute whose value is calculated from other attributes (e.g., Age derived from DateOfBirth)
- **Stored Attribute**: An attribute whose value is directly stored in the database (e.g., DateOfBirth)
- **Key Attribute**: An attribute that uniquely identifies an entity instance (e.g., StudentID, SSN)
- **Non-key Attribute**: An attribute that does not uniquely identify an entity (e.g., Name, Address)
- **Required Attribute**: An attribute that must have a value (cannot be null)
- **Optional Attribute**: An attribute that may or may not have a value (can be null)
- **Atomic Attribute**: An attribute that contains indivisible values (required for First Normal Form)

### Fundamental Key Concepts
- **Primary Key**: A unique identifier for each entity instance that cannot be null
- **Foreign Key**: An attribute that references the primary key of another entity
- **Candidate Key**: A minimal set of attributes that can uniquely identify each row in a table
- **Super Key**: Any set of attributes that can uniquely identify a row (may contain extra attributes)

### Key Types and Classifications
- **Natural Key**: A key that has business meaning and naturally occurs in the data
- **Surrogate Key**: An artificial key created for the sole purpose of being a primary key
- **Composite Key**: A primary key consisting of two or more columns that together uniquely identify each row
- **Junction Table**: A table that resolves many-to-many relationships between entities

### Functional Dependency Concepts
- **Functional Dependency**: When one attribute determines the value of another attribute (A → B)
- **Partial Dependency**: A non-key attribute depends on only part of a composite primary key
- **Transitive Dependency**: A non-key attribute depends on another non-key attribute

### Normalization Theory
- **Normalization**: Process of organizing data to reduce redundancy and improve data integrity
- **First Normal Form (1NF)**: Eliminates repeating groups and ensures atomic values
- **Second Normal Form (2NF)**: Achieves 1NF and eliminates partial dependencies
- **Third Normal Form (3NF)**: Achieves 2NF and eliminates transitive dependencies

## Candidate Keys

### What is a Candidate Key?

A **candidate key** is a minimal set of attributes that can uniquely identify each row in a table. "Minimal" means that if you remove any attribute from the candidate key, it would no longer be able to uniquely identify rows.

### Key Properties of Candidate Keys:

1. **Uniqueness**: No two rows can have the same candidate key value
2. **Minimality**: Cannot remove any attribute and still maintain uniqueness
3. **Non-null**: No part of a candidate key can be null
4. **Immutability**: Values should not change over time

### Examples of Candidate Keys:

**STUDENT Table:**
| student_id | ssn | email | first_name | last_name |
|------------|-----|-------|------------|-----------|
| 12345 | 123-45-6789 | john@email.com | John | Smith |
| 12346 | 987-65-4321 | jane@email.com | Jane | Doe |

**Candidate Keys:**
- `student_id` (chosen as primary key)
- `ssn` (Social Security Number)
- `email` (assuming emails are unique)

**ENROLLMENT Table (Junction Table):**
| student_id | course_id | semester | grade |
|------------|-----------|----------|-------|
| 12345 | CS101 | Fall2024 | A |
| 12345 | CS102 | Fall2024 | B+ |
| 12346 | CS101 | Fall2024 | A- |

**Candidate Keys:**
- `(student_id, course_id, semester)` - A student can only be enrolled in a specific course once per semester
- This is a **composite candidate key** because it requires multiple attributes

### Super Key vs. Candidate Key:

**Super Key**: Any set of attributes that can uniquely identify a row
- `{student_id}` - Super key and candidate key
- `{student_id, first_name}` - Super key but NOT candidate key (extra attribute)
- `{ssn, email}` - Super key but NOT candidate key (not minimal)

**Candidate Key**: Minimal super key
- `{student_id}` - Minimal, cannot remove any attribute
- `{ssn}` - Minimal, cannot remove any attribute

## Key Types with Detailed Examples

### Example Dataset 1: EMPLOYEE Table

| emp_id | ssn | email | badge_number | first_name | last_name | dept_id | hire_date |
|--------|-----|-------|--------------|------------|-----------|---------|-----------|
| 1001 | 123-45-6789 | john.smith@company.com | B001 | John | Smith | D101 | 2020-01-15 |
| 1002 | 987-65-4321 | jane.doe@company.com | B002 | Jane | Doe | D102 | 2019-03-22 |
| 1003 | 456-78-9123 | bob.wilson@company.com | B003 | Bob | Wilson | D101 | 2021-07-10 |
| 1004 | 789-12-3456 | alice.brown@company.com | B004 | Alice | Brown | D103 | 2018-11-05 |

### Key Analysis for EMPLOYEE Table:

#### **Natural Keys:**
Natural keys have business meaning and occur naturally in the data.

1. **`ssn` (Social Security Number)**
   - **Why Natural**: Has real-world meaning, used by government and business
   - **Pros**: Unique across all systems, meaningful to users
   - **Cons**: Privacy concerns, may change in rare cases, format variations

2. **`email`**
   - **Why Natural**: Has business meaning, used for communication
   - **Pros**: Meaningful, user-friendly
   - **Cons**: Can change when employee changes name or department

3. **`badge_number`**
   - **Why Natural**: Physical badge used in workplace
   - **Pros**: Visible identifier, meaningful to security systems
   - **Cons**: May be reassigned, could change with new badge systems

#### **Surrogate Key:**
Artificial keys created solely for database purposes.

**`emp_id`**
- **Why Surrogate**: No business meaning, created just for database identification
- **Pros**: Never changes, optimal performance, no privacy issues
- **Cons**: Meaningless to users, requires joins to get meaningful data

#### **Candidate Keys:**
All minimal sets of attributes that can uniquely identify rows.

For EMPLOYEE table:
- `{emp_id}` - Surrogate candidate key
- `{ssn}` - Natural candidate key
- `{email}` - Natural candidate key  
- `{badge_number}` - Natural candidate key

**Primary Key Choice**: `emp_id` (chosen for stability and performance)

#### **Super Keys:**
Any combination of attributes that can uniquely identify rows.

Examples of super keys:
- `{emp_id}` - Also a candidate key (minimal)
- `{ssn}` - Also a candidate key (minimal)
- `{emp_id, first_name}` - Super key but NOT candidate (has extra attribute)
- `{ssn, email}` - Super key but NOT candidate (has extra attributes)
- `{emp_id, ssn, email, badge_number}` - Super key but NOT candidate (way too many attributes)

### Example Dataset 2: ORDER_DETAILS Table (Junction Table)

| order_id | product_id | customer_id | quantity | unit_price | discount | order_date |
|----------|------------|-------------|----------|------------|----------|------------|
| 2001 | P101 | C501 | 2 | 25.99 | 0.10 | 2024-01-15 |
| 2001 | P102 | C501 | 1 | 45.50 | 0.00 | 2024-01-15 |
| 2002 | P101 | C502 | 3 | 25.99 | 0.05 | 2024-01-16 |
| 2003 | P103 | C501 | 1 | 12.75 | 0.00 | 2024-01-17 |

### Key Analysis for ORDER_DETAILS Table:

#### **Natural Composite Key:**
**`(order_id, product_id)`**
- **Why Natural**: Has business meaning - represents "this specific product in this specific order"
- **Why Composite**: Neither order_id nor product_id alone is unique in this table
- **Business Rule**: Each product can appear only once per order
- **Minimal**: Cannot remove either attribute and maintain uniqueness

#### **Alternative Surrogate Key Approach:**

| detail_id | order_id | product_id | customer_id | quantity | unit_price |
|-----------|----------|------------|-------------|----------|------------|
| 5001 | 2001 | P101 | C501 | 2 | 25.99 |
| 5002 | 2001 | P102 | C501 | 1 | 45.50 |
| 5003 | 2002 | P101 | C502 | 3 | 25.99 |

**`detail_id`** - Surrogate key
- **Pros**: Single column primary key, simpler to reference
- **Cons**: Requires additional business rule to ensure (order_id, product_id) remains unique

#### **Super Keys for ORDER_DETAILS:**
- `{order_id, product_id}` - Candidate key (minimal)
- `{order_id, product_id, quantity}` - Super key (extra attribute)
- `{order_id, product_id, customer_id}` - Super key (redundant since customer_id is determined by order_id)

### Example Dataset 3: COURSE_ENROLLMENT Table

| student_id | course_id | semester | year | grade | credits | enrollment_date |
|------------|-----------|----------|------|-------|---------|-----------------|
| S001 | CS101 | Fall | 2024 | A | 3 | 2024-08-15 |
| S001 | MATH201 | Fall | 2024 | B+ | 4 | 2024-08-15 |
| S001 | CS101 | Spring | 2025 | NULL | 3 | 2025-01-10 |
| S002 | CS101 | Fall | 2024 | A- | 3 | 2024-08-20 |

### Key Analysis for COURSE_ENROLLMENT:

#### **Natural Composite Key:**
**`(student_id, course_id, semester, year)`**
- **Why Composite**: A student can take the same course multiple times in different semesters
- **Business Meaning**: Represents a specific student's enrollment in a specific course during a specific semester/year
- **All Parts Required**: Remove any component and uniqueness is lost

#### **Candidate Keys Analysis:**

**Testing Minimality:**
- `{student_id, course_id}` - **NOT unique** (S001 took CS101 twice)
- `{student_id, course_id, semester}` - **NOT unique** (could repeat in different years)
- `{student_id, course_id, year}` - **NOT unique** (could take fall and spring)
- `{student_id, course_id, semester, year}` - **UNIQUE and minimal**

#### **Super Keys Examples:**
- `{student_id, course_id, semester, year}` - Candidate key
- `{student_id, course_id, semester, year, grade}` - Super key (extra attribute)
- `{student_id, course_id, semester, year, credits}` - Super key (extra attribute)

### Example Dataset 4: PRODUCT Table with Multiple Natural Keys

| product_id | sku | upc | isbn | name | category | price |
|------------|-----|-----|------|------|----------|-------|
| 1 | BOOK-CS-001 | 123456789012 | 978-0134685991 | Clean Code | Books | 39.99 |
| 2 | ELEC-LAP-045 | 987654321098 | NULL | MacBook Pro | Electronics | 1999.99 |
| 3 | BOOK-DB-012 | 456789123456 | 978-0321884497 | Database Systems | Books | 275.00 |

### Key Analysis for PRODUCT Table:

#### **Natural Keys Available:**
1. **`sku` (Stock Keeping Unit)**
   - **Business Meaning**: Internal inventory tracking code
   - **Unique**: Yes, company assigns unique SKUs
   - **Stable**: Usually stable but may change with rebranding

2. **`upc` (Universal Product Code)**
   - **Business Meaning**: Industry standard barcode
   - **Unique**: Yes, globally unique
   - **Stable**: Very stable, rarely changes

3. **`isbn` (for books only)**
   - **Business Meaning**: International Standard Book Number
   - **Unique**: Yes, but only applies to books (NULL for other products)
   - **Limitation**: Cannot be primary key due to NULLs

#### **Surrogate Key:**
**`product_id`**
- **Chosen as Primary Key**: Most stable, no business meaning to change
- **Performance**: Integer, optimal for indexing and foreign keys

#### **Candidate Keys:**
- `{product_id}` - Surrogate candidate key (chosen as primary)
- `{sku}` - Natural candidate key
- `{upc}` - Natural candidate key
- `{isbn}` - Would be candidate key if not nullable

### Key Selection Guidelines:

#### **When to Use Natural Keys:**
- **Small, stable datasets** where business meaning is important
- **Reference/lookup tables** (countries, states, currencies)
- **When business rules require it** (regulatory compliance)

#### **When to Use Surrogate Keys:**
- **Large transaction tables** (orders, payments, logs)
- **When natural keys are compound** (multi-column keys)
- **When natural keys might change** (email addresses, names)
- **For optimal performance** (integer keys are fastest)

#### **When to Use Composite Keys:**
- **Junction tables** resolving many-to-many relationships
- **When business rules require uniqueness across multiple attributes**
- **Time-series data** where entity + time period must be unique
- **When no suitable single natural key exists**

### Real-World Key Selection Example:

**E-commerce Order System Design:**

**CUSTOMERS Table:** Use surrogate key
- **Primary Key**: `customer_id` (surrogate)
- **Alternative Keys**: `email` (natural candidate key, but can change)
- **Reasoning**: Emails change, addresses change, surrogate key provides stability

**PRODUCTS Table:** Use surrogate key with natural alternatives
- **Primary Key**: `product_id` (surrogate, chosen for stability)
- **Alternative Keys**: `sku` (natural), `upc` (natural)
- **Reasoning**: Multiple natural options exist, but surrogate provides best performance

**ORDERS Table:** Use surrogate key for performance
- **Primary Key**: `order_id` (surrogate)
- **Alternative Keys**: `order_number` (natural, visible to customers)
- **Reasoning**: High transaction volume, integer keys perform better

**ORDER_ITEMS Table:** Use natural composite key
- **Primary Key**: `(order_id, product_id)` (natural composite)
- **Reasoning**: Business rule naturally enforces uniqueness - each product appears only once per order

This design balances performance, stability, and business meaning appropriately for each table's purpose.

## Functional Dependencies

### What is a Functional Dependency?

A **functional dependency** (FD) exists when the value of one attribute (or set of attributes) determines the value of another attribute. Written as: A → B (read as "A determines B" or "B is functionally dependent on A").

### Examples of Functional Dependencies:

**EMPLOYEE Table:**
| emp_id | emp_name | dept_id | dept_name | salary | manager_id |
|--------|----------|---------|-----------|--------|------------|
| 101 | John Smith | D1 | Engineering | 75000 | 201 |
| 102 | Jane Doe | D1 | Engineering | 68000 | 201 |
| 103 | Bob Wilson | D2 | Marketing | 62000 | 202 |

**Functional Dependencies:**
1. `emp_id → emp_name` (Employee ID determines employee name)
2. `emp_id → dept_id` (Employee ID determines which department)
3. `emp_id → salary` (Employee ID determines salary)
4. `dept_id → dept_name` (Department ID determines department name)
5. `dept_id → manager_id` (Department ID determines who the manager is)

### Types of Dependencies:

#### 1. Partial Dependency

A **partial dependency** occurs when a non-key attribute depends on only part of a composite primary key.

**Example - COURSE_ENROLLMENT Table:**
| student_id | course_id | student_name | course_name | grade |
|------------|-----------|--------------|-------------|-------|
| 101 | CS101 | John Smith | Programming | A |
| 101 | MATH201 | John Smith | Calculus | B+ |
| 102 | CS101 | Jane Doe | Programming | A- |

**Composite Primary Key**: `(student_id, course_id)`

**Partial Dependencies:**
- `student_id → student_name` (student_name depends on only part of the key)
- `course_id → course_name` (course_name depends on only part of the key)

**Problem**: These partial dependencies violate 2NF and cause redundancy.

#### 2. Transitive Dependency

A **transitive dependency** occurs when a non-key attribute depends on another non-key attribute.

**Example - EMPLOYEE Table:**
| emp_id | emp_name | dept_id | dept_name | dept_location |
|--------|----------|---------|-----------|---------------|
| 101 | John Smith | D1 | Engineering | Building A |
| 102 | Jane Doe | D1 | Engineering | Building A |
| 103 | Bob Wilson | D2 | Marketing | Building B |

**Primary Key**: `emp_id`

**Transitive Dependency:**
- `emp_id → dept_id` (direct dependency)
- `dept_id → dept_name` (direct dependency)
- `dept_id → dept_location` (direct dependency)
- `emp_id → dept_name` (transitive: through dept_id)
- `emp_id → dept_location` (transitive: through dept_id)

**Problem**: Changes to department information require updates in multiple rows.

## Normalization with Example

### Complete Normalization Example: Student Course Registration

**Unnormalized Data:**
| StudentID | StudentName | CourseList | Instructor | Department |
|-----------|-------------|------------|------------|------------|
| 001 | John Smith | CS101, CS102, MATH201 | Dr. Johnson, Dr. Lee, Dr. Brown | Computer Science, Computer Science, Mathematics |
| 002 | Jane Doe | CS101, PHYS101 | Dr. Johnson, Dr. White | Computer Science, Physics |

**Problems:**
- Repeating groups (multiple courses per student)
- Non-atomic values (comma-separated lists)
- Update anomalies (changing instructor name requires multiple updates)
- Insertion anomalies (can't add new course without student)
- Deletion anomalies (removing last student removes course information)

### Step 1: First Normal Form (1NF)
**Rule**: Eliminate repeating groups and ensure all attribute values are atomic.

**1NF Result:**
| StudentID | StudentName | CourseID | Instructor | Department |
|-----------|-------------|----------|------------|------------|
| 001 | John Smith | CS101 | Dr. Johnson | Computer Science |
| 001 | John Smith | CS102 | Dr. Lee | Computer Science |
| 001 | John Smith | MATH201 | Dr. Brown | Mathematics |
| 002 | Jane Doe | CS101 | Dr. Johnson | Computer Science |
| 002 | Jane Doe | PHYS101 | Dr. White | Physics |

**Composite Primary Key**: `(StudentID, CourseID)`

### Step 2: Second Normal Form (2NF)
**Rule**: Must be in 1NF AND eliminate partial dependencies.

**Partial Dependencies Identified:**
- `StudentID → StudentName` (StudentName depends only on StudentID, not CourseID)
- `CourseID → Instructor` (Instructor depends only on CourseID, not StudentID)
- `CourseID → Department` (Department depends only on CourseID, not StudentID)

**2NF Result:**

**STUDENT Table:**
| StudentID | StudentName |
|-----------|-------------|
| 001 | John Smith |
| 002 | Jane Doe |

**COURSE Table:**
| CourseID | Instructor | Department |
|----------|------------|------------|
| CS101 | Dr. Johnson | Computer Science |
| CS102 | Dr. Lee | Computer Science |
| MATH201 | Dr. Brown | Mathematics |
| PHYS101 | Dr. White | Physics |

**ENROLLMENT Table:**
| StudentID | CourseID |
|-----------|----------|
| 001 | CS101 |
| 001 | CS102 |
| 001 | MATH201 |
| 002 | CS101 |
| 002 | PHYS101 |

### Step 3: Third Normal Form (3NF)
**Rule**: Must be in 2NF AND eliminate transitive dependencies.

**Transitive Dependency Identified:**
- In COURSE table: `CourseID → Instructor` and `Instructor → Department`
- This creates: `CourseID → Department` (transitive through Instructor)

**3NF Result:**

**STUDENT Table:** (unchanged)
| StudentID | StudentName |
|-----------|-------------|
| 001 | John Smith |
| 002 | Jane Doe |

**INSTRUCTOR Table:**
| InstructorID | InstructorName | Department |
|--------------|----------------|------------|
| I001 | Dr. Johnson | Computer Science |
| I002 | Dr. Lee | Computer Science |
| I003 | Dr. Brown | Mathematics |
| I004 | Dr. White | Physics |

**COURSE Table:**
| CourseID | InstructorID |
|----------|--------------|
| CS101 | I001 |
| CS102 | I002 |
| MATH201 | I003 |
| PHYS101 | I004 |

**ENROLLMENT Table:** (unchanged)
| StudentID | CourseID |
|-----------|----------|
| 001 | CS101 |
| 001 | CS102 |
| 001 | MATH201 |
| 002 | CS101 |
| 002 | PHYS101 |

### Final Result Analysis:

**Benefits of 3NF Design:**
1. **No Redundancy**: Each piece of information is stored only once
2. **No Update Anomalies**: Changing an instructor's department requires only one update
3. **No Insertion Anomalies**: Can add new instructors without needing students
4. **No Deletion Anomalies**: Removing students doesn't lose instructor information
5. **Data Integrity**: Referential integrity maintained through foreign keys

**Functional Dependencies in Final Design:**
- `StudentID → StudentName`
- `InstructorID → InstructorName, Department`
- `CourseID → InstructorID`
- `(StudentID, CourseID) → ∅` (Enrollment relationship)