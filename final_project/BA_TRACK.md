# Business Analytics Track

Database design and analytics queries. **No backend required.**

---

## Important Dates

**November 26th - ERD Design Presentation**
- Submit ER diagram
- Present and describe design (5 minutes)
- Explain design choices and how your design is important/required for your project scope
- Discuss 1-2 key design decisions

**December 9th (EOD) - Final Submission**
- GitHub repository URL
- `docs/DATABASE_DESIGN.md` with ER diagram
- SQL scripts in `sql/` folder
- `docs/ANALYTICS_QUERIES.md` with demonstrations

**December 10th - Project Demo (In Class)**
- Live demonstration of SQL scripts and queries
- Explain analytical insights and design decisions

---

## Requirements

### 1. Database Design

- ER Diagram (entities, relationships, cardinalities, PKs/FKs)
- Normalized to 3NF minimum

### 2. GitHub Repository

- Public repository with incremental commits (minimum 10)

### 3. Database

- **Database**: Oracle (use provided credentials)

### 4. SQL Implementation

Required files:
- `sql/schema.sql` - DDL with CREATE TABLE statements, PKs/FKs/constraints
- `sql/sample_data.sql` - INSERT statements with representative data
- `sql/reset.sql` - DROP/TRUNCATE statements to reset database
- `docs/ANALYTICS_QUERIES.md` containing:
  - Core operations (INSERT/UPDATE/DELETE)
  - Analytics: JOINs, GROUP BY, window functions, CTEs
  - Views for reusable analytics
  - Indexing notes and one EXPLAIN PLAN example

### 5. Initial Focus

Start with:
1. ERD design
2. DDL design (CREATE TABLE statements)
3. Sample INSERT queries

**Note:** Visualization tools and additional requirements will be discussed in future weeks.

---

## Evaluation

1. **Database Design (30%)** - ER diagram, normalization, data isolation
2. **SQL Implementation (40%)** - Query complexity, analytics depth
3. **Data Quality (15%)** - Representative sample data
4. **Documentation (15%)** - Clear query explanations

