+++
title = "Joins"
+++

## Joins in PySpark

In PySpark Joins are used to combine two DataFrames based on a common column or condition. 

---

### Types of Joins in PySpark

- **Inner Join**: Matches rows from both DataFrames.
```python
df1.join(df2, df1.common_column == df2.common_column, "inner")
```
- **Left/Right Join**: Keeps all rows from the left or right DataFrame and matches where possible.
```python
 df1.join(df2, df1.common_column == df2.common_column, "left")
 df1.join(df2, df1.common_column == df2.common_column, "right")
 ```
- **Full Join**: Keeps all rows from both DataFrames.
```python
df1.join(df2, df1.common_column == df2.common_column, "outer")
```
- **Left Semi**: Filters `df1` to rows that match `df2` without including columns from `df2`.
```python
df1.join(df2, df1.common_column == df2.common_column, "left_semi") 
```
- **Left Anti**: Filters `df1` to rows that do not match `df2`. 
```python
df1.join(df2, df1.common_column == df2.common_column, "left_anti")
```
- **Cross Join**: Returns the Cartesian product, combining all rows of both DataFrames.
```python
df1.crossJoin(df2)
```
- **Explicit Condition Join**: Allows complex join conditions, including columns with different names.  
```python
df1.join(df2, df1.columnA == df2.columnB, "inner")
```

```python
df1.join(df2, df1.common_column == df2.common_column, "inner")
df1.join(df2, df1.common_column == df2.common_column, "left")
df1.join(df2, df1.common_column == df2.common_column, "right")
df1.join(df2, df1.common_column == df2.common_column, "outer")
df1.join(df2, df1.common_column == df2.common_column, "left_semi")
df1.join(df2, df1.common_column == df2.common_column, "left_anti")
df1.crossJoin(df2)
df1.join(df2, df1.columnA == df2.columnB, "inner")
```

---

### Practice-1
```python
from pyspark.sql import SparkSession
from pyspark.sql import Row
from pyspark.sql.functions import broadcast

# Initialize Spark session
spark = SparkSession.builder.appName("JoinsExample").getOrCreate()

# Sample DataFrames
data1 = [Row(id=0), Row(id=1), Row(id=1), Row(id=None), Row(id=None)]
data2 = [Row(id=1), Row(id=0), Row(id=None)]

df1 = spark.createDataFrame(data1)
df2 = spark.createDataFrame(data2)

# Inner Join
inner_join = df1.join(df2, on="id", how="inner")
print("Inner Join:")
inner_join.show()

# Right Join
right_join = df1.join(df2, on="id", how="right")
print("Right Join:")
right_join.show()

# Full (Outer) Join
full_join = df1.join(df2, on="id", how="outer")
print("Full (Outer) Join:")
full_join.show()

# Left Anti Join
left_anti_join = df1.join(df2, on="id", how="left_anti")
print("Left Anti Join:")
left_anti_join.show()

# Right Anti Join (Equivalent to swapping DataFrames and performing Left Anti Join)
right_anti_join = df2.join(df1, on="id", how="left_anti")
print("Right Anti Join:")
right_anti_join.show()

# Broadcast Join (Optimizing a join with a smaller DataFrame)
broadcast_join = df1.join(broadcast(df2), on="id", how="inner")
print("Broadcast Join:")
broadcast_join.show()

```

---

### Practice 2

#### PySpark Coding Questions

1. **Find employees whose location matches the location of their department**  
   - Display: `emp_id`, `emp_name`, `emp_location`, `dept_name`, `dept_location`.

2. **Find departments that have no employees assigned to them**  
   - Display: `dept_id`, `dept_name`, `dept_head`.

3. **Get the average salary of employees in each department**  
   - Display: `dept_name`, `average_salary`.

4. **List the employees who earn more than the average salary of their department**  
   - Display: `emp_id`, `emp_name`, `emp_salary`, `dept_name`, `dept_location`.

```python
# Sample DataFrames
emp_data = [
    Row(emp_id=1, emp_name="Alice", emp_salary=50000, emp_dept_id=101, emp_location="New York"),
    Row(emp_id=2, emp_name="Bob", emp_salary=60000, emp_dept_id=102, emp_location="Los Angeles"),
    Row(emp_id=3, emp_name="Charlie", emp_salary=55000, emp_dept_id=101, emp_location="Chicago"),
    Row(emp_id=4, emp_name="David", emp_salary=70000, emp_dept_id=103, emp_location="San Francisco"),
    Row(emp_id=5, emp_name="Eve", emp_salary=48000, emp_dept_id=102, emp_location="Houston")
]

dept_data = [
    Row(dept_id=101, dept_name="Engineering", dept_head="John", dept_location="New York"),
    Row(dept_id=102, dept_name="Marketing", dept_head="Mary", dept_location="Los Angeles"),
    Row(dept_id=103, dept_name="Finance", dept_head="Frank", dept_location="Chicago")
]

emp_columns = ["emp_id", "emp_name", "emp_salary", "emp_dept_id", "emp_location"]
dept_columns = ["dept_id", "dept_name", "dept_head", "dept_location"]

emp_df = spark.createDataFrame(emp_data, emp_columns)
dept_df = spark.createDataFrame(dept_data, dept_columns)

# Display emp data
print("emp_data:")
emp_df.show()

# Display dept data
print("dept_data:")
dept_df.show()
```
```python
# Inner Join on emp_dept_id and dept_id
inner_join = emp_df.join(dept_df, emp_df["emp_dept_id"] == dept_df["dept_id"], "inner")

# Display the result
print("Inner Join Result:")
inner_join.show()


# Inner Join with Filtering Columns and WHERE Condition
inner_join = emp_df.join(dept_df, emp_df["emp_dept_id"] == dept_df["dept_id"], "inner") \
    .select("emp_id", "emp_name", "emp_salary", "dept_name", "dept_location") \
    .filter("emp_salary > 55000")  # Add a WHERE condition

# Display the result
print("Inner Join with Filter and WHERE Condition:")
inner_join.show()


# Left Join with Filtering Columns and WHERE Condition
left_join_filtered = emp_df.join(dept_df, emp_df["emp_dept_id"] == dept_df["dept_id"], "left") \
    .select("emp_id", "emp_name", "dept_name", "dept_location") \
    .filter("emp_salary > 55000")  # Add a WHERE condition

# Display the result
print("Left Join with Filter and WHERE Condition:")
left_join_filtered.show()

# Left Anti Join
left_anti_join = emp_df.join(
    dept_df,
    emp_df["emp_dept_id"] == dept_df["dept_id"],
    "left_anti"
)

# Display the result
print("Left Anti Join Result:")
left_anti_join.show()
```
---

### Practice 3

#### PySpark Coding Questions

1. **List each employee along with their manager's name**  
   - Display: `employee`, `manager`.

2. **Find employees who do not have a manager (CEO-level employees)**  
   - Display: `employee`, `manager`.

3. **Find all employees who directly report to "Manager A"**  
   - Display: `empid`, `ename`, `mrgid`.

4. **Determine the hierarchy level of each employee**  
   - CEO → Level 1, direct reports to CEO → Level 2, and so on.  
   - Display: `empid`, `ename`, `mrgid`, `level`.

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, expr

# Create a Spark session
spark = SparkSession.builder.appName("EmployeeHierarchy").getOrCreate()

# Sample data
data = [
    (1, None, "CEO"),
    (2, 1, "Manager A"),
    (3, 1, "Manager B"),
    (4, 2, "Employee X"),
    (5, 3, "Employee Y"),
]
columns = ["empid", "mrgid", "ename"]

employee_df = spark.createDataFrame(data, columns)

# Display the result
print("emp_data:")
employee_df.show()


# Self-join to find the manager and CEO
manager_df = employee_df.alias("e") \
    .join(employee_df.alias("m"), col("e.mrgid") == col("m.empid"), "left") \
    .select(
        col("e.ename").alias("employee"),
        col("m.ename").alias("manager")
    )

# Display the result
print("mgr:")
manager_df.show()


# Filter for employees without a manager (CEO)
manager_df2 = employee_df.alias("e1") \
    .join(employee_df.alias("m1"), col("e1.mrgid") == col("m1.empid"), "left") \
    .select(
        col("e1.ename").alias("employee"),
        col("m1.ename").alias("manager")
    ) \
    .filter(col("manager").isNull())

# Display the result
manager_df2.show()

```


