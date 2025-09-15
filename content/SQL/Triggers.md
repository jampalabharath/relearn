---
title: "Triggers"
weight: 23
---


This script demonstrates:  
1. The creation of a **logging table**.  
2. A **trigger** on the `Sales.Employees` table.  
3. An **insert operation** that fires the trigger.  

The trigger logs details of newly added employees into the `Sales.EmployeeLogs` table.

---

## Step 1: Create Log Table

```sql
CREATE TABLE Sales.EmployeeLogs
(
    LogID      INT IDENTITY(1,1) PRIMARY KEY,
    EmployeeID INT,
    LogMessage VARCHAR(255),
    LogDate    DATE
);
GO
```

## Step 2: Create Trigger on Employees Table

```sql
CREATE TRIGGER trg_AfterInsertEmployee
ON Sales.Employees
AFTER INSERT
AS
BEGIN
    INSERT INTO Sales.EmployeeLogs (EmployeeID, LogMessage, LogDate)
    SELECT
        EmployeeID,
        'New Employee Added = ' + CAST(EmployeeID AS VARCHAR),
        GETDATE()
    FROM INSERTED;
END;
GO
```

## Step 3: Insert New Data Into Employees
```sql
INSERT INTO Sales.Employees
VALUES (6, 'Maria', 'Doe', 'HR', '1988-01-12', 'F', 80000, 3);
GO
```
## Step 4: Check the Logs
```sql
SELECT *
FROM Sales.EmployeeLogs;
GO
```