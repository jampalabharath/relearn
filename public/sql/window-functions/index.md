# Window Functions

SQL window functions enable advanced calculations across sets of rows related to the current row without needing complex subqueries or joins. They support clauses like **OVER, PARTITION, ORDER, FRAME**, along with important rules and group-based use cases.

## Table of Contents
1. SQL Window Basics  
2. SQL Window OVER Clause  
3. SQL Window PARTITION Clause  
4. SQL Window ORDER Clause  
5. SQL Window FRAME Clause  
6. SQL Window Rules  
7. SQL Window with GROUP BY  

---

## 1. SQL Window Basics

### TASK 1 – Calculate the Total Sales Across All Orders  
```sql
SELECT
    SUM(Sales) AS Total_Sales
FROM Sales.Orders;
```


### TASK 2 – Calculate the Total Sales for Each Product
```sql
SELECT 
    ProductID,
    SUM(Sales) AS Total_Sales
FROM Sales.Orders
GROUP BY ProductID;
```

## 2. SQL Window OVER Clause
### TASK 3 – Total sales across all orders with order details
```sql
SELECT
    OrderID,
    OrderDate,
    ProductID,
    Sales,
    SUM(Sales) OVER () AS Total_Sales
FROM Sales.Orders;
```

## 3. SQL Window PARTITION Clause
### TASK 4 – Total sales overall and per product
```sql
SELECT
    OrderID,
    OrderDate,
    ProductID,
    Sales,
    SUM(Sales) OVER () AS Total_Sales,
    SUM(Sales) OVER (PARTITION BY ProductID) AS Sales_By_Product
FROM Sales.Orders;
```
### TASK 5 – Total sales overall, per product, and per product-status combination
```sql
SELECT
    OrderID,
    OrderDate,
    ProductID,
    OrderStatus,
    Sales,
    SUM(Sales) OVER () AS Total_Sales,
    SUM(Sales) OVER (PARTITION BY ProductID) AS Sales_By_Product,
    SUM(Sales) OVER (PARTITION BY ProductID, OrderStatus) AS Sales_By_Product_Status
FROM Sales.Orders;
```
## 4. SQL Window ORDER Clause
### TASK 6 – Rank each order by sales (highest to lowest)
```sql
SELECT
    OrderID,
    OrderDate,
    Sales,
    RANK() OVER (ORDER BY Sales DESC) AS Rank_Sales
FROM Sales.Orders;
```
## 5. SQL Window FRAME Clause
### TASK 7 – Total sales by order status (current + next two orders)
```sql
SELECT
    OrderID,
    OrderDate,
    ProductID,
    OrderStatus,
    Sales,
    SUM(Sales) OVER (
        PARTITION BY OrderStatus 
        ORDER BY OrderDate 
        ROWS BETWEEN CURRENT ROW AND 2 FOLLOWING
    ) AS Total_Sales
FROM Sales.Orders;
```

### TASK 8 – Total sales by order status (current + previous two orders)
```sql
SELECT
    OrderID,
    OrderDate,
    ProductID,
    OrderStatus,
    Sales,
    SUM(Sales) OVER (
        PARTITION BY OrderStatus 
        ORDER BY OrderDate 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS Total_Sales
FROM Sales.Orders;
```

### TASK 9 – Total sales by order status (previous two orders only)
```sql
SELECT
    OrderID,
    OrderDate,
    ProductID,
    OrderStatus,
    Sales,
    SUM(Sales) OVER (
        PARTITION BY OrderStatus 
        ORDER BY OrderDate 
        ROWS 2 PRECEDING
    ) AS Total_Sales
FROM Sales.Orders;
```

### TASK 10 – Cumulative sales up to the current order
```sql
SELECT
    OrderID,
    OrderDate,
    ProductID,
    OrderStatus,
    Sales,
    SUM(Sales) OVER (
        PARTITION BY OrderStatus 
        ORDER BY OrderDate 
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS Total_Sales
FROM Sales.Orders;
```

### TASK 11 – Cumulative sales from start to current row
```sql
SELECT
    OrderID,
    OrderDate,
    ProductID,
    OrderStatus,
    Sales,
    SUM(Sales) OVER (
        PARTITION BY OrderStatus 
        ORDER BY OrderDate 
        ROWS UNBOUNDED PRECEDING
    ) AS Total_Sales
FROM Sales.Orders;
```

## 6. SQL Window Rules
### RULE 1 – Window functions can only be used in SELECT or ORDER BY
```sql
SELECT
    OrderID,
    OrderDate,
    ProductID,
    OrderStatus,
    Sales,
    SUM(Sales) OVER (PARTITION BY OrderStatus) AS Total_Sales
FROM Sales.Orders
WHERE SUM(Sales) OVER (PARTITION BY OrderStatus) > 100;  -- ❌ Invalid
```
### RULE 2 – Window functions cannot be nested
```sql
SELECT
    OrderID,
    OrderDate,
    ProductID,
    OrderStatus,
    Sales,
    SUM(SUM(Sales) OVER (PARTITION BY OrderStatus)) OVER (PARTITION BY OrderStatus) AS Total_Sales  -- ❌ Invalid nesting
FROM Sales.Orders;
```

## 7. SQL Window with GROUP BY
### TASK 12 – Rank customers by total sales
```sql
SELECT
    CustomerID,
    SUM(Sales) AS Total_Sales,
    RANK() OVER (ORDER BY SUM(Sales) DESC) AS Rank_Customers
FROM Sales.Orders
GROUP BY CustomerID;
```