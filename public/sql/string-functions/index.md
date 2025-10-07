# String Functions

This document provides an overview of **SQL string functions**, which allow manipulation, transformation, and extraction of text data efficiently.

---

## 1. Manipulations

### CONCAT() – String Concatenation
```sql
-- Concatenate first name and country into one column
SELECT 
    CONCAT(first_name, '-', country) AS full_info
FROM customers;
```

### LOWER() & UPPER() – Case Transformation
```sql
-- Convert the first name to lowercase
SELECT 
    LOWER(first_name) AS lower_case_name
FROM customers;

-- Convert the first name to uppercase
SELECT 
    UPPER(first_name) AS upper_case_name
FROM customers;
```

### TRIM() – Remove White Spaces

```sql
-- Find customers whose first name contains leading or trailing spaces
SELECT 
    first_name,
    LEN(first_name) AS len_name,
    LEN(TRIM(first_name)) AS len_trim_name,
    LEN(first_name) - LEN(TRIM(first_name)) AS flag
FROM customers
WHERE LEN(first_name) != LEN(TRIM(first_name));
-- Alternative:
-- WHERE first_name != TRIM(first_name)
```

### REPLACE() – Replace or Remove Values
```sql
-- Remove dashes (-) from a phone number
SELECT
    '123-456-7890' AS phone,
    REPLACE('123-456-7890', '-', '/') AS clean_phone;

-- Replace file extension from .txt to .csv
SELECT
    'report.txt' AS old_filename,
    REPLACE('report.txt', '.txt', '.csv') AS new_filename;
```
## 2. Calculation
### LEN() – String Length
```sql
-- Calculate the length of each customer's first name
SELECT 
    first_name, 
    LEN(first_name) AS name_length
FROM customers;
```

## 3. Substring Extraction
### LEFT() & RIGHT()
```sql
-- Retrieve the first two characters of each first name
SELECT 
    first_name,
    LEFT(TRIM(first_name), 2) AS first_2_chars
FROM customers;

-- Retrieve the last two characters of each first name
SELECT 
    first_name,
    RIGHT(first_name, 2) AS last_2_chars
FROM customers;
```

### SUBSTRING()
```sql
-- Retrieve a list of customers' first names after removing the first character
SELECT 
    first_name,
    SUBSTRING(TRIM(first_name), 2, LEN(first_name)) AS trimmed_name
FROM customers;
```
## 4. Nesting Functions
-- Nesting example
SELECT
    first_name, 
    UPPER(LOWER(first_name)) AS nesting
FROM customers;
