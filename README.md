🚀🚀 Project Overview
This repository demonstrates the successful implementation of a robust, incremental data loading pipeline for retail sales data. The solution utilizes Azure Data Factory (ADF) to extract data from a SQL database, process it through multiple refinement layers, and load the final aggregated data back into a target SQL DB.

The key innovation is the use of a dynamic watermark pattern and a central metadata table to manage incremental loads across 13 distinct tables, ensuring data integrity and optimizing transfer costs.

🎯 Architecture and Data Flow
The pipeline follows the industry-standard Medallion Architecture (Bronze, Silver, Gold layers) using ADF activities.

Layer	Description	Key Technologies
Bronze (Raw)	Incremental data extracted from the Source SQL DB using a dynamic query filter and loaded to Azure Data Lake Storage (ADLS) Gen2.	ADF Copy Activity
Silver (Pre-Processed)	Raw data is cleansed, files are standardized (e.g., Parquet), and essential transformations (null handling, column addition) are applied.	ADF Data Flow
Gold (Curated)	Business logic, aggregations, and key performance indicators (KPIs) are calculated. The final, business-ready data is then loaded to the Target SQL DB.	ADF Data Flow

Export to Sheets
🛠️ Key Technical Components
The project's complexity and efficiency are managed by these core components, many of which are visible in your reference images:

1. Dynamic Watermark Management (Metadata Table)
Component: ADF_Metadata SQL Table
Purpose: Centralized control for all 13 source tables. It stores the Table_ID, Table_Name, the Watermark_Column, and the Last_Load_Date (the previous end date for the load).
Initial State: The table is initialized with a starting date (e.g., '1900-01-01') to perform a full initial load.

ADF Execution: A Lookup Activity reads the Last_Load_Date and Watermark_Column dynamically based on the pipeline's table_id parameter.

2. Incremental Load Logic
Activity: Copy Data (Source Stage)
Logic: The source dataset uses a dynamic query to filter records based on the previous watermark (StartDate) and the current execution time (CurrentDate).
Filter Query Structure:

%SQL
SELECT * FROM @{TableName} 
WHERE @{WatermarkColumn} > '@{StartDate}' 
  AND @{WatermarkColumn} <= '@{CurrentDate}'
This ensures the pipeline only ingests newly added data, minimizing data transfer and execution time.

3. Pipeline Orchestration
The project uses a Master Pipeline (to iterate over the 13 tables) that calls a Child Pipeline for the core Extract, Transform, and Load (ETL) logic.

Pipeline Variables: Variables (StartDate, CurrentDate, TableName, WatermarkColumn) are dynamically set from the Lookup output and UTCnow() function, making the Child Pipeline generic and reusable for all 13 tables.

4. Watermark Update (Stored Procedure)
Activity: Stored Procedure Activity

Component: UpdateWatermark SQL Stored Procedure

Function: This activity is the final step of the Child Pipeline. It commits the new high watermark (CurrentDate) back into the ADF_Metadata table for the processed table ID, preparing the system for the next incremental run. This ensures that the next execution picks up exactly where the current one left off.

📂 Source Data (13 Tables)
The pipeline manages incremental loads for a comprehensive retail schema including: Order, Customer, Inventory, Payment, Review, Shipment, and supporting dimension tables.
