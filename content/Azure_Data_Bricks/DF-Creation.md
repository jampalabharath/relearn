+++
title = "DF Basics"
weight = 1
+++


## Creating DataFrame


### Creating DataFrame from Lists/Tuples
```python
# Sample Data
data = [(1, "Alice"), (2, "Bob"), (3, "Charlie"), (4, "David"), (5, "Eve")]
columns = ["ID", "Name"]

# Create DataFrame
df = spark.createDataFrame(data, columns)
# Show DataFrame
df.show()
```

### Creating DataFrame from Pandas
```python
import pandas as pd

# Sample Pandas DataFrame
pandas_df = pd.DataFrame(data, columns=columns)

# Convert to PySpark DataFrame
df_from_pandas = spark.createDataFrame(pandas_df)
df_from_pandas.show()
```

### Create DataFrame from Dictionary

```python
data_dict = [{"ID": 1, "Name": "Alice"}, {"ID": 2, "Name": "Bob"}]
df_from_dict = spark.createDataFrame(data_dict)
df_from_dict.show()
```

### Create Empty DataFrame

You can create an empty DataFrame with just schema definitions.
```python
from pyspark.sql.types import StructType, StructField, StringType, IntegerType

# Define Schema
schema = StructType([
    StructField("ID", IntegerType(), True),
    StructField("Name", StringType(), True)
])

# Create Empty DataFrame
empty_df = spark.createDataFrame([], schema)
empty_df.show()

```


### Creating DataFrame from Structured Data (CSV, JSON, Parquet)

```python
# Reading CSV file into DataFrame
df_csv = spark.read.csv("/path/to/file.csv", header=True, inferSchema=True)
df_csv.show()

# Reading JSON file into DataFrame
df_json = spark.read.json("/path/to/file.json")
df_json.show()

# Reading Parquet file into DataFrame
df_parquet = spark.read.parquet("/path/to/file.parquet")
df_parquet.show()
```

### show() Function in PySpark DataFrames

The `show()` function in PySpark displays the contents of a DataFrame in a tabular format. It has several useful parameters for customization:

### Parameters:

1. **n**: Number of rows to display (default is 20)
2. **truncate**: If set to True, it truncates column values longer than 20 characters (default is True)
3. **vertical**: If set to True, prints rows in a vertical format

### Usage Examples:

```python
# Show the first 3 rows, truncate columns to 25 characters, and display vertically:
df.show(n=3, truncate=25, vertical=True)

# Show entire DataFrame (default settings):
df.show()

# Show the first 10 rows:
df.show(10)

# Show DataFrame without truncating any columns:
df.show(truncate=False)
```

---

## Loading Data from CSV File into a DataFrame

Loading data into DataFrames is a fundamental step in any data processing workflow in PySpark. This document outlines how to load data from CSV files into a DataFrame, including using a custom schema and the implications of using the inferSchema option.


### 1. Import Required Libraries

Before loading the data, ensure you import the necessary modules:
```python
from pyspark.sql import SparkSession
from pyspark.sql.types import StructType, StructField, IntegerType, StringType, DoubleType
```

### 2. Define the Schema

You can define a custom schema for your CSV file. This allows you to explicitly set the data types for each column.
```python
# Define the schema for the CSV file
custom_schema = StructType([
    StructField("id", IntegerType(), True),
    StructField("name", StringType(), True),
    StructField("age", IntegerType(), True),
    StructField("salary", DoubleType(), True)
])
```

### 3. Read the CSV File

Load the CSV file into a DataFrame using the `read.csv()` method. Here, `header=True` treats the first row as headers, and `inferSchema=True` allows Spark to automatically assign data types to columns.

```python
# Read the CSV file with the custom schema
df = spark.read.csv("your_file.csv", schema=custom_schema, header=True)
```

### 4. Load Multiple CSV Files

To read multiple CSV files into a single DataFrame, you can pass a list of file paths. Ensure that the schema is consistent across all files.

```python
# List of file paths
file_paths = ["file1.csv", "file2.csv", "file3.csv"]

# Read multiple CSV files into a single DataFrame
df = spark.read.csv(file_paths, header=True, inferSchema=True)
```

### 5. Load a CSV from FileStore

Here is an example of loading a CSV file from Databricks FileStore:

```python
df = spark.read.csv("/FileStore/tables/Order.csv", header=True, inferSchema=True, sep=',')
```

### 6. Display the DataFrame

Use the following commands to check the schema and display the DataFrame:

```python
# Print the schema of the DataFrame
df.printSchema()

# Show the first 20 rows of the DataFrame
df.show()  # Displays only the first 20 rows

# Display the DataFrame in a tabular format
display(df)  # For Databricks notebooks
```

### Interview Question: How Does inferSchema Work?

**Behind the Scenes:** When you use `inferSchema`, Spark runs a job that scans the CSV file from top to bottom to identify the best-suited data type for each column based on the values it encounters.

### Does It Make Sense to Use inferSchema?

 **Pros:**
- Useful when the schema of the file keeps changing, as it allows Spark to automatically detect the data types.

**Cons:**
- **Performance Impact:** Spark must scan the entire file, which can take extra time, especially for large files.
- **Loss of Control:** You lose the ability to explicitly define the schema, which may lead to incorrect data types if the data is inconsistent.

### Conclusion

Loading data from CSV files into a DataFrame is straightforward in PySpark. Understanding how to define a schema and the implications of using `inferSchema` is crucial for optimizing your data processing workflows.

This document provides a comprehensive overview of how to load CSV data into DataFrames in PySpark, along with considerations for using schema inference. Let me know if you need any more details or adjustments!

## PySpark DataFrame Schema Definition

### 1. Defining Schema Programmatically with StructType

```python
from pyspark.sql.types import *

# Define the schema using StructType
employeeSchema = StructType([
   StructField("ID", IntegerType(), True),
   StructField("Name", StringType(), True),
   StructField("Age", IntegerType(), True),
   StructField("Salary", DoubleType(), True),
   StructField("Joining_Date", StringType(), True),  # Keeping as String for date issues
   StructField("Department", StringType(), True),
   StructField("Performance_Rating", IntegerType(), True),
   StructField("Email", StringType(), True),
   StructField("Address", StringType(), True),
   StructField("Phone", StringType(), True)
])

# Load the DataFrame with the defined schema
df = spark.read.load("/FileStore/tables/employees.csv",format="csv", header=True, schema=employeeSchema)

# Print the schema of the DataFrame
df.printSchema()

# Optionally display the DataFrame
# display(df)
```


### 2. Defining Schema as a String

```python
# Define the schema as a string
employeeSchemaString = '''
ID Integer,
Name String,
Age Integer,
Salary Double,
Joining_Date String,
Department String,
Performance_Rating Integer,
Email String,
Address String,
Phone String
'''

# Load the DataFrame with the defined schema
df = spark.read.load("dbfs:/FileStore/shared_uploads/imsvk11@gmail.com/employee_data.csv", 
                     format="csv", header=True, schema=employeeSchemaString)

# Print the schema of the DataFrame
df.printSchema()

# Optionally display the DataFrame
# display(df)
```


### Explanation

- **Schema Definition**: Both methods define a schema for the DataFrame, accommodating the dataset's requirements, including handling null values where applicable.
- **Data Types**: The Joining_Date column is defined as StringType to accommodate potential date format issues or missing values.
- **Loading the DataFrame**: The `spark.read.load` method is used to load the CSV file into a DataFrame using the specified schema.
- **Printing the Schema**: The `df.printSchema()` function allows you to verify that the DataFrame is structured as intended.
