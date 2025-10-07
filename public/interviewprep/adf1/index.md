# ADF 1

## 1. What are the different types of Integration runtimes(IR) in ADF, and when should you use each?

**Integration runtime (IR)** is a compute engine that provides computational resources to perform **data movement, transformation, and orchestration** in ADF.

### Types of Integration Runtimes

1. **Azure IR**  
   - Use when performing **data movement, transformation, and orchestration entirely within the cloud**.  

2. **Self-Hosted IR**  
   - Use when **any source or destination is an on-premises system**.  
   - Required to **pull data from on-prem systems**.  

3. **Azure SSIS IR**  
   - Specifically built to **lift and shift Microsoft SSIS packages**.  

### Managing Integration Runtimes in ADF

- Navigate to **ADF -> Manage tab -> Integration Runtimes**.  
- **AutoResolveIntegrationRuntime** is present by default.  

### Creating a New IR

1. Click on **Add**.  
2. Choose the type of IR: **Self-Hosted**, **SSIS**, or **Airflow**.



## 2. Platform for Creating Azure Self-Hosted IR

- **Self-Hosted IR** requires compute **outside of Azure**, so you can use:  
  - A **local machine**  
  - A **hosted virtual machine**  

## 3. Handling Schema Drift in ADF

When dealing with **evolving datasets**, e.g., a new column appears in the source not present in the destination:

- Use **Mapping Dataflows** to apply transformations.  

### Options in Mapping Dataflows

1. **Schema Drift**  
   - Available when creating the **source dataset** in Mapping Dataflows.  
   - Allows handling evolving schemas without errors.  
   - Example: If a 5th column appears in the source, it will be written to the destination, with **NULL for existing records**.  

2. **Auto Mapping**  
   - Automatically maps columns between source and destination, eliminating manual schema mapping.  

### Recommended Approach

- **Combine Schema Drift and Auto Mapping** to handle evolving datasets seamlessly.



## 4. How would you load multiple data files from REST APIs using SINGLE copy activity?

### Steps

1. **Create a new pipeline**
   - Go to **Author tab → Create a new pipeline** (name it `pipAPI`).

2. **Add a ForEach activity**
   - Drag a **ForEach** activity into the pipeline.

3. **Configure ForEach parameters**
   - In the **Parameters** tab of the ForEach activity, create a parameter:
     - **Name:** `loop_value`  
     - **Type:** `Array`  
     - **Default Value:**
       ```json
       [
         {"fname": "x.csv"},
         {"fname": "y.csv"},
         {"fname": "z.csv"}
       ]
       ```

4. **Set the ForEach items**
   - Go to the **Settings** tab → under **Items**, add dynamic content:
     ```text
     @pipeline().parameters.loop_value
     ```

5. **Add a Copy Data activity inside ForEach**
   - Drag a **Copy Data** activity inside the ForEach loop.

---

### Source Configuration

6. **Create Source Dataset**
   - Create a dataset `ds_api` with:
     - **Datastore:** HTTP
     - **Data format:** CSV
   - Choose or create a linked service `ls_api`:
     - **Integration Runtime:** AutoResolveIR  
     - **Base URL:** `https://raw.githubuserdata.com/`  
     - **Authentication type:** Anonymous  
     - Test and create the connection.

7. **Parameterize the Source Dataset**
   - Go to the **Parameters** tab:
     - Add parameter:  
       - **Name:** `p_filenm`  
       - **Type:** String  
       - **Default value:** `@item().fname`
   - Go to the **Connection** tab → under **Relative URL**, add dynamic content:
     ```text
     anshikagit/ADF/refs/heads/main/Data/@{dataset().p_filenm}
     ```

---

### Sink Configuration

8. **Create Sink Dataset**
   - Create a dataset `ds_adls` with:
     - **Datastore:** ADLS Gen2  
     - **Data format:** CSV
   - Choose or create a linked service `ls_adls`:
     - **Integration Runtime:** AutoResolveIR  
     - **Storage account name:** ansadfstrg11  
     - Test and create the connection.

9. **Parameterize the Sink Dataset**
   - Go to the **Parameters** tab:
     - Add parameters:
       - **p_folder:** string  
       - **p_file:** string  
     - Default values:
       ```text
       @item().fname
       ```
   - In the **Connection** tab, under **File path**, add dynamic content:
     - **Directory:** `@dataset().p_folder`
     - **Filename:** `@dataset().p_file.csv`

---

10. **Run the Pipeline**
    - Click **Debug** to execute the pipeline and verify multiple file copies from REST API to ADLS Gen2.



## 5. How would you design a fault tolerant ADF pipeline to process billions of records daily?

To handle large-scale data processing (billions of records per day) in **Azure Data Factory (ADF)** efficiently and reliably, the following mechanisms can be leveraged:

---

### a. Retry Policy
- ADF provides built-in **Retry Policy** for all activities.
- You can specify:
  - **Number of retries** — how many times an activity should retry upon failure.
  - **Retry interval** — time gap between retries.
  
**Example:**  
If a Copy Activity fails due to transient network issues, setting:
- **Retry count:** `3`  
- **Retry interval:** `30 seconds`  
ensures ADF automatically retries before marking the activity as failed.

---

### b. Parallelism in Mapping Data Flows

#### i. Parallelism within a Single Mapping Data Flow
- Enables **multiple sink operations** to run **simultaneously** within the same data flow.
- Example:  
  Processing customer data and writing to both:
  - A **SQL database** (for analytics)
  - A **Data Lake** (for archival)
  
By enabling **"Run in parallel"** in **Sink Properties**, both sinks execute concurrently instead of sequentially — improving overall performance.

---

#### ii. Parallelism Across Multiple Data Flow Activities in a Pipeline

**1. Independent Data Sources**  
- If two datasets (e.g., **Sales** and **Inventory**) are independent:
  - Create two Data Flow activities — one for each dataset.
  - Place them in **parallel branches** within the pipeline.
  - ADF executes both simultaneously, reducing total runtime.

**2. Parallel ForEach Processing**  
- When processing multiple files (e.g., CSVs in a folder) with identical transformation logic:
  - Use a **ForEach** activity to iterate through the files.
  - Enable **parallel execution** inside ForEach.
  - ADF will process multiple files concurrently, greatly accelerating throughput.

---

### c. Failure Notification and Alerts

To ensure quick response to failures:

1. Go to the **Monitor** tab in ADF.
2. Click **New Alert Rule**.
3. Provide details:
   - **Name:** `failAlert1`
   - **Severity:** Choose appropriate level.
4. In **Target criteria**:
   - Add metrics such as:
     - **Activity type(s)**
     - **Name(s)**
     - **Pipeline name(s)**
     - **Failure type(s)**
5. Define **condition**, **threshold**, **period**, and **frequency** for triggering alerts.
6. Configure **Notifications**:
   - Add Email, SMS, or both.
   - Provide recipient details.
   - Save to activate the alert.

> Example:  
> For the Copy Activity inside the ForEach (as in Question 4), include the **ForEach activity** itself in alert criteria since a child failure causes the parent to fail as well.

---

By combining **Retry Policies**, **Parallelism**, and **Automated Failure Alerts**, you can design a highly **fault-tolerant**, **scalable**, and **resilient** ADF pipeline capable of processing billions of records daily.

## 6. Your ADF Pipeline is Running Slow — How to Identify and Fix Performance Bottlenecks

### a. Initial Step: Monitor and Identify
- As a Data Engineer, the **first step** is to monitor the pipeline and **identify which activity** is causing delays.
- Use the **Monitor** tab in ADF to review activity runtimes and pinpoint the bottleneck.

### b. Common Root Causes and Fixes
1. **Slow Data Flows**
   - Issue: Data flow execution takes longer than expected.  
   - Fix: **Prune the data** in the early stages (apply filters or reduce columns before transformation).

2. **Slow Copy Activity**
   - Issue: Copying data from **on-premises to Azure** takes too long due to **Self-Hosted Integration Runtime (IR)** throttling.
   - Fix: Scale up the IR VM, allocate more CPU/memory, or distribute load across multiple IR nodes.

---

## 7. Implementing Incremental Data Load in ADF for Large Transactional Databases

### a. Watermark Approach — Step-by-Step

1. **Identify a Watermark Column**  
   Choose a column that updates with each new or modified record, e.g., `lastModifiedDate` or an increasing `ID`.

2. **Store the Watermark Value**  
   Maintain the last processed watermark value in a **control table** or **pipeline variable**.

3. **Filter Data**  
   - Use a **Lookup Activity** to fetch the last watermark from the control table.
   - Use a **Copy Activity** with a dynamic query:
     ```sql
     SELECT * FROM SourceTable WHERE lastModifiedDate > @{activity('Lookup').output.firstRow.LastWatermark}
     ```

4. **Update the Watermark**  
   After the successful copy, use a **Stored Procedure** or **Script Activity** to update the watermark table with the new maximum watermark value.

---

### b. CDC (Change Data Capture) Approach

1. **Enable CDC on Source**  
   Activate CDC on the source database and required tables — this tracks inserts, updates, and deletes in system tables.

2. **Use ADF’s Built-in CDC Feature**  
   ADF can directly read from CDC system tables to identify changed data automatically.

3. **Extract and Apply Changes**  
   - Use the `__$operation` field to determine change type:
     - `1`: Delete  
     - `2`: Insert  
     - `3`/`4`: Update
   - Apply logic accordingly in your sink.

4. **Manage Checkpoints Automatically**  
   ADF maintains **checkpoints** (like LSN) to ensure only new changes since the last run are processed.

---

## 8. Processing a JSON File with Unknown Structure in ADF

- Enable **Schema Drift** to handle schema evolution dynamically.
- In a **Mapping Data Flow**:
  1. Turn on **Debug Mode** to preview the schema.
  2. Apply **Flatten Transformation** to denormalize nested JSON structures for processing.

---

## 9. Implementing Real-Time Streaming Pipeline in ADF

### Scenario
A new file arrives in an ADLS container, and you need to **copy it automatically** to another container.

### Solution: Event-Based Trigger
- Use an **Event-Based Trigger** to automatically start the pipeline whenever a new file arrives.

### Configuration Steps
1. Register **Event Grid Services** under **Subscriptions**.
2. Create a new **Event-Based Trigger** in ADF:
   - Trigger type: **Blob Created**
   - Linked service: Connect to the ADLS container.
   - Action: Launch the desired pipeline (e.g., Copy Activity).

> **Note:** Registration of **Event Grid Services** is mandatory before using Event-Based Triggers.


## 10. How Would You Load Data to Different Locations Based on File Name in ADF

### a. Get File List (Get Metadata Activity)
- Start the pipeline with a **Get Metadata** activity.
- Configure it to retrieve **Child Items** from the source folder in your storage account.
- This activity outputs a list of all files present in the specified source location.

### b. Iterate Over Files (For Each Activity)
- Pass the output of the **Get Metadata** activity to a **For Each** activity.
- This allows the pipeline to **loop through each file** found in the source folder and process them one by one.

### c. Conditional Routing (If Condition Activity)
- Inside the **For Each** loop, add an **If Condition** activity.
- Use expressions to check the file name for specific patterns or keywords to determine where each file should go.
- Example expressions:

`@contains(item().name, 'sales')`
or 
`@startsWith(item().name, 'marketing')`

**Based on the Condition**

- If the file name contains **"sales"**, route it to a **Sales** destination folder.  
- If it contains **"marketing"**, route it to a **Marketing** destination folder.

---

### d. Load Data (Copy Data Activity)

- Each branch of the **If Condition** activity will contain a **Copy Data** activity.  
- Configure as follows:
  - **Source Dataset:** The current file being processed (`@item().name`)  
  - **Sink Dataset:** The target destination based on the file name (e.g., Sales or Marketing folder)
- The **Copy Data** activity will copy files to their respective destinations according to the defined condition logic.

---

### Example Pipeline Flow

1. **Get Metadata Activity** → Retrieve list of files from the source folder.  
2. **For Each Activity** → Iterate through each file.  
3. **If Condition Activity** → Check the file name for keywords like “sales” or “marketing.”  
4. **Copy Data Activity** → Load the file into the corresponding destination folder based on the condition.

## 11. Storing All File Names in a Variable & Counting Files in a Folder

### a. Get File List
- Use a **Get Metadata** activity configured with **Child Items**.  
- This returns a single array containing the names of all files and subfolders in the specified location.

### b. Iterate Over Files
- Use a **For Each** activity to loop through the array returned by Get Metadata.  
- Inside the loop, you can process each file individually.

### c. Store File Names
- Use **Set Variable** and **Append Variable** activities to store and manipulate the list of file names:
  - **Set Variable:** Assign an entire list of file names to a pipeline variable (e.g., `fileNamesArray`).  
  - **Append Variable:** When building the list dynamically within a loop, add one filename at a time.  
    - Useful if looping through multiple folders and combining all filenames into a single list.

### d. Count the Files
- Use a second **Set Variable** activity to store the count of files.
- Create a variable of type **Integer** (e.g., `fileCount`).
- Use the following expression in the **Value** field:

`@length(variables('fileNamesArray'))`

## 12. Pipeline Design in ADF to Copy Only Files Not Present in Destination

### a. Get Source File List
- Use a **Get Metadata** activity to retrieve a list of all files from the **source folder**.

### b. Get Destination File List
- Use a second **Get Metadata** activity to retrieve all files from the **destination folder**.  
- Store this list in a **pipeline variable** (e.g., `destinationFiles`).

### c. Iterate and Compare
- Use a **For Each** activity to loop through the array of source files obtained from the first **Get Metadata** activity.

### d. Implement the Condition
- Inside the loop, add an **If Condition** activity to check if the current file from the source exists in the destination variable.  
- Expression example:

`@not(contains(string(variables('destinationFiles')), item().name))`

### e. Copy the File

- Place a **Copy Data** activity inside the **True** block of the **If Condition**.  
- This ensures that only files **not present in the destination** are copied from the source.



## 13. Calling Secrets Stored in Key Vault in ADF

### Creation of Secret in Key Vault

1. **Create Key Vault**
   - In the Azure Portal, search for **"Key Vaults"** and click **Create**.
   - Fill in the basic details: Subscription, Resource Group, Name (globally unique), and Region.

2. **Assign Key Vault Administrator Role**
   - Navigate to **IAM** and assign the **Key Vault Administrator** role to the ID authorized to create secrets.

3. **Create a Secret**
   - Go to **Secrets** inside the Key Vault and create a new secret.

4. **(Optional) Provide 3rd Party Access**
   - To allow access to external services like Databricks:
     - Key Vault → **Access Configuration** → Choose **Vault Access Policy** to grant data plane access.  
   - By default, **Azure RBAC** is used. This step can also be configured during Key Vault creation.

5. **Set Access Policies for ADF**
   - Navigate to **Access Policies** or **Access Control (IAM)** in the Key Vault.
   - Grant your **Azure Data Factory** **Get** and **List** permissions for secrets via its **Managed Identity**.

---

### Calling the Secret in ADF

#### 1. Standard Method (Recommended)
- **Create a Key Vault Linked Service**
  - Go to **Manage → Linked Services → New** in your ADF instance.
  - Choose **Azure Key Vault** and point it to your Key Vault.

- **Reference the Secret in Another Linked Service**
  - When creating a linked service for a data store (e.g., SQL Database), select the option to **Reference a Secret** for sensitive fields like passwords.
  - Choose the Key Vault Linked Service created above and provide the **secret name** (e.g., `MyDatabasePassword`).

---

#### 2. Advanced Method (Using Web Activity)
- **Use Web Activity to Retrieve Secret Dynamically**
  1. Add a **Web Activity** to your pipeline.
  2. Configure REST API call to Key Vault:
     ```
     https://<your_key_vault_name>.vault.azure.net/secrets/<your_secret_name>?api-version=7.4
     ```
  3. **Authentication:** Managed Identity  
     **Resource:** `https://vault.azure.net`
  4. **Method:** GET

- **Use the Secret in Subsequent Activities**
  - The secret value will be in the Web Activity output.  
  - Reference it using an expression:
    ```text
    @activity('Web1').output.value
    ```
## 14. Avoiding Out-of-Memory (OOM) Errors in ADF Pipelines Processing Large Datasets

### a. Optimize Your Integration Runtime (IR)
- The **IR** is the compute engine for your pipeline. Proper configuration ensures enough memory for large workloads.

#### Mapping Data Flows
- Scale up the **Azure IR** used by your data flow.
- Increase **Core Count** and choose **Memory Optimized** compute type.
- This provides more resources to the underlying Spark cluster for large transformations.

#### Self-Hosted IR
- For on-premises data processing, add more nodes to your **Self-Hosted IR**.
- Distributes workload across machines to prevent a single node from running out of memory.

---

### b. Partition Large Datasets
- Breaking data into smaller chunks prevents OOM issues.

#### Files
- Use **Get Metadata** to retrieve all files in a folder.
- Use a **For Each** loop to process files individually via **Copy Data** or **Mapping Data Flow**.
- In **Data Flow source options**, set partitioning (single, multiple, or none) to enable parallel processing.

#### Database Tables
- Implement **iterative range copy** for large tables:
  - Use a **Lookup** activity to find min and max values of a key column (e.g., ID or date).
  - Use a **For Each** loop to copy smaller chunks (e.g., 100,000 rows at a time) with a **parameterized query**.

---

### c. Tune Activity-Specific Settings

#### Mapping Data Flow
- Be cautious with **Broadcast joins or lookups**:
  - Broadcasting improves performance but can cause OOM if the dataset is too large.
  - Turn off broadcasting for large streams if necessary.
- Prune or filter data early in the flow to reduce memory usage.

#### Copy Data Activity
- For extremely large single files (XML, JSON, Excel), OOM can occur.
- Use the **Binary Copy** feature to move files without loading content into memory.


## 15. Ensuring Idempotency in ADF Pipelines

**Idempotency** ensures that running the same process multiple times with the same input produces the **same result** as running it once, preventing duplicate processing of records. This can be achieved using **UPSERT** on file data, often with **inline datasets** (e.g., Delta Lake) in Mapping Data Flows.

---

### a. Source Transformations
- Configure **at least two source transformations**:
  1. **Incoming Data Source** – your new data.
  2. **Destination Data Source** – existing data in the target.
- Use **inline datasets** to define connection details and schema directly in the data flow.

---

### b. Exists Transformation
- Use an **Exists transformation** to implement idempotency.
- It acts like a `WHERE EXISTS` clause in SQL:
  - Checks if rows from the **incoming data stream** (left stream) exist in the **destination stream** (right stream) based on a key (e.g., primary key).

---

### c. Conditional Split
- Chain a **Conditional Split** transformation after the Exists transformation.
- Split the data into two streams:
  1. **New Records** – rows where the key does **not exist** in the destination; these will be inserted.
  2. **Existing Records** – rows where the key **does exist**; these will be updated.

---

### d. Alter Row & Sink
- Connect an **Alter Row** transformation to each stream from the Conditional Split:
  - **New Records** → set **Insert** policy.
  - **Existing Records** → set **Upsert** or **Update** policy.
- Connect both Alter Row outputs to a single **Sink** transformation.
  - Configure the sink to allow **Insert** and **Upsert/Update** policies.
  - Define the **key columns** for correct record matching.

This approach ensures that **duplicate processing of the same records does not occur**, maintaining idempotency in your ADF pipeline.


## 16. Enforcing File Processing Order in ADF

- To process files in a **specific order** (e.g., `sales.csv` → `product.csv` → `cust.csv`), use the **Sequential** option in the **For Each** activity.  
- This ensures that files are processed **one after another** rather than in parallel.

---

## 17. Processing Two Files with Different Schemas Using a Single Copy Data Activity

### a. Source Dataset
- Create a **source dataset** that takes the **file name as a parameter**.
- In the **file path settings**, use a dynamic expression:
  ```text
  @item().name

This passes the current file name from the For Each loop to the Copy Data activity.

### b. Dynamic Mapping

#### Delete Static Mapping
- In the **Mapping** tab of the Copy Data activity, remove existing column mappings.  
- You can either delete the entire mapping table or remove columns manually.

#### Enable Schema Drift
- In the **source settings** of the Copy Data activity, enable **Schema Drift**.  
- This allows the activity to **dynamically handle different schemas** at runtime.

#### Map Dynamically
**Option 1: Auto-Mapping**
- Leave the **Mapping** tab blank.  
- ADF will automatically map columns from source to sink based on matching names.

**Option 2: Explicit Dynamic Mapping**
- If column transformation or renaming is required, define **dynamic mappings** using expressions.  
- For simple copy scenarios, leaving the mapping blank is the most efficient approach.
