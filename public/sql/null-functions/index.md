# NULL Functions

This guide covers essential SQL functions for handling **NULL** values in different scenarios such as aggregation, mathematical operations, sorting, joins, and comparisons.


---

## 1. Handle NULL - Data Aggregation

Replace NULL values using `COALESCE` to ensure accurate averages.

```sql
SELECT
    CustomerID,
    Score,
    COALESCE(Score, 0) AS Score2,
    AVG(Score) OVER () AS AvgScores,
    AVG(COALESCE(Score, 0)) OVER () AS AvgScores2
FROM Sales.Customers;
```

## 2. Handle NULL - Mathematical Operators

Concatenate first and last names safely and add bonus points with COALESCE.

```sql
SELECT
    CustomerID,
    FirstName,
    LastName,
    FirstName + ' ' + COALESCE(LastName, '') AS FullName,
    Score,
    COALESCE(Score, 0) + 10 AS ScoreWithBonus
FROM Sales.Customers;
```

## 3. Handle NULL - Sorting Data

Order results so that NULL values appear last.
```sql
SELECT
    CustomerID,
    Score
FROM Sales.Customers
ORDER BY CASE WHEN Score IS NULL THEN 1 ELSE 0 END, Score;
```

## 4. NULLIF - Division by Zero

Prevent division errors by using NULLIF.
```sql
SELECT
    OrderID,
    Sales,
    Quantity,
    Sales / NULLIF(Quantity, 0) AS Price
FROM Sales.Orders;
```

## 5. IS NULL / IS NOT NULL

Identify rows with or without NULL values.
```sql
-- Customers with no scores
SELECT *
FROM Sales.Customers
WHERE Score IS NULL;

-- Customers with scores
SELECT *
FROM Sales.Customers
WHERE Score IS NOT NULL;
```
## 6. LEFT ANTI JOIN

Retrieve customers who have not placed any orders.
```sql
SELECT
    c.*,
    o.OrderID
FROM Sales.Customers AS c
LEFT JOIN Sales.Orders AS o
    ON c.CustomerID = o.CustomerID
WHERE o.CustomerID IS NULL;
```

## 7. NULLs vs Empty String vs Blank Spaces

Differentiate between NULL, empty strings (''), and blank spaces (' ').
```sql
WITH Orders AS (
    SELECT 1 AS Id, 'A' AS Category UNION
    SELECT 2, NULL UNION
    SELECT 3, '' UNION
    SELECT 4, '  '
)
SELECT 
    *,
    DATALENGTH(Category) AS LenCategory,
    TRIM(Category) AS Policy1,
    NULLIF(TRIM(Category), '') AS Policy2,
    COALESCE(NULLIF(TRIM(Category), ''), 'unknown') AS Policy3
FROM Orders;
```