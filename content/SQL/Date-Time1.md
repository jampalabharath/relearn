+++
title = "Date & Time Functions"
weight = 9
+++

This guide demonstrates various **date and time functions** in SQL.  
It covers `GETDATE`, `DATETRUNC`, `DATENAME`, `DATEPART`, `YEAR`, `MONTH`, `DAY`, `EOMONTH`, `FORMAT`, `CONVERT`, `CAST`, `DATEADD`, `DATEDIFF`, and `ISDATE`.

---

## 1. GETDATE() | Date Values
```sql
-- Display OrderID, CreationTime, a hard-coded date, and the current system date
SELECT
    OrderID,
    CreationTime,
    '2025-08-20' AS HardCoded,
    GETDATE() AS Today
FROM Sales.Orders;
```

## 2. Date Part Extractions (DATETRUNC, DATENAME, DATEPART, YEAR, MONTH, DAY)
```sql
-- Extract various parts of CreationTime
SELECT
    OrderID,
    CreationTime,
    DATETRUNC(year, CreationTime) AS Year_dt,
    DATETRUNC(day, CreationTime) AS Day_dt,
    DATETRUNC(minute, CreationTime) AS Minute_dt,
    DATENAME(month, CreationTime) AS Month_dn,
    DATENAME(weekday, CreationTime) AS Weekday_dn,
    DATENAME(day, CreationTime) AS Day_dn,
    DATENAME(year, CreationTime) AS Year_dn,
    DATEPART(year, CreationTime) AS Year_dp,
    DATEPART(month, CreationTime) AS Month_dp,
    DATEPART(day, CreationTime) AS Day_dp,
    DATEPART(hour, CreationTime) AS Hour_dp,
    DATEPART(quarter, CreationTime) AS Quarter_dp,
    DATEPART(week, CreationTime) AS Week_dp,
    YEAR(CreationTime) AS Year,
    MONTH(CreationTime) AS Month,
    DAY(CreationTime) AS Day
FROM Sales.Orders;
```
## 3. DATETRUNC() Data Aggregation
```sql
-- Aggregate orders by year
SELECT
    DATETRUNC(year, CreationTime) AS Creation,
    COUNT(*) AS OrderCount
FROM Sales.Orders
GROUP BY DATETRUNC(year, CreationTime);
```

## 4. EOMONTH()
```sql
-- Display end-of-month date for each order
SELECT
    OrderID,
    CreationTime,
    EOMONTH(CreationTime) AS EndOfMonth
FROM Sales.Orders;
```

## 5. Date Parts | Use Cases
```sql
-- Orders per year
SELECT YEAR(OrderDate) AS OrderYear, COUNT(*) AS TotalOrders
FROM Sales.Orders
GROUP BY YEAR(OrderDate);

-- Orders per month
SELECT MONTH(OrderDate) AS OrderMonth, COUNT(*) AS TotalOrders
FROM Sales.Orders
GROUP BY MONTH(OrderDate);

-- Orders per month (friendly names)
SELECT DATENAME(month, OrderDate) AS OrderMonth, COUNT(*) AS TotalOrders
FROM Sales.Orders
GROUP BY DATENAME(month, OrderDate);

-- Orders placed in February
SELECT * 
FROM Sales.Orders
WHERE MONTH(OrderDate) = 2;
```

## 6. FORMAT()
```sql
-- Format CreationTime into different formats
SELECT
    OrderID,
    CreationTime,
    FORMAT(CreationTime, 'MM-dd-yyyy') AS USA_Format,
    FORMAT(CreationTime, 'dd-MM-yyyy') AS EURO_Format,
    FORMAT(CreationTime, 'dd') AS dd,
    FORMAT(CreationTime, 'ddd') AS ddd,
    FORMAT(CreationTime, 'dddd') AS dddd,
    FORMAT(CreationTime, 'MM') AS MM,
    FORMAT(CreationTime, 'MMM') AS MMM,
    FORMAT(CreationTime, 'MMMM') AS MMMM
FROM Sales.Orders;

-- Custom format example
SELECT
    OrderID,
    CreationTime,
    'Day ' + FORMAT(CreationTime, 'ddd MMM') +
    ' Q' + DATENAME(quarter, CreationTime) + ' ' +
    FORMAT(CreationTime, 'yyyy hh:mm:ss tt') AS CustomFormat
FROM Sales.Orders;

-- Orders per month (formatted "MMM yy")
SELECT FORMAT(CreationTime, 'MMM yy') AS OrderDate, COUNT(*) AS TotalOrders
FROM Sales.Orders
GROUP BY FORMAT(CreationTime, 'MMM yy');
```

## 7. CONVERT()
```sql
-- Conversion using CONVERT
SELECT
    CONVERT(INT, '123') AS [String to Int CONVERT],
    CONVERT(DATE, '2025-08-20') AS [String to Date CONVERT],
    CreationTime,
    CONVERT(DATE, CreationTime) AS [Datetime to Date CONVERT],
    CONVERT(VARCHAR, CreationTime, 32) AS [USA Std. Style:32],
    CONVERT(VARCHAR, CreationTime, 34) AS [EURO Std. Style:34]
FROM Sales.Orders;
```

## 8. CAST()
```sql
-- Conversion using CAST
SELECT
    CAST('123' AS INT) AS [String to Int],
    CAST(123 AS VARCHAR) AS [Int to String],
    CAST('2025-08-20' AS DATE) AS [String to Date],
    CAST('2025-08-20' AS DATETIME2) AS [String to Datetime],
    CreationTime,
    CAST(CreationTime AS DATE) AS [Datetime to Date]
FROM Sales.Orders;
```

## 9. DATEADD / DATEDIFF
```sql
-- Date arithmetic
SELECT
    OrderID,
    OrderDate,
    DATEADD(day, -10, OrderDate) AS TenDaysBefore,
    DATEADD(month, 3, OrderDate) AS ThreeMonthsLater,
    DATEADD(year, 2, OrderDate) AS TwoYearsLater
FROM Sales.Orders;

-- Employee ages
SELECT
    EmployeeID,
    BirthDate,
    DATEDIFF(year, BirthDate, GETDATE()) AS Age
FROM Sales.Employees;

-- Avg shipping duration
SELECT
    MONTH(OrderDate) AS OrderMonth,
    AVG(DATEDIFF(day, OrderDate, ShipDate)) AS AvgShip
FROM Sales.Orders
GROUP BY MONTH(OrderDate);

-- Time gap analysis
SELECT
    OrderID,
    OrderDate AS CurrentOrderDate,
    LAG(OrderDate) OVER (ORDER BY OrderDate) AS PreviousOrderDate,
    DATEDIFF(day, LAG(OrderDate) OVER (ORDER BY OrderDate), OrderDate) AS NrOfDays
FROM Sales.Orders;
```

## 10. ISDATE()
```sql
-- Validate OrderDate using ISDATE
SELECT
    OrderDate,
    ISDATE(OrderDate) AS IsValidDate,
    CASE 
        WHEN ISDATE(OrderDate) = 1 THEN CAST(OrderDate AS DATE)
        ELSE '9999-01-01'
    END AS NewOrderDate
FROM (
    SELECT '2025-08-20' AS OrderDate UNION
    SELECT '2025-08-21' UNION
    SELECT '2025-08-23' UNION
    SELECT '2025-08'
) AS t;
-- WHERE ISDATE(OrderDate) = 0
```