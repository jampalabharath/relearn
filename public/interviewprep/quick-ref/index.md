# SQL & PySpark Queries

This guide pairs each **SQL** example with a **PySpark DataFrame API** equivalent.  
Assumptions: `df` is `employees`, and other tables are similarly named DataFrames (`departments`, `orders`, etc.).

> **Imports used in PySpark examples**
```python
from pyspark.sql import functions as F
from pyspark.sql.functions import col, lit, when, lower, upper, count, sum, avg, max, min, row_number, rank, dense_rank, ntile, lead, lag, coalesce
from pyspark.sql.window import Window
```

---

## 🧩 Section 1: Basic Queries (1–10)

### SQL
```sql
-- 1
SELECT * FROM employees;
-- 2
SELECT first_name, last_name, hire_date FROM employees;
-- 3
SELECT COUNT(*) FROM employees;
-- 4
SELECT COUNT(salary) FROM employees;
-- 5
SELECT DISTINCT department FROM employees;
-- 6
SELECT * FROM employees WHERE department = 'Sales';
-- 7
SELECT * FROM employees WHERE department = 'Sales' AND salary > 60000;
-- 8
SELECT * FROM employees WHERE department = 'Sales' OR department = 'Marketing';
-- 9
SELECT * FROM employees
WHERE (department = 'Sales' OR department = 'Marketing') AND salary > 70000;
-- 10
SELECT * FROM employees WHERE NOT department = 'IT';
```

### PySpark
```python
# 1
df.show()
# 2
df.select("first_name", "last_name", "hire_date").show()
# 3
df.count()
# 4
df.select(F.count("salary")).show()
# 5
df.select("department").distinct().show()
# 6
df.filter(col("department") == "Sales").show()
# 7
df.filter((col("department") == "Sales") & (col("salary") > 60000)).show()
# 8
df.filter((col("department") == "Sales") | (col("department") == "Marketing")).show()
# 9
df.filter(((col("department") == "Sales") | (col("department") == "Marketing")) & (col("salary") > 70000)).show()
# 10
df.filter(col("department") != "IT").show()
```

---

## 🔎 Section 2: Filtering & Pattern Matching (11–20)

### SQL
```sql
-- 11
SELECT * FROM employees WHERE department IN ('Sales', 'Marketing', 'IT');
-- 12
SELECT * FROM employees WHERE department NOT IN ('Sales', 'Marketing');
-- 13
SELECT * FROM employees WHERE salary BETWEEN 50000 AND 75000;
-- 14
SELECT * FROM employees WHERE first_name ILIKE '%jo%';
-- 15
SELECT * FROM employees WHERE last_name LIKE 'Smi%';
-- 16
SELECT * FROM employees WHERE email LIKE '%@gmail.com';
-- 17
SELECT * FROM employees WHERE first_name LIKE '_a%';
-- 18
SELECT * FROM employees WHERE manager_id IS NULL;
-- 19
SELECT * FROM employees WHERE manager_id IS NOT NULL;
-- 20
SELECT * FROM employees WHERE hire_date BETWEEN '2023-01-01' AND '2023-12-31';
```

### PySpark
```python
# 11
df.filter(col("department").isin("Sales", "Marketing", "IT")).show()
# 12
df.filter(~col("department").isin("Sales", "Marketing")).show()
# 13
df.filter(col("salary").between(50000, 75000)).show()
# 14  (case-insensitive search)
df.filter(lower(col("first_name")).like("%jo%")).show()
# 15
df.filter(col("last_name").startswith("Smi")).show()
# 16
df.filter(col("email").endswith("@gmail.com")).show()
# 17  (single char then 'a')
df.filter(col("first_name").rlike("^.a.*")).show()
# 18
df.filter(col("manager_id").isNull()).show()
# 19
df.filter(col("manager_id").isNotNull()).show()
# 20
df.filter(col("hire_date").between("2023-01-01", "2023-12-31")).show()
```

---

## ↕️ Section 3: Sorting & Limiting (21–30)

### SQL
```sql
-- 21
SELECT * FROM employees ORDER BY last_name ASC;
-- 22
SELECT * FROM employees ORDER BY salary DESC;
-- 23
SELECT * FROM employees ORDER BY department ASC, salary DESC;
-- 24
SELECT * FROM employees ORDER BY salary DESC LIMIT 10;
-- 25
SELECT * FROM employees ORDER BY hire_date DESC LIMIT 5;
-- 26
SELECT DISTINCT department FROM employees ORDER BY department;
-- 27
SELECT * FROM employees ORDER BY employee_id OFFSET 10 LIMIT 10;
-- 28
SELECT * FROM employees ORDER BY salary DESC LIMIT 1;
-- 29
SELECT DISTINCT salary FROM employees ORDER BY salary DESC LIMIT 1 OFFSET 1;
-- 30
SELECT first_name, last_name, (salary * 0.1) AS bonus
FROM employees ORDER BY bonus DESC;
```

### PySpark
```python
# 21
df.orderBy(col("last_name").asc()).show()
# 22
df.orderBy(col("salary").desc()).show()
# 23
df.orderBy(col("department").asc(), col("salary").desc()).show()
# 24
df.orderBy(col("salary").desc()).limit(10).show()
# 25
df.orderBy(col("hire_date").desc()).limit(5).show()
# 26
df.select("department").distinct().orderBy("department").show()
# 27  (rows 11–20): use ROW_NUMBER then filter
w = Window.orderBy("employee_id")
df.withColumn("rn", row_number().over(w)).filter((col("rn") >= 11) & (col("rn") <= 20)).drop("rn").show()
# 28
df.orderBy(col("salary").desc()).limit(1).show()
# 29  (second-highest salary) – two ways
df.select("salary").distinct().orderBy(col("salary").desc()).limit(2).orderBy(col("salary").asc()).limit(1).show()
# or using dense_rank
w2 = Window.orderBy(col("salary").desc())
df.select("salary", dense_rank().over(w2).alias("dr")).filter(col("dr") == 2).select("salary").distinct().show()
# 30
df.select("first_name", "last_name", (col("salary") * 0.1).alias("bonus")).orderBy(col("bonus").desc()).show()
```

---

## 📊 Section 4: Aggregation & Grouping (31–40)

### SQL
```sql
-- 31
SELECT department, COUNT(*) FROM employees GROUP BY department;
-- 32
SELECT department, AVG(salary) FROM employees GROUP BY department;
-- 33
SELECT department, SUM(salary) FROM employees GROUP BY department;
-- 34
SELECT department, MAX(salary) FROM employees GROUP BY department;
-- 35
SELECT department, MIN(salary) FROM employees GROUP BY department;
-- 36
SELECT department, AVG(salary)
FROM employees GROUP BY department
HAVING AVG(salary) > 65000;
-- 37
SELECT department, COUNT(*)
FROM employees GROUP BY department
HAVING COUNT(*) > 5;
-- 38
SELECT department, AVG(salary)
FROM employees
WHERE hire_date > '2022-01-01'
GROUP BY department;
-- 39
SELECT department, job_title, AVG(salary)
FROM employees GROUP BY department, job_title;
-- 40
SELECT department, MAX(salary) FROM employees GROUP BY department;
```

### PySpark
```python
# 31
df.groupBy("department").count().show()
# 32
df.groupBy("department").agg(avg("salary").alias("avg_salary")).show()
# 33
df.groupBy("department").agg(sum("salary").alias("total_salary")).show()
# 34
df.groupBy("department").agg(max("salary").alias("max_salary")).show()
# 35
df.groupBy("department").agg(min("salary").alias("min_salary")).show()
# 36
df.groupBy("department").agg(avg("salary").alias("avg_salary")).filter(col("avg_salary") > 65000).show()
# 37
df.groupBy("department").count().filter(col("count") > 5).show()
# 38
df.filter(col("hire_date") > "2022-01-01").groupBy("department").agg(avg("salary")).show()
# 39
df.groupBy("department", "job_title").agg(avg("salary").alias("avg_salary")).show()
# 40
df.groupBy("department").agg(max("salary").alias("max_salary")).show()
```

---

## 🔗 Section 5: Joins (41–60)

### SQL
```sql
-- 41
SELECT e.first_name, d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id;

-- 42
SELECT e.first_name, d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id;

-- 43
SELECT e.first_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id
WHERE d.department_id IS NULL;

-- 44
SELECT e.first_name, d.department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.department_id;

-- 45
SELECT e.first_name, d.department_name
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.department_id;

-- 46
SELECT e.first_name, p.project_name, d.department_name
FROM employees e
JOIN projects p ON e.employee_id = p.employee_id
JOIN departments d ON e.department_id = d.department_id;

-- 47
SELECT e.first_name AS employee, m.first_name AS manager
FROM employees e
JOIN employees m ON e.manager_id = m.employee_id;

-- 48
SELECT o.order_id, c.customer_name
FROM orders o
JOIN customers c ON o.customer_zip_code = c.customer_zip_code;

-- 49
SELECT * FROM employees CROSS JOIN departments;

-- 50
SELECT p.*
FROM products p
LEFT JOIN sales s ON p.product_id = s.product_id
WHERE s.sale_id IS NULL;

-- 51
SELECT e.first_name, e.last_name
FROM employees e
LEFT JOIN employee_projects ep ON e.employee_id = ep.employee_id
WHERE ep.project_id IS NULL;

-- 52
SELECT p.project_name
FROM projects p
LEFT JOIN employee_projects ep ON p.project_id = ep.project_id
WHERE ep.employee_id IS NULL;

-- 53
SELECT d.department_name, SUM(e.salary) AS total_salary
FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id
GROUP BY d.department_name;

-- 54
SELECT d.department_name, COUNT(e.employee_id) AS num_employees
FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id
GROUP BY d.department_name;

-- 55
SELECT o.*, c.customer_name
FROM orders o
JOIN customers c
  ON o.customer_id = c.customer_id
 AND o.order_date = c.last_purchase_date;

-- 56
SELECT e.first_name,
       m.first_name AS manager_name,
       d.department_name
FROM employees e
JOIN employees m ON e.manager_id = m.employee_id
JOIN departments d ON m.department_id = d.department_id;

-- 57
SELECT e.employee_id, h.salary_grade
FROM employees e
JOIN salary_grades h ON e.salary BETWEEN h.min_salary AND h.max_salary;

-- 58
SELECT e.first_name, d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.department_id
WHERE d.location_city = 'New York';

-- 59
SELECT d.department_name, AVG(e.salary) AS avg_salary
FROM employees e
JOIN departments d ON e.department_id = d.department_id
GROUP BY d.department_name;

-- 60
SELECT a.order_id, b.item_name
FROM table_a a
JOIN table_b b ON a.product_id = b.item_id;
```

### PySpark
```python
# 41
employees.join(departments, "department_id", "inner").select("first_name", "department_name").show()

# 42
employees.join(departments, "department_id", "left").select("first_name", "department_name").show()

# 43
employees.join(departments, "department_id", "left")\
         .filter(departments.department_id.isNull())\
         .select(employees.first_name).show()

# 44
employees.join(departments, "department_id", "right").select("first_name", "department_name").show()

# 45
employees.join(departments, "department_id", "outer").select("first_name", "department_name").show()

# 46
employees.join(projects, "employee_id")\
         .join(departments, "department_id")\
         .select("first_name", "project_name", "department_name").show()

# 47
e = employees.alias("e"); m = employees.alias("m")
e.join(m, col("e.manager_id") == col("m.employee_id"))\
 .select(col("e.first_name").alias("employee"), col("m.first_name").alias("manager")).show()

# 48
orders.join(customers, orders.customer_zip_code == customers.customer_zip_code)\
      .select("order_id", "customer_name").show()

# 49
employees.crossJoin(departments).show()

# 50
products.join(sales, "product_id", "left")\
        .filter(col("sale_id").isNull())\
        .select("product_id", *[c for c in products.columns if c != "product_id"]).show()

# 51
employees.join(employee_projects, "employee_id", "left")\
         .filter(col("project_id").isNull())\
         .select("first_name", "last_name").show()

# 52
projects.join(employee_projects, "project_id", "left")\
        .filter(col("employee_id").isNull())\
        .select("project_name").show()

# 53
departments.join(employees, "department_id", "left")\
          .groupBy("department_name").agg(sum("salary").alias("total_salary")).show()

# 54
departments.join(employees, "department_id", "left")\
          .groupBy("department_name").agg(count("employee_id").alias("num_employees")).show()

# 55
orders.join(customers, (orders.customer_id == customers.customer_id) & (orders.order_date == customers.last_purchase_date))\
      .select(orders["*"], customers.customer_name).show()

# 56
e = employees.alias("e"); m = employees.alias("m"); d = departments.alias("d")
e.join(m, col("e.manager_id") == col("m.employee_id"))\
 .join(d, col("m.department_id") == col("d.department_id"))\
 .select(col("e.first_name"), col("m.first_name").alias("manager_name"), col("d.department_name")).show()

# 57
employees.join(salary_grades, (employees.salary >= salary_grades.min_salary) & (employees.salary <= salary_grades.max_salary))\
         .select("employee_id", "salary_grade").show()

# 58
employees.join(departments, "department_id")\
         .filter(col("location_city") == "New York")\
         .select("first_name", "department_name").show()

# 59
employees.join(departments, "department_id")\
         .groupBy("department_name").agg(avg("salary").alias("avg_salary")).show()

# 60
table_a.join(table_b, table_a.product_id == table_b.item_id)\
       .select(table_a.order_id, table_b.item_name).show()
```

---

## 🧠 Section 6: Subqueries (61–75)

### SQL
```sql
-- 61
SELECT * FROM employees WHERE salary > (SELECT AVG(salary) FROM employees);
-- 62
SELECT * FROM employees WHERE department_id IN (
  SELECT department_id FROM departments WHERE location_city = 'San Francisco'
);
-- 63
SELECT d.department_name, s.avg_salary
FROM (
  SELECT department_id, AVG(salary) AS avg_salary
  FROM employees GROUP BY department_id
) AS s
JOIN departments d ON s.department_id = d.department_id;
-- 64
SELECT first_name, salary,
       (SELECT AVG(salary) FROM employees) AS company_avg_salary
FROM employees;
-- 65
SELECT * FROM employees e
WHERE salary > (SELECT AVG(salary) FROM employees WHERE department_id = e.department_id);
-- 66
SELECT d.department_name FROM departments d
WHERE EXISTS (SELECT 1 FROM employees e WHERE e.department_id = d.department_id AND e.salary > 100000);
-- 67
SELECT d.department_name FROM departments d
WHERE NOT EXISTS (SELECT 1 FROM employees e WHERE e.department_id = d.department_id);
-- 68
SELECT * FROM departments WHERE department_id IN (SELECT DISTINCT department_id FROM employees);
-- 69
SELECT * FROM employees e WHERE EXISTS (SELECT 1 FROM orders o WHERE o.employee_id = e.employee_id);
-- 70
SELECT d.department_name,
       (SELECT SUM(salary) FROM employees e WHERE e.department_id = d.department_id) AS total_department_salary
FROM departments d;
-- 71
SELECT department_name
FROM departments
WHERE department_id = (
  SELECT department_id
  FROM employees
  GROUP BY department_id
  ORDER BY AVG(salary) DESC
  LIMIT 1
);
-- 72
SELECT customer_name FROM customers
WHERE customer_id IN (SELECT customer_id FROM orders WHERE product_id = 123);
-- 73
SELECT COUNT(*) FROM employees WHERE salary > (SELECT AVG(salary) FROM employees);
-- 74
SELECT customer_name FROM customers
WHERE customer_id NOT IN (SELECT DISTINCT customer_id FROM orders);
-- 75
SELECT e.first_name, e.salary, e.job_title
FROM employees e
WHERE e.salary < (SELECT AVG(salary) FROM employees WHERE job_title = e.job_title);
```

### PySpark
```python
# 61
avg_sal = df.select(avg("salary").alias("avg")).first()["avg"]
df.filter(col("salary") > avg_sal).show()

# 62
sf_depts = departments.filter(col("location_city") == "San Francisco").select("department_id").distinct()
df.join(sf_depts, "department_id").show()

# 63
avg_df = df.groupBy("department_id").agg(avg("salary").alias("avg_salary"))
avg_df.join(departments, "department_id").select("department_name", "avg_salary").show()

# 64
avg_val = df.select(avg("salary").alias("avg")).first()["avg"]
df.select("first_name", "salary", lit(avg_val).alias("company_avg_salary")).show()

# 65
dept_avg = df.groupBy("department_id").agg(avg("salary").alias("dept_avg"))
df.join(dept_avg, "department_id").filter(col("salary") > col("dept_avg")).show()

# 66
high_paid = df.filter(col("salary") > 100000).select("department_id").distinct()
departments.join(high_paid, "department_id").select("department_name").distinct().show()

# 67
departments.join(df.select("department_id").distinct(), "department_id", "left_anti").select("department_name").show()

# 68
departments.join(df.select("department_id").distinct(), "department_id").show()

# 69
df.join(orders.select("employee_id").distinct(), "employee_id").select(df["*"]).distinct().show()

# 70
dept_totals = df.groupBy("department_id").agg(sum("salary").alias("total_department_salary"))
departments.join(dept_totals, "department_id").select("department_name", "total_department_salary").show()

# 71
dept_avg = df.groupBy("department_id").agg(avg("salary").alias("avg_salary"))
top_dept = dept_avg.orderBy(col("avg_salary").desc()).limit(1)
departments.join(top_dept, "department_id").select("department_name").show()

# 72
buyers = orders.filter(col("product_id") == 123).select("customer_id").distinct()
customers.join(buyers, "customer_id").select("customer_name").show()

# 73
df.filter(col("salary") > avg_sal).count()

# 74
customers.join(orders.select("customer_id").distinct(), "customer_id", "left_anti").select("customer_name").show()

# 75
job_avg = df.groupBy("job_title").agg(avg("salary").alias("job_avg"))
df.join(job_avg, "job_title").filter(col("salary") < col("job_avg")).select("first_name", "salary", "job_title").show()
```

---

## 🪟 Section 7: Window Functions (76–85)

### SQL
```sql
-- 76
SELECT first_name, department, salary,
       ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS rn
FROM employees;
-- 77
SELECT first_name, department, salary,
       RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rank
FROM employees;
-- 78
SELECT first_name, department, salary,
       DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS dense_rank
FROM employees;
-- 79
SELECT first_name, salary, NTILE(4) OVER (ORDER BY salary DESC) AS quartile
FROM employees;
-- 80
SELECT order_date, total_amount,
       LEAD(total_amount, 1) OVER (ORDER BY order_date) AS next_order_amount
FROM orders;
-- 81
SELECT order_date, total_amount,
       LAG(total_amount, 1, 0) OVER (ORDER BY order_date) AS previous_order_amount
FROM orders;
-- 82
SELECT order_date, total_amount,
       SUM(total_amount) OVER (ORDER BY order_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total
FROM orders;
-- 83
SELECT order_date, total_amount,
       AVG(total_amount) OVER (ORDER BY order_date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS moving_avg
FROM orders;
-- 84
SELECT * FROM (
  SELECT first_name, department, salary,
         RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rn
  FROM employees
) AS ranked_employees
WHERE rn <= 3;
-- 85
SELECT product_id, SUM(sales) AS product_sales,
       SUM(SUM(sales)) OVER () AS total_sales,
       (SUM(sales) / SUM(SUM(sales)) OVER ()) * 100 AS percentage_of_total
FROM sales GROUP BY product_id;
```

### PySpark
```python
w_dept = Window.partitionBy("department").orderBy(col("salary").desc())

# 76
df.select("first_name", "department", "salary",
          row_number().over(w_dept).alias("rn")).show()

# 77
df.select("first_name", "department", "salary",
          rank().over(w_dept).alias("rank")).show()

# 78
df.select("first_name", "department", "salary",
          dense_rank().over(w_dept).alias("dense_rank")).show()

# 79
df.select("first_name", "salary",
          ntile(4).over(Window.orderBy(col("salary").desc())).alias("quartile")).show()

# 80
orders.select("order_date", "total_amount",
              lead("total_amount", 1).over(Window.orderBy("order_date")).alias("next_order_amount")).show()

# 81
orders.select("order_date", "total_amount",
              lag("total_amount", 1, 0).over(Window.orderBy("order_date")).alias("previous_order_amount")).show()

# 82
orders.select("order_date", "total_amount",
              F.sum("total_amount").over(Window.orderBy("order_date")
                  .rowsBetween(Window.unboundedPreceding, Window.currentRow)).alias("running_total")).show()

# 83
orders.select("order_date", "total_amount",
              avg("total_amount").over(Window.orderBy("order_date").rowsBetween(-2, Window.currentRow)).alias("moving_avg")).show()

# 84
df.withColumn("rn", rank().over(w_dept)).filter(col("rn") <= 3).drop("rn").show()

# 85
prod_sales = sales.groupBy("product_id").agg(sum("sales").alias("product_sales"))
prod_sales.withColumn("total_sales", sum("product_sales").over(Window.partitionBy()))\
          .withColumn("percentage_of_total", (col("product_sales") / col("total_sales")) * 100)\
          .select("product_id", "product_sales", "total_sales", "percentage_of_total").show()
```

---

## 🧮 Section 8: CTEs & Data Manipulation (86–100)

### SQL
```sql
-- 86
WITH department_avg AS (
  SELECT department_id, AVG(salary) AS avg_dept_salary
  FROM employees GROUP BY department_id
)
SELECT e.first_name, e.salary, da.avg_dept_salary
FROM employees e
JOIN department_avg da ON e.department_id = da.department_id
WHERE e.salary > da.avg_dept_salary;

-- 87
INSERT INTO employees (first_name, last_name, salary) VALUES ('John', 'Doe', 60000);

-- 88
UPDATE employees SET salary = salary * 1.05 WHERE department = 'IT';

-- 89
DELETE FROM employees WHERE employee_id = 101;

-- 90
TRUNCATE TABLE old_data;

-- 91
SELECT first_name FROM employees
UNION
SELECT first_name FROM customers;

-- 92
SELECT first_name FROM employees
UNION ALL
SELECT first_name FROM customers;

-- 93
SELECT first_name, salary,
  CASE
    WHEN salary > 100000 THEN 'High Earner'
    WHEN salary BETWEEN 50000 AND 100000 THEN 'Mid-Range'
    ELSE 'Junior'
  END AS salary_level
FROM employees;

-- 94
SELECT department,
  COUNT(CASE WHEN salary > 70000 THEN 1 END) AS high_salary_count,
  COUNT(CASE WHEN salary <= 70000 THEN 1 END) AS low_salary_count
FROM employees
GROUP BY department;

-- 95
SELECT CAST(order_date AS DATE) FROM orders;

-- 96
SELECT COALESCE(email, 'No Email Provided') FROM employees;

-- 97
SELECT NULLIF(salary, 0) FROM employees;

-- 98 (Recursive CTE)
WITH RECURSIVE subordinates AS (
  SELECT employee_id, manager_id FROM employees WHERE employee_id = 1
  UNION ALL
  SELECT e.employee_id, e.manager_id
  FROM employees e JOIN subordinates s ON e.manager_id = s.employee_id
)
SELECT * FROM subordinates;

-- 99
SELECT department, job_title, SUM(salary)
FROM employees
GROUP BY GROUPING SETS ((department), (job_title), ());

-- 100
SELECT department, job_title, SUM(salary)
FROM employees
GROUP BY ROLLUP(department, job_title);
```

### PySpark
```python
# 86  (Use SQL CTE via temp view)
df.createOrReplaceTempView("employees")
spark.sql("""
WITH department_avg AS (
  SELECT department_id, AVG(salary) AS avg_dept_salary
  FROM employees GROUP BY department_id
)
SELECT e.first_name, e.salary, da.avg_dept_salary
FROM employees e
JOIN department_avg da ON e.department_id = da.department_id
WHERE e.salary > da.avg_dept_salary
""").show()

# 87  (Insert-like: union new rows)
df_new = spark.createDataFrame([("John", "Doe", 60000)], ["first_name", "last_name", "salary"])
df = df.unionByName(df_new)

# 88  (Update-like)
df = df.withColumn("salary", when(col("department") == "IT", col("salary") * 1.05).otherwise(col("salary")))

# 89  (Delete-like)
df = df.filter(col("employee_id") != 101)

# 90  (Truncate-like in-memory)
df = df.limit(0)  # for Delta table: spark.sql("TRUNCATE TABLE old_data")

# 91  UNION (distinct)
df_union_distinct = df1.union(df2).distinct()

# 92  UNION ALL
df_union_all = df1.union(df2)

# 93  CASE/WHEN
df.select("first_name", "salary",
          when(col("salary") > 100000, "High Earner")
          .when((col("salary") >= 50000) & (col("salary") <= 100000), "Mid-Range")
          .otherwise("Junior").alias("salary_level")).show()

# 94  Pivot without true pivot
df.groupBy("department").agg(
    count(when(col("salary") > 70000, 1)).alias("high_salary_count"),
    count(when(col("salary") <= 70000, 1)).alias("low_salary_count")
).show()

# 95  CAST
orders.select(col("order_date").cast("date").alias("order_date")).show()

# 96  COALESCE
df.select(coalesce(col("email"), lit("No Email Provided")).alias("email")).show()

# 97  NULLIF(salary,0) => NULL when equal
df.select(when(col("salary") == 0, lit(None)).otherwise(col("salary")).alias("salary_or_null")).show()

# 98  Recursive CTE not native; iterative approach example (hierarchy traversal)
# Start with seed
seed = df.filter(col("employee_id") == 1).select("employee_id", "manager_id")
result = seed
frontier = seed
while True:
    step = df.select("employee_id", "manager_id")\
             .join(frontier, df.manager_id == frontier.employee_id, "inner")\
             .select(df.employee_id, df.manager_id).dropDuplicates()
    new_nodes = step.join(result, ["employee_id", "manager_id"], "left_anti")
    if new_nodes.rdd.isEmpty():
        break
    result = result.unionByName(new_nodes).dropDuplicates()
    frontier = new_nodes
# result contains recursive closure

# 99  GROUPING SETS -> cube with filters
cube_df = df.cube("department", "job_title").agg(sum("salary").alias("sum_salary"))
# Keep only grouping sets: (department), (job_title), ()
from pyspark.sql.functions import grouping_id
cube_df.filter(
    (grouping_id("department", "job_title") == 1) |  # (department,NULL)
    (grouping_id("department", "job_title") == 2) |  # (NULL,job_title)
    (grouping_id("department", "job_title") == 3)    # (NULL,NULL)
).show()

# 100  ROLLUP
df.rollup("department", "job_title").agg(sum("salary").alias("sum_salary")).show()
```

---

### ✅ Notes
- Some SQL constructs (OFFSET, Recursive CTE) don’t map 1:1 to PySpark; patterns above show practical equivalents.
- Replace `.show()` with `.write.format("delta")...` to persist as needed.
