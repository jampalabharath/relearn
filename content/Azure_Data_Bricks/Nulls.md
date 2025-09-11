+++
title = "Handling Nulls"
weight = 5
+++

## Sample Sales Data with Null Values

```python
# Sample data: sales data with nulls
data = [
    ("John", "North", 100, None),
    ("Doe", "East", None, 50),
    (None, "West", 150, 30),
    ("Alice", None, 200, 40),
    ("Bob", "South", None, None),
    (None, None, None, None)
]

columns = ["Name", "Region", "UnitsSold", "Revenue"]

# Create DataFrame
df = spark.createDataFrame(data, columns)
df.show()
```

### 1. Detecting Null Values

- Use **`isNull()`** to identify rows where a column contains null values.    
- The output is a boolean flag indicating whether the value is null.
   
### 2. Dropping Rows with Null Values

- **`dropna()`** removes rows with nulls in any column (default mode). 
- Use **`how="all"`** to remove rows only if _all_ columns are null.
- Use **`subset=["col1", "col2"]`** to target specific columns.

### 3. Filling Null Values

- **`fillna()`** replaces nulls with specified default values.    
- Replace across all columns or selectively.
- Example:
    - Replace `Region` nulls with `"Unknown"`.
    - Replace `UnitsSold` and `Revenue` nulls with `0`.
        
### 4. Coalesce Function

- **`coalesce()`** returns the first non-null value among multiple columns.
- Useful when providing fallback values if some columns contain nulls.

### 5. Handling Nulls in Aggregations

- Nulls can distort aggregations like `mean()`.    
- Use **`coalesce()`** to substitute nulls with defaults (e.g., `0.0`).
- This prevents inaccurate results.

### 📌 Summary

1. **Detecting Nulls**: Use `isNull()` to find null values.
2. **Dropping Nulls**: Use `dropna()` to remove rows with nulls (all or specific columns).
3. **Filling Nulls**: Use `fillna()` to replace nulls with defaults.
4. **Coalesce Function**: Use `coalesce()` to return the first non-null value.
5. **Aggregations**: Use `coalesce()` in aggregations to handle nulls safely.