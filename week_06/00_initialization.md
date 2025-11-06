# Database Initialization - Week 6: SQL Joins

## Overview

This initialization script sets up the university enrollment database used throughout all Week 6 SQL Joins lessons.

## Database Schema

The database consists of four interconnected tables representing a university course enrollment system:

- **Students**: Student information
- **Instructors**: Faculty information
- **Courses**: Course catalog
- **Enrollments**: Student course enrollments (junction table)

## Setup Script

```sql
-- Create Students table
CREATE TABLE students (
    student_id INTEGER PRIMARY KEY,
    first_name VARCHAR2(50) NOT NULL,
    last_name VARCHAR2(50) NOT NULL,
    email VARCHAR2(100) UNIQUE NOT NULL,
    major VARCHAR2(50),
    enrollment_date DATE DEFAULT SYSDATE
);

-- Create Instructors table
CREATE TABLE instructors (
    instructor_id INTEGER PRIMARY KEY,
    instructor_name VARCHAR2(100) NOT NULL,
    department VARCHAR2(50),
    hire_date DATE
);

-- Create Courses table
CREATE TABLE courses (
    course_id VARCHAR2(10) PRIMARY KEY,
    course_name VARCHAR2(100) NOT NULL,
    department VARCHAR2(50),
    credits INTEGER,
    instructor_id INTEGER REFERENCES instructors(instructor_id)
);

-- Create Enrollments table (junction table)
CREATE TABLE enrollments (
    enrollment_id INTEGER PRIMARY KEY,
    student_id INTEGER REFERENCES students(student_id),
    course_id VARCHAR2(10) REFERENCES courses(course_id),
    semester VARCHAR2(20),
    grade VARCHAR2(5)
);

-- Insert Students
INSERT INTO students (student_id, first_name, last_name, email, major, enrollment_date) VALUES
(1, 'John', 'Smith', 'john.smith@university.edu', 'Computer Science', '2023-09-01');
INSERT INTO students (student_id, first_name, last_name, email, major, enrollment_date) VALUES
(2, 'Jane', 'Doe', 'jane.doe@university.edu', 'Mathematics', '2023-09-01');
INSERT INTO students (student_id, first_name, last_name, email, major, enrollment_date) VALUES
(3, 'Bob', 'Wilson', 'bob.wilson@university.edu', 'Computer Science', '2024-01-15');
INSERT INTO students (student_id, first_name, last_name, email, major, enrollment_date) VALUES
(4, 'Alice', 'Brown', 'alice.brown@university.edu', 'Physics', '2024-01-15');
INSERT INTO students (student_id, first_name, last_name, email, major, enrollment_date) VALUES
(5, 'Charlie', 'Davis', 'charlie.davis@university.edu', NULL, '2024-09-01');

-- Insert Instructors
INSERT INTO instructors (instructor_id, instructor_name, department, hire_date) VALUES
(10, 'Dr. Johnson', 'Computer Science', '2018-08-15');
INSERT INTO instructors (instructor_id, instructor_name, department, hire_date) VALUES
(11, 'Dr. Lee', 'Mathematics', '2019-01-10');
INSERT INTO instructors (instructor_id, instructor_name, department, hire_date) VALUES
(12, 'Dr. Martinez', 'Physics', '2020-09-01');
INSERT INTO instructors (instructor_id, instructor_name, department, hire_date) VALUES
(13, 'Dr. Taylor', 'Chemistry', '2021-06-15');

-- Insert Courses
INSERT INTO courses (course_id, course_name, department, credits, instructor_id) VALUES
('CS101', 'Introduction to Programming', 'Computer Science', 3, 10);
INSERT INTO courses (course_id, course_name, department, credits, instructor_id) VALUES
('CS201', 'Data Structures', 'Computer Science', 4, 10);
INSERT INTO courses (course_id, course_name, department, credits, instructor_id) VALUES
('MATH101', 'Calculus I', 'Mathematics', 4, 11);
INSERT INTO courses (course_id, course_name, department, credits, instructor_id) VALUES
('PHYS101', 'Physics I', 'Physics', 4, 12);
INSERT INTO courses (course_id, course_name, department, credits, instructor_id) VALUES
('CS301', 'Database Systems', 'Computer Science', 3, 10);
INSERT INTO courses (course_id, course_name, department, credits, instructor_id) VALUES
('ENG101', 'English Composition', 'English', 3, NULL);

-- Insert Enrollments
INSERT INTO enrollments (enrollment_id, student_id, course_id, semester, grade) VALUES
(101, 1, 'CS101', 'Fall 2023', 'A');
INSERT INTO enrollments (enrollment_id, student_id, course_id, semester, grade) VALUES
(102, 1, 'CS201', 'Spring 2024', 'B+');
INSERT INTO enrollments (enrollment_id, student_id, course_id, semester, grade) VALUES
(103, 2, 'MATH101', 'Fall 2023', 'A');
INSERT INTO enrollments (enrollment_id, student_id, course_id, semester, grade) VALUES
(104, 2, 'CS101', 'Fall 2023', 'A-');
INSERT INTO enrollments (enrollment_id, student_id, course_id, semester, grade) VALUES
(105, 3, 'CS101', 'Spring 2024', 'B');
INSERT INTO enrollments (enrollment_id, student_id, course_id, semester, grade) VALUES
(106, 3, 'CS201', 'Spring 2024', 'B+');
INSERT INTO enrollments (enrollment_id, student_id, course_id, semester, grade) VALUES
(107, 4, 'PHYS101', 'Spring 2024', 'A');
INSERT INTO enrollments (enrollment_id, student_id, course_id, semester, grade) VALUES
(108, 1, 'CS301', 'Fall 2024', NULL);
```

## Summary

This script creates a complete university enrollment database with:
- 5 students (including one without a declared major)
- 4 instructors across different departments
- 6 courses (including one without an assigned instructor)
- 8 enrollments (including one without a grade)

The data includes various scenarios useful for demonstrating different types of joins, including NULL values and missing relationships.

