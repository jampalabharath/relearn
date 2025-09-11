+++
title = "Date Functions"
weight = 4
+++


In PySpark, you can use various date functions to manipulate and analyze date and timestamp columns.  
We’ll explore:  
- `current_date`  
- `current_timestamp`  
- `date_add`  
- `date_sub`  
- `datediff`  
- `months_between`  

---

### Sample Code

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import current_date, current_timestamp, date_add, date_sub, col, datediff, months_between, to_date, lit

# Generate a DataFrame with 10 rows, adding "today" and "now" columns
dateDF = spark.range(10).withColumn("today", current_date()).withColumn("now", current_timestamp())

# Show the DataFrame with today and now columns
dateDF.show(truncate=False)
```

### Key Functions

- **`current_date()`** → Returns current date.
- **`current_timestamp()`** → Returns current timestamp (date + time).
- **`date_sub(col("today"), 5)`** → Subtracts 5 days.
- **`date_add(col("today"), 5)`** → Adds 5 days.
- **`datediff(date1, date2)`** → Returns difference in days.
- **`months_between(date1, date2)`** → Returns difference in months.

---


Working with dates and timestamps often requires converting formats and extracting components.  
We’ll explore:

- `to_date`
- `to_timestamp`
- `year`, `month`, `dayofmonth`
- `hour`, `minute`, `second`

---

### 1. to_date

- Converts a string to a date (default format: `yyyy-MM-dd`).    
- If format doesn’t match, returns **null**.
- Example:
    `to_date(lit("2017-12-11"), "yyyy-dd-MM")`

### 2. to_timestamp

- Converts a string with date & time into a timestamp.
- Allows extraction of time components.

### 3. Extracting Components

- **year()** → Extracts year.    
- **month()** → Extracts month.
- **dayofmonth()** → Extracts day.
- **hour()** → Extracts hour.
- **minute()** → Extracts minute.
- **second()** → Extracts second.

### Example Output
For input `"2017-12-11"` (format `yyyy-dd-MM`):
- Year: **2017**
- Month: **12**
- Day: **11**
- Hour: **0**
- Minute: **0**
- Second: **0**

For invalid input (e.g., `"2017-20-12"`):
- Result: **null**