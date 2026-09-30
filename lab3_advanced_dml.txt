-- ============================================================
-- Part A: Database and Table Setup
-- ============================================================

-- Task 1
CREATE DATABASE advanced_lab;

CREATE TABLE employees (
    emp_id SERIAL PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    department VARCHAR(50),
    salary INTEGER,
    hire_date DATE,
    status VARCHAR(20) DEFAULT 'Active'
);

CREATE TABLE departments (
    dept_id SERIAL PRIMARY KEY,
    dept_name VARCHAR(50),
    budget INTEGER,
    manager_id INTEGER
);

CREATE TABLE projects (
    project_id SERIAL PRIMARY KEY,
    project_name VARCHAR(100),
    dept_id INTEGER,
    start_date DATE,
    end_date DATE,
    budget INTEGER
);

-- ============================================================
-- Part B: Advanced INSERT Operations
-- ============================================================

-- Task 2
INSERT INTO employees (emp_id, first_name, last_name, department)
VALUES
    (1, 'Aidos', 'Nurgaliyev', 'IT'),
    (2, 'Aruzhan', 'Serikbayeva', 'IT');

SELECT setval(pg_get_serial_sequence('employees', 'emp_id'), (SELECT MAX(emp_id) FROM employees));

-- Task 3
INSERT INTO employees (first_name, last_name, department, salary, hire_date, status)
VALUES ('Yerlan', 'Abenov', 'HR', DEFAULT, '2021-06-01', DEFAULT);

-- Task 4
INSERT INTO departments (dept_name, budget, manager_id)
VALUES
    ('IT', 150000, 1),
    ('Sales', 80000, 2),
    ('Management', 200000, 3);

-- Task 5
INSERT INTO employees (first_name, last_name, department, salary, hire_date)
VALUES ('Madina', 'Kassymova', 'IT', 50000 * 1.1, CURRENT_DATE);

-- Task 6
CREATE TEMP TABLE temp_employees (LIKE employees);

INSERT INTO temp_employees
SELECT * FROM employees WHERE department = 'IT';

-- ============================================================
-- Part C: Complex UPDATE Operations
-- ============================================================

-- Sample data for Part C
INSERT INTO employees (first_name, last_name, department, salary, hire_date, status)
VALUES
    ('Nurlan', 'Zhaksylykov', 'Sales', 90000, '2018-05-10', 'Active'),
    ('Dana', 'Omarova', 'Sales', 65000, '2019-03-20', 'Active'),
    ('Bauyrzhan', 'Tulegenov', 'IT', 72000, '2017-08-15', 'Active'),
    ('Aigerim', 'Bekova', 'Sales', 55000, '2021-11-11', 'Terminated'),
    ('Timur', 'Sagyndykov', 'Finance', 45000, '2016-02-02', 'Inactive'),
    ('Zhanar', 'Mukhametova', 'HR', 62000, '2015-09-09', 'Active');

-- Task 7
UPDATE employees
SET salary = salary * 1.10;

-- Task 8
UPDATE employees
SET status = 'Senior'
WHERE salary > 60000 AND hire_date < '2020-01-01';

-- Task 9
UPDATE employees
SET department = CASE
    WHEN salary > 80000 THEN 'Management'
    WHEN salary BETWEEN 50000 AND 80000 THEN 'Senior'
    ELSE 'Junior'
END;

-- Task 10
UPDATE employees
SET department = DEFAULT
WHERE status = 'Inactive';

-- Task 11
UPDATE departments d
SET budget = (
    SELECT ROUND(AVG(e.salary) * 1.2)
    FROM employees e
    WHERE e.department = d.dept_name
)
WHERE EXISTS (
    SELECT 1 FROM employees e WHERE e.department = d.dept_name
);

-- Task 12
-- Sample data for Task 12
INSERT INTO employees (first_name, last_name, department, salary, hire_date)
VALUES
    ('Daulet', 'Akhmetov', 'Sales', 48000, '2022-01-15'),
    ('Amina', 'Suleimenova', 'Sales', 52000, '2022-07-01');

UPDATE employees
SET salary = salary * 1.15,
    status = 'Promoted'
WHERE department = 'Sales';

-- ============================================================
-- Part D: Advanced DELETE Operations
-- ============================================================

-- Task 13
DELETE FROM employees
WHERE status = 'Terminated';

-- Task 14
-- Sample data for Task 14
INSERT INTO employees (first_name, last_name, department, salary, hire_date)
VALUES ('Arman', 'Dauletov', NULL, 35000, '2023-08-01');

DELETE FROM employees
WHERE salary < 40000
  AND hire_date > '2023-01-01'
  AND department IS NULL;

-- Task 15
-- The task text compares dept_id (integer) with department (text).
-- PostgreSQL throws an error for that ("operator does not exist:
-- integer = character varying"), so dept_name is compared instead.
-- The "IS NOT NULL" filter is required: if the subquery returns even one
-- NULL, NOT IN never becomes true and nothing would be deleted.
DELETE FROM departments
WHERE dept_name NOT IN (
    SELECT DISTINCT department
    FROM employees
    WHERE department IS NOT NULL
);

-- Task 16
-- Sample data for Task 16
INSERT INTO projects (project_name, dept_id, start_date, end_date, budget)
VALUES
    ('Alpha', 1, '2022-01-01', '2022-12-31', 80000),
    ('Beta', 1, '2023-03-01', '2023-11-30', 60000),
    ('Gamma', 2, '2021-01-01', '2022-06-30', 30000),
    ('Delta', 3, '2024-01-01', '2024-12-31', 120000);

DELETE FROM projects
WHERE end_date < '2023-01-01'
RETURNING *;

-- ============================================================
-- Part E: Operations with NULL Values
-- ============================================================

-- Task 17
INSERT INTO employees (first_name, last_name, salary, department)
VALUES ('Gulmira', 'Sadykova', NULL, NULL);

-- Task 18
UPDATE employees
SET department = 'Unassigned'
WHERE department IS NULL;

-- Task 19
DELETE FROM employees
WHERE salary IS NULL OR department IS NULL;

-- ============================================================
-- Part F: RETURNING Clause Operations
-- ============================================================

-- Task 20
INSERT INTO employees (first_name, last_name, department, salary, hire_date)
VALUES ('Alikhan', 'Zhumabayev', 'HR', 47000, '2022-04-04')
RETURNING emp_id, first_name || ' ' || last_name AS full_name;

-- Task 21
-- Sample data for Task 21
INSERT INTO employees (first_name, last_name, department, salary, hire_date)
VALUES
    ('Yernar', 'Kenzhebayev', 'IT', 60000, '2021-05-05'),
    ('Saltanat', 'Baimukhanova', 'IT', 64000, '2022-02-02');

-- RETURNING shows only the NEW value of a row after UPDATE, so the old
-- salary has to be captured separately. The subquery "o" reads the old
-- salaries before the update, and the UPDATE ... FROM joins it by emp_id.
-- RETURNING can then use columns from both the updated table (e) and the
-- helper subquery (o).
UPDATE employees e
SET salary = e.salary + 5000
FROM (
    SELECT emp_id, salary AS old_salary
    FROM employees
    WHERE department = 'IT'
) o
WHERE e.emp_id = o.emp_id
RETURNING e.emp_id, o.old_salary, e.salary AS new_salary;

-- Task 22
DELETE FROM employees
WHERE hire_date < '2020-01-01'
RETURNING *;

-- ============================================================
-- Part G: Advanced DML Patterns
-- ============================================================

-- Task 23
INSERT INTO employees (first_name, last_name, department, salary, hire_date)
SELECT 'Asel', 'Nurmukhanbetova', 'HR', 45000, DATE '2022-03-15'
WHERE NOT EXISTS (
    SELECT 1
    FROM employees
    WHERE first_name = 'Asel' AND last_name = 'Nurmukhanbetova'
);

-- Task 24
-- Sample data for Task 24
INSERT INTO departments (dept_name, budget, manager_id)
VALUES ('IT', 150000, 1);

UPDATE employees e
SET salary = ROUND(
    CASE
        WHEN (SELECT d.budget FROM departments d WHERE d.dept_name = e.department LIMIT 1) > 100000
            THEN e.salary * 1.10
        ELSE e.salary * 1.05
    END
)
WHERE e.department IN (SELECT dept_name FROM departments);

-- Task 25
INSERT INTO employees (first_name, last_name, department, salary, hire_date)
VALUES
    ('Kairat', 'Ospanov', 'Operations', 40000, '2024-01-10'),
    ('Dinara', 'Tokhtarova', 'Operations', 42000, '2024-01-10'),
    ('Serik', 'Amanzholov', 'Operations', 44000, '2024-01-10'),
    ('Aizhan', 'Kaliyeva', 'Operations', 46000, '2024-01-10'),
    ('Askar', 'Rakhimov', 'Operations', 48000, '2024-01-10');

UPDATE employees
SET salary = ROUND(salary * 1.10)
WHERE department = 'Operations';

-- Task 26
-- Sample data for Task 26
INSERT INTO employees (first_name, last_name, department, salary, hire_date, status)
VALUES
    ('Samal', 'Iskakova', 'Finance', 51000, '2021-03-03', 'Inactive'),
    ('Marat', 'Yesenov', 'Finance', 53000, '2022-09-09', 'Inactive');

CREATE TABLE employee_archive (LIKE employees INCLUDING DEFAULTS);

INSERT INTO employee_archive
SELECT * FROM employees WHERE status = 'Inactive';

DELETE FROM employees
WHERE status = 'Inactive';

-- Task 27
-- Sample data for Task 27
INSERT INTO employees (first_name, last_name, department, salary, hire_date)
VALUES
    ('Ruslan', 'Kudaibergenov', 'IT', 61000, '2022-01-01'),
    ('Gaukhar', 'Zhunusova', 'IT', 62000, '2022-02-01'),
    ('Yerbol', 'Mamyrbayev', 'IT', 63000, '2022-03-01'),
    ('Ainur', 'Tastanova', 'IT', 64000, '2022-04-01');

INSERT INTO projects (project_name, dept_id, start_date, end_date, budget)
VALUES
    ('Epsilon', (SELECT dept_id FROM departments WHERE dept_name = 'IT' LIMIT 1), '2024-02-01', '2024-10-31', 90000),
    ('Zeta', (SELECT dept_id FROM departments WHERE dept_name = 'IT' LIMIT 1), '2024-03-01', '2024-09-30', 20000);

UPDATE projects p
SET end_date = end_date + 30
WHERE p.budget > 50000
  AND (
      SELECT COUNT(*)
      FROM employees e
      JOIN departments d ON d.dept_name = e.department
      WHERE d.dept_id = p.dept_id
  ) > 3;
