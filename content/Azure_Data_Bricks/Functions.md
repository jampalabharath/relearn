+++
title = "Functions"
weight = 3
+++

## Sorting and String Functions


```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, desc, asc, concat, concat_ws, initcap, lower, upper, instr, length, lit

# Create a Spark session
spark = SparkSession.builder.appName("SortingAndStringFunctions").getOrCreate()

# Sample data
data = [
   ("USA", "North America", 100, 50.5),
   ("India", "Asia", 300, 20.0),
   ("Germany", "Europe", 200, 30.5),
   ("Australia", "Oceania", 150, 60.0),
   ("Japan", "Asia", 120, 45.0),
   ("Brazil", "South America", 180, 25.0)
]

# Define the schema
columns = ["Country", "Region", "UnitsSold", "UnitPrice"]

# Create DataFrame
df = spark.createDataFrame(data, columns)

# Display the original DataFrame
df.show()
```
### Sorting the DataFrame

### 1. Sort by a single column (ascending order)

`df.orderBy("Country").show(5)`

> By default, sorting is ascending. This shows the first 5 countries alphabetically.

### 2. Sort by multiple columns

`df.orderBy("Country", "UnitsSold").show(5)`

> First sorts by `Country`, then within each country sorts by `UnitsSold`.

### 3. Sort by column in descending order and limit

`df.orderBy(desc("Country")).limit(3).show(5)`

> Sorts by `Country` in descending order and returns the top 3 rows.

### 4. Sorting with null values last

`df.orderBy(col("Country").desc(), nulls_last=True).show(5)`

> Ensures null values appear at the end when sorting.

**Key Functions:**

- Use `.orderBy()` or `.sort()` to sort DataFrames.
    
- Control order with `asc()` or `desc()`.
    

---

## String Functions

### 1. Capitalize first letter of each word

`df.select(initcap(col("Country"))).show()`

> Converts `"united states"` → `"United States"`.

### 2. Convert all text to lowercase

`df.select(lower(col("Country"))).show()`


### 3. Convert all text to uppercase

`df.select(upper(col("Country"))).show()`

**Key Functions:**

- `initcap()` → Capitalize first letter of each word.
    
- `lower()` → Convert to lowercase.
    
- `upper()` → Convert to uppercase.
    

---

## Concatenation Functions

### 1. Concatenate two columns

`df.select(concat(col("Region"), col("Country"))).show()`

> Joins `Region` and `Country` without separator.

### 2. Concatenate with a separator

`df.select(concat_ws(" | ", col("Region"), col("Country"))).show()`

> Joins with `" | "` as separator.

### 3. Create a new concatenated column

`df.withColumn("concatenated", concat(df["Region"], lit(" "), df["Country"])).show()`

> Adds a new column combining `Region` and `Country`.

**Key Functions:**

- `concat()` → Join columns directly.
    
- `concat_ws()` → Join columns with a separator.
    

---

### 📌 Summary

- **Sorting:** Use `.orderBy()` or `.sort()` with `asc()` / `desc()`.
- **String Manipulation:** Use `initcap()`, `lower()`, `upper()`.  
- **Concatenation:** Use `concat()` or `concat_ws()` for flexible joins.

---

## Split Function in DataFrame

Let’s create a PySpark DataFrame for employee data with columns such as **EmployeeID, Name, Department, and Skills**. We’ll explore `split`, `explode`, and other useful array functions.

### Sample Data Creation for Employee Data

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import split, explode, size, array_contains, col

# Sample employee data
data = [
   (1, "Alice", "HR", "Communication Management"),
   (2, "Bob", "IT", "Programming Networking"),
   (3, "Charlie", "Finance", "Accounting Analysis"),
   (4, "David", "HR", "Recruiting Communication"),
   (5, "Eve", "IT", "Cloud DevOps")
]

# Define the schema
columns = ["EmployeeID", "Name", "Department", "Skills"]

# Create DataFrame
df = spark.createDataFrame(data, columns)

# Display the original DataFrame
df.show(truncate=False)
```

### Examples

### 1. Split the `Skills` column

`df.select(col("EmployeeID"), col("Name"), split(col("Skills"), " ").alias("Skills_Array")).show(truncate=False)`

> Splits the `Skills` column into an array of skills using space as a delimiter.

---

### 2. Select the first skill from `Skills_Array`

`df2.select(col("EmployeeID"), col("Name"), col("Skills_Array")[0].alias("First_Skill")).show(truncate=False)`


> Uses index notation (`Skills_Array[0]`) to pick the first skill. Indexing starts from **0**.

---

### 3. Count the number of skills per employee


`df2.select(col("EmployeeID"), col("Name"), size(col("Skills_Array")).alias("Number_of_Skills")).show(truncate=False)`

> The `size()` function returns the number of elements in the array.

---

### 4. Check if the employee has "Cloud" skill

`df.select(col("EmployeeID"), col("Name"), array_contains(split(col("Skills"), " "), "Cloud").alias("Has_Cloud_Skill")).show(truncate=False)`
`

> `array_contains()` returns **True** if `"Cloud"` exists in the skill set.

---

### 5. Explode the `Skills_Array` into multiple rows

`df3 = df2.withColumn("Skill", explode(col("Skills_Array")))`

`df3.select("EmployeeID", "Name", "Skill").show(truncate=False)`


> `explode()` flattens the array into rows, where each skill becomes a separate row for that employee.

---

### 📌 Summary of Key Functions

- **split()** → Splits a string into an array.
- **explode()** → Converts an array into multiple rows.
- **size()** → Counts elements in an array.
- **array_contains()** → Checks if an array contains a value.
- **selectExpr()** → Lets you query arrays using SQL expressions like `Skills_Array[0]`.

---


---

## Trim Function in DataFrame

Let’s create a sample dataset for employees and demonstrate string trimming and padding functions in PySpark:  
- `ltrim()`  
- `rtrim()`  
- `trim()`  
- `lpad()`  
- `rpad()`  

---

### Sample Data Creation for Employees

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import lit, ltrim, rtrim, rpad, lpad, trim, col

# Sample employee data with leading and trailing spaces in the 'Name' column
data = [
   (1, " Alice   ", "HR"),
   (2, "  Bob", "IT"),
   (3, "Charlie  ", "Finance"),
   (4, "  David ", "HR"),
   (5, "Eve  ", "IT")
]

# Define the schema for the DataFrame
columns = ["EmployeeID", "Name", "Department"]

# Create DataFrame
df = spark.createDataFrame(data, columns)

# Show the original DataFrame
df.show(truncate=False)
```

### Applying Trimming and Padding Functions

### 1. Trimming Functions

- **`ltrim()`** → Removes leading spaces.
    
- **`rtrim()`** → Removes trailing spaces.
    
- **`trim()`** → Removes both leading and trailing spaces.
    

### 2. Padding Functions

- **`lpad()`** → Pads the left side of a string with a character up to a given length.
    
- **`rpad()`** → Pads the right side of a string with a character up to a given length.
    

---

### Example
```python
# Apply trimming and padding functions
result_df = df.select(
    col("EmployeeID"),
    col("Department"),
    ltrim(col("Name")).alias("ltrim_Name"),   # Remove leading spaces
    rtrim(col("Name")).alias("rtrim_Name"),   # Remove trailing spaces
    trim(col("Name")).alias("trim_Name"),     # Remove both leading & trailing spaces
    lpad(col("Name"), 10, "X").alias("lpad_Name"),  # Left pad with "X" to length 10
    rpad(col("Name"), 10, "Y").alias("rpad_Name")   # Right pad with "Y" to length 10
)

# Show the resulting DataFrame
result_df.show(truncate=False)
```
---

### Output Explanation

- **`ltrim_Name`** → Leading spaces removed.
    
- **`rtrim_Name`** → Trailing spaces removed.
    
- **`trim_Name`** → Both leading & trailing spaces removed.
    
- **`lpad_Name`** → Padded left with `"X"` until length = 10.
    
- **`rpad_Name`** → Padded right with `"Y"` until length = 10.
    

---

### 📌 Summary

- Use **trim functions** (`ltrim`, `rtrim`, `trim`) to clean up unwanted spaces.
- Use **padding functions** (`lpad`, `rpad`) to format strings with fixed lengths.