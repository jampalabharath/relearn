# Pyspark 1

## Section 1: Basic Queries (1–10)

Use these to select and filter data from a table.

```sql
-- 1. Select all columns from a table
SELECT * FROM employees;

-- 2. Select specific columns
SELECT first_name, last_name, hire_date FROM employees;

-- 3. Count all rows
SELECT COUNT(*) FROM employees;

-- 4. Count non-null values in a column
SELECT COUNT(salary) FROM employees;

-- 5. Find unique values in a column
SELECT DISTINCT department FROM employees;

-- 6. Filter data using WHERE
SELECT * FROM employees WHERE department = 'Sales';

-- 7. Filter with multiple conditions using AND
SELECT * FROM employees WHERE department = 'Sales' AND salary > 60000;

-- 8. Filter with OR
SELECT * FROM employees WHERE department = 'Sales' OR department = 'Marketing';

-- 9. Combine AND and OR with parentheses
SELECT * FROM employees
WHERE (department = 'Sales' OR department = 'Marketing') AND salary > 70000;

-- 10. Exclude a value using NOT
SELECT * FROM employees WHERE NOT department = 'IT';