+++
title = "100 SQL Queries"
weight = 4
+++

This cheat sheet provides a quick reference to common SQL queries and concepts used in data analysis, from fundamental commands to advanced techniques.

---

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

-- 8. Filter with multiple conditions using OR
SELECT * FROM employees WHERE department = 'Sales' OR department = 'Marketing';

-- 9. Combine AND and OR with parentheses
SELECT * FROM employees
WHERE (department = 'Sales' OR department = 'Marketing') AND salary > 70000;

-- 10. Exclude a value using NOT
SELECT * FROM employees WHERE NOT department = 'IT';
```

---

## Section 2: Filtering & Pattern Matching (11–20)

Advanced filtering using IN, BETWEEN, and LIKE.

```sql
-- 11. Filter by a list of values using IN
SELECT * FROM employees WHERE department IN ('Sales', 'Marketing', 'IT');

-- 12. Exclude a list of values using NOT IN
SELECT * FROM employees WHERE department NOT IN ('Sales', 'Marketing');

-- 13. Filter values within a range using BETWEEN
SELECT * FROM employees WHERE salary BETWEEN 50000 AND 75000;

-- 14. Case-insensitive substring search
SELECT * FROM employees WHERE first_name ILIKE '%jo%';

-- 15. Starts with a specific pattern
SELECT * FROM employees WHERE last_name LIKE 'Smi%';

-- 16. Ends with a specific pattern
SELECT * FROM employees WHERE email LIKE '%@gmail.com';

-- 17. Pattern at a specific position
SELECT * FROM employees WHERE first_name LIKE '_a%';

-- 18. Find rows with NULL values
SELECT * FROM employees WHERE manager_id IS NULL;

-- 19. Find rows with non-NULL values
SELECT * FROM employees WHERE manager_id IS NOT NULL;

-- 20. Filter by date range
SELECT * FROM employees WHERE hire_date BETWEEN '2023-01-01' AND '2023-12-31';
```

---

## Section 3: Sorting & Limiting (21–30)

Control the order of your results and limit the number of rows.

```sql
-- 21. Sort results in ascending order
SELECT * FROM employees ORDER BY last_name ASC;

-- 22. Sort results in descending order
SELECT * FROM employees ORDER BY salary DESC;

-- 23. Sort by multiple columns
SELECT * FROM employees ORDER BY department ASC, salary DESC;

-- 24. Top 10 highest paid employees
SELECT * FROM employees ORDER BY salary DESC LIMIT 10;

-- 25. Latest 5 hired employees
SELECT * FROM employees ORDER BY hire_date DESC LIMIT 5;

-- 26. DISTINCT and ORDER BY
SELECT DISTINCT department FROM employees ORDER BY department;

-- 27. Fetch rows 11–20
SELECT * FROM employees ORDER BY employee_id OFFSET 10 LIMIT 10;

-- 28. Employee with the highest salary
SELECT * FROM employees ORDER BY salary DESC LIMIT 1;

-- 29. Second-highest salary
SELECT DISTINCT salary FROM employees ORDER BY salary DESC LIMIT 1 OFFSET 1;

-- 30. Order by a calculated field
SELECT first_name, last_name, (salary * 0.1) AS bonus
FROM employees ORDER BY bonus DESC;
```

---

## Section 4: Aggregation & Grouping (31–40)

Summarize and analyze data with aggregate functions.

```sql
-- 31. Count rows for each group
SELECT department, COUNT(*) FROM employees GROUP BY department;

-- 32. Average salary per department
SELECT department, AVG(salary) FROM employees GROUP BY department;

-- 33. Total salary per department
SELECT department, SUM(salary) FROM employees GROUP BY department;

-- 34. Maximum salary in each department
SELECT department, MAX(salary) FROM employees GROUP BY department;

-- 35. Minimum salary in each department
SELECT department, MIN(salary) FROM employees GROUP BY department;

-- 36. Filter groups using HAVING
SELECT department, AVG(salary)
FROM employees
GROUP BY department
HAVING AVG(salary) > 65000;

-- 37. Count employees in departments with more than 5 employees
SELECT department, COUNT(*)
FROM employees
GROUP BY department
HAVING COUNT(*) > 5;

-- 38. Filter and then group
SELECT department, AVG(salary)
FROM employees
WHERE hire_date > '2022-01-01'
GROUP BY department;

-- 39. Group by multiple columns
SELECT department, job_title, AVG(salary)
FROM employees
GROUP BY department, job_title;

-- 40. Highest salary in each department (value)
SELECT department, MAX(salary) FROM employees GROUP BY department;
```

---

## Section 5: Joins (41–60)

Combine data from multiple tables.

```sql
-- 41. INNER JOIN
SELECT e.first_name, d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id;

-- 42. LEFT JOIN
SELECT e.first_name, d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id;

-- 43. LEFT JOIN to find rows in left table with no match in right
SELECT e.first_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id
WHERE d.department_id IS NULL;

-- 44. RIGHT JOIN
SELECT e.first_name, d.department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.department_id;

-- 45. FULL OUTER JOIN
SELECT e.first_name, d.department_name
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.department_id;

-- 46. Join three tables
SELECT e.first_name, p.project_name, d.department_name
FROM employees e
JOIN projects p ON e.employee_id = p.employee_id
JOIN departments d ON e.department_id = d.department_id;

-- 47. Self-Join (employee to manager)
SELECT e.first_name AS employee, m.first_name AS manager
FROM employees e
JOIN employees m ON e.manager_id = m.employee_id;

-- 48. Join on a non-key column
SELECT o.order_id, c.customer_name
FROM orders o
JOIN customers c ON o.customer_zip_code = c.customer_zip_code;

-- 49. CROSS JOIN
SELECT * FROM employees CROSS JOIN departments;

-- 50. LEFT JOIN with filtering (find products without sales)
SELECT p.*
FROM products p
LEFT JOIN sales s ON p.product_id = s.product_id
WHERE s.sale_id IS NULL;

-- 51. Employees not assigned to any project
SELECT e.first_name, e.last_name
FROM employees e
LEFT JOIN employee_projects ep ON e.employee_id = ep.employee_id
WHERE ep.project_id IS NULL;

-- 52. Projects without any employees assigned
SELECT p.project_name
FROM projects p
LEFT JOIN employee_projects ep ON p.project_id = ep.project_id
WHERE ep.employee_id IS NULL;

-- 53. Total salary of each department, including departments with no employees
SELECT d.department_name, SUM(e.salary) AS total_salary
FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id
GROUP BY d.department_name;

-- 54. Number of employees in each department (including zero)
SELECT d.department_name, COUNT(e.employee_id) AS num_employees
FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id
GROUP BY d.department_name;

-- 55. Join based on multiple conditions
SELECT o.*, c.customer_name
FROM orders o
JOIN customers c
  ON o.customer_id = c.customer_id
 AND o.order_date = c.last_purchase_date;

-- 56. Employees, their managers, and managers' departments
SELECT e.first_name,
       m.first_name AS manager_name,
       d.department_name
FROM employees e
JOIN employees m ON e.manager_id = m.employee_id
JOIN departments d ON m.department_id = d.department_id;

-- 57. Non-equi join (range join)
SELECT e.employee_id, h.salary_grade
FROM employees e
JOIN salary_grades h ON e.salary BETWEEN h.min_salary AND h.max_salary;

-- 58. Filter a joined table using WHERE
SELECT e.first_name, d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.department_id
WHERE d.location_city = 'New York';

-- 59. Aggregation on a joined table
SELECT d.department_name, AVG(e.salary) AS avg_salary
FROM employees e
JOIN departments d ON e.department_id = d.department_id
GROUP BY d.department_name;

-- 60. Join on common column with different names
SELECT a.order_id, b.item_name
FROM table_a a
JOIN table_b b ON a.product_id = b.item_id;
```

---

## Section 6: Subqueries (61–75)

Use a query within another query.

```sql
-- 61. Subquery in WHERE (single value)
SELECT * FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- 62. Subquery in WHERE (multiple values)
SELECT * FROM employees
WHERE department_id IN (
  SELECT department_id
  FROM departments
  WHERE location_city = 'San Francisco'
);

-- 63. Subquery in FROM (derived table)
SELECT d.department_name, s.avg_salary
FROM (
  SELECT department_id, AVG(salary) AS avg_salary
  FROM employees
  GROUP BY department_id
) AS s
JOIN departments d ON s.department_id = d.department_id;

-- 64. Subquery in SELECT (scalar subquery)
SELECT first_name, salary,
       (SELECT AVG(salary) FROM employees) AS company_avg_salary
FROM employees;

-- 65. Correlated subquery: higher than dept average
SELECT *
FROM employees e
WHERE salary > (
  SELECT AVG(salary) FROM employees WHERE department_id = e.department_id
);

-- 66. EXISTS to check existence
SELECT d.department_name
FROM departments d
WHERE EXISTS (
  SELECT 1 FROM employees e
  WHERE e.department_id = d.department_id AND e.salary > 100000
);

-- 67. NOT EXISTS to find non-matching rows
SELECT d.department_name
FROM departments d
WHERE NOT EXISTS (
  SELECT 1 FROM employees e
  WHERE e.department_id = d.department_id
);

-- 68. Departments with at least one employee
SELECT *
FROM departments
WHERE department_id IN (SELECT DISTINCT department_id FROM employees);

-- 69. Employees who have placed an order
SELECT *
FROM employees e
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.employee_id = e.employee_id);

-- 70. Total salary for each department using a subquery
SELECT d.department_name,
       (SELECT SUM(salary) FROM employees e WHERE e.department_id = d.department_id)
       AS total_department_salary
FROM departments d;

-- 71. Department with the highest average salary
SELECT department_name
FROM departments
WHERE department_id = (
  SELECT department_id
  FROM employees
  GROUP BY department_id
  ORDER BY AVG(salary) DESC
  LIMIT 1
);

-- 72. Customers who ordered a specific product
SELECT customer_name
FROM customers
WHERE customer_id IN (
  SELECT customer_id FROM orders WHERE product_id = 123
);

-- 73. Employees earning more than the average
SELECT COUNT(*)
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- 74. Customers who have not placed an order
SELECT customer_name
FROM customers
WHERE customer_id NOT IN (
  SELECT DISTINCT customer_id FROM orders
);

-- 75. Salary less than the average of their job title
SELECT e.first_name, e.salary, e.job_title
FROM employees e
WHERE e.salary < (
  SELECT AVG(salary) FROM employees WHERE job_title = e.job_title
);
```

---

## Section 7: Window Functions (76–85)

Perform calculations across a set of table rows related to the current row.

```sql
-- 76. ROW_NUMBER()
SELECT first_name, department, salary,
       ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS rn
FROM employees;

-- 77. RANK()
SELECT first_name, department, salary,
       RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rank
FROM employees;

-- 78. DENSE_RANK()
SELECT first_name, department, salary,
       DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS dense_rank
FROM employees;

-- 79. NTILE(n)
SELECT first_name, salary,
       NTILE(4) OVER (ORDER BY salary DESC) AS quartile
FROM employees;

-- 80. LEAD()
SELECT order_date, total_amount,
       LEAD(total_amount, 1) OVER (ORDER BY order_date) AS next_order_amount
FROM orders;

-- 81. LAG()
SELECT order_date, total_amount,
       LAG(total_amount, 1, 0) OVER (ORDER BY order_date) AS previous_order_amount
FROM orders;

-- 82. Running total
SELECT order_date, total_amount,
       SUM(total_amount) OVER (ORDER BY order_date
         ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total
FROM orders;

-- 83. Moving average (window = 3)
SELECT order_date, total_amount,
       AVG(total_amount) OVER (ORDER BY order_date
         ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS moving_avg
FROM orders;

-- 84. Top 3 employees in each department
SELECT *
FROM (
  SELECT first_name, department, salary,
         RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rn
  FROM employees
) AS ranked_employees
WHERE rn <= 3;

-- 85. Percentage of total sales for each product
SELECT product_id,
       SUM(sales) AS product_sales,
       SUM(SUM(sales)) OVER () AS total_sales,
       (SUM(sales) / SUM(SUM(sales)) OVER ()) * 100 AS percentage_of_total
FROM sales
GROUP BY product_id;
```

---

## Section 8: Common Table Expressions (CTEs) & Data Manipulation (86–100)

Organize complex queries and modify data.

```sql
-- 86. Use a CTE to simplify a query
WITH department_avg AS (
  SELECT department_id, AVG(salary) AS avg_dept_salary
  FROM employees GROUP BY department_id
)
SELECT e.first_name, e.salary, da.avg_dept_salary
FROM employees e
JOIN department_avg da ON e.department_id = da.department_id
WHERE e.salary > da.avg_dept_salary;

-- 87. INSERT INTO: Add a new row
INSERT INTO employees (first_name, last_name, salary)
VALUES ('John', 'Doe', 60000);

-- 88. UPDATE: Modify existing data
UPDATE employees SET salary = salary * 1.05 WHERE department = 'IT';

-- 89. DELETE: Remove rows
DELETE FROM employees WHERE employee_id = 101;

-- 90. TRUNCATE TABLE: Remove all rows quickly
TRUNCATE TABLE old_data;

-- 91. UNION: Combine distinct rows
SELECT first_name FROM employees
UNION
SELECT first_name FROM customers;

-- 92. UNION ALL: Combine including duplicates
SELECT first_name FROM employees
UNION ALL
SELECT first_name FROM customers;

-- 93. CASE statement for conditional logic
SELECT first_name, salary,
  CASE
    WHEN salary > 100000 THEN 'High Earner'
    WHEN salary BETWEEN 50000 AND 100000 THEN 'Mid-Range'
    ELSE 'Junior'
  END AS salary_level
FROM employees;

-- 94. Pivot data using CASE and GROUP BY
SELECT
  department,
  COUNT(CASE WHEN salary > 70000 THEN 1 END) AS high_salary_count,
  COUNT(CASE WHEN salary <= 70000 THEN 1 END) AS low_salary_count
FROM employees
GROUP BY department;

-- 95. CAST: Convert data type
SELECT CAST(order_date AS DATE) FROM orders;

-- 96. COALESCE: First non-null
SELECT COALESCE(email, 'No Email Provided') FROM employees;

-- 97. NULLIF: Returns NULL if equal
SELECT NULLIF(salary, 0) FROM employees;

-- 98. Recursive CTE
WITH RECURSIVE subordinates AS (
  SELECT employee_id, manager_id FROM employees WHERE employee_id = 1
  UNION ALL
  SELECT e.employee_id, e.manager_id
  FROM employees e
  JOIN subordinates s ON e.manager_id = s.employee_id
)
SELECT * FROM subordinates;

-- 99. GROUPING SETS
SELECT department, job_title, SUM(salary)
FROM employees
GROUP BY GROUPING SETS ((department), (job_title), ());

-- 100. ROLLUP
SELECT department, job_title, SUM(salary)
FROM employees
GROUP BY ROLLUP(department, job_title);
```
