### Week 1: Assignments
1. Define and explain the purpose of a database.

2. Create ERD diagrams for **two** simple business or organizational scenarios.
   - Complete both scenarios below (Library, Clinic).
   - Limit each diagram to a maximum of three entities (three boxes).
   - Use the assumptions and business activities to infer entities, attributes, and relationships.

   Steps:
   1) Read the assumptions and activities for the scenarios.
   2) Identify 2–3 entities for each scenario based on the activities.
   3) List 3 or more attributes for each entity
   4) Identify any primary or foreign key for each entity
   5) Draw each ERD using any standard notation
   6) Use appropriate attribute types and notation as needed (e.g. composite, derived, key etc).
   7) Write 2–3 sentences explaining your assumptions and key design choices.

   Deliverables:
   - ERDs as images or PDFs (filename: week1-erd-<lastname>.pdf).
   - A short note (3–5 bullets) listing entities, PKs/FKs, and the derived attribute(s).

   Expectations:
   - Grading focus: correctness in choosing entities, attributes, keys and clarity.

#### Scenario: Library

##### Background

We are digitizing a small public library's borrowing process. After speaking with library staff, we agreed on a few simplifying assumptions and captured the core activities to guide the ERD.

##### Assumptions

- The collection has one copy per item.
- Each borrowing involves exactly one item.
- Any late fee is recorded alongside the borrowing.

##### Business activities

- Register a new library user.
- Update a library user's contact information.
- Add a new title to the catalog.
- Mark an item as available or checked out.
- Check out an item to a library user.
- Record an item return and any late fee.
- Extend a borrowing period when renewed.
- Temporarily suspend borrowing privileges for unpaid fees.

#### Scenario: Clinic

##### Background

We are digitizing appointment scheduling for a small community clinic. After meeting with clinic staff, we agreed on a few simplifying assumptions and captured the core activities to guide the ERD.

##### Assumptions

- Each appointment links one person and one clinician.
- Each appointment covers a single visit type; visit types are not modeled separately.
- Appointments are scheduled for a specific calendar day and time; reusable time-slot catalogs are out of scope.

##### Business activities

- Register a new person seeking care.
- Update a person's contact and insurance information.
- Schedule an appointment on a specific day with a clinician.
- Change an existing appointment's time or day.
- Check in a person on arrival.
- Mark an appointment as completed or canceled and give a reason.
- Send reminders for upcoming appointments.

### Useful Reading Materials and References:
- [Crow’s Foot notation: relationship symbols and how to read diagrams](https://www.freecodecamp.org/news/crows-foot-notation-relationship-symbols-and-how-to-read-diagrams/)
- [ER diagram symbols and notation guide](https://creately.com/guides/er-diagram-symbols/)
- [Types of attributes in ER model](https://www.geeksforgeeks.org/dbms/types-of-attributes-in-er-model/)
