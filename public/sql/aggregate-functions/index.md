# Aggregate Functions

SQL aggregate functions perform calculations on multiple rows of data and return summary results.

---

## 1. Basic Aggregate Functions

### COUNT – Count rows  
```sql
SELECT COUNT(*) AS total_customers
FROM customers;
```

### SUM – Total of values
```sql
SELECT SUM(sales) AS total_sales
FROM orders;
```

### AVG – Average of values
```sql
SELECT AVG(sales) AS avg_sales
FROM orders;
```

### MAX – Maximum value
```sql
SELECT MAX(score) AS max_score
FROM customers;
```

### MIN – Minimum value
```sql
SELECT MIN(score) AS min_score
FROM customers;
```
## 2. Grouped Aggregations – GROUP BY

Aggregate results per group.
```sql
SELECT
    customer_id,
    COUNT(*) AS total_orders,
    SUM(sales) AS total_sales,
    AVG(sales) AS avg_sales,
    MAX(sales) AS highest_sales,
    MIN(sales) AS lowest_sales
FROM orders
GROUP BY customer_id;
```