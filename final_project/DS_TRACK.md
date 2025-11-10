# Data Science Track

Database design with large-scale data population and analysis.

**See main file for general requirements, important dates, and project options.**

---

## Deliverables
- `sql/schema.sql` (5 core + 10 data science enhancements)
- `sql/sample_data.sql`, `sql/reset.sql`
- `scripts/` (reproducible data population)
- `docs/ANALYSIS.md` (15 queries with performance notes, 1 EXPLAIN PLAN)

---

## Track-Specific Requirements

### 1. Data Science Schema Design

- **Goal:** Extend the core schema to support feature engineering and large-scale analysis.
- **Task:** Implement the 5 core features and add **at least 10 enhancements** for data science.
- **Examples (illustrative, not prescriptive):**
  - Fine‑grained event/behavior tracking tables
  - Tables to store engineered features
  - Historical snapshot tables for model training
  - Pre‑aggregated statistical summary tables

### 2. Data Population at Scale

- **Goal:** Build reproducible scripts to populate a realistic, scalable dataset.
- **Task:** Create documented, automated scripts (Python, SQL, etc.) with realistic patterns/distributions.
- **Note on Scale:** You’re graded on script quality and scalability design, not raw data volume.

### 3. Analytical & Statistical Queries

- **Goal:** Answer complex questions and evaluate performance at scale.
- **Task:** In `docs/ANALYSIS.md`, provide **at least 15 analytical queries**.
- **Each query includes:** question, commented SQL, performance notes.
- **Quality bar:** Avoid trivial variants—each query must answer a distinct question.
- **Technical:** Use advanced SQL (window functions, CTEs) and statistical queries (distributions, correlations); include one `EXPLAIN PLAN` focused on optimization for large datasets.

**Presentation:** Prepare 2–3 slides or a 1‑minute script summarizing schema changes, dataset generation, and key insights. Use DBeaver or Apache Superset to visualize and demo your analytics.

---

## Evaluation

**To be updated, provided later.**

Confirmed criteria:
- **ERD and DB Design (30%)**
- **DB Initialization and Population (20%)**

Additional criteria will be added later.

