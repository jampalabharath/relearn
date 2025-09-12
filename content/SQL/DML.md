+++
title = "DML"
weight = 3
+++

This guide covers the essential **DML (Data Manipulation Language)** commands used for inserting, updating, and deleting data in database tables.

---

## 1. INSERT – Adding Data to Tables

### Method 1: Manual INSERT using VALUES
```sql
-- Insert new records into the customers table
INSERT INTO customers (id, first_name, country, score)
VALUES 
    (6, 'Anna', 'USA', NULL),
    (7, 'Sam', NULL, 100);

-- Incorrect column order 
INSERT INTO customers (id, first_name, country, score)
VALUES 
    (8, 'Max', 'USA', NULL);

-- Incorrect data type in values
INSERT INTO customers (id, first_name, country, score)
VALUES 
    ('Max', 9, 'Max', NULL);

-- Insert a new record with full column values
INSERT INTO customers (id, first_name, country, score)
VALUES (8, 'Max', 'USA', 368);

-- Insert without specifying column names (not recommended)
INSERT INTO customers 
VALUES 
    (9, 'Andreas', 'Germany', NULL);

-- Insert a record with only id and first_name
INSERT INTO customers (id, first_name)
VALUES 
    (10, 'Sahra');
```

### Method 2: INSERT using SELECT (Copying Data)
```sql
-- Copy data from customers table into persons
INSERT INTO persons (id, person_name, birth_date, phone)
SELECT
    id,
    first_name,
    NULL,
    'Unknown'
FROM customers;
```


## 2. UPDATE – Modifying Existing Data
```sql
-- Change the score of customer with ID 6 to 0
UPDATE customers
SET score = 0
WHERE id = 6;

-- Change score and country for customer with ID 10
UPDATE customers
SET score = 0,
    country = 'UK'
WHERE id = 10;

-- Update all NULL scores to 0
UPDATE customers
SET score = 0
WHERE score IS NULL;

-- Verify the update
SELECT *
FROM customers
WHERE score IS NULL;
```

## 3. DELETE – Removing Data from Tables
```sql
-- Select customers with ID greater than 5
SELECT *
FROM customers
WHERE id > 5;

-- Delete customers with ID greater than 5
DELETE FROM customers
WHERE id > 5;

-- Delete all rows from persons
DELETE FROM persons;

-- Faster method to delete all rows
TRUNCATE TABLE persons;
```