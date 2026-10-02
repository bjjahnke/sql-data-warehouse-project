# Data Warehouse and Analytics Project

Built while following Baraa Salkini's (Data With Baraa) [SQL Data Warehouse tutorial](https://www.youtube.com/watch?v=9GVqKuTVANE). The original project assumes Windows (SQL Server Express + SSMS). This version documents how I ran it on a **Mac**, using **SQL Server in Docker** and **VS Code**, and the changes that required.

> **Credit:** Original project and course by [Data With Baraa](https://github.com/DataWithBaraa/sql-data-warehouse-project) (MIT License). This repo is my learning copy plus Mac-specific adaptations.

---
## 🏗️ Data Architecture

The data architecture for this project follows Medallion Architecture (**Bronze**, **Silver**, and **Gold** layers):
![Data Architecture](docs/data_architecture.png)

1. **Bronze**: raw data loaded as-is from the ERP and CRM CSV files into SQL Server.
2. **Silver**: cleansing, standardization, and normalization.
3. **Gold**: business-ready star schema for reporting and analytics.

---
## 🍎 Running This on a Mac

SQL Server Express and SSMS are Windows-only. My setup:

| Original (Windows) | My Mac setup |
|---|---|
| SQL Server Express installed locally | SQL Server 2022 Express in a **Docker** container |
| SSMS | **VS Code** + the SQL Server (mssql) extension |
| Windows Authentication | SQL Login (`sa`) |
| Server `.\SQLEXPRESS` | `localhost,1433` |
| F5 to run a query | Cmd + Shift + E |

**Prerequisites:** Docker Desktop, VS Code with the mssql extension, and (Apple Silicon) *Use Rosetta for x86/amd64 emulation* enabled in Docker Desktop settings.

**Create the container (one time):**
```bash
docker run --platform linux/amd64 \
  -e 'ACCEPT_EULA=Y' -e 'MSSQL_SA_PASSWORD=<choose-a-strong-password>' -e 'MSSQL_PID=Express' \
  -p 1433:1433 -v sqlserver_data:/var/opt/mssql \
  --name sqlserver -d mcr.microsoft.com/mssql/server:2022-latest
```
- Use single quotes: in zsh, `!` inside double quotes causes `event not found`.
- `-v sqlserver_data:/var/opt/mssql` keeps your data if the container is deleted.

**Connect in VS Code:** server `localhost,1433`, SQL Login `sa`, **Trust server certificate = Yes**. Test with `SELECT @@VERSION;`.

Full details and troubleshooting: [docs/mac-setup.md](docs/mac-setup.md) · Daily workflow: [docs/sql-workflow.md](docs/sql-workflow.md)

### Changes needed to the original scripts

**1. Loading the CSVs (`scripts/bronze/proc_load_bronze.sql`)**
The original uses `BULK INSERT` with Windows paths (`C:\sql\dwh_project\datasets\...`). SQL Server runs *inside the container*, so it can't see your Mac folders. Fix:
```bash
docker exec -u 0 sqlserver mkdir -p /var/opt/mssql/datasets
docker cp datasets/source_crm sqlserver:/var/opt/mssql/datasets/
docker cp datasets/source_erp sqlserver:/var/opt/mssql/datasets/
```
Then change each path in the script, e.g.:
```sql
-- original
FROM 'C:\sql\dwh_project\datasets\source_crm\cust_info.csv'
-- Mac / Docker
FROM '/var/opt/mssql/datasets/source_crm/cust_info.csv'
```

**2. Case-sensitive filenames**
The container is Linux, so filenames are case-sensitive. The ERP files are uppercase (`CUST_AZ12.csv`, `LOC_A101.csv`, `PX_CAT_G1V2.csv`) but the script references lowercase. Match the paths to the real filenames.

**3. Line endings (if a load fails)**
If `BULK INSERT` errors on the last column, check the row terminator (`\n` vs `\r\n`) against the file.

> ⚠️ *Status: items 1-3 are written from reading the scripts. Update with what actually happened when you ran it.*

---
## 📖 Project Overview

This project involves:

1. **Data Architecture**: Designing a Modern Data Warehouse Using Medallion Architecture **Bronze**, **Silver**, and **Gold** layers.
2. **ETL Pipelines**: Extracting, transforming, and loading data from source systems into the warehouse.
3. **Data Modeling**: Developing fact and dimension tables optimized for analytical queries.
4. **Analytics & Reporting**: Creating SQL-based reports and dashboards for actionable insights.

🎯 This repository is meant to showcase expertise in:
- SQL Development
- Data Architecture
- Data Engineering  
- ETL Pipeline Developement
- Data Modeling  
- Data Analytics  

---

## 🛠️ Important Links & Tools:

Everything is for Free!
- **[Datasets](datasets/):** Access to the project dataset (csv files).
- **[SQL Server Express](https://www.microsoft.com/en-us/sql-server/sql-server-downloads):** Lightweight server for hosting your SQL database.
- **[SQL Server Management Studio (SSMS)](https://learn.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver16):** GUI for managing and interacting with databases.
- **[Git Repository](https://github.com/):** Set up a GitHub account and repository to manage, version, and collaborate on your code efficiently.
- **[DrawIO](https://www.drawio.com/):** Design data architecture, models, flows, and diagrams.
- **[Notion](https://www.notion.com/templates/sql-data-warehouse-project):** Get Data With Baraa's Project Template from Notion 
- **[Notion Project Steps](https://thankful-pangolin-2ca.notion.site/SQL-Data-Warehouse-Project-16ed041640ef80489667cfe2f380b269?pvs=4):** Access to All Project Phases and Tasks.

---

## 🚀 Project Requirements

### Building the Data Warehouse (Data Engineering)

#### Objective
Develop a modern data warehouse using SQL Server to consolidate sales data, enabling analytical reporting and informed decision-making.

#### Specifications
- **Data Sources**: Import data from two source systems (ERP and CRM) provided as CSV files.
- **Data Quality**: Cleanse and resolve data quality issues prior to analysis.
- **Integration**: Combine both sources into a single, user-friendly data model designed for analytical queries.
- **Scope**: Focus on the latest dataset only; historization of data is not required.
- **Documentation**: Provide clear documentation of the data model to support both business stakeholders and analytics teams.

---

### BI: Analytics & Reporting (Data Analysis)

#### Objective
Develop SQL-based analytics to deliver detailed insights into:
- **Customer Behavior**
- **Product Performance**
- **Sales Trends**

These insights empower stakeholders with key business metrics, enabling strategic decision-making.  

For more details, refer to [docs/requirements.md](docs/requirements.md).

## 📂 Repository Structure
```
data-warehouse-project/
│
├── datasets/                           # Raw datasets used for the project (ERP and CRM data)
│
├── docs/                               # Project documentation and architecture details
│   ├── etl.drawio                      # Draw.io file shows all different techniquies and methods of ETL
│   ├── data_architecture.drawio        # Draw.io file shows the project's architecture
│   ├── data_catalog.md                 # Catalog of datasets, including field descriptions and metadata
│   ├── data_flow.drawio                # Draw.io file for the data flow diagram
│   ├── data_models.drawio              # Draw.io file for data models (star schema)
│   ├── naming-conventions.md           # Consistent naming guidelines for tables, columns, and files
│
├── scripts/                            # SQL scripts for ETL and transformations
│   ├── bronze/                         # Scripts for extracting and loading raw data
│   ├── silver/                         # Scripts for cleaning and transforming data
│   ├── gold/                           # Scripts for creating analytical models
│
├── tests/                              # Test scripts and quality files
│
├── README.md                           # Project overview and instructions
├── LICENSE                             # License information for the repository
├── .gitignore                          # Files and directories to be ignored by Git
└── requirements.txt                    # Dependencies and requirements for the project
```