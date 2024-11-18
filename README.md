# Azure End-To-End Data Pipeline

## Scope
The goal of this project is to demonstrate the implementation of a modern data engineering workflow by integrating an on-premises SQL Server database with a cloud-based analytics solution. Using Azure as the cloud platform, this project showcases how data can be seamlessly ingested, processed, and visualized to support business intelligence needs. The final dataset is served to end-users via Power BI for reporting and analytics.

This project leverages the AdventureWorks LT17 sample database as a source to illustrate key concepts and capabilities, focusing on the technical architecture and process rather than specific data insights. The architecture includes Azure services such as Data Factory, Data Lake, Synapse Analytics, and integrates authentication and security through Azure Key Vault. By completing this workflow, I aim to highlight my expertise in cloud-based data engineering and my ability to build scalable, end-to-end data pipelines.

## Architechture
![image](https://github.com/user-attachments/assets/9910cb0a-9cba-4a40-a807-28b1110dc37a)

The architecture begins by ingesting data from the on-premises SQL Server database using Azure Data Factory into a Data Lake Gen2, following the Bronze, Silver, and Gold tiering pattern, making simple transformations in between tiers using Azure Databricks. Finally, Azure Synapse Analytics with serverless SQL pools is used to create views on the Gold tier, enabling integration with Power BI for visualization and reporting.

##Ingestion

