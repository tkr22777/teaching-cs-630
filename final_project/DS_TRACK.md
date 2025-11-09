# Data Science Track

Database design with large-scale data population and analysis.

---

## Requirements

### 1. Database Design

- ER Diagram (entities, relationships, cardinalities, PKs/FKs)
- Normalized to 3NF minimum

### 2. GitHub Repository

- Public repository with incremental commits (minimum 10)

### 3. Database

- **Database**: Oracle (use provided credentials)

### 4. Data Population Scripts

Create scripts to populate database with large datasets:
- Python, SQL, or other scripting languages
- Generate or import substantial data volumes for analysis
- Scripts must be reproducible and documented

### 5. SQL Implementation

Required files:
- `sql/schema.sql` - DDL with CREATE TABLE statements, PKs/FKs/constraints
- `sql/sample_data.sql` - Initial INSERT statements
- `sql/reset.sql` - DROP/TRUNCATE statements to reset database
- `scripts/` - Data population scripts (Python/SQL/other) for large datasets
- `docs/ANALYSIS.md` containing:
  - Data population methodology
  - Analytics: JOINs, GROUP BY, window functions, CTEs
  - Statistical analysis queries
  - Performance considerations for large datasets

### 6. Initial Focus

Start with:
1. ERD design
2. DDL design (CREATE TABLE statements)
3. Sample INSERT queries

**Note:** Data population scripts and visualization tools will be discussed in future weeks.

---

## Evaluation

1. **Database Design (25%)** - ER diagram, normalization, data isolation
2. **Data Population (25%)** - Script quality, data volume, reproducibility
3. **SQL & Analysis (30%)** - Query complexity, analytical depth
4. **Documentation (20%)** - Clear explanations, methodology

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
- Data population scripts in `scripts/` folder
- `docs/ANALYSIS.md` with methodology and queries

**December 10th - Project Demo (In Class)**
- Live demonstration of data population scripts
- Show analytical queries and results
- Explain design decisions and data analysis approach

