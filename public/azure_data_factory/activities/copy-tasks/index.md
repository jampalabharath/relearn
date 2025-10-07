# Copy Activity Tasks

## 📝 Copy Data Activity – Hands-on Task Checklist
### 1. Basics & Setup
 
Create a pipeline with Copy Activity to copy data from Azure Blob Storage → Azure SQL Database.

Try copying structured (CSV), semi-structured (JSON), and binary files.

### 2. Connectors

Copy data from Azure Blob → ADLS Gen2.

Copy data from On-prem SQL Server → Azure SQL DB using Self-hosted IR.

Copy data from a REST API → ADLS.

### 3. Integration Runtime

Use Auto-resolve IR for cloud-to-cloud copy.

Set up a Self-hosted IR to copy on-prem SQL → Blob.

### 4. Mapping & Schema Handling

Test auto-mapping (matching column names).

Configure explicit mapping (manual mapping between different schemas).

Try a scenario with schema drift (extra columns in source) and handle it.

### 5. Performance Optimization

Enable parallel copy and observe throughput improvement.

Implement partitioned copy (range partition on an integer/date column).

Use staging (Blob/ADLS) when copying into Azure Synapse.

### 6. Transformations

Copy CSV → Parquet with compression (snappy).

Perform data type conversion (string → int, float → decimal).

Try copying compressed files (gzip, zip) and decompress during load.

### 7. Fault Tolerance & Error Handling

Configure retry policy for a failed copy.

Use fault tolerance settings to skip bad rows and log errors to a file.

### 8. Security

Store connection strings in Azure Key Vault and link them in Copy Activity.

Use Managed Identity authentication to access Azure Storage.

Test with service principal authentication for an external system.

### 9. Monitoring

Run the pipeline and review monitoring tab: throughput, data read/written, duration.

Introduce an error (e.g., wrong column mapping) and analyze detailed error logs.

Enable logging to Azure Monitor / Log Analytics.

### 10. Cost & Optimization

Compare runtime & cost for:

Single-threaded copy vs parallel copy.

Direct copy vs staged copy into Synapse.

### 11. Advanced Features

Enable Preserve folder hierarchy when copying nested directories.

Test binary copy for images or PDFs.

Configure incremental copy using a watermark column (LastModifiedDate).