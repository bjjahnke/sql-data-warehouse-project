# Data Warehouse and Analytics Project

Built following Baraa Salkini's (Data With Baraa) [SQL Data Warehouse tutorial](https://www.youtube.com/watch?v=9GVqKuTVANE). The original project assumes Windows (SQL Server Express + SSMS). This version documents how I ran it on a **Mac**, using **SQL Server in Docker** and **VS Code**, and the changes that required.

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
  -e 'ACCEPT_EULA=Y' -e 'MSSQL_SA_PASSWORD=YourStrong!Passw0rd' -e 'MSSQL_PID=Express' \
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

1. **Data Architecture**: Data warehouse designed with Medallion Architecture (Bronze, Silver, Gold layers).
2. **ETL Pipelines**: Extract, transform, and load data into the warehouse from sources (ERP, CRM).
3. **Data Modeling**: Develop fact and dimension tables for analytics.
4. **Analytics & Reporting**: SQL-based reports and dashboards for insights.

**Skills demonstrated:**
- SQL Development
- Data Architecture
- Data Engineering  
- ETL Pipeline Developement
- Data Modeling  
- Data Analytics  

---

## 🛠️ Tools

- **[Datasets](datasets/):** ERP and CRM CSV files.
- **[SQL Server Express](https://www.microsoft.com/en-us/sql-server/sql-server-downloads)** (running in Docker on Mac)
- **[Docker Desktop](https://www.docker.com/products/docker-desktop/)**
- **[VS Code](https://code.visualstudio.com/)** + mssql extension (replaces SSMS)
- **[DrawIO](https://www.drawio.com/):** architecture and model diagrams.
- **[Original Notion project steps](https://thankful-pangolin-2ca.notion.site/SQL-Data-Warehouse-Project-16ed041640ef80489667cfe2f380b269?pvs=4)** by Data With Baraa.

---

## 🚀 Project Requirements

### Building the Data Warehouse (Data Engineering)
- **Objective:** consolidate sales data from two sources into a SQL Server warehouse for analytical reporting.
- **Data sources:** ERP and CRM CSV files.
- **Data quality:** cleanse and resolve issues before analysis.
- **Integration:** one user-friendly model built for analytical queries.
- **Scope:** latest dataset only, no historization.
- **Documentation:** clear data model documentation.

### Analytics & Reporting (Data Analysis)
SQL analytics on **customer behavior**, **product performance**, and **sales trends**. Details: [docs/requirements.md](docs/requirements.md).

---
## 📂 Repository Structure
```
sql-data-warehouse-project/
│
├── datasets/                  # Source data (from Data With Baraa)
│   ├── source_crm/            # CRM CSVs: cust_info, prd_info, sales_details
│   └── source_erp/            # ERP CSVs: CUST_AZ12, LOC_A101, PX_CAT_G1V2
│
├── docs/                      # My documentation
│   ├── mac-setup.md           # Docker + VS Code setup for Mac
│   └── sql-workflow.md        # Working with .sql files, backups
│
├── .gitignore                 # Ignores .DS_Store, *.bak, .env
├── LICENSE                    # MIT, original copyright retained
└── README.md                  # Project overview and Mac adaptations
```

---
## 📝 My Progress & Notes

- [x] Docker + SQL Server Express running on Mac, connected from VS Code
- [x] Source datasets added
- [ ] Bronze layer loaded
- [ ] Silver layer
- [ ] Gold layer
- [ ] My own additions (extra quality checks, analysis queries)

## 🛡️ License

MIT. Original © 2024 Baraa Khatib Salkini; see [LICENSE](LICENSE).
