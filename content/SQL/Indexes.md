---
title: "Indexes"
weight: 24
---

This script demonstrates various **index types in SQL Server** including clustered, non-clustered, columnstore, unique, and filtered indexes.  
It also covers **index monitoring techniques** like usage stats, missing/duplicate indexes, updating statistics, and fragmentation.  

---

## 📑 Table of Contents

### Index Types
- Clustered and Non-Clustered Indexes  
- Leftmost Prefix Rule Explanation  
- Columnstore Indexes  
- Unique Indexes  
- Filtered Indexes  

### Index Monitoring
- Monitor Index Usage  
- Monitor Missing Indexes  
- Monitor Duplicate Indexes  
- Update Statistics  
- Fragmentations  

---

## Index Types

### Clustered and Non-Clustered Indexes

```sql
-- Create a Heap Table as a copy of Sales.Customers 
SELECT *
INTO Sales.DBCustomers
FROM Sales.Customers;

-- Test Query
SELECT *
FROM Sales.DBCustomers
WHERE CustomerID = 1;

-- Create a Clustered Index
CREATE CLUSTERED INDEX idx_DBCustomers_CustomerID
ON Sales.DBCustomers (CustomerID);

-- Attempt to create a second Clustered Index (will fail)
CREATE CLUSTERED INDEX idx_DBCustomers_CustomerID
ON Sales.DBCustomers (CustomerID);

-- Drop the Clustered Index
DROP INDEX idx_DBCustomers_CustomerID
ON Sales.DBCustomers;

-- Query using LastName filter
SELECT *
FROM Sales.DBCustomers
WHERE LastName = 'Brown';

-- Create a Non-Clustered Index on LastName
CREATE NONCLUSTERED INDEX idx_DBCustomers_LastName
ON Sales.DBCustomers (LastName);

-- Additional Non-Clustered Index on FirstName
CREATE INDEX idx_DBCustomers_FirstName
ON Sales.DBCustomers (FirstName);

-- Composite Index on Country and Score
CREATE INDEX idx_DBCustomers_CountryScore
ON Sales.DBCustomers (Country, Score);

-- Query that uses Composite Index
SELECT *
FROM Sales.DBCustomers
WHERE Country = 'USA'
  AND Score > 500;

-- Query that may not use Composite Index due to column order
SELECT *
FROM Sales.DBCustomers
WHERE Score > 500
  AND Country = 'USA';
```

### Leftmost Prefix Rule Explanation

For a composite index (A, B, C, D) the index is useful when filtering on:

A only

A, B

A, B, C

It is not effective when filtering on:

B only

A, C

A, B, D

### Columnstore Indexes

```sql
-- Create Clustered Columnstore Index
CREATE CLUSTERED COLUMNSTORE INDEX idx_DBCustomers_CS
ON Sales.DBCustomers;
GO

-- Non-Clustered Columnstore Index on FirstName
CREATE NONCLUSTERED COLUMNSTORE INDEX idx_DBCustomers_CS_FirstName
ON Sales.DBCustomers (FirstName);
GO

-- Switch context to AdventureWorksDW2022
USE AdventureWorksDW2022;

-- Create Heap Table
SELECT *
INTO FactInternetSales_HP
FROM FactInternetSales;

-- Create RowStore Table
SELECT *
INTO FactInternetSales_RS
FROM FactInternetSales;

-- Clustered Index on RowStore
CREATE CLUSTERED INDEX idx_FactInternetSales_RS_PK
ON FactInternetSales_RS (SalesOrderNumber, SalesOrderLineNumber);

-- Create Columnstore Table
SELECT *
INTO FactInternetSales_CS
FROM FactInternetSales;

-- Clustered Columnstore Index
CREATE CLUSTERED COLUMNSTORE INDEX idx_FactInternetSales_CS_PK
ON FactInternetSales_CS;
```

### Unique Indexes
```sql
-- Attempt Unique Index on Category (fails if duplicates exist)
CREATE UNIQUE INDEX idx_Products_Category
ON Sales.Products (Category);

-- Unique Index on Product
CREATE UNIQUE INDEX idx_Products_Product
ON Sales.Products (Product);

-- Test Insert (should fail if duplicate constraint holds)
INSERT INTO Sales.Products (ProductID, Product)
VALUES (106, 'Caps');
```

### Filtered Indexes
```sql
-- Test Query
SELECT *
FROM Sales.Customers
WHERE Country = 'USA';

-- Filtered Index
CREATE NONCLUSTERED INDEX idx_Customers_Country
ON Sales.Customers (Country)
WHERE Country = 'USA';
```

## Index Monitoring

### Monitor Index Usage
```sql
-- List all indexes on a table
sp_helpindex 'Sales.DBCustomers';

-- Monitor Index Usage
SELECT 
	tbl.name AS TableName,
    idx.name AS IndexName,
    idx.type_desc AS IndexType,
    idx.is_primary_key AS IsPrimaryKey,
    idx.is_unique AS IsUnique,
    idx.is_disabled AS IsDisabled,
    s.user_seeks AS UserSeeks,
    s.user_scans AS UserScans,
    s.user_lookups AS UserLookups,
    s.user_updates AS UserUpdates,
    COALESCE(s.last_user_seek, s.last_user_scan) AS LastUpdate
FROM sys.indexes idx
JOIN sys.tables tbl
    ON idx.object_id = tbl.object_id
LEFT JOIN sys.dm_db_index_usage_stats s
    ON s.object_id = idx.object_id
    AND s.index_id = idx.index_id
ORDER BY tbl.name, idx.name;
```

### Monitor Missing Indexes
```sql
SELECT * 
FROM sys.dm_db_missing_index_details;
```

### Monitor Duplicate Indexes
```sql
SELECT  
	tbl.name AS TableName,
	col.name AS IndexColumn,
	idx.name AS IndexName,
	idx.type_desc AS IndexType,
	COUNT(*) OVER (PARTITION BY tbl.name, col.name) ColumnCount
FROM sys.indexes idx
JOIN sys.tables tbl ON idx.object_id = tbl.object_id
JOIN sys.index_columns ic ON idx.object_id = ic.object_id AND idx.index_id = ic.index_id
JOIN sys.columns col ON ic.object_id = col.object_id AND ic.column_id = col.column_id
ORDER BY ColumnCount DESC;
```
### Update Statistics
```sql
SELECT 
    SCHEMA_NAME(t.schema_id) AS SchemaName,
    t.name AS TableName,
    s.name AS StatisticName,
    sp.last_updated AS LastUpdate,
    DATEDIFF(day, sp.last_updated, GETDATE()) AS LastUpdateDay,
    sp.rows AS 'Rows',
    sp.modification_counter AS ModificationsSinceLastUpdate
FROM sys.stats AS s
JOIN sys.tables AS t
    ON s.object_id = t.object_id
CROSS APPLY sys.dm_db_stats_properties(s.object_id, s.stats_id) AS sp
ORDER BY sp.modification_counter DESC;

-- Update specific system statistic
UPDATE STATISTICS Sales.DBCustomers _WA_Sys_00000001_6EF57B66;
GO

-- Update all statistics for a table
UPDATE STATISTICS Sales.DBCustomers;
GO

-- Update all statistics in the database
EXEC sp_updatestats;
GO
```

### Fragmentations

```sql
-- Retrieve fragmentation stats
SELECT 
    tbl.name AS TableName,
    idx.name AS IndexName,
    s.avg_fragmentation_in_percent,
    s.page_count
FROM sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, 'LIMITED') AS s
INNER JOIN sys.tables tbl 
    ON s.object_id = tbl.object_id
INNER JOIN sys.indexes AS idx 
    ON idx.object_id = s.object_id
    AND idx.index_id = s.index_id
ORDER BY s.avg_fragmentation_in_percent DESC;

-- Reorganize index (lightweight)
ALTER INDEX idx_Customers_CS_Country 
ON Sales.Customers REORGANIZE;
GO

-- Rebuild index (full)
ALTER INDEX idx_Customers_Country 
ON Sales.Customers REBUILD;
GO
```