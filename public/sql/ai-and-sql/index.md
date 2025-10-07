# AI and SQL

This document contains a series of **AI-powered prompts** designed to help SQL developers and learners improve skills in writing, optimizing, and understanding SQL queries.  
The prompts cover **tasks, readability, performance, debugging, interview/exam prep**, and more.  
Each section provides clear **instructions and sample code** to support **self-learning** and **real-world application**.

---

## Table of Contents
1. [Solve an SQL Task](#1-solve-an-sql-task)  
2. [Improve the Readability](#2-improve-the-readability)  
3. [Optimize the Performance Query](#3-optimize-the-performance-query)  
4. [Optimize Execution Plan](#4-optimize-execution-plan)  
5. [Debugging](#5-debugging)  
6. [Explain the Result](#6-explain-the-result)  
7. [Styling & Formatting](#7-styling--formatting)  
8. [Documentations & Comments](#8-documentations--comments)  
9. [Improve Database DDL](#9-improve-database-ddl)  
10. [Generate Test Dataset](#10-generate-test-dataset)  
11. [Create SQL Course](#11-create-sql-course)  
12. [Understand SQL Concept](#12-understand-sql-concept)  
13. [Comparing SQL Concepts](#13-comparing-sql-concepts)  
14. [SQL Questions with Options](#14-sql-questions-with-options)  
15. [Prepare for a SQL Interview](#15-prepare-for-a-sql-interview)  
16. [Prepare for a SQL Exam](#16-prepare-for-a-sql-exam)  

---

## 1. Solve an SQL Task
Prompt:  
In my SQL Server database, we have two tables:  

- `orders(order_id, sales, customer_id, product_id)`  
- `customers(customer_id, first_name, last_name, country)`  

Tasks:  
- Write a query to rank customers based on their sales.  
- Result must include: `customer_id`, full name, country, total sales, rank.  
- Write **3 different versions** of the query.  
- Include comments (avoid obvious ones).  
- Evaluate which version is best in terms of **readability** and **performance**.  

---

## 2. Improve the Readability

The following SQL Server query is long and hard to understand.  

**Task:**
- Improve its readability.  
- Remove any redundancy and consolidate it.  
- Include comments (avoid obvious ones).  
- Explain each improvement.

**Original Query:**
```sql
WITH CTE_Total_Sales_By_Customer AS (
    SELECT 
        c.CustomerID, 
        c.FirstName + ' ' + c.LastName AS FullName,  SUM(o.Sales) AS TotalSales
    FROM  Sales.Customers c
    INNER JOIN 
        Sales.Orders o ON c.CustomerID = o.CustomerID GROUP BY  c.CustomerID, c.FirstName, c.LastName
),CTE_Highest_Order_Product AS (
    SELECT 
        o.CustomerID, 
        p.Product, ROW_NUMBER() OVER (PARTITION BY o.CustomerID ORDER BY o.Sales DESC) AS rn
    FROM Sales.Orders o
    INNER JOIN Sales.Products p ON o.ProductID = p.ProductID
),
CTE_Highest_Category AS (  SELECT 
        o.CustomerID,  p.Category, 
        ROW_NUMBER() OVER (PARTITION BY o.CustomerID ORDER BY SUM(o.Sales) DESC) AS rn
    FROM Sales.Orders o
    INNER JOIN Sales.Products p ON o.ProductID = p.ProductID GROUP BY  o.CustomerID, p.Category
),
CTE_Last_Order_Date AS (
    SELECT 
        CustomerID, 
        MAX(OrderDate) AS LastOrderDate
    FROM  Sales.Orders
    GROUP BY CustomerID
),
CTE_Total_Discounts_By_Customer AS (
    SELECT o.CustomerID,  SUM(o.Quantity * p.Price * 0.1) AS TotalDiscounts
    FROM  Sales.Orders o INNER JOIN Sales.Products p ON o.ProductID = p.ProductID
    GROUP BY o.CustomerID
)
SELECT 
    ts.CustomerID, ts.FullName,
    ts.TotalSales,hop.Product AS HighestOrderProduct,hc.Category AS HighestCategory,
    lod.LastOrderDate,
    td.TotalDiscounts
FROM CTE_Total_Sales_By_Customer ts
LEFT JOIN (SELECT CustomerID, Product FROM CTE_Highest_Order_Product WHERE rn = 1) hop ON ts.CustomerID = hop.CustomerID
LEFT JOIN (SELECT CustomerID, Category FROM CTE_Highest_Category WHERE rn = 1) hc ON ts.CustomerID = hc.CustomerID
LEFT JOIN CTE_Last_Order_Date lod ON ts.CustomerID = lod.CustomerID
LEFT JOIN  CTE_Total_Discounts_By_Customer td ON ts.CustomerID = td.CustomerID
WHERE  ts.TotalSales > 0
ORDER BY  ts.TotalSales DESC
```

## 3. Optimize the Performance Query

The following query is slow.

**Task:**

Propose optimizations.

Provide an improved query.

Explain each improvement.

**Original Query:**
```sql
SELECT 
    o.OrderID,
    o.CustomerID,
    c.FirstName AS CustomerFirstName,
    (SELECT COUNT(o2.OrderID)
     FROM Sales.Orders o2
     WHERE o2.CustomerID = c.CustomerID) AS OrderCount
FROM 
    Sales.Orders o
LEFT JOIN 
    Sales.Customers c ON o.CustomerID = c.CustomerID
WHERE 
    LOWER(o.OrderStatus) = 'delivered'
    OR YEAR(o.OrderDate) = 2025
    OR o.CustomerID =1 OR o.CustomerID =2 OR o.CustomerID =3
    OR o.CustomerID IN (
        SELECT CustomerID
        FROM Sales.Customers
        WHERE Country LIKE '%USA%'
    )
```

## 4. Optimize Execution Plan

**Task:**

Describe the execution plan step by step.

Identify performance bottlenecks.

Suggest optimizations.

## 5. Debugging

The following query causes an error:
Msg 8120, Level 16, State 1, Line 5

**Task:**

Explain the error.

Find the root cause.

Suggest a fix.

```sql
SELECT 
    C.CustomerID,
    C.Country,
    SUM(O.Sales) AS TotalSales,
    RANK() OVER (PARTITION BY C.Country ORDER BY O.Sales DESC) AS RankInCountry
FROM Sales.Customers C
LEFT JOIN Sales.Orders O 
ON C.CustomerID = O.CustomerID
GROUP BY C.CustomerID, C.Country
```

## 6. Explain the Result

**Query:**
```sql
WITH Series AS (
	-- Anchor Query
	SELECT 1 AS MyNumber
	UNION ALL
	-- Recursive Query
	SELECT MyNumber + 1
	FROM Series
	WHERE MyNumber < 20
)
SELECT * FROM Series
```

**Task:**

Break down how SQL processes this query step by step.

Explain how the result is formed.

## 7. Styling & Formatting

The following query is hard to read.

**Task:**

Restyle for readability.

Align column aliases.

Keep compact.

**Original Query:**

```sql
with CTE_Total_Sales as 
(Select 
CustomerID, sum(Sales) as TotalSales 
from Sales.Orders 
group by CustomerID),
cte_customer_segments as 
(SELECT CustomerID, 
case when TotalSales > 100 then 'High Value' 
when TotalSales between 50 and 100 then 'Medium Value' 
else 'Low Value' end as CustomerSegment 
from CTE_Total_Sales)
select c.CustomerID, c.FirstName, c.LastName, 
cts.TotalSales, ccs.CustomerSegment 
FROM sales.customers c 
left join CTE_Total_Sales cts 
ON cts.CustomerID = c.CustomerID 
left JOIN cte_customer_segments ccs ON ccs.CustomerID = c.CustomerID
```

## 8. Documentations & Comments

The following query lacks comments.

**Task:**

Add a leading comment.

Insert clarifying comments where needed.

Create two documents:

Business rules implemented.

How the query works.

```sql
WITH CTE_Total_Sales AS 
(
SELECT 
    CustomerID,
    SUM(Sales) AS TotalSales
FROM Sales.Orders 
GROUP BY CustomerID
),
CTE_Customer_Segements AS (
SELECT 
	CustomerID,
	CASE 
		WHEN TotalSales > 100 THEN 'High Value'
		WHEN TotalSales BETWEEN 50 AND 100 THEN 'Medium Value'
		ELSE 'Low Value'
	END CustomerSegment
FROM CTE_Total_Sales
)
SELECT 
c.CustomerID, 
c.FirstName,
c.LastName,
cts.TotalSales,
ccs.CustomerSegment
FROM Sales.Customers c
LEFT JOIN CTE_Total_Sales cts
ON cts.CustomerID = c.CustomerID
LEFT JOIN CTE_Customer_Segements ccs
ON ccs.CustomerID = c.CustomerID 
```

## 9. Improve Database DDL

**Task:**

Check naming consistency.

Optimize data types.

Verify PK/FK integrity.

Review indexes.

Ensure normalization.

## 10. Generate Test Dataset

**Task:**

Generate realistic test dataset with INSERT statements.

Keep it small.

Ensure valid PK/FK relationships.

Avoid NULL.

## 11. Create SQL Course

**Task:**

Build a beginner-to-advanced SQL roadmap.

Include analytics-focused topics.

Use real-world scenarios.

## 12. Understand SQL Concept

**Task:**

Explain SQL Window Functions.

Provide an analogy.

Describe when/why to use them.

Show syntax & examples.

List top 3 use cases.

## 13. Comparing SQL Concepts

**Task:**

Compare Window Functions vs GROUP BY.

Explain key differences.

Show examples.

Provide pros/cons.

Summarize in a side-by-side table.

## 14. SQL Questions with Options

**Task:**

Act as SQL trainer.

Provide sample dataset.

Create progressive tasks.

Simulate SQL Server results.

Review solutions & suggest improvements.

## 15. Prepare for a SQL Interview

**Task:**

Act as interviewer.

Ask common SQL questions.

Progress to advanced topics.

Evaluate answers & give feedback.

## 16. Prepare for a SQL Exam

**Task:**

Ask SQL exam-style questions.

Progress gradually.

Evaluate answers & give feedback.