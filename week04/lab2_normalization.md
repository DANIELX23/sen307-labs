# SEN307 Week 4 Lab 2: Database Design and Normalization

Matric number: 202412674
Database: sen307_202412674

## Unnormalized table (given)

| matric_no | student_name | student_dept | course_codes | course_titles | lecturers | lecturer_depts | session |
|---|---|---|---|---|---|---|---|
| 20231234 | Ada Obi | Software Engineering | SEN307, SEN301 | Database Systems, Software Architecture | Dr A. Okafor, Dr M. Bello | Software Eng., Software Eng. | 2026/2027 |
| 20231235 | John Musa | Computer Science | SEN307, COS301 | Database Systems, Operating Systems | Dr A. Okafor, Mrs F. Eze | Software Eng., Computer Sci. | 2026/2027 |
| 20231236 | Grace Okon | Software Engineering | SEN301 | Software Architecture | Dr M. Bello | Software Eng. | 2026/2027 |

Problems: cells hold several values, and student, course, lecturer and department details are repeated.

## Step 1: First Normal Form (1NF)

| matric_no | student_name | student_dept | course_code | course_title | lecturer | lecturer_dept | session |
|---|---|---|---|---|---|---|---|
| 20231234 | Ada Obi | Software Engineering | SEN307 | Database Systems | Dr A. Okafor | Software Eng. | 2026/2027 |
| 20231234 | Ada Obi | Software Engineering | SEN301 | Software Architecture | Dr M. Bello | Software Eng. | 2026/2027 |
| 20231235 | John Musa | Computer Science | SEN307 | Database Systems | Dr A. Okafor | Software Eng. | 2026/2027 |
| 20231235 | John Musa | Computer Science | COS301 | Operating Systems | Mrs F. Eze | Computer Sci. | 2026/2027 |
| 20231236 | Grace Okon | Software Engineering | SEN301 | Software Architecture | Dr M. Bello | Software Eng. | 2026/2027 |

Primary key: (matric_no, course_code, session). No single column is unique on its own.

**What I fixed:** I removed the multi-valued cells so every cell holds one value, giving each student-course pair its own row.

## Step 2: Second Normal Form (2NF)

Partial dependencies found:
- student_name and student_dept depend only on matric_no, not on the whole key.
- course_title depends only on course_code.
- lecturer and lecturer_dept depend on the course being taught (course_code and session), not on which student is enrolled.

**Student**

| matric_no (PK) | student_name | student_dept |
|---|---|---|
| 20231234 | Ada Obi | Software Engineering |
| 20231235 | John Musa | Computer Science |
| 20231236 | Grace Okon | Software Engineering |

**Course**

| course_code (PK) | course_title |
|---|---|
| SEN307 | Database Systems |
| SEN301 | Software Architecture |
| COS301 | Operating Systems |

**Lecturer**

| lecturer_id (PK) | lecturer_name | lecturer_dept |
|---|---|---|
| 1 | Dr A. Okafor | Software Eng. |
| 2 | Dr M. Bello | Software Eng. |
| 3 | Mrs F. Eze | Computer Sci. |

**Enrolment**

| matric_no (PK, FK) | course_code (PK, FK) | session (PK) |
|---|---|---|
| 20231234 | SEN307 | 2026/2027 |
| 20231234 | SEN301 | 2026/2027 |
| 20231235 | SEN307 | 2026/2027 |
| 20231235 | COS301 | 2026/2027 |
| 20231236 | SEN301 | 2026/2027 |

**Teaching**

| lecturer_id (PK, FK) | course_code (PK, FK) | session (PK) |
|---|---|---|
| 1 | SEN307 | 2026/2027 |
| 2 | SEN301 | 2026/2027 |
| 3 | COS301 | 2026/2027 |

**What I fixed:** I moved each attribute that depended on only part of the key into its own table (Student, Course, Lecturer), and used Enrolment and Teaching to record which student takes which course and which lecturer teaches it in which session.

## Step 3: Third Normal Form (3NF)

Transitive dependency found: student_dept and lecturer_dept are really details of a department, not of the student or lecturer. If a department were renamed, many rows would need updating. The sheet also writes the same department two ways ("Software Engineering" and "Software Eng."), which a Department table prevents.

**Department**

| dept_id (PK) | dept_name |
|---|---|
| 1 | Software Engineering |
| 2 | Computer Science |

**Student** (after 3NF)

| matric_no (PK) | student_name | dept_id (FK) |
|---|---|---|
| 20231234 | Ada Obi | 1 |
| 20231235 | John Musa | 2 |
| 20231236 | Grace Okon | 1 |

**Lecturer** (after 3NF)

| lecturer_id (PK) | lecturer_name | dept_id (FK) |
|---|---|---|
| 1 | Dr A. Okafor | 1 |
| 2 | Dr M. Bello | 1 |
| 3 | Mrs F. Eze | 2 |

Course, Enrolment and Teaching stay as they were in 2NF.

**What I fixed:** I moved department details into a Department table and linked to it with a foreign key, so each department name is stored once.

## Step 4: Final schema

- Department(dept_id PK, dept_name)
- Student(matric_no PK, student_name, dept_id FK -> Department)
- Lecturer(lecturer_id PK, lecturer_name, dept_id FK -> Department)
- Course(course_code PK, course_title)
- Enrolment(matric_no PK/FK -> Student, course_code PK/FK -> Course, session PK)
- Teaching(lecturer_id PK/FK -> Lecturer, course_code PK/FK -> Course, session PK)

## Relationships

- Department to Student: 1:N
- Department to Lecturer: 1:N
- Student to Course: M:N, resolved by Enrolment
- Lecturer to Course: M:N, resolved by Teaching
