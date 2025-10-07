# Copy Activity

## Copy Activity in Azure Data Factory

### What is Copy Activity?

Copy Activity is the core data movement activity in **Azure Data Factory (ADF)**. It allows you to securely and efficiently copy data from a source data store to a destination (sink) data store. This is widely used for **data ingestion**, **ETL pipelines**, and **data migration** across on-premises, cloud, and hybrid environments.

---

### Key Features

- **Supports 100+ connectors** for structured, semi-structured, and unstructured data.
    
- **Scalable data movement** – can handle gigabytes to terabytes of data.
    
- **Secure integration** with managed identities, Key Vault, and encrypted channels.
    
- **Rich transformation support** like column mapping, schema drift handling, and format conversions.
    
- **Parallelism & partitioning** for high-performance copying.
    

---

### Components of Copy Activity

1. **Source (From)** – The dataset or linked service where the data resides (e.g., SQL DB, Blob storage, REST API).
    
2. **Sink (To)** – The destination dataset where the data will be written (e.g., ADLS, Synapse, Cosmos DB).
    
3. **Mapping** – Defines how columns/fields from the source map to the sink.
    
4. **Settings** – Performance tuning options such as degree of parallelism, staging, and fault tolerance.
    

---

### Workflow of Copy Activity

1. A pipeline triggers the Copy Activity.
    
2. ADF reads data from the **source** using a linked service.
    
3. Data is optionally transformed (format, schema mapping, partitioning).
    
4. Data is written to the **sink**.
    
5. The process is logged and monitored in the ADF monitoring dashboard.
    

---

### Supported Data Stores

- **Azure Sources/Sinks:** Blob Storage, Data Lake (ADLS Gen1/Gen2), Azure SQL, Synapse, Cosmos DB, Table Storage.
    
- **On-Premises:** SQL Server, Oracle, SAP, File system (via Self-hosted IR).
    
- **SaaS Applications:** Salesforce, Dynamics 365, ServiceNow, Google Analytics, etc.
    
- **Big Data & NoSQL:** Amazon S3, MongoDB, Cassandra, Hadoop.
    

---

### Performance Tuning Options

- **Parallel Copy:** Enable parallelism to copy partitions of data simultaneously.
    
- **Staging:** Use Azure Blob/ADLS as a staging area for complex transformations.
    
- **Batch Size & Block Size:** Adjust for optimal throughput.
    
- **Self-hosted IR Scaling:** Scale out compute for large hybrid data transfers.
    

---

### Monitoring & Troubleshooting

- **Activity Runs View:** Shows status (Success, Failed, In Progress).
    
- **Detailed Metrics:** Throughput, rows read/written, time taken.
    
- **Retry Policies:** Configure automatic retries on transient failures.
    
- **Integration with Log Analytics:** Centralized monitoring and alerting.
    

---

### Example Use Cases

- Migrating on-prem SQL Server data to Azure SQL Database.
    
- Ingesting CSV/JSON/XML files from Blob Storage to Synapse for reporting.
    
- Copying SaaS application data (e.g., Salesforce) into Data Lake for analytics.
    
- Hybrid ETL: Moving data from on-prem Oracle DB to ADLS for big data processing.
    

---
### Best Practices

- Use **Integration Runtime** close to data sources for better performance.
    
- Leverage **incremental loads** instead of full loads to optimize costs.
    
- Enable **fault tolerance** for large jobs.
    
- Store sensitive connection strings in **Azure Key Vault**.
    
- Use **parameterized datasets** for reusability.




==========================================


## 📂 Source Options for ADLS in Copy Activity

### 1. **File Path Settings**

- **File path type** → Choose how to select files:
    
    - _File path_ → Specific file (container/folder/file).
        
    - _Wildcard file path_ → Use `*` or `?` to match multiple files. Example: `input/*.csv`.
        
    - _Folder path_ → Copy all files in a folder.
        
- **Recursive** → If enabled, copies files from subfolders as well.
    

---

### 2. **File Filter Options**

- **File name wildcard** → Filter by pattern (e.g., `*.json`, `2025-09-*.parquet`).
    
- **Modified date filter** → Pick files based on _last modified time_. Useful for **incremental loads**.
    
- **List of files** → Provide explicit list of files (parameterized).
    

---

### 3. **Binary Copy**

- **Binary Copy Mode** → If enabled, files are copied as-is (no parsing). Used for images, videos, executables.
    

---

### 4. **Compression**

- If files are compressed, choose:
    
    - **Gzip, Bzip2, Deflate, Zip** → Auto-decompress on read.
        
    - _None_ → If files are plain text or structured formats.
        

---

### 5. **File Format Settings** (if not binary copy)

- **Delimited Text (CSV/TSV):**
    
    - Row delimiter, column delimiter
        
    - Escape character, quote character
        
    - First row as header (yes/no)
        
- **JSON:**
    
    - Single object or Array of objects
        
- **Parquet / ORC / Avro:**
    
    - Schema auto-detection
        
    - Support for column projection
        

---

### 6. **Partition Options** (for performance)

- **Max concurrent connections** → Number of parallel threads to read files.
    
- **Partition by folder/file** → ADF can split work by file or by folder for faster ingestion.
    

---

### 7. **Additional Options**

- **Preserve hierarchy** → Decide how to keep folder structure when moving data:
    
    - _PreserveHierarchy_ → Keeps folder structure.
        
    - _FlattenHierarchy_ → Drops folder paths, copies only files.
        
    - _MergeFiles_ → Combines multiple files into a single output file.
        
- **Ignore empty files** → Skip empty files instead of failing.
    
- **Max concurrent connections** → Speed up large dataset ingestion.
    

---

✅ Example Scenarios:

- **Incremental ingestion:** Use _last modified filter_ and wildcards (`2025-09-16*.csv`).
    
- **Big Data migration:** Enable _recursive_ + _parallel reads_ + _PreserveHierarchy_.
    
- **Data Lake to Synapse pipeline:** Use _Parquet/CSV reader_ with schema mapping.


## 🗄️ Source Options for SQL Server in Copy Activity

### 1. **Data Retrieval Options**

- **Table** → Pick a specific table from the connected database.
    
- **Query** → Provide a custom SQL query (parameterized if needed).
    
- **Stored Procedure** (only in some cases) → Call a stored procedure and use its result set as source.
    

---

### 2. **Query Behavior**

- **Use Query from Dataset or Inline** → You can store SQL queries in the dataset definition or write inline queries in the activity.
    
- **Parameterization** → Pass pipeline parameters into the query (e.g., `SELECT * FROM Sales WHERE OrderDate = '@{pipeline().parameters.Date}'`).
    

---

### 3. **Batching & Fetch Options**

- **Batch Size** → Number of rows fetched in one go (default varies, typically ~10,000). Helps tune performance for large datasets.
    
- **Isolation Level** → Read Uncommitted, Read Committed, Snapshot, etc. (controls concurrency & locking).
    

---

### 4. **Partition Options (for parallel reads)**

- **Partition Option** → Split source data into multiple partitions for faster reads. Options:
    
    - _None_ → Single-threaded copy.
        
    - _Physical Partitions_ → Use table partitions (if available).
        
    - _Dynamic Range_ → Partition data based on numeric/date column (e.g., `OrderID BETWEEN 1–10000`, etc.).
        
- **Partition Column** → Column used for partitioning (usually numeric or date).
    
- **Number of Partitions** → Controls parallelism (e.g., 4 partitions = 4 parallel queries).
    

---

### 5. **Additional Options**

- **SQL Command Timeout** → Maximum time allowed for query execution.
    
- **Detect Changes (Incremental Load)** → Use watermark columns (like `LastModifiedDate`) for incremental copies.
    
- **Enable CDC (Change Data Capture)** → If SQL Server has CDC enabled, ADF can use it to capture only changed rows.
    

---

### 6. **Data Consistency & Validation**

- **Table/Query Validation** → Validate schema before copying.
    
- **Null Handling** → How NULL values are handled when mapping.
    
- **String Handling** → Options for trimming, padding, or encoding.
    

---

✅ **Example Use Cases**

- Full table load → Choose _Table_ option.
    
- Incremental load → Use _Query_ with a `WHERE LastModified > @{pipeline().parameters.LastRunTime}` filter.
    
- High-volume migration → Use _Dynamic Range partitioning_ on an indexed column for parallel ingestion


# 1️⃣ Sink: **CSV (Delimited Text in Blob/ADLS)**

When you choose **CSV** as the **sink format**, you’ll see file-related settings.

### **File Path Options**

- **File name option**
    
    - _Output to single file_ → All data is written into one file.
        
    - _Per partition/file_ → Each partition creates a separate file (e.g., `part-0001.csv`).
        
    - _Preserve source hierarchy_ → Keeps folder/file structure from source.
        
- **File naming pattern** → Custom names like `data_{date}.csv`.
    
- **Folder path** → Target container/folder in Blob/ADLS.
    

### **File Format Settings**

- **Row delimiter** → `\n` (default), `\r\n`, or custom.
    
- **Column delimiter** → `,` (default), `;`, `|`, `\t`, etc.
    
- **Quote character** → Usually `"`.
    
- **Escape character** → Usually `\`.
    
- **Null value representation** → For example, empty string, `\N`, or `"NULL"`.
    
- **Write headers** → Yes/No (include column names as first row).
    

### **Compression**

- None, Gzip, Bzip2, Deflate, Zip.
    

### **Copy Behavior**

- _Overwrite_ (replace existing files).
    
- _Append_ (add new file without deleting old ones).
    
- _Merge_ (combine into existing file – limited scenarios).
    

---

# 2️⃣ Sink: **SQL Server**

When **SQL Server** is the **sink**, the options are more database-related.

### **Table & Write Options**

- **Table name** → Select or provide the target table.
    
- **Auto-create table** → ADF can auto-generate the table if it doesn’t exist (based on source schema).
    
- **Pre-copy script** → SQL script that runs before insert (e.g., `TRUNCATE TABLE TargetTable`).
    
- **Stored procedure sink** → Instead of direct inserts, call a stored procedure with input parameters.
    

### **Write Behavior**

- **Insert** → Default (append new rows).
    
- **Upsert** → Insert new + update existing (requires key column).
    
- **Update** → Update only existing rows.
    
- **Stored procedure** → Custom merge logic via proc.
    

### **Batching & Performance**

- **Batch size** → Number of rows per batch insert.
    
- **Bulk insert (PolyBase/BCP)** → Faster loading for large volumes.
    
- **Table lock** → Lock the table during write (improves speed but reduces concurrency).
    

### **Additional Options**

- **Identity insert** → Control whether identity columns are written or auto-generated.
    
- **Null handling** → Define how nulls map to SQL columns.
    
- **Transaction settings** → Commit mode (per batch or per entire load).
    
- **SQL command timeout** → Max allowed time for inserts.
    

---

✅ **When to use what?**

- **CSV Sink** → Good for **data lake staging**, archival, or feeding BI/ML pipelines.
    
- **SQL Server Sink** → Good for **operational databases, reporting, or direct analytics**.
  
  
  