+++
title = "Pyspark 1"
weight = 2
+++



## 1. Write a PySpark query using below input to get below output?

#### Input

| name  | Hobbies              |
|-------|-----------------------|
| Alice | Badminton, Tennis    |
| Bob   | Tennis, Cricket      |
| Julie | Cricket, Carroms     |

#### Output

| name  | Hobbies    |
|-------|------------|
| Alice | Badminton  |
| Alice | Tennis     |
| Bob   | Tennis     |
| Bob   | Cricket    |
| Julie | Cricket    |
| Julie | Carroms    |

---

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import split, explode

# Create Spark session
spark = SparkSession.builder.appName("HobbiesSplit").getOrCreate()

# Sample data
data = [
    ("Alice", "Badminton, Tennis"),
    ("Bob", "Tennis, Cricket"),
    ("Julie", "Cricket, Carroms")
]

columns = ["name", "Hobbies"]

# Create DataFrame
df = spark.createDataFrame(data, columns)
```

{{% expand title="Solution" %}}
```python
# Split and explode hobbies
result = df.withColumn("Hobbies", explode(split(df["Hobbies"], ",\s*")))

result.show(truncate=False)
```
{{% /expand %}}


## 2. Write a PySpark query using the below input to get the required output.

#### Input

| City1 | City2 | City3 |
|-------|-------|-------|
| Goa   | null  | Ap    |
| null  | AP    | null  |
| null  | null  | Bglr  |

#### Expected Output

| Result |
|--------|
| Goa    |
| AP     |
| Bglr   |

---

```python
from pyspark.sql import functions as F

# Sample Input Data
data = [
    ("Goa", None, "Ap"),
    (None, "AP", None),
    (None, None, "Bglr")
]

df = spark.createDataFrame(data, ["City1", "City2", "City3"])

```

{{% expand title="Solution" %}}
```python
# Use COALESCE to pick the first non-null city
result_df = df.select(
    F.coalesce("City1", "City2", "City3").alias("Result")
)

result_df.show()
```
{{% /expand %}}


## 3. Question – Student Result Classification

You have a PySpark DataFrame `df` containing student marks in multiple subjects:

| Id | Name  | Subject | Mark |
|----|-------|----------|------|
| 1  | Steve | SQL      | 90   |
| 1  | Steve | PySpark  | 100  |
| 2  | David | SQL      | 70   |
| 2  | David | PySpark  | 60   |
| 3  | John  | SQL      | 30   |
| 3  | John  | PySpark  | 20   |
| 4  | Shree | SQL      | 50   |

1. Write a PySpark query to calculate the **average percentage** for each student across all subjects.  
2. Based on the percentage, assign a **Result** column using the following rules:  

   - **Distinction** → Percentage ≥ 70  
   - **First Class** → 60 ≤ Percentage < 70  
   - **Second Class** → 50 ≤ Percentage < 60  
   - **Third Class** → 40 ≤ Percentage < 50  
   - **Fail** → Percentage < 40  

Show the final DataFrame with columns: `Id`, `Name`, `Percentage`, `Result`.

---

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, avg, when

spark = SparkSession.builder.appName("StudentMarks").getOrCreate()

# Sample DataFrame
data = [
    (1, 'Steve', 'SQL', 90),
    (1, 'Steve', 'PySpark', 100),
    (2, 'David', 'SQL', 70),
    (2, 'David', 'PySpark', 60),
    (3, 'John', 'SQL', 30),
    (3, 'John', 'PySpark', 20),
    (4, 'Shree', 'SQL', 50)
]

columns = ["Id", "Name", "Subject", "Mark"]
df = spark.createDataFrame(data, columns)

```

{{% expand title="Solution" %}}
```python
# Calculate average percentage per student
df_per = df.groupBy("Id", "Name").agg(avg("Mark").alias("Percentage"))

# Assign Result based on percentage
df_final = df_per.select(
    '*',
    when(col("Percentage") >= 70, "Distinction")
    .when((col("Percentage") < 70) & (col("Percentage") >= 60), "First Class")
    .when((col("Percentage") < 60) & (col("Percentage") >= 50), "Second Class")
    .when((col("Percentage") < 50) & (col("Percentage") >= 40), "Third Class")
    .otherwise("Fail").alias("Result")
)

df_final.show()
```
{{% /expand %}}


## 4. Department-wise Employee Ranking

#### **Question**

You are given the following employee dataset:

| EmpId | EmpName | Salary | DeptName |
|-------|----------|---------|----------|
| 1     | A        | 1000    | IT       |
| 2     | B        | 1500    | IT       |
| 3     | C        | 2500    | IT       |
| 4     | D        | 3000    | HR       |
| 5     | E        | 2000    | HR       |
| 6     | F        | 1000    | HR       |
| 7     | G        | 4000    | Sales    |
| 8     | H        | 4000    | Sales    |
| 9     | I        | 1000    | Sales    |
| 10    | J        | 2000    | Sales    |

You need to:
1. Assign a **dense rank** to employees based on their salary **within each department**, sorted in **descending order**.  
2. Display only those employees who hold the **2nd highest salary** in each department.

---


```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import *
from pyspark.sql.window import *

spark = SparkSession.builder.appName("EmployeeRanking").getOrCreate()

# Input Data
data1 = [
    (1,"A",1000,"IT"),
    (2,"B",1500,"IT"),
    (3,"C",2500,"IT"),
    (4,"D",3000,"HR"),
    (5,"E",2000,"HR"),
    (6,"F",1000,"HR"),
    (7,"G",4000,"Sales"),
    (8,"H",4000,"Sales"),
    (9,"I",1000,"Sales"),
    (10,"J",2000,"Sales")
]
schema1 = ["EmpId","EmpName","Salary","DeptName"]
df = spark.createDataFrame(data1, schema1)
```

{{% expand title="Solution" %}}
```python
# Apply dense_rank() by department based on descending salary
df_rank = df.select(
    '*',
    dense_rank().over(Window.partitionBy(df.DeptName).orderBy(df.Salary.desc())).alias('rank')
)

# Filter employees with 2nd highest salary
df_second_highest = df_rank.filter(df_rank.rank == 2)

df_second_highest.show()
```
{{% /expand %}}




## 5. Employee Salary Report by Department and Manager

#### **Question**

You are given two DataFrames:

#### **1️⃣ Employee Salary Information**

| EmpId | EmpName | Mgrid | deptid | salarydt  | salary |
|-------|----------|-------|---------|------------|--------|
| 100   | Raj      | null  | 1       | 01-04-23   | 50000  |
| 200   | Joanne   | 100   | 1       | 01-04-23   | 4000   |
| 200   | Joanne   | 100   | 1       | 13-04-23   | 4500   |
| 200   | Joanne   | 100   | 1       | 14-04-23   | 4020   |

#### **2️⃣ Department Information**

| deptid | deptname |
|--------|-----------|
| 1      | IT        |
| 2      | HR        |

---

#### **Task**

1. Convert the `salarydt` column to **date type**.  
2. Join both DataFrames to add **department name**.  
3. Perform a **self-join** to get each employee’s **manager name** (based on `Mgrid = EmpId`).  
4. Group the data to calculate the **total salary paid** for each employee, month, and year within their department, along with their manager’s name.  

---

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, to_date, year, date_format

spark = SparkSession.builder.appName("EmployeeSalaryReport").getOrCreate()

# Employee Salary DataFrame
data1 = [
    (100, "Raj", None, 1, "01-04-23", 50000),
    (200, "Joanne", 100, 1, "01-04-23", 4000),
    (200, "Joanne", 100, 1, "13-04-23", 4500),
    (200, "Joanne", 100, 1, "14-04-23", 4020)
]
schema1 = ["EmpId", "EmpName", "Mgrid", "deptid", "salarydt", "salary"]
df_salary = spark.createDataFrame(data1, schema1)

# Department DataFrame
data2 = [(1, "IT"), (2, "HR")]
schema2 = ["deptid", "deptname"]
df_dept = spark.createDataFrame(data2, schema2)
```

{{% expand title="Solution" %}}
```python
# Step 1: Convert salarydt to Date Type
df = df_salary.withColumn("Newsaldt", to_date("salarydt", "dd-MM-yy"))

# Step 2: Join department data
df1 = df.join(df_dept, ["deptid"])

# Step 3: Self join to get Manager Name
df2 = (
    df1.alias("a")
    .join(df1.alias("b"), col("a.Mgrid") == col("b.EmpId"), "left")
    .select(
        col("a.deptname"),
        col("b.EmpName").alias("ManagerName"),
        col("a.EmpName"),
        col("a.Newsaldt"),
        col("a.salary")
    )
)

# Step 4: Group by Dept, Manager, Employee, Year, and Month
df3 = (
    df2.groupBy(
        "deptname",
        "ManagerName",
        "EmpName",
        year("Newsaldt").alias("Year"),
        date_format("Newsaldt", "MMMM").alias("Month")
    )
    .sum("salary")
)

df3.show()
```
{{% /expand %}}


## 6. Month-over-Month Sales Growth Calculation

#### **Question**

You are given the following sales order dataset:

| SOId | SODate     | ItemId | ItemQty | ItemValue |
|------|-------------|--------|----------|------------|
| 1    | 2024-01-01  | I1     | 10       | 1000       |
| 2    | 2024-01-15  | I2     | 20       | 2000       |
| 3    | 2024-02-01  | I3     | 10       | 1500       |
| 4    | 2024-02-15  | I4     | 20       | 2500       |
| 5    | 2024-03-01  | I5     | 30       | 3000       |
| 6    | 2024-03-10  | I6     | 40       | 3500       |
| 7    | 2024-03-20  | I7     | 20       | 2500       |
| 8    | 2024-03-30  | I8     | 10       | 1000       |

---

#### **Task**

1. Convert the `SODate` column into a **DateType**.  
2. Extract **Month** and **Year** from the sales order date.  
3. Calculate **total sales per month** (`sum(ItemValue)`).  
4. Use a **window function (`lag`)** to get the **previous month’s total sales**.  
5. Compute the **month-over-month sales growth percentage** using:

   \[
   Growth\% = \frac{(CurrentMonth - PrevMonth) * 100}{CurrentMonth}
   \]

---

```python
from pyspark.sql import SparkSession
from pyspark.sql.types import *
from pyspark.sql.functions import *
from pyspark.sql.window import *

spark = SparkSession.builder.appName("MonthlySalesGrowth").getOrCreate()

# Sample Data
data = [
    (1, '2024-01-01', "I1", 10, 1000),
    (2, "2024-01-15", "I2", 20, 2000),
    (3, "2024-02-01", "I3", 10, 1500),
    (4, "2024-02-15", "I4", 20, 2500),
    (5, "2024-03-01", "I5", 30, 3000),
    (6, "2024-03-10", "I6", 40, 3500),
    (7, "2024-03-20", "I7", 20, 2500),
    (8, "2024-03-30", "I8", 10, 1000)
]
schema = ["SOId", "SODate", "ItemId", "ItemQty", "ItemValue"]

df1 = spark.createDataFrame(data, schema)
```


{{% expand title="Solution" %}}
```python
# Step 1: Convert SODate to DateType
df1 = df1.withColumn("SODate", df1.SODate.cast(DateType()))

# Step 2: Extract Month, Year
df2 = df1.select(
    month("SODate").alias("Month"),
    year("SODate").alias("Year"),
    col("ItemValue")
)

# Step 3: Aggregate monthly total sales
df3 = df2.groupBy("Month", "Year").agg(sum("ItemValue").alias("TotalSale"))

# Step 4: Use lag() to find previous month's sale
df4 = df3.select(
    '*',
    lag("TotalSale").over(Window.orderBy("Month", "Year")).alias("PrevSale")
)

# Step 5: Calculate Month-over-Month growth %
df_final = df4.select(
    "*",
    ((col("TotalSale") - col("PrevSale")) * 100 / col("TotalSale")).alias("Growth_Percentage")
)

df_final.show()
```
{{% /expand %}}


## 7. Average Machine Processing Time

#### **Question**

You are given the following dataset representing machine process activity logs:

| Machine_id | processid | activityid | timestamp |
|-------------|------------|-------------|------------|
| 0           | 0          | start       | 0.712      |
| 0           | 0          | end         | 1.520      |
| 0           | 1          | start       | 3.140      |
| 0           | 1          | end         | 4.120      |
| 1           | 0          | start       | 0.550      |
| 1           | 0          | end         | 1.550      |
| 1           | 1          | start       | 0.430      |
| 1           | 1          | end         | 1.420      |
| 2           | 0          | start       | 4.100      |
| 2           | 0          | end         | 4.512      |
| 2           | 1          | start       | 2.500      |
| 2           | 1          | end         | 5.000      |

---

#### **Task**

1. Identify **start** and **end** times for each `processid` and `Machine_id`.  
2. Calculate the **processing duration** (`endtime - starttime`) for each process.  
3. Find the **average processing time per machine** across all its processes.

---

#### **PySpark Solution**

```python
from pyspark.sql import SparkSession
from pyspark.sql.types import *
from pyspark.sql.functions import *

spark = SparkSession.builder.appName("MachineProcessingTime").getOrCreate()

# Step 1: Input Data
data = [
    (0, 0, 'start', 0.712), (0, 0, 'end', 1.520),
    (0, 1, 'start', 3.140), (0, 1, 'end', 4.120),
    (1, 0, 'start', 0.550), (1, 0, 'end', 1.550),
    (1, 1, 'start', 0.430), (1, 1, 'end', 1.420),
    (2, 0, 'start', 4.100), (2, 0, 'end', 4.512),
    (2, 1, 'start', 2.500), (2, 1, 'end', 5.000)
]
schema = ["Machine_id", "processid", "activityid", "timestamp"]
df1 = spark.createDataFrame(data, schema)
```


{{% expand title="Solution" %}}
```python
# Step 2: Split start and end timestamps
df2 = df1.select(
    df1.Machine_id,
    df1.processid,
    when(df1.activityid == 'start', df1.timestamp).alias('starttime'),
    when(df1.activityid == 'end', df1.timestamp).alias('endtime')
)

# Step 3: Aggregate start and end timestamps per process
df3 = df2.groupBy("Machine_id", "processid").agg(
    max("starttime").alias("starttime"),
    max("endtime").alias("endtime")
)

# Step 4: Calculate processing duration
df4 = df3.select(
    df3.Machine_id,
    (df3.endtime - df3.starttime).alias("diff")
)

# Step 5: Calculate average processing time per machine
df_final = df4.groupBy("Machine_id").agg(
    avg("diff").alias("avg_processing_time")
)

df_final.show()
```
{{% /expand %}}


## 8. Combine Multiple Skills into a Single Column

#### **Question**

You are given the following employee skill dataset:

| EmpId | EmpName | Skill            |
|-------|----------|------------------|
| 1     | John     | ADF              |
| 1     | John     | ADB              |
| 1     | John     | PowerBI          |
| 2     | Joanne   | ADF              |
| 2     | Joanne   | SQL              |
| 2     | Joanne   | Crystal Report   |
| 3     | Vikas    | ADF              |
| 3     | Vikas    | SQL              |
| 3     | Vikas    | SSIS             |
| 4     | Monu     | SQL              |
| 4     | Monu     | SSIS             |
| 4     | Monu     | SSAS             |
| 4     | Monu     | ADF              |

---

#### **Task**

1. Group the records by **employee name**.  
2. Collect all skills of each employee into a single list.  
3. Concatenate those skills into a single comma-separated string under a new column **`Skills`**.

---

#### **PySpark Solution**

```python
from pyspark.sql import SparkSession
from pyspark.sql.types import *
from pyspark.sql.functions import collect_list, concat_ws

spark = SparkSession.builder.appName("EmployeeSkills").getOrCreate()

# Step 1: Create DataFrame
data = [
    (1, 'John', 'ADF'),
    (1, 'John', 'ADB'),
    (1, 'John', 'PowerBI'),
    (2, 'Joanne', 'ADF'),
    (2, 'Joanne', 'SQL'),
    (2, 'Joanne', 'Crystal Report'),
    (3, 'Vikas', 'ADF'),
    (3, 'Vikas', 'SQL'),
    (3, 'Vikas', 'SSIS'),
    (4, 'Monu', 'SQL'),
    (4, 'Monu', 'SSIS'),
    (4, 'Monu', 'SSAS'),
    (4, 'Monu', 'ADF')
]
schema = ["EmpId", "EmpName", "Skill"]
df1 = spark.createDataFrame(data, schema)
```

{{% expand title="Solution" %}}
```python
# Step 2: Collect all skills per employee into a list
df2 = df1.groupBy("EmpName").agg(collect_list("Skill").alias("Skill"))

# Step 3: Convert list to comma-separated string
df3 = df2.select("EmpName", concat_ws(",", "Skill").alias("Skills"))

df3.show(truncate=False)
```
{{% /expand %}}


## 9. Split Phone Numbers into STD Code and Phone Number

#### **Question**

You are given the following dataset containing employee names and their phone numbers:

| name   | phone         |
|--------|----------------|
| Joanne | 040-20215632   |
| Tom    | 044-23651023   |
| John   | 086-12456782   |

Each phone number is stored in the format:  
`<STD Code>-<Phone Number>`

---

#### **Task**

1. Split the `phone` column into two new columns:
   - **`std_code`** → Contains the STD code before the hyphen.  
   - **`phone_num`** → Contains the actual phone number after the hyphen.  

2. Display the final DataFrame.

---

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import split

spark = SparkSession.builder.appName("PhoneNumberSplit").getOrCreate()

# Step 1: Create DataFrame
data = [
    ('Joanne', "040-20215632"),
    ('Tom', "044-23651023"),
    ('John', "086-12456782")
]
schema = ["name", "phone"]
df = spark.createDataFrame(data, schema)
```
{{% expand title="Solution" %}}
```python
# Step 2: Split phone number into STD code and phone number
df = df.withColumn("std_code", split(df.phone, "-").getItem(0))
df = df.withColumn("phone_num", split(df.phone, "-").getItem(1))

df.show()
```
{{% /expand %}}

## 10.Find Actor-Director Pairs with More Than Two Collaborations

#### **Question**

You are given a dataset containing details of **actors** and **directors**, where each record represents a collaboration between them:

| ActorId | DirectorId | timestamp |
|----------|-------------|------------|
| 1        | 1           | 0          |
| 1        | 1           | 1          |
| 1        | 1           | 2          |
| 1        | 2           | 3          |
| 1        | 2           | 4          |
| 1        | 2           | 5          |
| 2        | 1           | 6          |

---

#### **Task**

Find all **Actor–Director pairs** that have worked together on **more than two occasions**.

---

#### **PySpark Solution**

```python
from pyspark.sql import SparkSession
from pyspark.sql.types import *

spark = SparkSession.builder.appName("ActorDirectorCollaboration").getOrCreate()

# Step 1: Define schema and data
schema = StructType([
    StructField("ActorId", IntegerType(), True),
    StructField("DirectorId", IntegerType(), True),
    StructField("timestamp", IntegerType(), True)
])

data = [
    (1, 1, 0),
    (1, 1, 1),
    (1, 1, 2),
    (1, 2, 3),
    (1, 2, 4),
    (1, 2, 5),
    (2, 1, 6)
]

df = spark.createDataFrame(data, schema)
```

{{% expand title="Solution" %}}
```python
# Step 2: Group by ActorId and DirectorId and count occurrences
result_df = df.groupBy("ActorId", "DirectorId").count()

# Step 3: Filter pairs with count > 2
df1 = result_df.filter(result_df["count"] > 2)

# Step 4: Display Actor-Director pairs with more than 2 collaborations
df1.select("ActorId", "DirectorId").show()
```
{{% /expand %}}