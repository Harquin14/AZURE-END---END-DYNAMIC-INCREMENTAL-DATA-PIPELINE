# AZURE-END---END-DYNAMIC-INCREMENTAL-DATA-PIPELINE

Medallion Architecture (Bronze → Silver → Gold) with Incremental Loading






![incremental sales](https://github.com/user-attachments/assets/d4257664-6017-4767-90f1-570a0ada0384)



# This Azure data pipeline demonstrates how I built a scalable data engineering workflow using:

  ✔ Azure SQL Database
  
  ✔ Azure Data Lake Gen2
  
  ✔ Azure Data Factory
  
  ✔ Azure Databricks
  
  ✔ Azure Access Connector / Service Principal

  
  ✔ Parquet & Delta Lake formats




Below is a polished breakdown of the entire process:

![WhatsApp Image 2025-11-03 at 18 20 36_76d793b8](https://github.com/user-attachments/assets/3d005566-650f-454f-be88-601121b78cc6)



# 1. Source & Ingestion (Azure Data Factory) 


## Data Source

The raw dataset is hosted on GitHub. 


Azure Data Factory (ADF) pulls the data dynamically using parameters, allowing for incremental loads.


Landing in Azure SQL


Ingested data is copied into Azure SQL Database, where it becomes the official source system.


This design supports efficient querying and incremental logic using SQL expressions.



# 2. Bronze Layer – Raw Data

From Azure SQL, data is read into Data Lake Gen2 (Bronze folder).


Data is stored in Parquet format for fast reads and optimized storage.


No heavy transformations occur here—Bronze is a clean replica of the source.


# 3. Incremental Loading Pipeline (ADF)


To ensure only new/updated records are processed, an incremental ingestion pipeline was built.


Lookup Activities


✔ Last Load Lookup


    SELECT * FROM water_table;

 

water_table stores the last successful load timestamp.


✔ Current Load Lookup

    SELECT MAX(Date) AS Max_Date FROM source_table;


 
 Retrieves the latest timestamp from the source.


Copy Activity (Incremental Extract)


Data is filtered using:

    SELECT * 
    FROM source_table 
    WHERE Date > @last_load 
      AND Date <= @current_load;


Stored Procedure Activity

 
After successful loading, the pipeline executes a stored procedure:

This procedure updates the water_table with the current load timestamp.


This ensures the next pipeline run continues from where it left off.



# 4. Azure Databricks (Transformations)


Databricks connects to Data Lake using an Access Connector with permissions granted at the Workspace and Catalog level.


Workspace Setup


Created workspace


Created notebooks for Bronze → Silver → Gold processing


Registered tables in the Unity Catalog

Bronze Notebook


Reads raw Parquet data from Data Lake Gen2


Registers it in the catalog (optional)


Silver Notebook


Loads Bronze data


Applies transformations, cleaning, type casting, de-duplication


Writes back to Data Lake in Parquet format


Gold Notebook

# Loads Silver data


Applies business rules and aggregations


Writes output as Delta tables



Supports star-schema modeling (Fact + Dimensions)




![WhatsApp Image 2025-11-10 at 12 45 17_4c38ffe7](https://github.com/user-attachments/assets/bc2e0b5f-e183-4b88-97f6-ed43dd2f2f95)



#  5. Databricks Workflow (Automation)



A Databricks Workflow/Pipeline orchestrates the notebooks:

  ➡ Bronze → Silver → Gold
  ➡ Fact tables and dimension tables created
  ➡ All dependencies linked in a DAG-like view
  ➡ Ensures reliable, repeatable transformations



# 6. File Formats


Bronze: Parquet


Silver: Parquet


Gold: Delta Lake (ACID-compliant, versioned)


# Summary


This project implements a fully scalable data engineering pipeline on Azure using:


Dynamic pipelines in ADF


Incremental loading


Data Lake storage layers


PySpark transformations


Hive & Catalog integration

Delta Lake for analytics

Fact & Dimension modeling
