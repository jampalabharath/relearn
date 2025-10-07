# Joins

This document provides an overview of **SQL Joins**, which allow combining data from multiple tables to retrieve meaningful insights.

---

## 1. Basic Joins

### No Join
```sql
-- Retrieve all data from customers and orders as separate results
SELECT * FROM customers;
SELECT * FROM orders;
```
### INNER JOIN

```sql
-- Get all customers along with their orders (only those who placed an order)
SELECT
    c.id,
    c.first_name,
    o.order_id,
    o.sales
FROM customers AS c
INNER JOIN orders AS o
ON c.id = o.customer_id;
```

### LEFT JOIN
```sql
-- Get all customers along with their orders (including customers without orders)
SELECT
    c.id,
    c.first_name,
    o.order_id,
    o.sales
FROM customers AS c
LEFT JOIN orders AS o
ON c.id = o.customer_id;
```

### RIGHT JOIN
```sql
-- Get all customers along with their orders (including orders without customers)
SELECT
    c.id,
    c.first_name,
    o.order_id,
    o.customer_id,
    o.sales
FROM customers AS c 
RIGHT JOIN orders AS o 
ON c.id = o.customer_id;

Alternative to RIGHT JOIN (using LEFT JOIN)
SELECT
    c.id,
    c.first_name,
    o.order_id,
    o.sales
FROM orders AS o 
LEFT JOIN customers AS c
ON c.id = o.customer_id;
```

### FULL JOIN

```sql
-- Get all customers and all orders, even if there’s no match
SELECT
    c.id,
    c.first_name,
    o.order_id,
    o.customer_id,
    o.sales
FROM customers AS c 
FULL JOIN orders AS o 
ON c.id = o.customer_id;
```
## 2. Advanced Joins
### LEFT ANTI JOIN
```sql
-- Customers who haven't placed any order
SELECT *
FROM customers AS c
LEFT JOIN orders AS o
ON c.id = o.customer_id
WHERE o.customer_id IS NULL;
```

### RIGHT ANTI JOIN
```sql
-- Orders without matching customers
SELECT *
FROM customers AS c
RIGHT JOIN orders AS o
ON c.id = o.customer_id
WHERE c.id IS NULL;

Alternative to RIGHT ANTI JOIN (using LEFT JOIN)
SELECT *
FROM orders AS o 
LEFT JOIN customers AS c
ON c.id = o.customer_id
WHERE c.id IS NULL;

Alternative to INNER JOIN (using LEFT JOIN)
SELECT *
FROM customers AS c
LEFT JOIN orders AS o
ON c.id = o.customer_id
WHERE o.customer_id IS NOT NULL;
```

### FULL ANTI JOIN
```sql
-- Find customers without orders and orders without customers
SELECT
    c.id,
    c.first_name,
    o.order_id,
    o.customer_id,
    o.sales
FROM customers AS c 
FULL JOIN orders AS o 
ON c.id = o.customer_id
WHERE o.customer_id IS NULL OR c.id IS NULL;
```
### CROSS JOIN
```sql
-- Generate all possible combinations of customers and orders
SELECT *
FROM customers
CROSS JOIN orders;
```

## 3. Multiple Table Joins (4 Tables)

**Task**: Using SalesDB, retrieve a list of all orders along with related customer, product, and employee details.
For each order, display:

Order ID

Customer's name

Product name

Sales amount

Product price

Salesperson's name

```sql
USE SalesDB;

SELECT 
    o.OrderID,
    o.Sales,
    c.FirstName AS CustomerFirstName,
    c.LastName AS CustomerLastName,
    p.Product AS ProductName,
    p.Price,
    e.FirstName AS EmployeeFirstName,
    e.LastName AS EmployeeLastName
FROM Sales.Orders AS o
LEFT JOIN Sales.Customers AS c
ON o.CustomerID = c.CustomerID
LEFT JOIN Sales.Products AS p
ON o.ProductID = p.ProductID
LEFT JOIN Sales.Employees AS e
ON o.SalesPersonID = e.EmployeeID;
```