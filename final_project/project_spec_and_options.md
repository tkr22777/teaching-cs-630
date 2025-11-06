# Final Project: Database-Backed Application

## Overview

Design and implement a database-backed application demonstrating database design, SQL implementation, and proper application architecture.

---

## Requirements

### 1. Database Design

- Complete ER Diagram (entities, relationships, cardinalities, PKs/FKs)
- Normalized to minimum 3NF

### 2. GitHub Repository

- Public repository with incremental commits (minimum 10)
- Include repository link in submission

### 3. Documentation Structure

```
project/
├── docs/
│   ├── DATABASE_DESIGN.md    # ER diagram, schema, normalization
│   ├── SETUP.md              # Setup and configuration
│   ├── ARCHITECTURE.md       # Layer descriptions, design decisions
│   └── USER_GUIDE.md         # How to use features
├── sql/
│   ├── schema.sql
│   └── sample_data.sql
└── README.md                  # Overview, tech stack, links to docs
```

README.md should link to separate documentation files above.

### 4. Code Architecture

Separate layers clearly:

- **UI Layer**: User interface
- **Controller Layer**: Request handling, routing
- **Business Logic Layer**: Business rules, validation
- **Data Access Layer**: SQL queries, database connections

Folder names can vary by framework (routes/, services/, repositories/, etc.) but layers must be separated.

### 5. Technical Stack

- **Backend**: Python (Flask/Django/FastAPI) OR TypeScript/JavaScript (Node.js/Express)
- **Database**: Oracle, PostgreSQL, MySQL, or SQLite
- **SQL**: Direct SQL queries - **NO ORM abstractions**
- **AuthNZ**: Optional

### 6. AI Tools

- Use GitHub Copilot, Cursor AI, or similar tools
- **CRITICAL**: Review and understand every line of generated code
- Must explain design decisions and code during demo
- "Vibe coding" without understanding will result in grade penalties

---

### Business Analytics Track (DB-only option)

For Business Analytics students, implementing a backend service is not required.

Deliverables:
- ER Diagram (min 3NF)
- `sql/schema.sql` with CREATE TABLE statements, PK/FK/constraints
- `sql/sample_data.sql` with representative rows
- `docs/ANALYTICS_QUERIES.md` containing SQL that demonstrates:
  - Core operations (INSERT/UPDATE/DELETE where relevant)
  - Analysis: JOINs, GROUP BY aggregates, window functions, subqueries/CTEs
  - Views (or materialized views if supported) for reusable analytics
  - Brief notes on indexing and one EXPLAIN/EXPLAIN ANALYZE discussion

Demo: run the SQL scripts and analytics queries live in the last class and explain results.

---

## Project Ideas

Choose one project. Each includes **5 core features** - you must design and implement **5 additional features** based on real-world needs.

<details>
<summary><strong>1. Donation & Fundraising Platform</strong> - Nonprofit campaign management</summary>

**Core Features:**

1. Multiple charities/organizations can register and create profiles
2. Create fundraising campaigns with goals and descriptions
3. Donor profiles and donation tracking
4. Campaign progress tracking and analytics
5. Receipt generation for donations

**Your Additional Features (5)**: Design based on real-world needs (recurring donations, matching campaigns, donor recognition, etc.).

</details>

<details>
<summary><strong>2. Learning Management System</strong> - Course and assignment platform</summary>

**Core Features:**

1. Multiple schools/institutions can use the platform
2. Course catalog with instructor assignments
3. Student enrollment and course access
4. Assignment submission and grading
5. Grade tracking and reporting

**Your Additional Features (5)**: Design based on real-world needs (parent portal, attendance, discussion forums, etc.).

</details>

<details>
<summary><strong>3. Property Rental Management</strong> - Landlord and tenant coordination</summary>

**Core Features:**

1. Multiple property management companies can use the system
2. Property and unit listings with details
3. Tenant applications and lease agreements
4. Rent payment tracking
5. Maintenance request management

**Your Additional Features (5)**: Design based on real-world needs (late fees, inspections, vendor management, etc.).

</details>

<details>
<summary><strong>4. Restaurant Order Management</strong> - Food ordering and delivery platform</summary>

**Core Features:**

1. Multiple restaurants can register and manage profiles
2. Menu management with items and pricing
3. Customer orders from specific restaurants
4. Order status tracking (received, preparing, ready, delivered)
5. Order history and analytics

**Your Additional Features (5)**: Design based on real-world needs (delivery zones, table reservations, loyalty programs, etc.).

</details>

<details>
<summary><strong>5. Product Review & Rating Platform</strong> - Consumer feedback and recommendations</summary>

**Core Features:**

1. Multiple vendors/brands can register products
2. Product catalog with details and categories
3. User reviews with ratings and text feedback
4. Review moderation and flagging system
5. Product ranking and filtering by ratings

**Your Additional Features (5)**: Design based on real-world needs (verified purchases, helpful votes, comparison features, etc.).

</details>

<details>
<summary><strong>6. Event Management Platform</strong> - Conference and event coordination</summary>

**Core Features:**

1. Multiple organizations can create and manage events
2. Event creation with details, location, and capacity
3. Attendee registration and ticket purchasing
4. Multiple ticket types and pricing
5. Check-in and attendance tracking

**Your Additional Features (5)**: Design based on real-world needs (sponsors, speakers, sessions, surveys, etc.).

</details>

<details>
<summary><strong>7. Help Desk Ticket System</strong> - IT support and knowledge base</summary>

**Core Features:**

1. Multiple companies can use the system
2. Submit support tickets with priority and category
3. Assign tickets to technicians
4. Track ticket status and resolution time
5. Knowledge base articles

**Your Additional Features (5)**: Design based on real-world needs (SLA tracking, escalation, satisfaction ratings, etc.).

</details>

<details>
<summary><strong>8. Job Board & Career Portal</strong> - Employment listing and application tracking</summary>

**Core Features:**

1. Multiple companies can post job openings
2. Job listings with requirements, salary range, and location
3. Candidate profiles with resume and experience
4. Job applications with status tracking
5. Company profiles and reviews

**Your Additional Features (5)**: Design based on real-world needs (skill matching, saved jobs, application analytics, etc.).

</details>

<details>
<summary><strong>9. Freelance Project Marketplace</strong> - Connect clients with freelancers</summary>

**Core Features:**

1. Multiple agencies or independent freelancers can join
2. Project postings with requirements and budget
3. Freelancer profiles with portfolio and skills
4. Proposal submissions and tracking
5. Project completion and reviews

**Your Additional Features (5)**: Design based on real-world needs (time tracking, invoicing, milestone payments, etc.).

</details>

<details>
<summary><strong>10. Library Management System</strong> - Catalog and circulation tracking</summary>

**Core Features:**

1. Support multiple library branches in the system
2. Book catalog with details and availability
3. Member registration and profiles
4. Check-out and check-in tracking
5. Hold requests and inter-branch transfers

**Your Additional Features (5)**: Design based on real-world needs (overdue fines, digital content, event rooms, etc.).

</details>

---

## Evaluation Criteria

1. **Database Design (30%)** - ER diagram, normalization, proper data isolation between organizations
2. **SQL Implementation (25%)** - Direct SQL, JOINs, subqueries, transactions, correct data filtering
3. **Code Quality & Architecture (20%)** - Layer separation, clean code, understanding
4. **Features & Functionality (15%)** - All 10 features working correctly, creativity
5. **Documentation & Repository (10%)** - Complete docs, setup instructions, commits

---

## Submission

- GitHub repository URL
- Complete `docs/` folder with all markdown files
- ER diagram in `docs/DATABASE_DESIGN.md`
- SQL scripts in `sql/` folder with sample data for multiple organizations/users
- Live demo of the running project during the last class
  - For Business Analytics track: demo the SQL schema and analytics queries live (no backend required)

---

## Tips

✓ Design ER diagram and all 10 features before coding
✓ Write and understand your SQL queries
✓ Separate layers - SQL only in data access layer
✓ Use AI tools but review everything
✓ Commit frequently with clear messages
✓ Be ready to explain your design decisions in demo
