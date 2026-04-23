# Netflix Data Engineering Project

An end-to-end data engineering pipeline built on Microsoft Azure,
using Databricks, Azure Data Factory, and Delta Live Tables
to process Netflix dataset through a medallion architecture.

---

## Architecture Overview

GitHub / Data Lake → ADF → Bronze → Silver → Gold → Azure Synapse → Power BI

- **Source 1:** Netflix CSV files pulled from Data Lake using Databricks Autoloader
- **Source 2:** Lookup/mapping files pulled from GitHub using ADF (HTTP connector)
- **Destination:** Azure Synapse Analytics + Power BI for reporting

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| Azure Data Factory | Data ingestion from GitHub via REST API |
| Azure Data Lake Gen2 | Storage (Bronze, Silver, Gold, Raw containers) |
| Azure Databricks | Data transformation using PySpark |
| Delta Live Tables | Gold layer ETL pipeline |
| Azure Synapse Analytics | Data warehousing |
| Power BI | Reporting and visualization |
| Unity Catalog | Governance and access control |

---

## 📁 Project Structure
netflix-databricks-de-project/
│
├── notebooks/
│   ├── 1_Autoloader.py          # Bronze layer - incremental file ingestion
│   ├── 2_silver.py              # Silver layer - parameterized transformation
│   ├── 3_lookupNotebook.py      # Lookup - holds source/dest folder array
│   ├── 4_Silver.py              # PySpark transformations on movie titles
│   ├── 5_lookupNotebook.py      # Returns weekday number as parameter
│   ├── 6_falsenotebook.py       # Handles else condition (non-Sunday runs)
│   └── 7_DLT_Notebook.py        # Delta Live Tables - Gold layer
│
├── adf_pipelines/
│   ├── pipeline1.json           # GitHub ingestion pipeline (ADF)
│   └── pipeline2.json           # Weekday-based conditional pipeline
│
├── screenshots/
│   ├── 1_ADF_Pipeline_Run.png
│   ├── 2_Autoloader_Streaming.png
│   ├── 3_Pipeline_Run.png
│   ├── 4_Aggregate_Viz.png
│   ├── 5_Pipeline2.png
│   └── Arch.png
│
└── README.md


---

## Bronze Layer — Data Ingestion

### Pipeline 1 (ADF)
- Ingests lookup/mapping files from **GitHub** using HTTP connector
- Uses **parameterized + dynamic** Copy Activity
- Activities: Web (GitHubMetadata) → Set Variable → Validation → ForEach → Copy
- Data validation: only copies if "titles" file exists in raw folder

### Notebook 1 — Autoloader
- Reads new files incrementally from the **raw** container in ADLS Gen2
- Uses **Spark Structured Streaming** (readStream)
- Writes to the **bronze** container
- Two modes: Directory Listing / File Notifications

---

## Silver Layer — Transformation

### Notebook 2 — Silver (Parameterized)
- Takes source and destination folder as parameters via `dbutils.widgets`
- Processes mapping files and master data
- Values passed at runtime via Databricks Workflows

### Notebook 3 — Lookup Notebook
- Returns an array of source + destination folder paths
- Output is consumed by NB2 via Databricks Jobs (`dbutils.jobs.taskValues`)

### Notebook 4 — PySpark Transformations
Transforms `movie_titles` data with:
- ✅ Remove nulls (`withColumn`, `fillna`)
- ✅ Change data types (`cast`, `printSchema`)
- ✅ Extract values using `split()`
- ✅ Conditional column (Movie → 1, TV Show → 2) using `when/otherwise`
- ✅ Rank data using **Window functions** (`dense_rank`)
- ✅ Aggregation and writing the final data

### Notebook 5 — Weekday Lookup
- Returns today's weekday number as a parameter
- Used to conditionally trigger silver notebook

### Notebook 6 — False Notebook
- Handles the **else condition** of Pipeline 2
- Runs when today is NOT Sunday (weekday ≠ 7)

### Pipeline 2 (Databricks Workflow)
Scenario: Run silver notebook only on Sundays

---

## 🥇 Gold Layer — Delta Live Tables

### Notebook 7 — DLT Notebook
- Creates **streaming DataFrames** on top of all Silver tables
- Creates **DLT** for all 4 Silver tables
- Defines **expectations** (data quality rules):
  - `expect_all`
  - `expect_all_or_drop`
  - `expect_all_or_fail`
- Includes staging tables, transforms, and streaming views
- Run via: Jobs & Pipelines → ETL Pipeline

---

## Dataset

**Netflix Dataset** (sourced from Tableau/AppLearn sample data)

- **Main file:** movie name, release date, rating, description, ID
  - Pulled from Data Lake incrementally using Databricks Autoloader
- **Lookup tables:** Various mapping/reference files
  - Pulled from GitHub using ADF

---

## Azure Setup

### Resources (inside Resource Group)
- Azure Data Lake Gen2 (Storage Account)
  - Containers: `raw`, `bronze`, `silver`, `gold`
- Azure Data Factory
- Azure Databricks
  - Unity Catalog enabled
  - Access Connector configured
  - External Locations: Bronze, Silver, Gold, Raw
- Azure Synapse Analytics

### Databricks Setup Steps
1. Create Unity Catalog + metastore
2. Configure Access Connector (for ADLS access)
3. Create External Locations (one per container)
4. Create Compute (15.4 LTS)
5. Create notebooks and attach to cluster



## Screenshots

| Screenshot | Description |
|-----------|-------------|
| ADF Pipeline Run | Successful pipeline with ForEach + Copy activities |
| Autoloader Streaming | Input vs Processing rate dashboard |
| Pipeline Run (DBx) | Lookup + Silver notebook job run |
| Aggregate Viz | Movie vs TV Show donut chart (68% vs 32%) |
| Pipeline 2 | Weekday conditional branch logic |
| Architecture | Full medallion architecture diagram |

---

## How to Run

1. Set up Azure resources (RG, ADLS, ADF, Databricks, Synapse)
2. Configure Unity Catalog and External Locations in Databricks
3. Run **Pipeline 1** in ADF to ingest GitHub lookup files
4. Run **Notebook 1** (Autoloader) to stream raw files to Bronze
5. Run **Pipeline 1** (Databricks Workflow) for Silver transformation
6. Run **Pipeline 2** (Databricks Workflow) for conditional Silver run
7. Run **DLT Pipeline** (NB7) to populate Gold layer
8. Connect Gold layer to Azure Synapse
9. Connect Synapse to Power BI for reporting

---

## 👤 Author

**Nuha Aquil**
- Azure Databricks Workspace: `netflix-adb-nuha`
- ADF Instance: `adfnetflixnuha`