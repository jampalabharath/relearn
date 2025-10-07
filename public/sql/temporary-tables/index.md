# Temporary Tables

This script provides a **generic example of data migration** using a temporary table.  

---

## Step 1: Create Temporary Table (`#Orders`)

```sql
SELECT
    *
INTO #Orders
FROM Sales.Orders;
```

## Step 2: Clean Data in Temporary Table
```sql
DELETE FROM #Orders
WHERE OrderStatus = 'Delivered';
```
## Step 3: Load Cleaned Data into Permanent Table
```sql
SELECT
    *
INTO Sales.OrdersTest
FROM #Orders;
```