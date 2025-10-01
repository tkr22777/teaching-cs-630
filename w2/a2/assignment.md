### Week 2: Assignment - Supertype/Subtype ERD Design

Create an ERD diagram for a transportation company business scenario that demonstrates advanced entity relationship concepts including inheritance hierarchies.

**Steps:**

1. Read the background, assumptions, and business activities below
2. Identify supertypes and their respective subtypes
3. Design entity hierarchies with appropriate discriminators
4. Determine and apply appropriate completeness and disjoint/overlapping constraints
5. Create relationships between entities from different hierarchies
6. Draw the complete ERD using standard notation
7. Create sample data tables that demonstrate constraint enforcement

#### Scenario: TransLogistics Inc. Fleet Management

##### Background

TransLogistics Inc. is a transportation company managing a fleet of cars and trucks and employing drivers and mechanics. The company needs a database system to track their vehicles, employees, and vehicle assignments. The system must handle these different vehicle types and employee roles with distinct responsibilities.

##### Assumptions

- The company operates two types of vehicles: cars and trucks, each with distinct characteristics
- Employees are classified into two roles: drivers who operate vehicles and mechanics who maintain them
- Each vehicle can be assigned to one driver at any given time
- Mechanics maintain vehicles but are not assigned to specific vehicles for routes

##### Business Activities

- Register new vehicles (cars or trucks) with type-specific specifications
- Track passenger capacity and fuel efficiency for cars used in client transport
- Monitor cargo capacity and weight limits for trucks used in freight delivery
- Hire new employees and classify them as drivers or mechanics
- Verify driver license classes and track years of driving experience
- Track mechanic certification levels and qualifications
- Assign drivers to vehicles based on license compatibility and experience
- Assign vehicle maintenance to mechanics
- Update vehicle and employee information
- View current vehicle assignments

## Deliverables

### ERD Diagram

Your ERD must include:

- Supertype/subtype hierarchies based on the business scenario
- Subtype discriminators clearly labeled for each hierarchy
- Completeness and disjoint constraints properly marked using standard notation
- Entity relationships
- Core attributes (3-4 per entity) including primary keys, foreign keys, and discriminators

### Sample Data

- Sample data for all tables
- Tables demonstrate constraint enforcement

## Expectations

- Grading focus: correct supertype/subtype modeling, constraint representation, relationship design, and sample data that demonstrates constraints
