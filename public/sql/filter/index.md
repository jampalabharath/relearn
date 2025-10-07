# Filtering Data

This document provides an overview of SQL filtering techniques using `WHERE` and various operators for precise data retrieval.

---

## 1. Comparison Operators (=, <>, >, >=, <, <=)
```sql
-- Retrieve all customers from Germany
SELECT *
FROM customers
WHERE country = 'Germany';

-- Retrieve all customers who are not from Germany
SELECT *
FROM customers
WHERE country <> 'Germany';

-- Retrieve all customers with a score greater than 500
SELECT *
FROM customers
WHERE score > 500;

-- Retrieve all customers with a score of 500 or more
SELECT *
FROM customers
WHERE score >= 500;

-- Retrieve all customers with a score less than 500
SELECT *
FROM customers
WHERE score < 500;

-- Retrieve all customers with a score of 500 or less
SELECT *
FROM customers
WHERE score <= 500;
```

## 2. Logical Operators (AND, OR, NOT)

```sql
-- Customers from the USA and score > 500
SELECT *
FROM customers
WHERE country = 'USA' AND score > 500;

-- Customers from the USA or score > 500
SELECT *
FROM customers
WHERE country = 'USA' OR score > 500;

-- Customers with score not less than 500
SELECT *
FROM customers
WHERE NOT score < 500;
```

## 3. Range Filtering – BETWEEN
```sql
-- Customers with score between 100 and 500
SELECT *
FROM customers
WHERE score BETWEEN 100 AND 500;

-- Equivalent to BETWEEN
SELECT *
FROM customers
WHERE score >= 100 AND score <= 500;
```

## 4. Set Filtering – IN
```sql
-- Customers from Germany or USA
SELECT *
FROM customers
WHERE country IN ('Germany', 'USA');
```

## 5. Pattern Matching – LIKE

```sql
-- First name starts with 'M'
SELECT *
FROM customers
WHERE first_name LIKE 'M%';

-- First name ends with 'n'
SELECT *
FROM customers
WHERE first_name LIKE '%n';

-- First name contains 'r'
SELECT *
FROM customers
WHERE first_name LIKE '%r%';

-- First name has 'r' in the third position
SELECT *
FROM customers
WHERE first_name LIKE '__r%';
```