+++
title = "Azure Data Factory"
linkTitle = "ADF"
type = "chapter"
weight = 2
+++

{{% children depth="999" %}}

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


