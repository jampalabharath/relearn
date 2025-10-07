# DF Operations

## Column Selection & Manipulation

### 1. Different Methods to Select Columns

In PySpark, you can select specific columns in multiple ways:

```python
# Using col() function
df.select(col("Name")).show()

# Using column() function
df.select(column("Age")).show()

# Directly using string name
df.select("Salary").show()
```

### 2. Selecting Multiple Columns Together

You can combine different methods to select multiple columns:

```python
# multiple column
df2 = df.select("ID", "Name", col("Salary"), column("Department"), df.Phone)
df2.show()
```
### 3. Listing All Columns in a DataFrame

To get a list of all the column names:

```python
# get all column name
df.columns
```

### 4. Renaming Columns with alias()

You can rename columns using the `alias()` method:


```python
df.select(
  col("Name").alias('EmployeeName'),  # Rename "Name" to "EmployeeName"
  col("Salary").alias('EmployeeSalary'),  # Rename "Salary" to "EmployeeSalary"
  column("Department"),  # Select "Department"
  df.Joining_Date  # Select "Joining_Date"
).show()
```

### 5. Using selectExpr() for Concise Column Selection

`selectExpr()` allows you to use SQL expressions directly and rename columns concisely:

```python
df.selectExpr("Name as EmployeeName", "Salary as EmployeeSalary", "Department").show()
```

### Summary

- Use `col()`, `column()`, or string names to select columns.
- Use `expr()` and `selectExpr()` for SQL-like expressions and renaming.
- Use `alias()` to rename columns.
- Get the list of columns using `df.columns`.

---

## Adding, Renaming, and Dropping Columns

### 1. Adding New Columns with withColumn()

In PySpark, the `withColumn()` function is widely used to add new columns to a DataFrame. You can either assign a constant value using `lit()` or perform transformations using existing columns.

#### Add a constant value column:
```python
newdf = df.withColumn("NewColumn", lit(1))
```

#### Add a column based on an expression:

```python
newdf = df.withColumn("withinCountry", expr("Country == 'India'"))
```

This function allows adding multiple columns, including calculated ones:

**Example:**

- Assign a constant value with `lit()`.
- Perform calculations using existing columns like multiplying values.

### 2. Renaming Columns with withColumnRenamed()

PySpark provides the `withColumnRenamed()` method to rename columns. This is especially useful when you want to change the names for clarity or to follow naming conventions:

### Renaming a column:

python

```python
new_df = df.withColumnRenamed("oldColumnName", "newColumnName")
```

#### Handling column names with special characters or spaces:

If a column has special characters or spaces, you need to use backticks (`) to escape it:

python

```python
newdf.select("`New Column Name`").show()
```

### 3. Dropping Columns with drop()

To remove unwanted columns, you can use the `drop()` method:

#### Drop a single column:

```python
df2 = df.drop("Country")
```

#### Drop multiple columns:

```python
df2 = df.drop("Country", "Region")
```

Dropping columns creates a new DataFrame, and the original DataFrame remains unchanged.

### 4. Immutability of DataFrames

In Spark, DataFrames are immutable by nature. This means that after creating a DataFrame, its contents cannot be changed. All transformations like adding, renaming, or dropping columns result in a new DataFrame, keeping the original one intact.

#### For instance, dropping columns creates a new DataFrame without altering the original:

```python
newdf = df.drop("ItemType", "SalesChannel")
```

This immutability ensures data consistency and supports Spark's parallel processing, as transformations do not affect the source data.

### Key Points

- Use `withColumn()` for adding columns, with `lit()` for constant values and expressions for computed values.
- Use `withColumnRenamed()` to rename columns and backticks for special characters or spaces.
- Use `drop()` to remove one or more columns.
- DataFrames are immutable in Spark—transformations result in new DataFrames, leaving the original unchanged.


---

## Data Types, Filtering, and Unique Values

Here's a structured set of notes with code to cover changing data types, filtering data, and handling unique/distinct values in PySpark using the employee data:

### 1. Changing Data Types (Schema Transformation)

In PySpark, you can change the data type of a column using the `cast()` method. This is helpful when you need to convert data types for columns like Salary or Phone.

```python
from pyspark.sql.functions import col

# Change the 'Salary' column from integer to double
df = df.withColumn("Salary", col("Salary").cast("double"))

# Convert 'Phone' column to string
df = df.withColumn("Phone", col("Phone").cast("string"))

df.printSchema()
```

### 2. Filtering Data

You can filter rows based on specific conditions. For instance, to filter employees with a salary greater than 50,000:

```python
# Filter rows where Salary is greater than 50,000
filtered_df = df.filter(col("Salary") > 50000)
filtered_df.show()

# Filtering rows where Age is not null
filtered_df = df.filter(df["Age"].isNotNull())
filtered_df.show()
```

### 3. Multiple Filters (Chaining Conditions)

You can also apply multiple conditions using `&` or `|` (AND/OR) to filter data. For example, finding employees over 30 years old and in the IT department:

```python
# Filter rows where Age > 30 and Department is 'IT'
filtered_df = df.filter((df["Age"] > 30) & (df["Department"] == "IT"))
filtered_df.show()
```

### 4. Filtering on Null or Non-Null Values

Filtering based on whether a column has NULL values or not is crucial for data cleaning:

```python
# Filter rows where 'Address' is NULL
filtered_df = df.filter(df["Address"].isNull())
filtered_df.show()

# Filter rows where 'Email' is NOT NULL
filtered_df = df.filter(df["Email"].isNotNull())
filtered_df.show()
```

### 5. Handling Unique or Distinct Data

To get distinct rows or unique values from your dataset:

```python
# Get distinct rows from the entire DataFrame
unique_df = df.distinct()
unique_df.show()

# Get distinct values from the 'Department' column
unique_departments_df = df.select("Department").distinct()
unique_departments_df.show()
```

To remove duplicates based on specific columns, such as Email or Phone, use `dropDuplicates()`:

```python
# Remove duplicates based on 'Email' column
unique_df = df.dropDuplicates(["Email"])
unique_df.show()

# Remove duplicates based on both 'Phone' and 'Email'
unique_df = df.dropDuplicates(["Phone", "Email"])
unique_df.show()
```

### 6. Counting Distinct Values

You can count distinct values in a particular column, or combinations of columns:

```python
# Count distinct values in the 'Department' column
distinct_count_department = df.select("Department").distinct().count()
print("Distinct Department Count:", distinct_count_department)

# Count distinct combinations of 'Department' and 'Performance_Rating'
distinct_combinations_count = df.select("Department", "Performance_Rating").distinct().count()
print("Distinct Department and Performance Rating Combinations:", distinct_combinations_count)
```

This set of operations will help you efficiently manage and transform your data in PySpark, ensuring data integrity and accuracy for your analysis!

### Mastering PySpark DataFrame Operations

1. **Changing Data Types**: Easily modify column types using `.cast()`. E.g., change 'Salary' to double or 'Phone' to string for better data handling.
2. **Filtering Data**: Use `.filter()` or `.where()` to extract specific rows. For example, filter employees with a salary over 50,000 or non-null Age.
3. **Multiple Conditions**: Chain filters with `&` and `|` to apply complex conditions, such as finding employees over 30 in the IT department.
4. **Handling NULLs**: Use `.isNull()` and `.isNotNull()` to filter rows with missing or available values, such as missing addresses or valid emails.
5. **Unique/Distinct Values**: Use `.distinct()` to get unique rows or distinct values in a column. Remove duplicates based on specific fields like Email or Phone using `.dropDuplicates()`.
6. **Count Distinct Values**: Count distinct values in one or multiple columns to analyze data diversity, such as counting unique departments or combinations of Department and Performance_Rating.