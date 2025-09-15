---
title: "Performance Tips"
weight: 26
---


This guide demonstrates best practices for:  
- Fetching data  
- Filtering  
- Joins  
- UNION  
- Aggregations  
- Subqueries/CTE  
- DDL  
- Indexing  

It covers techniques such as selecting only necessary columns, proper filtering methods, explicit joins, avoiding redundant logic, and efficient indexing strategies.

---

## Fetching Data

### Tip 1: Select Only What You Need
```sql
-- Bad Practice
SELECT * FROM Sales.Customers;

-- Good Practice
SELECT CustomerID, FirstName, LastName FROM Sales.Customers;
```


### Tip 2: Avoid unnecessary DISTINCT & ORDER BY
```sql
-- Bad Practice
SELECT DISTINCT FirstName 
FROM Sales.Customers 
ORDER BY FirstName;

-- Good Practice
SELECT FirstName 
FROM Sales.Customers;
```

### Tip 3: For Exploration Purpose, Limit Rows!
```sql
-- Bad Practice
SELECT OrderID, Sales 
FROM Sales.Orders;

-- Good Practice
SELECT TOP 10 OrderID, Sales 
FROM Sales.Orders;
```
## Filtering
### Tip 4: Create nonclustered index on frequently used columns
```sql
SELECT * 
FROM Sales.Orders
WHERE OrderStatus = 'Delivered';

CREATE NONCLUSTERED INDEX Idx_Orders_OrderStatus 
ON Sales.Orders(OrderStatus);
```

### Tip 5: Avoid applying functions in WHERE
```sql
-- Bad
SELECT * FROM Sales.Orders 
WHERE LOWER(OrderStatus) = 'delivered';

-- Good
SELECT * FROM Sales.Orders 
WHERE OrderStatus = 'Delivered';

-- Bad
SELECT * FROM Sales.Customers
WHERE SUBSTRING(FirstName, 1, 1) = 'A';

-- Good
SELECT * FROM Sales.Customers
WHERE FirstName LIKE 'A%';

-- Bad
SELECT * FROM Sales.Orders 
WHERE YEAR(OrderDate) = 2025;

-- Good
SELECT * FROM Sales.Orders 
WHERE OrderDate BETWEEN '2025-01-01' AND '2025-12-31';
```

### Tip 6: Avoid leading wildcards
```sql
-- Bad
SELECT * FROM Sales.Customers 
WHERE LastName LIKE '%Gold%';

-- Good
SELECT * FROM Sales.Customers 
WHERE LastName LIKE 'Gold%';
```
### Tip 7: Use IN instead of multiple OR
```sql
-- Bad
SELECT * FROM Sales.Orders
WHERE CustomerID = 1 OR CustomerID = 2 OR CustomerID = 3;

-- Good
SELECT * FROM Sales.Orders
WHERE CustomerID IN (1, 2, 3);
```
## Joins
### Tip 8: Use INNER JOIN when possible
```sql
-- Best
SELECT c.FirstName, o.OrderID
FROM Sales.Customers c 
INNER JOIN Sales.Orders o ON c.CustomerID = o.CustomerID;

-- Slower
SELECT c.FirstName, o.OrderID
FROM Sales.Customers c 
LEFT JOIN Sales.Orders o ON c.CustomerID = o.CustomerID;

-- Worst
SELECT c.FirstName, o.OrderID
FROM Sales.Customers c 
OUTER JOIN Sales.Orders o ON c.CustomerID = o.CustomerID;
```

### Tip 9: Use explicit (ANSI) joins
```sql
-- Bad
SELECT o.OrderID, c.FirstName
FROM Sales.Customers c, Sales.Orders o
WHERE c.CustomerID = o.CustomerID;

-- Good
SELECT o.OrderID, c.FirstName
FROM Sales.Customers c
INNER JOIN Sales.Orders o ON c.CustomerID = o.CustomerID;
```

### Tip 10: Index columns used in ON clause
```sql
SELECT c.FirstName, o.OrderID
FROM Sales.Orders o
INNER JOIN Sales.Customers c ON c.CustomerID = o.CustomerID;

CREATE NONCLUSTERED INDEX IX_Orders_CustomerID 
ON Sales.Orders(CustomerID);
```

### Tip 11: Filter before joining big tables
```sql
-- Best for big tables
SELECT c.FirstName, o.OrderID
FROM Sales.Customers c
INNER JOIN (
    SELECT OrderID, CustomerID
    FROM Sales.Orders
    WHERE OrderStatus = 'Delivered'
) o ON c.CustomerID = o.CustomerID;
```

### Tip 12: Aggregate before joining
```sql
-- Best for big tables
SELECT c.CustomerID, c.FirstName, o.OrderCount
FROM Sales.Customers c
INNER JOIN (
    SELECT CustomerID, COUNT(OrderID) AS OrderCount
    FROM Sales.Orders
    GROUP BY CustomerID
) o ON c.CustomerID = o.CustomerID;
```

### Tip 13: Use UNION instead of OR in joins
```sql
-- Bad
SELECT o.OrderID, c.FirstName
FROM Sales.Customers c
INNER JOIN Sales.Orders o
ON c.CustomerID = o.CustomerID
   OR c.CustomerID = o.SalesPersonID;

-- Good
SELECT o.OrderID, c.FirstName
FROM Sales.Customers c
INNER JOIN Sales.Orders o ON c.CustomerID = o.CustomerID
UNION
SELECT o.OrderID, c.FirstName
FROM Sales.Customers c
INNER JOIN Sales.Orders o ON c.CustomerID = o.SalesPersonID;
```

### Tip 14: Use SQL hints for optimization
```sql
SELECT o.OrderID, c.FirstName
FROM Sales.Customers c
INNER JOIN Sales.Orders o
ON c.CustomerID = o.CustomerID
OPTION (HASH JOIN);
```

## Union
### Tip 15: Use UNION ALL when duplicates are acceptable
```sql
-- Bad
SELECT CustomerID FROM Sales.Orders
UNION
SELECT CustomerID FROM Sales.OrdersArchive;

-- Good
SELECT CustomerID FROM Sales.Orders
UNION ALL
SELECT CustomerID FROM Sales.OrdersArchive;
```

### Tip 16: Use UNION ALL + DISTINCT when duplicates not allowed
```sql
SELECT DISTINCT CustomerID
FROM (
    SELECT CustomerID FROM Sales.Orders
    UNION ALL
    SELECT CustomerID FROM Sales.OrdersArchive
) CombinedData;
```

## Aggregations
### Tip 17: Use columnstore indexes
```sql
SELECT CustomerID, COUNT(OrderID) AS OrderCount
FROM Sales.Orders 
GROUP BY CustomerID;

CREATE CLUSTERED COLUMNSTORE INDEX Idx_Orders_Columnstore 
ON Sales.Orders;
```
### Tip 18: Pre-aggregate data
```sql
SELECT MONTH(OrderDate) AS OrderYear, SUM(Sales) AS TotalSales
INTO Sales.SalesSummary
FROM Sales.Orders
GROUP BY MONTH(OrderDate);

SELECT OrderYear, TotalSales 
FROM Sales.SalesSummary;
```

## Subqueries & CTE

### Tip 19: Prefer JOIN or EXISTS over IN
```sql
-- Good
SELECT o.OrderID, o.Sales
FROM Sales.Orders o
WHERE EXISTS (
    SELECT 1
    FROM Sales.Customers c
    WHERE c.CustomerID = o.CustomerID
      AND c.Country = 'USA'
);

-- Bad
SELECT o.OrderID, o.Sales
FROM Sales.Orders o
WHERE o.CustomerID IN (
    SELECT CustomerID FROM Sales.Customers WHERE Country = 'USA'
);
```

### Tip 20: Avoid redundant logic
```sql
-- Good
SELECT EmployeeID, FirstName,
    CASE 
        WHEN Salary > AVG(Salary) OVER () THEN 'Above Average'
        WHEN Salary < AVG(Salary) OVER () THEN 'Below Average'
        ELSE 'Average'
    END AS Status
FROM Sales.Employees;
```

## DDL

### Tip 21: Avoid VARCHAR(MAX) unless necessary.

### Tip 22: Avoid overly large lengths.

### Tip 23: Use NOT NULL when possible.

### Tip 24: All tables should have a clustered primary key.

### Tip 25: Create nonclustered indexes on foreign keys when frequently used.
```sql
-- Good Practice
CREATE TABLE CustomersInfo (
    CustomerID INT PRIMARY KEY CLUSTERED,
    FirstName VARCHAR(50) NOT NULL,
    LastName VARCHAR(50) NOT NULL,
    Country VARCHAR(50) NOT NULL,
    TotalPurchases FLOAT,
    Score INT,
    BirthDate DATE,
    EmployeeID INT,
    CONSTRAINT FK_CustomersInfo_EmployeeID 
        FOREIGN KEY (EmployeeID) REFERENCES Sales.Employees(EmployeeID)
);

CREATE NONCLUSTERED INDEX IX_CustomersInfo_EmployeeID
ON CustomersInfo(EmployeeID);
```

## Indexing

### Tip 26: Avoid over-indexing (slows down writes).

### Tip 27: Drop unused indexes regularly.

### Tip 28: Update table statistics weekly.

### Tip 29: Reorganize/rebuild fragmented indexes weekly.

### Tip 30: For very large tables, partition + columnstore index.