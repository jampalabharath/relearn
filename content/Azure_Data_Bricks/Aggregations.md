+++
title = "Aggregate functions"
weight = 6
+++


## Basic Aggregate Functions

### Sample Data

```python
from pyspark.sql import Row

# Create sample data
data = [
    Row(id=1, value=10),
    Row(id=2, value=20),
    Row(id=3, value=30),
    Row(id=4, value=None),
    Row(id=5, value=40),
    Row(id=6, value=20)
]

# Create DataFrame
df = spark.createDataFrame(data)

# Show the DataFrame
df.show()
```

### Aggregate Functions in PySpark

1. **Summation (`sum`)** – Adds up the values in a column. 
2. **Average (`avg`)** – Computes the average of values in a column.
3. **Count (`count`)** – Counts the number of non-null values in a column.
4. **Maximum (`max`) / Minimum (`min`)** – Finds the highest and lowest values.
5. **Distinct Count (`countDistinct`)** – Counts unique values in a column.

### Notes

- **Handling Nulls**:    
    - `count()` counts only **non-null** values.
    - `sum()`, `avg()`, `max()`, and `min()` ignore null values.
        
- **Performance**:  
    Aggregate functions can be expensive on large datasets; partitioning improves performance.
    
- **Use Cases**:
    - **Summation**: Total sales, total revenue.
    - **Average**: Average sales per day.
    - **Count**: Number of transactions.
    - **Max/Min**: Highest and lowest values (e.g., max sales in a day).
    - **Distinct Count**: Unique customers, unique products.
---

## Advanced Aggregation Functions

### Sample Data
```python
from pyspark.sql import SparkSession
from pyspark.sql import functions as F
from pyspark.sql.types import StructType, StructField, StringType, IntegerType

# Create Spark session
spark = SparkSession.builder.appName("AggregationExamples").getOrCreate()

# Sample data
data = [
    ("HR", 10000, 500, "John"),
    ("Finance", 20000, 1500, "Doe"),
    ("HR", 15000, 1000, "Alice"),
    ("Finance", 25000, 2000, "Eve"),
    ("HR", 20000, 1500, "Mark")
]

# Define schema
schema = StructType([
    StructField("department", StringType(), True),
    StructField("salary", IntegerType(), True),
    StructField("bonus", IntegerType(), True),
    StructField("employee_name", StringType(), True)
])

# Create DataFrame
df = spark.createDataFrame(data, schema)
df.show()
```
### 1. Grouped Aggregation

Perform aggregation within groups based on a column.

- **`sum()`** → Adds values within the group. 
- **`avg()`** → Computes group average.
- **`max()`** → Finds maximum value.
- **`min()`** → Finds minimum value.

### 2. Multiple Aggregations

Perform several aggregations in one step.
- **`count()`** → Number of rows in each group.
- **`avg()`** → Average of values.
- **`max()`** → Maximum value in group.

### 3. Concatenate Strings

- **`concat_ws()`** → Concatenates string values within a column, separated by a delimiter (`,`).

### 4. First and Last

- **`first()`** → Retrieves the first value of a column in a group.    
- **`last()`** → Retrieves the last value of a column in a group.

### 5. Standard Deviation and Variance

- **`stddev()`** → Standard deviation of values.
- **`variance()`** → Variance of values.

### 6. Aggregation with Alias

- **`.alias()`** → Rename the result columns after aggregation.

### 7. Sum of Distinct Values

- **`sumDistinct()`** → Sums only **unique values** in a column (avoids double-counting duplicates).

### 📌 Summary

- Use basic aggregations (`sum`, `avg`, `count`, `max`, `min`, `countDistinct`) for general metrics.
- Apply advanced aggregations (`grouped`, `concat_ws`, `first`, `last`, `stddev`, `variance`, `sumDistinct`) for deeper analysis.

- Always consider **null handling** and **performance optimizations** when using aggregate functions in PySpark.