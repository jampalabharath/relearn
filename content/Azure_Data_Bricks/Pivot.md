+++
title = "Pivot"
weight = 11
+++

The **pivot** operation in PySpark is used to **transpose rows into columns** based on a specified column's unique values.  
It's particularly useful for creating **wide-format data**, where values in one column become new column headers, and corresponding values from another column fill those headers.

---

## Key Concepts

1. **groupBy and pivot**
   - The `pivot` method is typically used in combination with `groupBy`.  
   - You group by certain columns and pivot one column to create new columns.

2. **Aggregation Function**
   - You need to specify an aggregation function (like `sum`, `avg`, `count`, etc.) to fill the values in the pivoted columns.

3. **Performance Consideration**
   - Pivoting can be **computationally expensive**, especially with a high number of unique values in the pivot column.  
   - For better performance, explicitly specify the values to pivot if possible.

4. **Syntax**

```python
dataframe.groupBy("group_column").pivot("pivot_column").agg(aggregation_function)
```


## Example Code: Pivot in PySpark

### Sample Data
Imagine we have a DataFrame of sales data with the following schema:
| Product | Region | Sales |
|---------|--------|-------|
| A       | North  | 100   |
| B       | North  | 150   |
| A       | South  | 200   |
| B       | South  | 300   |

We want to pivot the data so that regions (North, South) become columns and the sales values are aggregated.

### Code Implementation

# Example: Pivot in PySpark

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import sum

# Create a Spark session
spark = SparkSession.builder.appName("PivotExample").getOrCreate()

# Create a sample DataFrame
data = [
    ("A", "North", 100),
    ("B", "North", 150),
    ("A", "South", 200),
    ("B", "South", 300)
]

columns = ["Product", "Region", "Sales"]

df = spark.createDataFrame(data, columns)

# Pivot the DataFrame
pivoted_df = df.groupBy("Product").pivot("Region").agg(sum("Sales"))

# Show the results
pivoted_df.show()
```


## Output

| Product | North | South |
|---------|-------|-------|
| A       | 100   | 200   |
| B       | 150   | 300   |


## Explanation of Code

1. **groupBy("Product")**  
   - Groups the data by the `Product` column.

2. **pivot("Region")**  
   - Transforms unique values in the `Region` column (`North`, `South`) into new columns.

3. **agg(sum("Sales"))**  
   - Computes the sum of `Sales` for each combination of `Product` and the new columns created by the pivot.

---

## Notes

- **Explicit Pivot Values**: To improve performance, you can specify the pivot values explicitly.  

```df.groupBy("Product").pivot("Region", ["North", "South"]).agg(sum("Sales"))```
- **Handling Null Values**: If some combinations of `groupBy` and pivot values have no corresponding rows, the resulting cells will contain `null`.  
- **Alternative Aggregations**: You can use other aggregation functions like `avg`, `max`, `min`, etc.  

---

This approach is commonly used in creating **summary reports** or preparing data for **machine learning models** where wide-format data is required.


## Unpivot in PySpark

The unpivot operation (also called melting) is used to transform a wide-format table into a long-format table. This means columns are turned into rows, effectively reversing the pivot operation. PySpark doesn't have a direct unpivot function like Pandas' melt, but you can achieve it using the selectExpr method or a combination of stack and other DataFrame transformations.

### Key Concepts

### 1. Purpose of Unpivot:

- Simplifies data analysis by converting column headers into a single column (e.g., categorical variables).
    
- Ideal for scenarios where you need to aggregate data further or visualize it in a long format.
    

### 2. Syntax Overview:

- Use the stack function inside a selectExpr to unpivot.
    
- Stack reshapes the DataFrame by creating multiple rows for specified columns.
    

### 3. Performance:

- Unpivoting can generate many rows, especially if the original DataFrame is wide with numerous columns. Ensure your environment can handle the resulting data volume.


### Example: Unpivot in PySpark
**Sample Data**
Suppose we have the following DataFrame:
| Product | North | South | East | West |
| ------- | ----- | ----- | ---- | ---- |
| A       | 100   | 200   | 150  | 130  |
| B       | 150   | 300   | 200  | 180  |

```python
from pyspark.sql import SparkSession

# Create a Spark session
spark = SparkSession.builder.appName("UnpivotExample").getOrCreate()

# Sample data
data = [
    ("A", 100, 200, 150, 130),
    ("B", 150, 300, 200, 180)
]
columns = ["Product", "North", "South", "East", "West"]

# Create the DataFrame
df = spark.createDataFrame(data, columns)

# Unpivot the DataFrame using stack
unpivoted_df = df.selectExpr(
    "Product",
    "stack(4, 'North', North, 'South', South, 'East', East, 'West', West) as (Region, Sales)"
)

# Show the results
unpivoted_df.show()
```

## Explanation of Code

1. **Input DataFrame**:  
   - Each column (North, South, East, West) represents a region's sales for each product.

2. **selectExpr with stack**:  
   - The stack function takes two arguments:  
     - The number of columns being unpivoted (4 in this case).  
     - A sequence of column-value pairs: 'ColumnName1', ColumnValue1, 'ColumnName2', ColumnValue2, ....  
   - The result is two new columns: the first contains the column names (now rows, Region), and the second contains the corresponding values (Sales).

3. **Aliasing Columns**:  
   - The stack result is aliased as (Region, Sales) to give meaningful names to the new columns.

---

## Alternative Methods

**Using withColumn and union**:  
If stack isn't flexible enough, you can manually combine rows for each column:

```python
from pyspark.sql import functions as F

# Create a DataFrame with union operations for unpivoting
north = df.select("Product", F.lit("North").alias("Region"), F.col("North").alias("Sales"))
south = df.select("Product", F.lit("South").alias("Region"), F.col("South").alias("Sales"))
east = df.select("Product", F.lit("East").alias("Region"), F.col("East").alias("Sales"))
west = df.select("Product", F.lit("West").alias("Region"), F.col("West").alias("Sales"))

# Combine all rows using union
unpivoted_df = north.union(south).union(east).union(west)

# Show results
unpivoted_df.show()
```


## Notes

1. **Performance Considerations**  
   - `stack` is efficient for unpivoting a large number of columns.  
   - The `union` method may become unwieldy for many columns, but it offers more control over the transformation process.

2. **Dynamic Column Unpivoting**  
   - If the column names are not fixed (dynamic), you can:  
     - Collect the column names dynamically using `df.columns`.  
     - Construct the `selectExpr` or `union` queries programmatically.

3. **Resulting Format**  
   - After unpivoting, the data will have more rows but fewer columns.  
   - Ensure downstream processes are optimized to handle the increased row count.

---

Unpivoting is a powerful operation for **restructuring data** and is frequently used in **data preprocessing, reporting, and machine learning pipelines**.
