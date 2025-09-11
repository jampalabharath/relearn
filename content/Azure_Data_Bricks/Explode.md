+++
title = "Explode"
weight = 10
+++

## Explode vs Explode_outer in PySpark

In PySpark, `explode` and `explode_outer` are functions used to work with nested data structures, like **arrays** or **maps**, by *“exploding”* (flattening) each element of an array or key-value pair in a map into separate rows.  

The key difference between **explode** and **explode_outer** is in handling **null** or **empty arrays**, which makes them useful in different scenarios.

---

### 1. explode()

The `explode()` function takes a column with array or map data and creates a new row for each element in the array (or each key-value pair in the map).  
If the array is empty or null, `explode()` will **drop the row entirely**.

#### Key Characteristics
- Converts each element in an array or each entry in a map into its own row.
- Drops rows with **null** or **empty arrays**.


## Example: Using `explode()` in PySpark

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import explode

# Initialize Spark session
spark = SparkSession.builder.appName("ExplodeExample").getOrCreate()

# Sample DataFrame with arrays
data = [
    ("Alice", ["Math", "Science"]),
    ("Bob", ["History"]),
    ("Cathy", []),   # Empty array
    ("David", None)  # Null array
]

df = spark.createDataFrame(data, ["Name", "Subjects"])
df.show()

# Use explode to flatten the array
exploded_df = df.select("Name", explode("Subjects").alias("Subject"))

# Show the result
exploded_df.show()
```


---

## 1. explode()

The `explode()` function expands the `Subjects` array into individual rows.  
Rows with **empty (`[]`)** or **null (`None`)** arrays are removed, which is why **Cathy** and **David** do not appear in the output.

### Key Characteristics
- Converts each element in an array or each entry in a map into its own row.
- **Drops** rows with null or empty arrays.

---

## 2. explode_outer()

The `explode_outer()` function works similarly to `explode()`, but it **keeps rows** with null or empty arrays.  
When `explode_outer()` encounters a null or empty array, it still generates a row for that entry, with **null** as the value in the resulting column.

### Key Characteristics
- Converts each element in an array or each entry in a map into its own row.
- **Retains** rows with null or empty arrays, using `null` values in the exploded column.


```python
# Use explode_outer to flatten the array while keeping null or empty rows
exploded_outer_df = df.select(
    "Name",
    F.explode_outer("Subjects").alias("Subject")
)

# Show the result
exploded_outer_df.show()
```


## Explanation: explode_outer()

- `explode_outer()` expands the **Subjects** array into individual rows.  
- Unlike `explode()`, rows with empty (`[]`) or null arrays (`None`) are **kept** in the result, with **null values** in the `Subject` column for these cases.

---

## Summary Table of Differences

| Function           | Description                                         | Null/Empty Arrays Behavior                |
|--------------------|-----------------------------------------------------|-------------------------------------------|
| **explode()**      | Expands each element of an array or map into rows   | **Drops** rows with null or empty arrays  |
| **explode_outer()**| Similar to `explode()`, but retains null/empty      | **Keeps** rows with null/empty arrays, fills with `null` |

---

These functions are very useful when working with **complex, nested data structures**, especially when dealing with **JSON** or other hierarchical data.
