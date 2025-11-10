# Business Analytics Track

Database design and analytics queries. **No backend required.**

**See main file for general requirements, important dates, and project options.**

---

## Deliverables
- `sql/schema.sql` (5 core + 10 analytics enhancements)
- `sql/sample_data.sql`, `sql/reset.sql`
- `docs/ANALYTICS_QUERIES.md` (15 queries, 3+ views/materialized views, 1 EXPLAIN PLAN)

---

## Track-Specific Requirements

### 1. Analytical Schema Design

- **Goal:** Extend the core project schema to enable business intelligence and analytics.
- **Task:** Implement the 5 core features and add **at least 10 enhancements** for analytics.
- **Examples (illustrative, not prescriptive):**
  - Dimensional tables (e.g., time, location)
  - Pre-aggregated summary tables or materialized views
  - History/audit tables for trends
  - Lookup tables for categories/classifications

### 2. Business Intelligence Queries

- **Goal:** Answer meaningful business questions using your enhanced schema.
- **Task:** In `docs/ANALYTICS_QUERIES.md`, provide **at least 15 analytical queries**.
- **Each query includes:** business question, commented SQL, brief expected insight.
- **Quality bar:** Avoid trivial variants—each query must answer a distinct question.
- **Technical:** Use advanced SQL (window functions, CTEs, subqueries); include 3+ views or materialized views (depending on privileges); include one `EXPLAIN PLAN` with optimization notes.

**Presentation:** Prepare 2–3 slides or a 1‑minute script summarizing schema changes and top insights.

**Note:** Visualization guidance will follow in upcoming weeks.

---

## Evaluation

1. **Database Design (25%)** - ER diagram, normalization, 5 core features + 10 analytics enhancements
2. **Business Questions (35%)** - Quality and relevance of 15 analytical queries
3. **SQL Implementation (25%)** - Query complexity, advanced SQL features, optimization
4. **Documentation (15%)** - Clear explanations, insights, and business context

