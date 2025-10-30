# Urban Mobility — Yellow Taxi Analysis (Databricks)

<p align="center">
   <img src="img/yellow_taxi.png" alt="Yellow Taxi" width="1000" height="700" />
</p>

## Project overview

This project implements a data processing pipeline in Azure Databricks to analyze large public Yellow Taxi trip datasets (Parquet files) for the years 2024 and 2025. The goal is to produce a set of monthly indicators describing mobility patterns and service usage so they can be consumed by BI or downstream applications.

Key analytics delivered by the solution:
- Top 10 departure zones per month
- Average trip duration per month
- Average trip distance by payment type per month
- Average fare amount by passenger count per month
- Total tip amount per month

The attached solution notebook demonstrates reading Parquet files, extracting metadata (year/month), computing aggregations using Spark SQL and DataFrame APIs, writing results as Parquet/Delta, and persisting final tables to Azure SQL.

## File included

- `notebook/yellow_taxi_analysis_databricks.ipynb` — the Databricks-focused notebook that contains the production logic for reading parquet files, extracting year/month from file names, computing metrics and writing outputs (see `notebook/yellow_taxi_analysis_databricks.ipynb`).

## Data

- Source: Yellow Taxi trip data in Parquet format.
- The parquet files are expected to follow a naming pattern like `yellow_tripdata_YYYY-MM.parquet` (e.g. `yellow_tripdata_2024-01.parquet`).
- The pipeline reads all parquet files in a source folder and extracts the year and month from the file path or file name to compute monthly aggregations.

Typical columns used in the analysis :
- `PULocationID` — pickup location (used to identify departure zones)
- `trip_distance` — distance of the trip
- `tpep_pickup_datetime` / `tpep_dropoff_datetime` — used to compute trip duration
- `payment_type` — payment method (card, cash, etc.)
- `passenger_count` — number of passengers
- `total_amount` — total fare
- `tip_amount` — tip value


## High-level architecture

1. Databricks cluster (Spark) reads Parquet files from DBFS (or mounted storage such as ADLS Gen2 / Blob Storage).
2. Notebook extracts year/month metadata per file and normalizes/cleans columns as needed.
3. Aggregations and transforms are computed using Spark SQL and DataFrame API (distributed processing).
4. Results are written to outputs as Parquet/Delta in DBFS (or a volume path) and optionally persisted to Azure SQL via JDBC for downstream consumption.

## Prerequisites

- Azure subscription with a Databricks workspace (or another Spark environment that supports Spark 3.x / Spark 4.x behavior used in notebook).
- Cluster with Spark runtime compatible with your PySpark version used in the notebook.
- Parquet dataset uploaded to DBFS or accessible storage mounted to Databricks (path used in the notebook: `/Volumes/workspace/default/yellow_tripdata/` — update to match your environment).
- Databricks secrets configured for database credentials (if you want to write to Azure SQL). The notebook expects secret scope `my-secrets` with keys `sql-url`, `sql-user`, `sql-pass` as an example — change to your scope/keys.
- JDBC driver for Azure SQL present on cluster (or use the built-in driver via Maven coordinate in cluster libraries). Example driver: `com.microsoft.sqlserver:mssql-jdbc:11.2.1.jre11`.


## How to run (Databricks)

1. Upload the Parquet files into DBFS or mount your storage account to Databricks. Place files under a folder such as `/Volumes/workspace/default/yellow_tripdata/`.
2. Import the notebook (`notebook/yellow_taxi_analysis_databricks.ipynb`) into your Databricks workspace.
3. Create or start an interactive cluster. Attach the notebook to that cluster.
4. Set up secrets (Databricks CLI or UI) if you want to write to Azure SQL. Example:

   - Secret scope name: `my-secrets`
   - Keys: `sql-url`, `sql-user`, `sql-pass`

5. Open the notebook and run cells in order. Typical flow in the notebook:
   - Read parquet files into a DataFrame (the notebook demonstrates reading the full folder).
   - Add a `source_file` / `year` / `month` column by extracting from file path.
   - Compute aggregations using Spark SQL or DataFrame transformations.
   - Write intermediate outputs to Parquet/Delta and optionally write final tables to Azure SQL via JDBC.


## What the Databricks notebook computes (summary)

- `most_frequent_departure` — top 10 pickup zones per year/month (row_number windowed by year/month ordered by trip_count)
- `avg_trip_duration` — average trip duration per year/month
- `avg_trip_distance` — average trip distance per year/month
- `avg_trip_distance_by_payment` — average trip distance grouped by payment type per month
- `avg_fees_by_passenger_count` — average total_amount by passenger count per month
- `total_tip_paid` — total tips summed per month

Each result is displayed in the notebook (for exploration) and stored as a named DataFrame variable that can then be written to Delta / Parquet / Azure SQL.

## Troubleshooting

- If the notebook fails to start Spark locally, ensure the local Java version is compatible with PySpark used for testing (matching JDK major version recommended). Databricks runtimes handle Java configuration automatically.
- If JDBC writes fail, verify driver availability and secret values. Test connectivity from a small Spark job before running the full pipeline.

## Author

[Michae Adebayo](https://github.com/MichAdebayo)


