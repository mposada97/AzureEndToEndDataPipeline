# Azure End-To-End Data Pipeline

## Scope
The goal of this project is to demonstrate the implementation of a modern data engineering workflow by integrating an on-premises SQL Server database with a cloud-based analytics solution. Using Azure as the cloud platform, this project showcases how data can be seamlessly ingested, processed, and visualized to support business intelligence needs. The final dataset is served to end-users via Power BI for reporting and analytics.

This project leverages the AdventureWorks LT17 sample database as a source to illustrate key concepts and capabilities, focusing on the technical architecture and process rather than specific data insights. The architecture includes Azure services such as Data Factory, Data Lake, Synapse Analytics, and integrates authentication and security through Azure Key Vault. By completing this workflow, I aim to highlight my expertise in cloud-based data engineering and my ability to build scalable, end-to-end data pipelines.

## Architechture
![image](https://github.com/user-attachments/assets/9910cb0a-9cba-4a40-a807-28b1110dc37a)

The architecture begins by ingesting data from the on-premises SQL Server database using Azure Data Factory into a Data Lake Gen2, following the Bronze, Silver, and Gold tiering pattern, making simple transformations in between tiers using Azure Databricks. Finally, Azure Synapse Analytics with serverless SQL pools is used to create views on the Gold tier, enabling integration with Power BI for visualization and reporting.

## Data Pipelines
First a pipeline was created using ADF consuming the data from the On prem SQL server and getting it up to the gold tier level in the datalake. A schedule trigger was added to run the pipeline daily. 
![image](https://github.com/user-attachments/assets/6f23fad9-2bec-4d8e-b210-b3c2960a496e)

A second pipeline was built on synapse, this pipeline creates views out of the tables in the datalake so that they can be consumed by powerBI. 
![image](https://github.com/user-attachments/assets/39445f70-0f19-42a1-bf46-8e6a53b28f1d)


## Ingestion
To enable data ingestion from the on-premises SQL Server database to the cloud, I installed a Self-Hosted Integration Runtime on the same machine as the SQL Server. This runtime acts as a secure bridge, allowing Azure Data Factory to access the on-premises data.

To automate the ingestion process, I used a Lookup activity in Azure Data Factory with the following query to dynamically retrieve all tables under the SalesLT schema:

```sql
SELECT
    s.name AS SchemaName,
    t.name AS TableName
FROM sys.tables t 
INNER JOIN sys.schemas s ON t.schema_id = s.schema_id
WHERE s.name = 'SalesLT'
The Lookup activity provided a list of tables, which was passed into a For Each activity. Inside the loop, a Copy activity was used to load each table from SQL Server into the Bronze tier of the Data Lake Gen2, preserving the raw data for further transformations.
```

