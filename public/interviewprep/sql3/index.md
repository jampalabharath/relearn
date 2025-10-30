# SQL cheat sheet

## 🧩 Data Manipulation Language (DML)

| **Command** | **Description** | **Syntax** | **Example** |
|--------------|----------------|-------------|--------------|
| **SELECT** | Retrieves data from a database. | `SELECT column1, column2 FROM table_name;` | `SELECT first_name, last_name FROM customers;` |
| **INSERT** | Adds new records to a table. | `INSERT INTO table_name (column1, column2) VALUES (value1, value2);` | `INSERT INTO customers (first_name, last_name) VALUES ('Mary', 'Doe');` |
| **UPDATE** | Modifies existing records in a table. | `UPDATE table_name SET column1 = value1, column2 = value2 WHERE condition;` | `UPDATE employees SET employee_name = 'John Doe', department = 'Marketing';` |
| **DELETE** | Removes records from a table. | `DELETE FROM table_name WHERE condition;` | `DELETE FROM employees WHERE employee_name = 'John Doe';` |

---

## 🏗️ Data Definition Language (DDL)

| **Command** | **Description** | **Syntax** | **Example** |
|--------------|----------------|-------------|--------------|
| **CREATE** | Creates a new database object (table, view, etc.). | `CREATE TABLE table_name (column1 datatype1, column2 datatype2, ...);` | `CREATE TABLE employees (employee_id INT PRIMARY KEY, first_name VARCHAR(50), last_name VARCHAR(50), age INT);` |
| **ALTER** | Adds, deletes, or modifies columns in an existing table. | `ALTER TABLE table_name ADD column_name datatype;` | `ALTER TABLE customers ADD email VARCHAR(100);` |
| **DROP** | Deletes an existing table or database object. | `DROP TABLE table_name;` | `DROP TABLE customers;` |
| **TRUNCATE** | Removes all rows from a table but keeps the structure. | `TRUNCATE TABLE table_name;` | `TRUNCATE TABLE customers;` |

---

## 🔐 Data Control Language (DCL)

| **Command** | **Description** | **Syntax** | **Example** |
|--------------|----------------|-------------|--------------|
| **GRANT** | Gives privileges to users or roles. | `GRANT SELECT, INSERT ON table_name TO user_name;` | `GRANT SELECT, INSERT ON employees TO 'John Doe';` |
| **REVOKE** | Removes privileges previously granted. | `REVOKE SELECT, INSERT ON table_name FROM user_name;` | `REVOKE SELECT, INSERT ON employees FROM 'John Doe';` |

---

## 🔍 Querying Data

| **Command** | **Description** | **Syntax** | **Example** |
|--------------|----------------|-------------|--------------|
| **SELECT Statement** | Retrieves data from one or more tables. | `SELECT column1, column2 FROM table_name;` | `SELECT first_name, last_name FROM customers;` |
| **WHERE Clause** | Filters rows based on specific conditions. | `SELECT * FROM table_name WHERE condition;` | `SELECT * FROM customers WHERE age > 30;` |
| **ORDER BY Clause** | Sorts the result set by one or more columns. | `SELECT * FROM table_name ORDER BY column_name ASC/DESC;` | `SELECT * FROM products ORDER BY price DESC;` |
| **GROUP BY Clause** | Groups rows sharing a property and is used with aggregates. | `SELECT column_name, COUNT(*) FROM table_name GROUP BY column_name;` | `SELECT category, COUNT(*) FROM products GROUP BY category;` |
| **HAVING Clause** | Filters grouped results (used with GROUP BY). | `SELECT column_name, COUNT(*) FROM table_name GROUP BY column_name HAVING condition;` | `SELECT category, COUNT(*) FROM products GROUP BY category HAVING COUNT(*) > 5;` |


## 🔗 Joining Commands

| **Command** | **Description** | **Syntax** | **Example** |
|--------------|----------------|-------------|--------------|
| **INNER JOIN** | Returns rows with matching values in both tables. | `SELECT * FROM table1 INNER JOIN table2 ON table1.column = table2.column;` | `SELECT * FROM employees INNER JOIN departments ON employees.department_id = departments.id;` |
| **LEFT JOIN / LEFT OUTER JOIN** | Returns all rows from the left table and matching rows from the right table. | `SELECT * FROM table1 LEFT JOIN table2 ON table1.column = table2.column;` | `SELECT * FROM employees LEFT JOIN departments ON employees.department_id = departments.id;` |
| **RIGHT JOIN / RIGHT OUTER JOIN** | Returns all rows from the right table and matching rows from the left table. | `SELECT * FROM table1 RIGHT JOIN table2 ON table1.column = table2.column;` | `SELECT * FROM employees RIGHT JOIN departments ON employees.department_id = departments.department_id;` |
| **FULL JOIN / FULL OUTER JOIN** | Returns all rows when there’s a match in either table. | `SELECT * FROM table1 FULL JOIN table2 ON table1.column = table2.column;` | ```sql SELECT * FROM employees LEFT JOIN departments ON employees.employee_id = departments.employee_id UNION SELECT * FROM employees RIGHT JOIN departments ON employees.employee_id = departments.employee_id; ``` |
| **CROSS JOIN** | Combines every row from the first table with every row from the second (Cartesian product). | `SELECT * FROM table1 CROSS JOIN table2;` | `SELECT * FROM employees CROSS JOIN departments;` |
| **SELF JOIN** | Joins a table with itself. | `SELECT * FROM table1 t1, table1 t2 WHERE t1.column = t2.column;` | `SELECT * FROM employees t1, employees t2 WHERE t1.employee_id = t2.employee_id;` |
| **NATURAL JOIN** | Matches columns with the same name in both tables. | `SELECT * FROM table1 NATURAL JOIN table2;` | `SELECT * FROM employees NATURAL JOIN departments;` |

---

## 🧮 Subqueries in SQL

| **Command** | **Description** | **Syntax** | **Example** |
|--------------|----------------|-------------|--------------|
| **IN** | Checks if a value matches any in a subquery result. | `SELECT column(s) FROM table WHERE value IN (subquery);` | `SELECT * FROM customers WHERE city IN (SELECT city FROM suppliers);` |
| **ANY** | Compares a value to any returned by a subquery (used with =, >, <, etc.). | `SELECT column(s) FROM table WHERE value < ANY (subquery);` | `SELECT * FROM products WHERE price < ANY (SELECT unit_price FROM supplier_products);` |
| **ALL** | Compares a value to all values returned by a subquery. | `SELECT column(s) FROM table WHERE value > ALL (subquery);` | `SELECT * FROM orders WHERE order_amount > ALL (SELECT total_amount FROM previous_orders);` |

---

## 📊 Aggregate Functions

| **Command** | **Description** | **Syntax** | **Example** |
|--------------|----------------|-------------|--------------|
| **COUNT()** | Counts rows or non-null values in a column. | `SELECT COUNT(column_name) FROM table_name;` | `SELECT COUNT(age) FROM employees;` |
| **SUM()** | Calculates the sum of values in a column. | `SELECT SUM(column_name) FROM table_name;` | `SELECT SUM(revenue) FROM sales;` |
| **AVG()** | Calculates the average of values in a column. | `SELECT AVG(column_name) FROM table_name;` | `SELECT AVG(price) FROM products;` |
| **MIN()** | Returns the minimum value in a column. | `SELECT MIN(column_name) FROM table_name;` | `SELECT MIN(price) FROM products;` |
| **MAX()** | Returns the maximum value in a column. | `SELECT MAX(column_name) FROM table_name;` | `SELECT MAX(price) FROM products;` |

---

## 🔤 String Functions

| **Command** | **Description** | **Syntax** | **Example** |
|--------------|----------------|-------------|--------------|
| **CONCAT()** | Concatenates two or more strings. | `SELECT CONCAT(string1, string2, ...) AS concatenated_string FROM table_name;` | `SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM employees;` |
| **SUBSTRING() / SUBSTR()** | Extracts a substring from a string. | `SELECT SUBSTRING(string FROM start_position [FOR length]) AS substring FROM table_name;` | `SELECT SUBSTRING(product_name FROM 1 FOR 5) AS substring FROM products;` |
| **CHAR_LENGTH() / LENGTH()** | Returns the number of characters in a string. | `SELECT CHAR_LENGTH(string) AS length FROM table_name;` | `SELECT CHAR_LENGTH(product_name) AS length FROM products;` |
| **UPPER()** | Converts all characters to uppercase. | `SELECT UPPER(string) AS uppercase_string FROM table_name;` | `SELECT UPPER(first_name) AS uppercase_first_name FROM employees;` |
| **LOWER()** | Converts all characters to lowercase. | `SELECT LOWER(string) AS lowercase_string FROM table_name;` | `SELECT LOWER(last_name) AS lowercase_last_name FROM employees;` |
| **TRIM()** | Removes specified prefixes, suffixes, or whitespace. | `SELECT TRIM([LEADING / TRAILING / BOTH] characters FROM string) AS trimmed_string FROM table_name;` | `SELECT TRIM(TRAILING ' ' FROM full_name) AS trimmed_full_name FROM customers;` |
| **LEFT()** | Returns characters from the left of a string. | `SELECT LEFT(string, num_characters) AS left_string FROM table_name;` | `SELECT LEFT(product_name, 5) AS left_product_name FROM products;` |
| **RIGHT()** | Returns characters from the right of a string. | `SELECT RIGHT(string, num_characters) AS right_string FROM table_name;` | `SELECT RIGHT(order_number, 4) AS right_order_number FROM orders;` |
| **REPLACE()** | Replaces occurrences of a substring within a string. | `SELECT REPLACE(string, old_substring, new_substring) AS replaced_string FROM table_name;` | `SELECT REPLACE(description, 'old_string', 'new_string') AS replaced_description FROM product_descriptions;` |


## 🕒 Date and Time SQL Commands

| **Command** | **Description** | **Syntax** | **Example** |
|--------------|----------------|-------------|--------------|
| **CURRENT_DATE()** | Returns the current date. | `SELECT CURRENT_DATE() AS current_date;` | — |
| **CURRENT_TIME()** | Returns the current time. | `SELECT CURRENT_TIME() AS current_time;` | — |
| **CURRENT_TIMESTAMP()** | Returns the current date and time. | `SELECT CURRENT_TIMESTAMP() AS current_timestamp;` | — |
| **DATE_PART()** | Extracts a specific part (year, month, day, etc.) from a date or time. | `SELECT DATE_PART('part', date_expression) AS extracted_part;` | — |
| **DATE_ADD() / DATE_SUB()** | Adds or subtracts a specified interval (days, months, years) to/from a date. | `SELECT DATE_ADD(date_expression, INTERVAL value unit) AS new_date;` | — |
| **EXTRACT()** | Extracts a specific component from a date or time. | `SELECT EXTRACT(part FROM date_expression) AS extracted_part;` | — |
| **TO_CHAR()** | Converts a date or time to a specific string format. | `SELECT TO_CHAR(date_expression, 'format') AS formatted_date;` | — |
| **TIMESTAMPDIFF()** | Calculates the difference between two timestamps in a specified unit. | `SELECT TIMESTAMPDIFF(unit, timestamp1, timestamp2) AS difference;` | — |
| **DATEDIFF()** | Returns the number of days between two dates. | `SELECT DATEDIFF(date1, date2) AS difference_in_days;` | — |

---

## ⚙️ Conditional Expressions

| **Command** | **Description** | **Syntax** | **Example** |
|--------------|----------------|-------------|--------------|
| **CASE Statement** | Performs conditional logic within a query. | ```sql SELECT column1, column2, CASE WHEN condition1 THEN result1 WHEN condition2 THEN result2 ELSE default_result END AS alias FROM table_name; ``` | ```sql SELECT order_id, total_amount, CASE WHEN total_amount > 1000 THEN 'High Value Order' WHEN total_amount > 500 THEN 'Medium Value Order' ELSE 'Low Value Order' END AS order_status FROM orders; ``` |
| **IF() Function** | Evaluates a condition and returns a value based on the evaluation. | `SELECT IF(condition, true_value, false_value) AS alias FROM table_name;` | `SELECT name, age, IF(age > 50, 'Senior', 'Junior') AS employee_category FROM employees;` |
| **COALESCE() Function** | Returns the first non-null value from a list. | `SELECT COALESCE(value1, value2, ...) AS alias FROM table_name;` | `SELECT COALESCE(first_name, middle_name) AS preferred_name FROM employees;` |
| **NULLIF() Function** | Returns NULL if two expressions are equal. | `SELECT NULLIF(expression1, expression2) AS alias FROM table_name;` | `SELECT NULLIF(total_amount, discounted_amount) AS diff_amount FROM orders;` |

---

## 🔁 Set Operations

| **Command** | **Description** | **Syntax** | **Example** |
|--------------|----------------|-------------|--------------|
| **UNION** | Combines result sets of two or more `SELECT` statements (removes duplicates). | ```sql SELECT column1, column2 FROM table1 UNION SELECT column1, column2 FROM table2; ``` | ```sql SELECT first_name, last_name FROM customers UNION SELECT first_name, last_name FROM employees; ``` |
| **INTERSECT** | Returns common rows that appear in both result sets. | ```sql SELECT column1, column2 FROM table1 INTERSECT SELECT column1, column2 FROM table2; ``` | ```sql SELECT first_name, last_name FROM customers INTERSECT SELECT first_name, last_name FROM employees; ``` |
| **EXCEPT** | Returns rows from the first query that are not present in the second. | ```sql SELECT column1, column2 FROM table1 EXCEPT SELECT column1, column2 FROM table2; ``` | ```sql SELECT first_name, last_name FROM customers EXCEPT SELECT first_name, last_name FROM employees; ``` |

---


## 💾 Transaction Control Commands (TCL)

| **Command** | **Description** | **Syntax** | **Example** |
|--------------|----------------|-------------|--------------|
| **COMMIT** | Saves all the changes made during the current transaction and makes them permanent. | `COMMIT;` | ```sql BEGIN TRANSACTION; -- SQL statements and changes within the transaction INSERT INTO employees (name, age) VALUES ('Alice', 30); UPDATE products SET price = 25.00 WHERE category = 'Electronics'; COMMIT; ``` |
| **ROLLBACK** | Undoes all the changes made during the current transaction and discards them. | `ROLLBACK;` | ```sql BEGIN TRANSACTION; -- SQL statements and changes within the transaction INSERT INTO employees (name, age) VALUES ('Bob', 35); UPDATE products SET price = 30.00 WHERE category = 'Electronics'; ROLLBACK; ``` |
| **SAVEPOINT** | Sets a point within a transaction to which you can later roll back. | `SAVEPOINT savepoint_name;` | ```sql BEGIN TRANSACTION; INSERT INTO employees (name, age) VALUES ('Carol', 28); SAVEPOINT before_update; UPDATE products SET price = 40.00 WHERE category = 'Electronics'; SAVEPOINT after_update; DELETE FROM customers WHERE age > 60; ROLLBACK TO before_update; -- DELETE is rolled back, but UPDATE remains COMMIT; ``` |
| **ROLLBACK TO SAVEPOINT** | Rolls back the transaction to a specific savepoint without affecting prior operations. | `ROLLBACK TO SAVEPOINT savepoint_name;` | ```sql BEGIN TRANSACTION; INSERT INTO employees (name, age) VALUES ('David', 42); SAVEPOINT before_update; UPDATE products SET price = 50.00 WHERE category = 'Electronics'; SAVEPOINT after_update; DELETE FROM customers WHERE age > 60; ROLLBACK TO SAVEPOINT before_update; -- UPDATE is rolled back, but INSERT remains COMMIT; ``` |
| **SET TRANSACTION** | Configures properties for the current transaction, such as isolation level or mode. | `SET TRANSACTION [ISOLATION LEVEL { READ COMMITTED / SERIALIZABLE }];` | ```sql BEGIN TRANSACTION; -- Set isolation level to READ COMMITTED SET TRANSACTION ISOLATION LEVEL READ COMMITTED; INSERT INTO employees (name, age) VALUES ('Emily', 35); UPDATE products SET price = 60.00 WHERE category = 'Electronics'; COMMIT; ``` |




