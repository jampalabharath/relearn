# Window Aggregate Functions

These functions allow you to perform aggregate calculations over a set of rows without the need for complex subqueries. They enable you to compute counts, sums, averages, minimums, and maximums while still retaining access to individual row details.

## Table of Contents
1. COUNT
2. SUM
3. AVG
4. MAX / MIN
5. ROLLING SUM & AVERAGE Use Case

---

## COUNT

### Task 1: Find the Total Number of Orders and the Total Number of Orders for Each Customer
```sql
SELECT
    OrderID,
    OrderDate,
    CustomerID,
    COUNT(*) OVER() AS TotalOrders,
    COUNT(*) OVER(PARTITION BY CustomerID) AS OrdersByCustomers
FROM Sales.Orders
```

### Task 2: Find the Total Number of Customers, Scores, and Countries
```sql
SELECT
    *,
    COUNT(*) OVER () AS TotalCustomersStar,
    COUNT(1) OVER () AS TotalCustomersOne,
    COUNT(Score) OVER() AS TotalScores,
    COUNT(Country) OVER() AS TotalCountries
FROM Sales.Customers
```

### Task 3: Check whether the table 'OrdersArchive' contains any duplicate rows
```sql
SELECT 
    * 
FROM (
    SELECT 
        *,
        COUNT(*) OVER(PARTITION BY OrderID) AS CheckDuplicates
    FROM Sales.OrdersArchive
) t
WHERE CheckDuplicates > 1
```
## SUM
### Task 4: Find the Total Sales Across All Orders and per Product
```sql
SELECT
    OrderID,
    OrderDate,
    Sales,
    ProductID,
    SUM(Sales) OVER () AS TotalSales,
    SUM(Sales) OVER (PARTITION BY ProductID) AS SalesByProduct
FROM Sales.Orders
```

### Task 5: Find the Percentage Contribution of Each Product's Sales to the Total Sales
```sql
SELECT
    OrderID,
    ProductID,
    Sales,
    SUM(Sales) OVER () AS TotalSales,
    ROUND(CAST(Sales AS FLOAT) / SUM(Sales) OVER () * 100, 2) AS PercentageOfTotal
FROM Sales.Orders
```

## AVG
### Task 6: Find the Average Sales Across All Orders and per Product
```sql
SELECT
    OrderID,
    OrderDate,
    Sales,
    ProductID,
    AVG(Sales) OVER () AS AvgSales,
    AVG(Sales) OVER (PARTITION BY ProductID) AS AvgSalesByProduct
FROM Sales.Orders
```

### Task 7: Find the Average Scores of Customers
```sql
SELECT
    CustomerID,
    LastName,
    Score,
    COALESCE(Score, 0) AS CustomerScore,
    AVG(Score) OVER () AS AvgScore,
    AVG(COALESCE(Score, 0)) OVER () AS AvgScoreWithoutNull
FROM Sales.Customers
```

### Task 8: Find all orders where Sales exceed the average Sales across all orders
```sql
SELECT
    *
FROM (
    SELECT
        OrderID,
        ProductID,
        Sales,
        AVG(Sales) OVER () AS Avg_Sales
    FROM Sales.Orders
) t 
WHERE Sales > Avg_Sales
```
## MAX / MIN
### Task 9: Find the Highest and Lowest Sales across all orders
```sql
SELECT 
    MIN(Sales) AS MinSales, 
    MAX(Sales) AS MaxSales 
FROM Sales.Orders
```

### Task 10: Find the Lowest Sales across all orders and by Product
```sql
SELECT 
    OrderID,
    ProductID,
    OrderDate,
    Sales,
    MIN(Sales) OVER () AS LowestSales,
    MIN(Sales) OVER (PARTITION BY ProductID) AS LowestSalesByProduct
FROM Sales.Orders
```

### Task 11: Show the employees who have the highest salaries
```sql
SELECT *
FROM (
	SELECT *,
		   MAX(Salary) OVER() AS HighestSalary
	FROM Sales.Employees
) t
WHERE Salary = HighestSalary
```

### Task 12: Find the deviation of each Sale from the minimum and maximum Sales
```sql
SELECT
    OrderID,
    OrderDate,
    ProductID,
    Sales,
    MAX(Sales) OVER () AS HighestSales,
    MIN(Sales) OVER () AS LowestSales,
    Sales - MIN(Sales) OVER () AS DeviationFromMin,
    MAX(Sales) OVER () - Sales AS DeviationFromMax
FROM Sales.Orders
```
## ROLLING SUM & AVERAGE Use Case
### Task 13: Calculate the moving average of Sales for each Product over time
```sql
SELECT
    OrderID,
    ProductID,
    OrderDate,
    Sales,
    AVG(Sales) OVER (PARTITION BY ProductID) AS AvgByProduct,
    AVG(Sales) OVER (PARTITION BY ProductID ORDER BY OrderDate) AS MovingAvg
FROM Sales.Orders
```

### Task 14: Calculate the moving average of Sales for each Product over time, including only the next order
```sql
SELECT
    OrderID,
    ProductID,
    OrderDate,
    Sales,
    AVG(Sales) OVER (PARTITION BY ProductID ORDER BY OrderDate ROWS BETWEEN CURRENT ROW AND 1 FOLLOWING) AS RollingAvg
FROM Sales.Orders
```