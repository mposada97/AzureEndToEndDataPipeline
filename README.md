# Azure End-To-End Data Pipeline

## Scope
The goal of this project is to demonstrate the implementation of a modern data engineering workflow by integrating an on-premises SQL Server database with a cloud-based analytics solution. Using Azure as the cloud platform, this project showcases how data can be seamlessly ingested, processed, and visualized to support business intelligence needs. The final dataset is served to end-users via Power BI for reporting and analytics.

This project leverages the AdventureWorks LT17 sample database as a source to illustrate key concepts and capabilities, focusing on the technical architecture and process rather than specific data insights. The architecture includes Azure services such as Data Factory, Data Lake, Synapse Analytics, and integrates authentication and security through Azure Key Vault. By completing this workflow, I aim to highlight my expertise in cloud-based data engineering and my ability to build scalable, end-to-end data pipelines.

## Architechture
![image](https://github.com/user-attachments/assets/9910cb0a-9cba-4a40-a807-28b1110dc37a)

The architecture begins by ingesting data from the on-premises SQL Server database using Azure Data Factory into a Data Lake Gen2, following the Bronze, Silver, and Gold tiering pattern, making simple transformations in between tiers using Azure Databricks. Finally, Azure Synapse Analytics with serverless SQL pools is used to create views on the Gold tier, enabling integration with Power BI for visualization and reporting.

The following services were created for this project under one resource group:
![image](https://github.com/user-attachments/assets/feffbe73-ba3e-4b0b-b9cb-9aacb10503e2)


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
```
The Lookup activity provided a list of tables, which was passed into a For Each activity. Inside the loop, a Copy activity was used to load each table from SQL Server into the Bronze tier of the Data Lake Gen2, preserving the raw data for further transformations.
![image](https://github.com/user-attachments/assets/584b5b5c-93db-4f0e-ae95-a832419e0c14)

Within the For Each activity, the retrieved table and schema names from the Lookup activity were dynamically passed to the Copy activity to determine file names and folder paths. The dataset in the Copy activity utilized these values to save the data in the Bronze tier of the Data Lake Gen2, ensuring proper organization.

Each table's data was written into a folder structure based on its schema and table name, using the following dynamic expressions:

- Folder Path: @{concat(dataset().schemaname, '/')}
- File Name: @{concat(dataset().tablename, '.parquet')}
  
This structure allowed for a clear, logical organization of the raw ingested data in Parquet format with Snappy compression, facilitating easy navigation and further transformations.

Its also important to mention that by default the copy activity will overwrite files if they already exist, to have different versions of files folder with date information can also be added, or date information can be added to the file name

## Transformation
The notebooks used for data transformations can be found in the repository linked to this README. The transformation pipeline follows a structured approach, moving data through Bronze, Silver, and Gold tiers in the data lake.

Bronze Tier:
- Data is ingested as-is from the source and saved in Parquet format in the Bronze tier.
- This layer acts as a raw storage zone, preserving the data in its original state for traceability and auditing.

Silver Tier:
- Minor transformations are applied in the Silver tier, such as data cleaning, normalization, and deduplication.
- This layer prepares the data for further analysis and aligns it with standard business rules.

Gold Tier:
- Final transformations are performed to organize the data into a Star Schema, facilitating analytics and reporting.
- The Gold tier is saved in Delta format, enabling efficient querying and integration with BI tools.
This tiered approach demonstrates the best practices for managing and transforming data in a cloud-based data lake architecture.

## Loading Data into Synapse
The final step in the pipeline involves creating views in Azure Synapse serverless SQL pools to expose the transformed data in the Gold tier for analytics and reporting. This process was automated using a For Each activity in Azure Data Factory, which iterates over the list of tables in the Gold tier.

A Get Metadata activity was used to retrieve all child items (table names) from the Gold tier folder in the Data Lake.
The output of the metadata activity was passed into the For Each activity, which executed a stored procedure for each table. The stored procedure dynamically creates views in the Synapse serverless SQL pool.
Here’s the code for the stored procedure used:

```sql
USE gold_db
GO

CREATE OR ALTER PROC CreateSQLServerlessView_gold @ViewName nvarchar(100)
AS
BEGIN

DECLARE @statement VARCHAR(MAX)

    SET @statement = N'CREATE OR ALTER VIEW ' + @ViewName + ' AS
        SELECT *
        FROM 
            OPENROWSET(
            BULK ''https://dlgen2mposada.dfs.core.windows.net/gold/SalesLT/' + @ViewName + '/'',
            FORMAT = ''DELTA''
        ) as [result]
    '   
EXEC (@statement)

END
GO
```

## Data Serving / Data Visualization
Finally the serverless sql pool database called gold_db was connected in powerBI. A simple dashboard was created to demonstrate the functionality of this workflow.
![image](https://github.com/user-attachments/assets/27f0fc47-015d-44e5-a07f-3cf9cf7a7be2)

## Next Steps
This project demonstrates an end-to-end workflow for integrating on-premises data into a cloud-based analytics solution using Azure. While the implementation showcases key capabilities, there are opportunities to enhance and optimize the solution further:

Incremental Data Ingestion and Transformations:
- Currently, both the ingestion and transformations are done in overwrite mode. This approach, while simple, is not optimal for managing large datasets or controlling cloud costs.
- To improve efficiency, incremental methods should be implemented, where only new or changed data is processed and ingested. This would significantly reduce the volume of data being handled.
  
Slowly Changing Dimensions (SCD Type 2):
- The Gold tier is currently modeled with a basic star schema. A logical next step is to refine dimension tables using SCD Type 2 techniques to track historical changes and maintain data accuracy over time. This is particularly useful for building robust analytical models.
  
Partitioning for Performance:
- Partitioning data in the Silver and Gold tiers can improve query performance, especially when dealing with large datasets. By organizing the data based on frequently queried columns (e.g., date or region), you can reduce the amount of data scanned during queries.



