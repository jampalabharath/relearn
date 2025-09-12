+++
title = "SELECT Query"
weight = 1
+++


This guide covers various `SELECT` query techniques used for retrieving, filtering, sorting, and aggregating data efficiently.

---

## Comments

```sql
-- This is a single-line comment.

/* This
   is
   a multiple-line
   comment */
```

## 1. SELECT ALL COLUMNS
```sql
-- Retrieve All Customer Data
SELECT * FROM customers;

-- Retrieve All Order Data
SELECT * FROM orders;
```

## 2. SELECT SPECIFIC COLUMNS
```sql
-- Retrieve each customer's name, country, and score
SELECT 
    first_name,
    country, 
    score
FROM customers;
```

## 3. WHERE CLAUSE
```sql
-- Retrieve customers with a score not equal to 0
SELECT * 
FROM customers
WHERE score != 0;

-- Retrieve customers from Germany
SELECT * 
FROM customers
WHERE country = 'Germany';

-- Retrieve the name and country of customers from Germany
SELECT
    first_name,
    country
FROM customers
WHERE country = 'Germany';
```

## 4. ORDER BY
```sql
-- Sort by highest score first
SELECT * 
FROM customers
ORDER BY score DESC;

-- Sort by lowest score first
SELECT * 
FROM customers
ORDER BY score ASC;

-- Sort by country
SELECT * 
FROM customers
ORDER BY country ASC;

-- Sort by country, then highest score
SELECT * 
FROM customers
ORDER BY country ASC, score DESC;

-- Customers with score != 0, sorted by highest score
SELECT
    first_name,
    country,
    score
FROM customers
WHERE score != 0
ORDER BY score DESC;
```

## 5. GROUP BY
```sql
-- Find total score for each country
SELECT 
    country,
    SUM(score) AS total_score
FROM customers
GROUP BY country;

-- ❌ Invalid: first_name not grouped or aggregated
SELECT 
    country,
    first_name,
    SUM(score) AS total_score
FROM customers
GROUP BY country;

-- Find total score & number of customers per country
SELECT 
    country,
    SUM(score) AS total_score,
    COUNT(id) AS total_customers
FROM customers
GROUP BY country;
```

## 6. HAVING
```sql
-- Average score > 430 for each country
SELECT
    country,
    AVG(score) AS avg_score
FROM customers
GROUP BY country
HAVING AVG(score) > 430;

-- Only consider customers with score != 0
SELECT
    country,
    AVG(score) AS avg_score
FROM customers
WHERE score != 0
GROUP BY country
HAVING AVG(score) > 430;
```

## 7. DISTINCT
```sql
-- Unique list of countries
SELECT DISTINCT country
FROM customers;
```

## 8. TOP
```sql
-- Retrieve only 3 customers
SELECT TOP 3 * 
FROM customers;

-- Top 3 customers with highest scores
SELECT TOP 3 *
FROM customers
ORDER BY score DESC;

-- Lowest 2 customers by score
SELECT TOP 2 *
FROM customers
ORDER BY score ASC;

-- Two most recent orders
SELECT TOP 2 *
FROM orders
ORDER BY order_date DESC;
```

## 9. All Together
```sql
-- Average score per country, excluding score=0,
-- only > 430, ordered by highest avg_score
SELECT
    country,
    AVG(score) AS avg_score
FROM customers
WHERE score != 0
GROUP BY country
HAVING AVG(score) > 430
ORDER BY AVG(score) DESC;
```

## 10. COOL STUFF – Additional SQL Features
```sql
-- Execute multiple queries at once
SELECT * FROM customers;
SELECT * FROM orders;

-- Select static values
SELECT 123 AS static_number;
SELECT 'Hello' AS static_string;

-- Assign a constant value in query results
SELECT
    id,
    first_name,
    'New Customer' AS customer_type
FROM customers;
```