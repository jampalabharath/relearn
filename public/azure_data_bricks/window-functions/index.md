# Window Functions

## Windows Function in PySpark
### 1. Introduction to Window Functions

Window functions allow you to perform calculations across a set of rows related to the current row within a specified partition.

Unlike groupBy functions, window functions do not reduce the number of rows in the result; instead, they calculate a value for each row based on the specified window.

### 2. Importing Required Libraries

To use window functions, import the necessary modules from PySpark:
```python
from pyspark.sql import SparkSession
from pyspark.sql import functions as F
from pyspark.sql.window import Window
```
### 3. Creating a Window Specification

A window specification defines how rows will be grouped (partitioned) and ordered within each group.

**Example – Basic Window Specification:**

`window_spec = Window.partitionBy("category").orderBy("timestamp")`


**Example – Advanced Window Specification:**

`window_spec = Window.partitionBy("category", "sub_category").orderBy(F.col("timestamp"), F.col("score"))`

### 4. Common Window Functions
#### a. Row Number

**Function:** `row_number()`

**Description:** Assigns a unique integer to each row within the partition (starting from 1).

`df = df.withColumn("row_number", F.row_number().over(window_spec))`

#### b. Rank

**Function:** `rank()`

**Description:** Assigns the same rank to rows with the same values in the order criteria. The next rank has a gap.

`df = df.withColumn("rank", F.rank().over(window_spec))`

#### c. Dense Rank

**Function:** `dense_rank()`

**Description:** Similar to rank, but does not leave gaps in ranking.

`df = df.withColumn("dense_rank", F.dense_rank().over(window_spec))`

#### d. Lead and Lag

**Functions:** `lead()`, `lag()`

**Description:**

`lead()` → value of the next row within the window.

`lag()` → value of the previous row within the window.

`df = df.withColumn("next_value", F.lead("value").over(window_spec))`
`df = df.withColumn("previous_value", F.lag("value").over(window_spec))`

#### e. Aggregation Functions

Window functions can also compute aggregated values across the specified window.

`df = df.withColumn("avg_value", F.avg("value").over(window_spec))`


Other common aggregations:

**Sum:** `F.sum("column_name").over(window_spec)`

**Min:** `F.min("column_name").over(window_spec)`

**Max:** `F.max("column_name").over(window_spec)`

### 5. Putting It All Together – Example
```python
from pyspark.sql import SparkSession
from pyspark.sql import functions as F
from pyspark.sql.window import Window

# Initialize Spark session
spark = SparkSession.builder.appName("WindowFunctionsExample").getOrCreate()

# Sample DataFrame
data = [
    ("A", "X", 1, "2023-01-01"),
    ("A", "X", 2, "2023-01-02"),
    ("A", "Y", 3, "2023-01-01"),
    ("A", "Y", 3, "2023-01-02"),
    ("B", "X", 5, "2023-01-01"),
    ("B", "X", 4, "2023-01-02"),
]
columns = ["category", "sub_category", "value", "timestamp"]
df = spark.createDataFrame(data, columns)

# Define window specification
window_spec = Window.partitionBy("category", "sub_category") \
                   .orderBy(F.col("timestamp"), F.col("value"))

# Apply window functions
df = df.withColumn("row_number", F.row_number().over(window_spec))
df = df.withColumn("rank", F.rank().over(window_spec))
df = df.withColumn("dense_rank", F.dense_rank().over(window_spec))
df = df.withColumn("next_value", F.lead("value").over(window_spec))
df = df.withColumn("previous_value", F.lag("value").over(window_spec))
df = df.withColumn("avg_value", F.avg("value").over(window_spec))

df.show()
```

### 6. Conclusion

Window functions in PySpark are powerful tools for analyzing data within groups while retaining row-level details.

By defining window specifications and applying functions like rank, dense_rank, lead, lag, and aggregations, you can perform complex analytics efficiently.

## Windows Function in PySpark – Part 2
```python
from pyspark.sql import SparkSession
from pyspark.sql.window import Window
import pyspark.sql.functions as F

# Sample data
data = [
    ("Alice", 100),
    ("Bob", 200),
    ("Charlie", 200),
    ("David", 300),
    ("Eve", 400),
    ("Frank", 500),
    ("Grace", 500),
    ("Hank", 600),
    ("Ivy", 700),
    ("Jack", 800)
]
columns = ["Name", "Score"]
df = spark.createDataFrame(data, columns)

# Define window
window_spec = Window.orderBy("Score")

# Ranking functions
df1 = df.withColumn("Rank", F.rank().over(window_spec))
df2 = df.withColumn("DenseRank", F.dense_rank().over(window_spec))
df3 = df.withColumn("RowNumber", F.row_number().over(window_spec))

# Lead & Lag
df4 = df.withColumn("ScoreDifferenceWithNext", F.lead("Score").over(window_spec) - df["Score"])
df5 = df.withColumn("ScoreDifferenceWithPrevious", df["Score"] - F.lag("Score").over(window_spec))
```
## Windows Function in PySpark – Part 3 (Student Marks Analysis)
```python
from pyspark.sql import SparkSession
from pyspark.sql.window import Window
import pyspark.sql.functions as F

# Updated sample data
data = [
    ("Alice", "Math", 90, 1),
    ("Alice", "Science", 85, 1),
    ("Alice", "History", 78, 1),
    ("Bob", "Math", 80, 1),
    ("Bob", "Science", 81, 1),
    ("Bob", "History", 77, 1),
    ("Charlie", "Math", 75, 1),
    ("Charlie", "Science", 82, 1),
    ("Charlie", "History", 79, 1),
    ("Alice", "Physics", 86, 2),
    ("Alice", "Chemistry", 92, 2),
    ("Alice", "Biology", 80, 2),
    ("Bob", "Physics", 94, 2),
    ("Bob", "Chemistry", 91, 2),
    ("Bob", "Biology", 96, 2),
    ("Charlie", "Physics", 89, 2),
    ("Charlie", "Chemistry", 88, 2),
    ("Charlie", "Biology", 85, 2),
    ("Alice", "Computer Science", 95, 3),
    ("Alice", "Electronics", 91, 3),
    ("Alice", "Geography", 97, 3),
    ("Bob", "Computer Science", 88, 3),
    ("Bob", "Electronics", 66, 3),
    ("Bob", "Geography", 92, 3),
    ("Charlie", "Computer Science", 92, 3),
    ("Charlie", "Electronics", 97, 3),
    ("Charlie", "Geography", 99, 3)
]

columns = ["First Name", "Subject", "Marks", "Semester"]
df = spark.createDataFrame(data, columns)

# 1. Max marks in each semester
window_spec_max_marks = Window.partitionBy("Semester").orderBy(F.desc("Marks"))
max_marks_df = df.withColumn("Rank", F.rank().over(window_spec_max_marks))
top_scorer = max_marks_df.filter(max_marks_df["Rank"] == 1)

# 2. Percentage of each student
window_spec_total_marks = Window.partitionBy("First Name", "Semester")
df = df.withColumn("TotalMarks", F.sum("Marks").over(window_spec_total_marks))
df = df.withColumn("Percentage", (F.col("TotalMarks") / (3*100)).cast("decimal(5,2)")*100)
df2 = df.groupBy("First Name","Semester").agg(F.max("TotalMarks").alias("TotalMarks"), F.max("Percentage").alias("Percentage"))

# 3. Top rank holder in each semester
window_spec_rank = Window.partitionBy("Semester").orderBy(F.desc("Percentage"))
rank_df = df.withColumn("Rank", F.rank().over(window_spec_rank))
top_rank_holder = rank_df.filter(rank_df["Rank"] == 1).select("First Name","Semester","Rank","Percentage").distinct()

# 4. Max marks in each subject in each semester
window_spec_max_subject_marks = Window.partitionBy("Semester","Subject").orderBy(F.desc("Marks"))
max_subject_marks_df = df.withColumn("Rank", F.rank().over(window_spec_max_subject_marks))
max_subject_scorer = max_subject_marks_df.filter(max_subject_marks_df["Rank"] == 1)
```
## Windows Function in PySpark – Part 4 (Highest Salary per Department)
```python
from pyspark.sql import functions as F
from pyspark.sql.window import Window

# Employee Data
emp_data = [
    (1, "Alice", 1, 6300),
    (2, "Bob", 1, 6200),
    (3, "Charlie", 2, 7000),
    (4, "David", 2, 7200),
    (5, "Eve", 1, 6300),
    (6, "Frank", 2, 7100)
]

# Department Data
dept_data = [
    (1, "HR"),
    (2, "Finance")
]

# Create DataFrames
emp_df = spark.createDataFrame(emp_data, ["EmpId","EmpName","DeptId","Salary"])
dept_df = spark.createDataFrame(dept_data, ["DeptId","DeptName"])

# Window for salary ranking
window_spec = Window.partitionBy("DeptId").orderBy(F.desc("Salary"))

# Add rank & filter top salary
ranked_salary_df = emp_df.withColumn("Rank", F.rank().over(window_spec))
result_df = ranked_salary_df.filter(F.col("Rank") == 1)

# Join department names
result_df = result_df.join(dept_df, ["DeptId"], "left")

# Final Output
result_df.select("EmpName","DeptName","Salary").show()
```