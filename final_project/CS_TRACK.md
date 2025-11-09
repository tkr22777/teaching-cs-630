# Computer Science Track (Engineering Track)

Full-stack implementation with UI, backend, and database.

---

## Important Dates

**November 26th - ERD Design Presentation**
- Submit ER diagram
- Present and describe design (5 minutes)
- Explain design choices and how your design is important/required for your project scope
- Discuss 1-2 key design decisions

**December 9th (EOD) - Final Submission**
- GitHub repository URL
- Complete `docs/` folder and `sql/` scripts
- All code and documentation

**December 10th - Project Demo (In Class)**
- Live demonstration
- Explain design decisions and architecture

---

## Requirements

### 1. Database Design

- Complete ER Diagram (entities, relationships, cardinalities, PKs/FKs)
- Normalized to minimum 3NF

### 2. GitHub Repository

- Public repository with incremental commits (minimum 10)
- Include repository link in submission

### 3. Documentation

```
project/
├── docs/
│   ├── DATABASE_DESIGN.md
│   ├── SETUP.md
│   ├── ARCHITECTURE.md
│   └── USER_GUIDE.md
├── sql/
│   ├── schema.sql
│   ├── sample_data.sql
│   └── reset.sql
└── README.md
```

Required SQL files:
- `schema.sql` - DDL with CREATE TABLE statements, PKs/FKs/constraints
- `sample_data.sql` - INSERT statements with representative data
- `reset.sql` - DROP/TRUNCATE statements to reset database

### 4. Code Architecture

Required layers:
- **UI Layer**
- **Controller Layer** 
- **Business Logic Layer**
- **Data Access Layer** (SQL queries only)

### 5. Technical Stack

- **Backend**: Python (Flask/Django/FastAPI) OR TypeScript/JavaScript (Node.js/Express)
- **Database**: Oracle (use provided credentials)
- **SQL**: Direct SQL queries - **NO ORMs**

### 6. Initial Focus

Start with:
1. ERD design
2. DDL design (CREATE TABLE statements)
3. Sample INSERT queries

**Note:** Use AI assistants but understand all generated code. Must explain design decisions during demo.

---

## Evaluation

1. **Database Design (30%)** - ER diagram, normalization, data isolation
2. **SQL Implementation (25%)** - JOINs, subqueries, transactions
3. **Code Quality & Architecture (20%)** - Layer separation, understanding
4. **Features & Functionality (15%)** - All 10 features working
5. **Documentation & Repository (10%)** - Complete docs, commits

