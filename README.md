# HR Database — From Scratch

A simple, educational HR (Human Resources) database built from scratch. This repository provides the schema, sample data, and example queries you can use to learn relational database design, practice SQL, and prototype common HR reporting tasks (headcount, compensation, tenure, org structure, etc.).

> Note: This README is intentionally generic so it can be adapted to different SQL engines (PostgreSQL, SQLite, MySQL). Update commands and file names below to match the actual files in this repository.

Table of contents
- [About](#about)
- [Features](#features)
- [Repository layout](#repository-layout)
- [Prerequisites](#prerequisites)
- [Quickstart (Postgres)](#quickstart-postgres)
- [Quickstart (SQLite)](#quickstart-sqlite)
- [Schema overview](#schema-overview)
- [Example queries](#example-queries)
- [Development workflow](#development-workflow)
- [Testing](#testing)
- [Contributing](#contributing)
- [License](#license)
- [Contact](#contact)

## About
This project is intended for students, instructors, or engineers who want to:
- Learn how to design a normalized HR schema
- Practice writing SQL queries for business metrics
- Seed and explore sample HR data
- Build reporting queries and simple analytics

## Features
- Relational schema for common HR concepts:
  - employees, departments, jobs/roles, hires, salaries, managers, locations
- Seed scripts with example records (configurable)
- Example reporting queries and stored procedures
- (Optional) Docker Compose for reproducible database environment
- (Optional) ETL scripts for loading CSV data

## Repository layout
The exact file names may vary. Typical layout:
- schema/
  - schema.sql           — CREATE TABLE statements and constraints
  - migrations/          — optional migrations (versioned)
- seed/
  - seed.sql             — INSERT statements or CSV loaders
  - sample-data.csv      — sample dataset(s)
- queries/
  - report_headcount.sql
  - report_avg_salary.sql
  - report_tenure.sql
- tools/
  - load_data.py         — helper scripts (optional)
- docker-compose.yml     — optional DB services for local dev
- README.md

Update these paths if your repository uses different names.

## Prerequisites
Choose one of the supported backends and install the client tools:
- PostgreSQL (psql)
- SQLite (sqlite3)
- MySQL (mysql client)
- Docker & Docker Compose (recommended for isolated environment)

## Quickstart (Postgres)
1. Clone the repo:
   ```bash
   git clone https://github.com/<owner>/<repo>.git
   cd <repo>
   ```
2. Create a database:
   ```bash
   createdb hr_demo
   ```
3. Run schema and seed scripts:
   ```bash
   psql -d hr_demo -f schema/schema.sql
   psql -d hr_demo -f seed/seed.sql
   ```
4. Run example query:
   ```bash
   psql -d hr_demo -f queries/report_headcount.sql
   ```

## Quickstart (SQLite)
1. Create and load a SQLite database:
   ```bash
   sqlite3 hr.db < schema/schema.sql
   sqlite3 hr.db < seed/seed.sql
   ```
2. Run an example query:
   ```bash
   sqlite3 hr.db < queries/report_headcount.sql
   ```

## Schema overview
A typical HR schema will include tables similar to:

- employees
  - id (PK), first_name, last_name, email, hire_date, job_id, manager_id, department_id
- departments
  - id (PK), name, location_id
- jobs (or roles)
  - id (PK), title, min_salary, max_salary
- salaries (optional history)
  - id, employee_id (FK), salary, from_date, to_date
- locations
  - id, city, state, country
- dependents (optional)
  - id, employee_id, name, relationship, birth_date

Adjust types, constraints, and FK behavior to your target RDBMS.

## Example queries
Below are a few useful queries you can adapt. Replace table/column names if different.

Top-level headcount by department:
```sql
SELECT d.name AS department,
       COUNT(e.id) AS headcount
FROM departments d
LEFT JOIN employees e ON e.department_id = d.id
GROUP BY d.name
ORDER BY headcount DESC;
```

Average salary by job:
```sql
SELECT j.title,
       ROUND(AVG(s.salary)::numeric, 2) AS avg_salary
FROM jobs j
JOIN employees e ON e.job_id = j.id
JOIN salaries s ON s.employee_id = e.id
WHERE s.to_date IS NULL OR s.to_date > CURRENT_DATE
GROUP BY j.title
ORDER BY avg_salary DESC;
```

Employees hired in the last 6 months:
```sql
SELECT id, first_name, last_name, hire_date, department_id
FROM employees
WHERE hire_date >= (CURRENT_DATE - INTERVAL '6 months')
ORDER BY hire_date DESC;
```

Manager span (direct reports):
```sql
SELECT m.id AS manager_id, m.first_name || ' ' || m.last_name AS manager_name,
       COUNT(e.id) AS direct_reports
FROM employees m
LEFT JOIN employees e ON e.manager_id = m.id
GROUP BY m.id, manager_name
ORDER BY direct_reports DESC;
```

(If using SQLite remove ::numeric casting and INTERVAL syntax or adapt accordingly.)

## Development workflow
- Use feature branches: git checkout -b feat/your-change
- Add/modify schema under schema/ or migrations/
- Add seed data as needed under seed/
- Add new reports under queries/
- Run tests and linting before opening a PR

If you use migrations (Flyway, Alembic, Liquibase, etc.), include instructions here for applying/updating them.

## Testing
- Add SQL-based integration tests that run against a test DB (CI job)
- Optionally use Docker Compose to spin up a database in CI
- Example: run tests with psql or a small test runner script

## Contributing
Contributions are welcome. Please:
1. Open an issue describing the change or idea
2. Branch from main: git checkout -b feat/your-feature
3. Add tests where applicable
4. Open a PR with a clear description of the change

Include a CONTRIBUTING.md if you want more detailed contributor guidelines.

## License
Specify your license here (e.g., MIT, Apache-2.0). Example:
This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

## Contact
Project maintainer: <your-name> — replace with actual contact info or GitHub username.

---

If you'd like, I can:
- tailor this README to the exact files in your repository (I can read the repo and update commands and paths),
- generate badges, a Docker Compose example, or CI workflow,
- produce a CONTRIBUTING.md and LICENSE file for you.
