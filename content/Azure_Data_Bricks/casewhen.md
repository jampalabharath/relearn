+++
title="When|Cast|Union"
+++


## `when` and `otherwise`

The `when` and `otherwise` functions in **PySpark** provide a way to create conditional expressions within a DataFrame, allowing you to specify different values for new or existing columns based on specific conditions.

- **`when`**:  
  The `when` function in PySpark is used to define a condition.  
  If the condition is met, it returns the specified value.  
  You can chain multiple `when` conditions to handle various cases.

- **`otherwise`**:  
  The `otherwise` function specifies a default value to return if none of the conditions in the `when` statements are met.

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import when
from pyspark.sql.types import StructType, StructField, IntegerType, StringType

# Initialize Spark session
spark = SparkSession.builder.appName("WhenOtherwiseExample").getOrCreate()

# Define the schema for the dataset
schema = StructType([
    StructField("name", StringType(), True),
    StructField("age", IntegerType(), True),
    StructField("salary", IntegerType(), True)
])

# Create a sample dataset
data = [
    ("Alice", 25, 3000),
    ("Bob", 35, 4000),
    ("Charlie", 40, 5000),
    ("David", 28, 4500),
    ("Eve", 32, 3500)
]

# Create DataFrame
df = spark.createDataFrame(data, schema)
df.show()

# Apply 'when' and 'otherwise' to add new columns based on conditions
df = (
    df.withColumn("status", when(df.age < 30, "Young").otherwise("Adult"))
      .withColumn("income_bracket",
                  when(df.salary < 4000, "Low")
                  .when((df.salary >= 4000) & (df.salary <= 4500), "Medium")
                  .otherwise("High"))
)

# Show the result
df.show()
```

---

#### Explanation

1. **`status` column**  
   - Assigns `"Young"` if `age < 30`.  
   - Otherwise assigns `"Adult"`.  

2. **`income_bracket` column**  
   - Assigns `"Low"` if `salary < 4000`.  
   - Assigns `"Medium"` if `4000 <= salary <= 4500`.  
   - Assigns `"High"` for any other salary values.  

This approach allows for flexible handling of multiple conditions in PySpark DataFrames using `when` and `otherwise`.

## `cast()` and `printSchema()`

In PySpark, the `cast()` function is used to change the data type of a column within a DataFrame.  
This is helpful when you need to standardize column data types for data processing, schema consistency, or compatibility with other operations.

- **Purpose**:  
  The `cast()` function allows you to change the data type of a column, useful in situations like standardizing formats (e.g., converting strings to dates or integers).

- **Syntax**:  
  The `cast()` function is applied on individual columns and requires specifying the target data type in quotes.

- **Multiple Columns**:  
  You can cast multiple columns at once by using a list of cast expressions and passing them to `select()`.

- **Supported Data Types**:  
  PySpark supports various data types for casting, including:
  - `StringType`
  - `IntegerType` (or `"int"`)
  - `DoubleType` (or `"double"`)
  - `DateType`
  - `TimestampType`
  - `BooleanType`
  - Others, based on the data types available in PySpark.

```python
from pyspark.sql.functions import col

# Single column cast
df = df.withColumn("column_name", col("column_name").cast("target_data_type"))

# Multiple columns cast with select
cast_expr = [
    col("column1_name").cast("target_data_type1"),
    col("column2_name").cast("target_data_type2"),
    # More columns and data types as needed
]

df = df.select(*cast_expr)
```

#### Example
Let's create a dataset and apply `cast()` to change the data types of multiple columns:

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col
from pyspark.sql.types import StructType, StructField, StringType, IntegerType, FloatType

# Initialize Spark session
spark = SparkSession.builder.appName("CastExample").getOrCreate()

# Define the schema for the dataset
schema = StructType([
    StructField("name", StringType(), True),
    StructField("age", StringType(), True),    # Stored as StringType initially
    StructField("height", StringType(), True)  # Stored as StringType initially
])

# Create a sample dataset
data = [
    ("Alice", "25", "5.5"),
    ("Bob", "35", "6.1"),
    ("Charlie", "40", "5.8"),
]

# Create DataFrame
df = spark.createDataFrame(data, schema)

# Show schema and data before casting
df.printSchema()
df.show()
```

```python
# Define cast expressions for multiple columns
cast_expr = [
    col("name").cast("string"),
    col("age").cast("int"),       # Casting age to IntegerType
    col("height").cast("double")  # Casting height to DoubleType
]

# Apply the cast expressions to the DataFrame
df = df.select(*cast_expr)

# Show the result
df.printSchema()
df.show()
```

#### Explanation

- **`age` column**: Initially stored as `StringType`, it’s cast to `IntegerType` (or `"int"`).  
- **`height` column**: Initially stored as `StringType`, it’s cast to `DoubleType` (or `"double"`).  

---

#### Advantages of Using `cast()`

- **Schema Alignment**: Ensures data types in different tables or DataFrames are compatible for joining or union operations.  
- **Data Consistency**: Ensures all columns conform to expected data types for downstream data processing.  
- **Error Reduction**: Minimizes issues arising from mismatched data types in computations or transformations.  

This approach using `cast()` provides a flexible and powerful way to manage data types in PySpark.  

---

#### `printSchema()` Method in PySpark

- **Purpose**:  
  - To display the schema of a DataFrame, which includes the column names, data types, and nullability of each column.  

- **Output Structure**:  
  The schema is presented in a tree-like structure showing:  
  - **Column Name**: The name of the column.  
  - **Data Type**: The data type of the column (e.g., `string`, `integer`, `double`, `boolean`, etc.).  
  - **Nullability**: Indicates whether the column can contain null values (e.g., `nullable = true`).  

- **Usage**:  
  - Call `df.printSchema()` on a DataFrame `df` to see its structure.  
  - Useful for verifying the structure of the DataFrame after operations like `select()`, `withColumn()`, or `cast()`. 


## `union` and `unionAll`

#### Overview
- **Purpose**: Both `union` and `unionAll` are used to combine two DataFrames into a single DataFrame.  
- **DataFrame Compatibility**: The two DataFrames must have the same schema (i.e., the same column names and data types) to perform the union operation.  

---

#### `union()`
- **Functionality**:  
  - Combines two DataFrames and retains all rows, including duplicates.  

- **Behavior**:  
  - The `union()` method does not remove duplicate rows, resulting in a DataFrame that may contain duplicates.  

---

#### `unionAll()`
- **Functionality**:  
  - Combines two DataFrames and retains all rows, including duplicates.  

- **Behavior**:  
  - The `unionAll()` method performs the union operation but does not eliminate duplicate rows (similar to `union`).  

---

#### Syntax

```python
# Using union to retain all rows including duplicates
unioned_df = df1.union(df2)

# Using unionAll to retain all rows including duplicates
unionAll_df = df1.unionAll(df2)
```
 
 #### Example: Using `union` and `unionAll` in PySpark

```python
from pyspark.sql import SparkSession

# Initialize Spark session
spark = SparkSession.builder.appName("UnionExample").getOrCreate()

# Sample DataFrames
data1 = [("Alice", 25), ("Bob", 30), ("Charlie", 35)]
data2 = [("David", 40), ("Eve", 45), ("Alice", 25)]
columns = ["name", "age"]

df1 = spark.createDataFrame(data1, columns)
df2 = spark.createDataFrame(data2, columns)

# Using union to retain all rows including duplicates
unioned_df = df1.union(df2)

# Using unionAll to retain all rows
unionAll_df = df1.unionAll(df2)

# Show the results
print("unioned_df (No duplicates removed):")
unioned_df.show()

print("unionAll_df (duplicates retained):")
unionAll_df.show()


## Removing Duplicate Rows in PySpark

# Remove duplicate rows and create a new DataFrame
unique_df = unioned_df.dropDuplicates()
# or
unique_df = unioned_df.distinct()

print("unique_df (after removing duplicates):")
unique_df.show()

```

#### Union and UnionByName

In PySpark, both `union` and `unionByName` are operations that allow you to combine two or more DataFrames. However, they do this in slightly different ways, particularly regarding how they handle column names.

---

##### 1. `union`

**Definition**:  
The `union()` function is used to combine two DataFrames with the same schema (i.e., the same number of columns with the same data types).  
It appends the rows of one DataFrame to the other.

**Key Characteristics**:
- The DataFrames must have the same number of columns.  
- The columns must have compatible data types.  
- It does **not** automatically handle column names that differ between DataFrames.  

#### Syntax

```python
DataFrame.union(otherDataFrame)
```
```python
from pyspark.sql import SparkSession

# Create a Spark session
spark = SparkSession.builder.appName("Union Example").getOrCreate()

# Create two DataFrames with the same schema
data1 = [("Alice", 1), ("Bob", 2)]
data2 = [("Cathy", 3), ("David", 4)]
columns = ["Name", "Id"]

df1 = spark.createDataFrame(data1, columns)
df2 = spark.createDataFrame(data2, columns)

# Perform union
result_union = df1.union(df2)

# Show the result
result_union.show()
```
#### 2. `unionByName`

**Definition**:  
The `unionByName()` function allows you to combine two DataFrames by matching column names.  
If the DataFrames do not have the same schema, it will fill in missing columns with `null`.

**Key Characteristics**:
- Matches DataFrames by **column names** rather than position.  
- If the DataFrames have different columns, it will include all columns and fill in `null` for missing values in any DataFrame.  
- You can specify `allowMissingColumns=True` to ignore missing columns.  

#### Syntax

```python
DataFrame.unionByName(otherDataFrame, allowMissingColumns=False)
```

```python
# Create two DataFrames with different schemas
data3 = [("Eve", 5), ("Frank", 6)]
data4 = [("Grace", "New York"), ("Hannah", "Los Angeles")]

columns1 = ["Name", "Id"]
columns2 = ["Name", "City"]

df3 = spark.createDataFrame(data3, columns1)
df4 = spark.createDataFrame(data4, columns2)

# Perform unionByName (with allowMissingColumns=True to handle schema differences)
result_union_by_name = df3.unionByName(df4, allowMissingColumns=True)

# Show the result result_union_by_name.show()
result_union_by_name.show()

```
#### Summary of Differences

- **`union()`**: Requires DataFrames to have the **same schema** (same number of columns and compatible data types). It combines rows without checking column names.  
- **`unionByName()`**: Matches DataFrames by **column names**. It can handle different schemas and fill missing columns with `null` (when `allowMissingColumns=True`).  

---

| Feature                  | Union       | UnionByName                    |
|--------------------------|------------|-------------------------------|
| Column Matching          | Positional | By Name                        |
| Missing Columns Handling | Does not allow | Allows with `null` for missing |
| Schema Requirement       | Must be identical | Can differ                 |


#### Conclusion

In PySpark:  
- Use **`union()`** when you have DataFrames with the **same schema** and need a straightforward concatenation.  
- Use **`unionByName()`** when your DataFrames have **different schemas** and you want to combine them by matching column names while handling missing columns.  




