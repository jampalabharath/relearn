+++
title = "Set Operations"
weight = 6
+++

SQL **set operations** enable you to combine results from multiple queries into a single result set.  
This guide demonstrates the rules and usage of **UNION, UNION ALL, EXCEPT, and INTERSECT**.

---

## 1. SQL Operation Rules

### Rule: Data Types
The data types of columns in each query should match.
```sql
SELECT
    FirstName,
    LastName,
    Country
FROM Sales.Customers
UNION
SELECT
    FirstName,
    LastName
FROM Sales.Employees;
```

### Rule: Data Types (Example)
```sql
SELECT
    CustomerID,
    LastName
FROM Sales.Customers
UNION
SELECT
    FirstName,
    LastName
FROM Sales.Employees;
```

### Rule: Column Order

The order of the columns in each query must be the same.

```sql
SELECT
    LastName,
    CustomerID
FROM Sales.Customers
UNION
SELECT
    EmployeeID,
    LastName
FROM Sales.Employees;
```

### Rule: Column Aliases

The column names in the result set are determined by the column names specified in the first SELECT statement.

```sql
SELECT
    CustomerID AS ID,
    LastName AS Last_Name
FROM Sales.Customers
UNION
SELECT
    EmployeeID,
    LastName
FROM Sales.Employees;
```

### Rule: Correct Columns

Ensure that the correct columns are used to maintain data consistency.

```sql
SELECT
    FirstName,
    LastName
FROM Sales.Customers
UNION
SELECT
    LastName,
    FirstName
FROM Sales.Employees;
```

## 2. UNION, UNION ALL, EXCEPT, INTERSECT
### Task 1: UNION

Combine the data from Employees and Customers into one table.
```sql
SELECT
    FirstName,
    LastName
FROM Sales.Customers
UNION
SELECT
    FirstName,
    LastName
FROM Sales.Employees;
```

### Task 2: UNION ALL

Combine the data from Employees and Customers into one table, including duplicates.
```sql
SELECT
    FirstName,
    LastName
FROM Sales.Customers
UNION ALL
SELECT
    FirstName,
    LastName
FROM Sales.Employees;
```

### Task 3: EXCEPT

Find employees who are NOT customers.

```sql
SELECT
    FirstName,
    LastName
FROM Sales.Employees
EXCEPT
SELECT
    FirstName,
    LastName
FROM Sales.Customers;
```

### Task 4: INTERSECT

Find employees who are also customers.
```sql
SELECT
    FirstName,
    LastName
FROM Sales.Employees
INTERSECT
SELECT
    FirstName,
    LastName
FROM Sales.Customers;
```

### Task 5: UNION with Orders

Combine order data from Orders and OrdersArchive into one report without duplicates.
```sql
SELECT
    'Orders' AS SourceTable,
    OrderID,
    ProductID,
    CustomerID,
    SalesPersonID,
    OrderDate,
    ShipDate,
    OrderStatus,
    ShipAddress,
    BillAddress,
    Quantity,
    Sales,
    CreationTime
FROM Sales.Orders
UNION
SELECT
    'OrdersArchive' AS SourceTable,
    OrderID,
    ProductID,
    CustomerID,
    SalesPersonID,
    OrderDate,
    ShipDate,
    OrderStatus,
    ShipAddress,
    BillAddress,
    Quantity,
    Sales,
    CreationTime
FROM Sales.OrdersArchive
ORDER BY OrderID;
```