# Subquery Functions

This script demonstrates various subquery techniques in SQL.  
It covers result types, subqueries in the `FROM` clause, in `SELECT`, in `JOIN` clauses, with comparison operators, `IN`, `ANY`, correlated subqueries, and `EXISTS`.

---

## Table of Contents
1. SUBQUERY - RESULT TYPES  
2. SUBQUERY - FROM CLAUSE  
3. SUBQUERY - SELECT  
4. SUBQUERY - JOIN CLAUSE  
5. SUBQUERY - COMPARISON OPERATORS  
6. SUBQUERY - IN OPERATOR  
7. SUBQUERY - ANY OPERATOR  
8. SUBQUERY - CORRELATED  
9. SUBQUERY - EXISTS OPERATOR  

---

## SUBQUERY | RESULT TYPES

### Scalar Query
```sql
SELECT
    AVG(Sales)
FROM Sales.Orders;
```

### Row Query
```sql
SELECT
    CustomerID
FROM Sales.Orders;
```

### Table Query
```sql
SELECT
    OrderID,
    OrderDate
FROM Sales.Orders;
```

## SUBQUERY | FROM CLAUSE
### Task 1: Products with Price Higher than Average Price
```sql
SELECT
*
FROM (
    SELECT
        ProductID,
        Price,
        AVG(Price) OVER () AS AvgPrice
    FROM Sales.Products
) AS t
WHERE Price > AvgPrice;
```

### Task 2: Rank Customers by Total Sales
```sql
SELECT
    *,
    RANK() OVER (ORDER BY TotalSales DESC) AS CustomerRank
FROM (
    SELECT
        CustomerID,
        SUM(Sales) AS TotalSales
    FROM Sales.Orders
    GROUP BY CustomerID
) AS t;
```

## SUBQUERY | SELECT
### Task 3: Product Details with Total Number of Orders
```sql
SELECT
    ProductID,
    Product,
    Price,
    (SELECT COUNT(*) FROM Sales.Orders) AS TotalOrders
FROM Sales.Products;
```

## SUBQUERY | JOIN CLAUSE
### Task 4: Customer Details with Total Sales
```sql
SELECT
    c.*,
    t.TotalSales
FROM Sales.Customers AS c
LEFT JOIN ( 
    SELECT
        CustomerID,
        SUM(Sales) AS TotalSales
    FROM Sales.Orders
    GROUP BY CustomerID
) AS t
    ON c.CustomerID = t.CustomerID;
```

### Task 5: Customer Details with Total Orders
```sql
SELECT
    c.*,
    o.TotalOrders
FROM Sales.Customers AS c
LEFT JOIN (
    SELECT
        CustomerID,
        COUNT(*) AS TotalOrders
    FROM Sales.Orders
    GROUP BY CustomerID
) AS o
    ON c.CustomerID = o.CustomerID;
```

## SUBQUERY | COMPARISON OPERATORS
### Task 6: Products with Price Higher than Average Price
```sql
SELECT
    ProductID,
    Price,
    (SELECT AVG(Price) FROM Sales.Products) AS AvgPrice
FROM Sales.Products
WHERE Price > (SELECT AVG(Price) FROM Sales.Products);
```

## SUBQUERY | IN OPERATOR
### Task 7: Orders Made by Customers in Germany
```sql
SELECT
    *
FROM Sales.Orders
WHERE CustomerID IN (
    SELECT
        CustomerID
    FROM Sales.Customers
    WHERE Country = 'Germany'
);
```

### Task 8: Orders Made by Customers Not in Germany
```sql
SELECT
    *
FROM Sales.Orders
WHERE CustomerID NOT IN (
    SELECT
        CustomerID
    FROM Sales.Customers
    WHERE Country = 'Germany'
);
```

## SUBQUERY | ANY OPERATOR
### Task 9: Female Employees with Salaries Greater than Any Male Employee
```sql
SELECT
    EmployeeID, 
    FirstName,
    Salary
FROM Sales.Employees
WHERE Gender = 'F'
  AND Salary > ANY (
      SELECT Salary
      FROM Sales.Employees
      WHERE Gender = 'M'
  );
```

## CORRELATED SUBQUERY
### Task 10: Customer Details with Total Orders (Correlated)
```sql
SELECT
    *,
    (SELECT COUNT(*)
     FROM Sales.Orders o
     WHERE o.CustomerID = c.CustomerID) AS TotalSales
FROM Sales.Customers AS c;
```

## SUBQUERY | EXISTS OPERATOR
### Task 11: Orders Made by Customers in Germany
```sql
SELECT
    *
FROM Sales.Orders AS o
WHERE EXISTS (
    SELECT 1
    FROM Sales.Customers AS c
    WHERE Country = 'Germany'
      AND o.CustomerID = c.CustomerID
);
```

### Task 12: Orders Made by Customers Not in Germany
```sql
SELECT
    *
FROM Sales.Orders AS o
WHERE NOT EXISTS (
    SELECT 1
    FROM Sales.Customers AS c
    WHERE Country = 'Germany'
      AND o.CustomerID = c.CustomerID
);
```