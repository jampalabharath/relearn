+++
title = "Pyspark 1"
weight = 2
+++



## 1. Write a PySpark query using below input to get below output?

### Input

| name  | Hobbies              |
|-------|-----------------------|
| Alice | Badminton, Tennis    |
| Bob   | Tennis, Cricket      |
| Julie | Cricket, Carroms     |

### Output

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

# Split and explode hobbies
result = df.withColumn("Hobbies", explode(split(df["Hobbies"], ",\s*")))

result.show(truncate=False)
```


## 2. Write a PySpark query using the below input to get the required output.

### Input

| City1 | City2 | City3 |
|-------|-------|-------|
| Goa   | null  | Ap    |
| null  | AP    | null  |
| null  | null  | Bglr  |

### Expected Output

| Result |
|--------|
| Goa    |
| AP     |
| Bglr   |

---

## PySpark Solution

```python
from pyspark.sql import functions as F

# Sample Input Data
data = [
    ("Goa", None, "Ap"),
    (None, "AP", None),
    (None, None, "Bglr")
]

df = spark.createDataFrame(data, ["City1", "City2", "City3"])

# Use COALESCE to pick the first non-null city
result_df = df.select(
    F.coalesce("City1", "City2", "City3").alias("Result")
)

result_df.show()
```
