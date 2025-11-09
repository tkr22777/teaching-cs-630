# Data Science Track

Database design with large-scale data population and analysis.

**See main file for general requirements, important dates, and project options.**

---

## Track-Specific Requirements

### 1. Schema Design

- Implement the **5 core features** from your chosen project
- Add **10 data science enhancements** to support analysis:
  - Time-series data tables
  - Event/behavioral tracking tables
  - Feature engineering tables
  - Aggregated metrics tables
  - Historical snapshot tables
  - Statistical summary tables

### 2. Data Population Scripts

Create scripts to populate database with large datasets:
- Python, SQL, or other scripting languages
- Generate or import substantial data volumes (design for scale)
- Scripts must be reproducible and documented
- Include realistic data distributions and patterns

### 3. Analysis Documentation

Required file `docs/ANALYSIS.md` containing:

**Data Population Methodology:**
- Description of data generation approach
- Data volume and distribution strategy
- Data quality considerations

**Business Questions & Queries:**
- Minimum **15 analytical queries** answering business/domain questions
- Each query must include:
  - Business question being answered
  - SQL query with comments
  - Performance notes for large datasets

**Query Requirements:**
- Demonstrate: JOINs, GROUP BY, window functions, CTEs, subqueries
- Include statistical analysis queries (percentiles, distributions, correlations)
- Performance considerations and optimizations
- One EXPLAIN PLAN analysis with optimization discussion

**Note:** Data population scripts and visualization tools will be discussed in future weeks.

---

## Evaluation

1. **Database Design (20%)** - ER diagram, normalization, 5 core features + 10 data science enhancements
2. **Data Population (25%)** - Script quality, data volume, realism, reproducibility
3. **Business Questions (30%)** - Quality and relevance of 15 analytical queries with performance considerations
4. **SQL & Analysis (15%)** - Statistical queries, advanced SQL features, optimization
5. **Documentation (10%)** - Clear methodology, insights, and business context

