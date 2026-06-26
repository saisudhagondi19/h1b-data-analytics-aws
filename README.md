# H-1B Visa Data Analytics Platform

End-to-end OLTP + OLAP pipeline on AWS — processes 70,000+ H-1B visa applications through a normalized MySQL database, Snowflake data warehouse, and Tableau dashboards.

![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-232F3E?style=flat&logo=amazonaws&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=flat&logo=mysql&logoColor=white)
![Snowflake](https://img.shields.io/badge/Snowflake-29B5E8?style=flat&logo=snowflake&logoColor=white)
![AWS Glue](https://img.shields.io/badge/AWS_Glue-FF9900?style=flat&logo=amazonaws&logoColor=white)
![Tableau](https://img.shields.io/badge/Tableau-E97627?style=flat&logo=tableau&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green?style=flat)

---

## Architecture

```
USCIS Raw CSV Data (154 cols)
        │
        ▼
  Data Cleaning & Normalization (Python)
        │
        ▼
  Amazon S3 (us-east-1)          ← Normalized CSVs stored here
        │
        ▼
  AWS Glue Workflow (on-demand)
   ├── Job 1: Create & load MySQL tables (OLTP)
   └── Job 2: ETL → Extract from MySQL → Transform → Load to Snowflake
        │                    │
        ▼                    ▼
  RDS MySQL (OLTP)     Snowflake (OLAP / DWH)
  3NF · 8 tables       Star Schema · Fact + 5 Dims
                              │
                              ▼
                        Tableau Dashboards
                        (auto-refreshed on pipeline run)
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| Cloud Platform | AWS (S3, RDS MySQL, Glue, IAM, VPC) |
| OLTP Database | Amazon RDS — MySQL (us-east-1b) |
| ETL Orchestration | AWS Glue (Apache Spark Structured Streaming) |
| Data Warehouse | Snowflake — Star Schema |
| ETL Scripts | Python (pandas, snowflake-connector-python, pymysql) |
| Visualization | Tableau |
| Data Source | USCIS + US Dept of Labor + USPS (70,000+ records, 64 columns) |

---

## Key Results

| Metric | Finding |
|---|---|
| Total applications analyzed | 70,000+ (FY 2022) |
| Approval rate | **95.01%** approved |
| Denial rate | 1.45% denied |
| Withdrawal rate | 3.54% withdrawn |
| Top applicant country | **India** (majority of all applicants) |
| Top employer | Cognizant, Google, Microsoft, Apple (30%+ combined) |
| Top state | **California** — 2,006 Software Engineer applications |
| Average approved salary | **$112,350** |
| Highest avg salary state | California |
| Top education fields | Computer Science, Computer Engineering, Business Administration |

---

## Database Design

### OLTP — 3NF Relational Schema (MySQL)

Data normalized from 154 raw columns into **8 tables** satisfying 1NF, 2NF, and 3NF:

```
Application (fact)
    ├── Applicant
    ├── Employer
    ├── Agent
    ├── Job Information
    ├── Prevailing Wage
    ├── Education Info
    └── Geography (postal codes from USPS)
```

### OLAP — Star Schema (Snowflake)

```
Fact: Application
Dimensions: Geography · Prevailing Wage · Education Info · Agent · Employer
```

---

## Analysis Performed

1. Top 10 applicant countries
2. Popular education backgrounds across all applicants
3. Top 10 employers by application volume
4. Top 10 employers by H-1B approvals
5. State-wise application distribution across the US
6. Average wage by job title
7. State vs. maximum applications per job title
8. Distribution of applications by prevailing wage range
9. Total applications by fiscal year
10. Wage distribution by educational qualification

---

## ETL Pipeline

### Extract
- Connects to RDS MySQL using credentials stored securely in S3 (not hardcoded)
- Reads tables into pandas DataFrames using `read_sql`
- Connection closed immediately after extraction

### Transform
- Renames columns for warehouse consistency
- Handles null values and removes unwanted columns
- Formats string-type columns
- Strips special characters from revenue fields
- Adds `refreshed_date` timestamp to every table

### Load
- Connects to Snowflake using secure credential file (stored in S3)
- Dumps transformed DataFrames into Snowflake tables via `snowflake-connector-python`
- Full exception handling at every step (S3 → MySQL → Snowflake)

---

## AWS Glue Workflow

The entire pipeline runs as an **on-demand AWS Glue Workflow** — no manual steps required:

```
Workflow Start
    │
    ├── Job 1: Create MySQL tables → Load CSVs from S3 into OLTP
    │
    └── Job 2: ETL — Extract MySQL → Transform → Load Snowflake
                │
                └── Trigger: Tableau data source refresh
```

Re-running the workflow automatically appends new fiscal year data to the warehouse.

---

## Setup

**1. Clone the repo**
```bash
git clone https://github.com/saisudhagondi19/h1b-data-analytics-aws.git
cd h1b-data-analytics-aws
```

**2. Install dependencies**
```bash
pip install -r requirements.txt
```

**3. Configure credentials**

Create a `.env` file (never commit this — it's in `.gitignore`):
```
MYSQL_HOST=your-rds-endpoint
MYSQL_DB=h1b_oltp

SNOWFLAKE_ACCOUNT=your-account
SNOWFLAKE_DB=H1B_DWH
SNOWFLAKE_SCHEMA=PUBLIC
```


**4. Upload normalized CSVs to S3**
```bash
aws s3 cp data/ s3://your-bucket/h1b-data/ --recursive
```

**5. Run the Glue workflow** via AWS Console or CLI:
```bash
aws glue start-workflow-run --name h1b-pipeline-workflow
```

---

## Requirements

```
pandas
pymysql
snowflake-connector-python
boto3
sqlalchemy
```

---

## Conclusions

- **India dominates** H-1B applications — the largest immigrant group by a significant margin
- **Tech majors lead** — Computer Science and Computer Engineering are the top education backgrounds
- **Big Tech files the most** — Cognizant, Google, Microsoft, and Apple together account for 30%+ of all applications
- **California is the hub** — highest application volume, highest average salary ($112,350+)
- **95% approval rate** in this dataset, though real-world rates are lower due to data composition

---

## Lessons Learned

- Designed and implemented a full OLTP → OLAP pipeline on AWS from scratch
- Learned AWS Glue workflows and Spark-based serverless ETL orchestration
- Applied database normalization (1NF → 3NF) to a 154-column real-world dataset
- Built a Snowflake star schema and created SQL views for Tableau reporting
- Implemented secure credential management — no keys exposed in code

---

## Future Work

- Extend to multi-visa type analysis (H-2B, L-1, O-1)
- Add streaming ingestion for real-time USCIS updates
- Build a QuickSight or Power BI dashboard as a Tableau alternative
- Automate pipeline on a quarterly schedule using AWS EventBridge

---

## License

MIT License — see [LICENSE](LICENSE) for details.
