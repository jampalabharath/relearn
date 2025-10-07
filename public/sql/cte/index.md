# Common Table Expressions (CTEs)

This script demonstrates the use of **Common Table Expressions (CTEs)** in SQL Server.  
It includes examples of non-recursive CTEs for data aggregation and segmentation, as well as recursive CTEs for generating sequences and building hierarchical data.

---


## NON-RECURSIVE CTE

```sql
-- Step 1 → Total Sales Per Customer
WITH CTE_Total_Sales AS
(
    SELECT
        CustomerID,
        SUM(Sales) AS TotalSales
    FROM Sales.Orders
    GROUP BY CustomerID
)

-- Step 2 → Last Order Date for Each Customer
, CTE_Last_Order AS
(
    SELECT
        CustomerID,
        MAX(OrderDate) AS Last_Order
    FROM Sales.Orders
    GROUP BY CustomerID
)

-- Step 3 → Rank Customers by Total Sales
, CTE_Customer_Rank AS
(
    SELECT
        CustomerID,
        TotalSales,
        RANK() OVER (ORDER BY TotalSales DESC) AS CustomerRank
    FROM CTE_Total_Sales
)

-- Step 4 → Segment Customers by Sales
, CTE_Customer_Segments AS
(
    SELECT
        CustomerID,
        TotalSales,
        CASE 
            WHEN TotalSales > 100 THEN 'High'
            WHEN TotalSales > 80  THEN 'Medium'
            ELSE 'Low'
        END AS CustomerSegments
    FROM CTE_Total_Sales
)

-- Final Query Combining All CTEs
SELECT
    c.CustomerID,
    c.FirstName,
    c.LastName,
    cts.TotalSales,
    clo.Last_Order,
    ccr.CustomerRank,
    ccs.CustomerSegments
FROM Sales.Customers AS c
LEFT JOIN CTE_Total_Sales AS cts
    ON cts.CustomerID = c.CustomerID
LEFT JOIN CTE_Last_Order AS clo
    ON clo.CustomerID = c.CustomerID
LEFT JOIN CTE_Customer_Rank AS ccr
    ON ccr.CustomerID = c.CustomerID
LEFT JOIN CTE_Customer_Segments AS ccs
    ON ccs.CustomerID = c.CustomerID;
```

## RECURSIVE CTE | GENERATE SEQUENCE
### Task 2: Generate Numbers 1 to 20
```sql
WITH Series AS (
    SELECT 1 AS MyNumber
    UNION ALL
    SELECT MyNumber + 1
    FROM Series
    WHERE MyNumber < 20
)
SELECT *
FROM Series;
```

### Task 3: Generate Numbers 1 to 1000
```sql
WITH Series AS
(
    SELECT 1 AS MyNumber
    UNION ALL
    SELECT MyNumber + 1
    FROM Series
    WHERE MyNumber < 1000
)
SELECT *
FROM Series
OPTION (MAXRECURSION 5000);
```
## RECURSIVE CTE | BUILD HIERARCHY
### Task 4: Build Employee Hierarchy

Display each employee's level within the organization.

Anchor Query: Select employees with no manager.

Recursive Query: Select subordinates and increment the level.

```sql
WITH CTE_Emp_Hierarchy AS
(
    -- Anchor Query: Top-level employees (no manager)
    SELECT
        EmployeeID,
        FirstName,
        ManagerID,
        1 AS Level
    FROM Sales.Employees
    WHERE ManagerID IS NULL

    UNION ALL

    -- Recursive Query: Get subordinate employees and increment level
    SELECT
        e.EmployeeID,
        e.FirstName,
        e.ManagerID,
        Level + 1
    FROM Sales.Employees AS e
    INNER JOIN CTE_Emp_Hierarchy AS ceh
        ON e.ManagerID = ceh.EmployeeID
)
SELECT *
FROM CTE_Emp_Hierarchy;

```