+++
title = "Azure Data Factory"
linkTitle = "ADF"
type = "chapter"
weight = 2
+++

## 1. Introduction
- Overview of Azure Data Factory
- Data Integration in Cloud
- ADF Architecture
- ADF Components (Pipelines, Activities, Datasets, Linked Services, Triggers)
- ADF vs SSIS vs Synapse Pipelines

## 2. ADF Basics
- Creating an ADF Instance
- ADF Studio Overview
- Linked Services
- Datasets
- Pipelines
- Activities
- Triggers (Schedule, Tumbling Window, Event-based, Manual)

## 3. Data Movement
- Copy Activity
- Integration Runtime (IR) Types
  - Azure IR
  - Self-hosted IR
  - Azure-SSIS IR
- Data Movement Performance
- Parallelism & Batch Copy

## 4. Data Transformation
- Mapping Data Flows
- Wrangling Data Flows
- Data Flow Debugging
- Joins, Aggregations, Filters, Derived Columns
- Surrogate Keys & Window Functions
- Data Flow Performance Tuning

## 5. Orchestration
- Control Activities (Execute Pipeline, ForEach, If Condition, Until, Switch, Wait)
- Parameterization
- Variables in Pipelines
- Expressions & Functions
- Dynamic Content
- Error Handling & Retry Policies

## 6. Data Sources & Destinations
- Azure Blob Storage
- Azure Data Lake Storage (ADLS)
- Azure SQL Database
- Azure Synapse Analytics
- On-Premises SQL Server
- Cosmos DB
- REST API
- SAP, Oracle, Teradata, Snowflake
- Amazon S3
- Google Cloud Storage

## 7. Integration with Other Azure Services
- Azure Key Vault Integration
- Azure Monitor & Log Analytics
- Power BI Integration
- Event Grid & Event Hub
- Logic Apps & Functions
- Azure Machine Learning

## 8. CI/CD & DevOps with ADF
- Source Control with Git (GitHub, Azure Repos)
- Branching & Collaboration
- Publishing & ARM Templates
- Continuous Integration & Deployment
- Automated Testing
- Environment Promotion (Dev → Test → Prod)

## 9. Security
- Managed Identity in ADF
- Service Principals
- Access Control (RBAC)
- Data Encryption
- Network Security (VNET Integration, Private Endpoints)
- Credential Management with Key Vault

## 10. Monitoring & Troubleshooting
- Pipeline Monitoring
- Activity Run Details
- Debugging Failures
- Alerts & Notifications
- Logging with Log Analytics
- Performance Optimization

## 11. Advanced Features
- Incremental Data Loading (Watermarking, Change Data Capture)
- Slowly Changing Dimensions (SCD) Implementation
- Parameterized Pipelines & Templates
- Global Parameters
- Data Lineage & Impact Analysis
- ADF REST API
- Managed VNET Data Integration

## 12. Best Practices
- Designing Efficient Pipelines
- Naming Conventions
- Error Handling Framework
- Cost Optimization in ADF
- Scalability & Performance Tuning
- Version Control Strategies

## 13. Real-World Use Cases
- Building an ETL Pipeline with ADF
- Data Lake to Synapse ETL
- Real-Time Event Processing
- Hybrid Data Integration (On-Prem + Cloud)
- Machine Learning Model Deployment Orchestration

## 14. Capstone Project
- End-to-End Data Warehouse Load using ADF
- Incremental Data Pipeline with Delta Processing
- Enterprise Data Integration with ADF and Synapse



ADF is the data movement and transformation engine that takes raw data and converts it into structured, consumable insights for decision-making.


Features of Azure Data Factory
Data Compression: During the Data Copy activity, it is possible to compress the data and write the compressed data to the target data source. This feature helps optimize bandwidth usage in data copying.

Extensive Connectivity Support for Different Data Sources: Azure Data Factory provides broad connectivity support for connecting to different data sources. This is useful when you want to pull or write data from different data sources.

Custom Event Triggers: Azure Data Factory allows you to automate data processing using custom event triggers. This feature allows you to automatically execute a certain action when a certain event occurs.

Data Preview and Validation: During the Data Copy activity, tools are provided for previewing and validating data. This feature helps you ensure that data is copied correctly and written to the target data source correctly.

Customizable Data Flows: Azure Data Factory allows you to create customizable data flows. This feature allows you to add custom actions or steps for data processing.

Integrated Security: Azure Data Factory offers integrated security features such as Entra ID integration and role-based access control to control access to dataflows. This feature increases security in data processing and protects your data.

Monitor
After you have successfully built and deployed your data integration pipeline, providing business value from refined data, monitor the scheduled activities and pipelines for success and failure rates. Azure Data Factory has built-in support for pipeline monitoring via Azure Monitor, API, PowerShell, Azure Monitor logs, and health panels on the Azure portal.

Top-level concepts
An Azure subscription might have one or more Azure Data Factory instances (or data factories). Azure Data Factory is composed of the following key components:
Pipelines
Activities
Datasets
Linked services
Data Flows
Integration Runtimes

These components work together to provide the platform on which you can compose data-driven workflows with steps to move and transform data.

Pipeline:

The benefit of this is that the pipeline allows you to manage the activities as a set instead of managing each one individually. The activities in a pipeline can be chained together to operate sequentially, or they can operate independently in parallel.

Activities:

Activities represent a processing step in a pipeline. For example, you might use a copy activity to copy data from one data store to another data store.
Data Factory supports three types of activities:
1.Data movement activities
2.Data transformation activities
3.Control activities

Datasets:

Datasets represent data structures within the data stores, which simply point to or reference the data you want to use in your activities as inputs or outputs.

Linked services:

Linked services are define the connection information that's needed for Data Factory to connect to external resources.
To represent a data store that includes, but isn't limited to, a SQL Server database, Oracle database, file share, or Azure blob storage account. For a list of supported data stores
