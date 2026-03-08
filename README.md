# 🏥 Healthcare Claims Fraud Detection System
### Provider-Level Anomaly Detection Using CMS Medicare Data, MySQL & Power BI

![Project Status](https://img.shields.io/badge/Status-Complete-brightgreen)
![Data Source](https://img.shields.io/badge/Data-CMS%20Medicare%202023-blue)
![Database](https://img.shields.io/badge/Database-MySQL%208.0-orange)
![Visualization](https://img.shields.io/badge/Dashboard-Power%20BI-yellow)
![Records](https://img.shields.io/badge/Records%20Analyzed-13.6M%2B-red)

---

## 📋 Table of Contents
- [Project Overview](#-project-overview)
- [The Problem](#-the-problem)
- [The Solution](#-the-solution)
- [Key Findings](#-key-findings)
- [Data Sources](#-data-sources)
- [Project Architecture](#-project-architecture)
- [Technology Stack](#-technology-stack)
- [Database Schema](#-database-schema)
- [SQL Views & Feature Engineering](#-sql-views--feature-engineering)
- [Fraud Risk Scoring Model](#-fraud-risk-scoring-model)
- [Power BI Dashboard](#-power-bi-dashboard)
- [How to Reproduce](#-how-to-reproduce)
- [Important Disclaimer](#%EF%B8%8F-important-disclaimer)

---

## 🎯 Project Overview

This project builds a **provider-level anomaly detection system** to identify physicians and prescribers whose billing behavior is statistically abnormal compared to their peers. Using over 13.6 million rows of publicly available CMS Medicare data, the system calculates fraud risk scores for **691,000+ providers** across all 50 US states and Washington DC.

The output is an interactive Power BI dashboard that allows investigators to:
- Identify the **130 highest risk providers** in the 2023 Medicare dataset
- Drill down by specialty, state, and individual provider
- Understand **which specific billing behaviors** are driving each provider's risk score
- Compare providers against their **peer group** (same specialty + same state)

---

## ❗ The Problem

Medicare is the largest health insurance program in the United States, processing **hundreds of billions of dollars** in claims annually. This scale makes it a prime target for fraud, waste, and abuse.

**Why fraud is hard to detect:**
- Fraudulent billing rarely looks obviously wrong — it typically appears as slightly elevated charges across many claims over time
- With 691,000+ active Medicare providers, manual review is impossible
- Traditional auditing is random or complaint-driven, meaning most fraud goes undetected for years
- No single metric reveals fraud — it requires analyzing multiple billing behaviors simultaneously

**Common Medicare fraud schemes:**
| Scheme | Description |
|--------|-------------|
| Upcoding | Billing for more expensive procedures than were actually performed |
| Unbundling | Billing separately for services that should be combined at a lower rate |
| Phantom Billing | Submitting claims for services never rendered |
| Prescription Fraud | Prescribing unnecessary or excessive medications to inflate drug costs |
| Ping-Ponging | Referring patients between providers unnecessarily to generate additional claims |

---

## ✅ The Solution

A **6-stage data pipeline** that automatically scores every Medicare provider for fraud risk:

```
Raw CMS Data → MySQL Database → Feature Engineering → Z-Score Analysis → Risk Scoring → Power BI Dashboard
```

| Stage | Description |
|-------|-------------|
| 1️⃣ Data Engineering | Loaded and cleaned 13.6M+ rows into MySQL. Standardized numeric fields, removed non-US territories |
| 2️⃣ Peer Group Definition | Grouped providers by specialty type + state for fair comparison |
| 3️⃣ Feature Engineering | Calculated 6 fraud indicators per provider from raw billing data |
| 4️⃣ Z-Score Analysis | Measured how many standard deviations each provider is above their peer group |
| 5️⃣ Risk Scoring | Combined Z-scores into a weighted 0-100 composite fraud risk score |
| 6️⃣ Dashboard | Built interactive Power BI dashboard with drill-down capability |

---

## 🔍 Key Findings

> All findings are statistical in nature. See [Important Disclaimer](#important-disclaimer).

- **130 providers** classified as High Risk out of 691,000+ analyzed
- **Wyoming** had the highest average fraud risk score among all US states
- **California** and **Pennsylvania** had the highest counts of High Risk providers
- **Micrographic Dermatologic Surgery** had the highest average fraud risk score by specialty
- **538 providers** triggered 3 or more simultaneous fraud flags
- Top flagged provider had a services-per-beneficiary Z-score of **7.97** and drug cost Z-score of **11.01** — extreme outliers in both Part B and Part D
- Highest single metric Z-score in the dataset: **25.66** for drug cost per beneficiary (Emergency Medicine, AZ)

---

## 📊 Data Sources

| Dataset | Source | Records | Description |
|---------|--------|---------|-------------|
| Medicare Part B | CMS Open Data | 5,686,356 rows | Physician & Other Practitioners by Provider and Service 2023 |
| Medicare Part D | CMS Open Data | 8,000,000+ rows | Prescribers by Provider and Drug 2023 |

**Download the data:**
- [Part B Data](https://data.cms.gov/provider-summary-by-type-of-service/medicare-physician-other-practitioners/medicare-physician-other-practitioners-by-provider-and-service)
- [Part D Data](https://data.cms.gov/provider-summary-by-type-of-service/medicare-part-d-prescribers/medicare-part-d-prescribers-by-provider-and-drug)

---

## 🏗 Project Architecture

```
cms-medicare-fraud-detection/
│
├── sql/
│   ├── 01_create_tables.sql          # Table definitions for Part B and Part D
│   ├── 02_load_data.sql              # LOAD DATA INFILE statements
│   ├── 03_clean_data.sql             # Data cleaning and type conversion
│   ├── 04_views.sql                  # Provider summary and peer benchmark views
│   └── 05_fraud_risk_scores.sql      # Final risk scoring view
│
├── dashboard/
│   └── fraud_detection_dashboard.pbix  # Power BI dashboard file
│
├── docs/
│   ├── Fraud_Detection_Problem_and_Solution.docx
│   └── Fraud_Detection_Data_Reference_Guide.docx
│
└── README.md
```

---

## 🛠 Technology Stack

| Technology | Tool | Purpose |
|------------|------|---------|
| Database | MySQL 8.0 | Data storage, cleaning, aggregation, Z-score calculation |
| Query Tool | MySQL Workbench | SQL execution and database management |
| Visualization | Microsoft Power BI | Interactive 5-page fraud detection dashboard |
| Statistical Method | Z-Score Analysis | Peer group outlier detection |
| Data Engineering | SQL Views | Layered aggregation and feature engineering |
| Data Source | CMS Open Data | Publicly available Medicare claims data |

---

## 🗄 Database Schema

### Part B Table (`P Table`)
Stores raw Medicare Part B physician billing data.

| Column | Type | Description |
|--------|------|-------------|
| Rndrng_NPI | VARCHAR(15) | National Provider Identifier (primary key) |
| Rndrng_Prvdr_Type | VARCHAR(255) | Provider specialty type |
| Rndrng_Prvdr_State_Abrvtn | VARCHAR(255) | Provider state |
| HCPCS_Cd | VARCHAR(255) | Procedure code |
| Tot_Benes | INT | Total beneficiaries |
| Tot_Srvcs | DECIMAL(15,2) | Total services rendered |
| Avg_Mdcr_Pymt_Amt | DECIMAL(15,2) | Average Medicare payment |
| Avg_Sbmtd_Chrg | DECIMAL(15,2) | Average submitted charge |

### Part D Table (`prescribers_table`)
Stores raw Medicare Part D drug prescribing data.

| Column | Type | Description |
|--------|------|-------------|
| Prscrbr_NPI | VARCHAR(15) | National Provider Identifier (primary key) |
| Prscrbr_Type | VARCHAR(255) | Prescriber specialty type |
| Prscrbr_State_Abrvtn | VARCHAR(255) | Prescriber state |
| Gnrc_Name | TEXT | Generic drug name |
| Tot_Clms | INT | Total claims |
| Tot_Benes | INT | Total beneficiaries |
| Tot_Drug_Cst | DECIMAL(15,2) | Total drug cost |
| Tot_30day_Fills | DECIMAL(15,2) | Total 30-day fills |

---

## 🔧 SQL Views & Feature Engineering

The project uses **6 layered SQL views** to transform raw billing data into fraud risk scores:

### View 1 & 2: Provider Summaries
Aggregates raw procedure/drug level data to the provider level and calculates fraud indicators:

| Indicator | Formula | What It Detects |
|-----------|---------|-----------------|
| services_per_bene | SUM(Tot_Srvcs) / SUM(Tot_Benes) | Unnecessary procedure inflation |
| avg_payment_per_service | SUM(payment) / SUM(services) | Upcoding |
| charge_to_payment_ratio | SUM(charges) / SUM(payment) | Aggressive overbilling |
| drug_cost_per_bene | SUM(drug_cost) / SUM(benes) | Expensive drug overprescribing |
| claims_per_bene | SUM(claims) / SUM(benes) | Overprescribing |
| drug_cost_per_claim | SUM(drug_cost) / SUM(claims) | High-cost drug abuse |

### View 3 & 4: Peer Benchmarks
Calculates AVG and STD for each metric grouped by **provider_type + state**:
```sql
SELECT
    Rndrng_Prvdr_Type,
    Rndrng_Prvdr_State_Abrvtn,
    AVG(services_per_bene) AS avg_services_per_bene,
    STD(services_per_bene) AS std_services_per_bene
    -- ... additional metrics
FROM part_b_provider_summary
GROUP BY Rndrng_Prvdr_Type, Rndrng_Prvdr_State_Abrvtn;
```

### View 5 & 6: Z-Score Calculation
```sql
-- Z-Score formula
(provider_value - peer_group_mean) / peer_group_std_deviation

-- Example
ROUND((p.services_per_bene - b.avg_services_per_bene)
    / NULLIF(b.std_services_per_bene, 0), 2) AS z_services_per_bene
```

**Z-Score interpretation:**
| Z-Score | Meaning | Action |
|---------|---------|--------|
| Z < 1 | Normal variation | No concern |
| 1 ≤ Z < 2 | Slightly above average | Monitor |
| 2 ≤ Z < 3 | Notably above average | Review |
| Z ≥ 3 | Extreme outlier — flagged | Priority audit |

---

## 📈 Fraud Risk Scoring Model

The composite fraud risk score combines all 6 Z-scores with weighted contributions:

```
Risk Score = 
  MIN(5, |z_services_per_bene|)    / 5 × 20  +
  MIN(5, |z_payment_per_service|)  / 5 × 20  +
  MIN(5, |z_charge_to_payment|)    / 5 × 10  +
  MIN(5, |z_drug_cost_per_bene|)   / 5 × 20  +
  MIN(5, |z_claims_per_bene|)      / 5 × 20  +
  MIN(5, |z_drug_cost_per_claim|)  / 5 × 10
```

Scores are capped at 100. Z-scores are capped at 5 to prevent single extreme outliers from dominating the score.

**Risk Tiers:**
| Tier | Score Range | Count | Action |
|------|-------------|-------|--------|
| 🔴 High | 60 - 100 | 130 providers | Priority audit target |
| 🟡 Medium | 30 - 59 | 7,900 providers | Monitor for trends |
| 🟢 Low | 0 - 29 | 683,473 providers | Normal billing behavior |

---

## 📊 Power BI Dashboard

The dashboard contains **5 interactive pages:**

| Page | Description |
|------|-------------|
| Project Overview | Landing page with project summary and key stats 
!(CMS-medicare-fraud-detecttion)Dashboard Images/Project Overview.png |
| Executive Summary | KPI cards, risk tier donut, top specialties and states |
| Provider Drilldown | Searchable table of all providers with risk scores and flags |
| Geographic Analysis | Map and state-level breakdown of high risk providers |
| Metric Deep Dive | Scatter plots, Z-score comparisons, and flag distribution |

---

## 🚀 How to Reproduce

### Prerequisites
- MySQL 8.0+
- MySQL Workbench
- Microsoft Power BI Desktop
- MySQL Connector for Power BI (ODBC)

### Step 1 — Download the Data
Download both CMS datasets from the links in [Data Sources](#data-sources) and save as CSV files.

### Step 2 — Set Up the Database
```sql
CREATE DATABASE cms;
USE cms;
```

Run the SQL files in order:
```
sql/01_create_tables.sql
sql/02_load_data.sql
sql/03_clean_data.sql
sql/04_views.sql
sql/05_fraud_risk_scores.sql
```

### Step 3 — Load Data
Update the file paths in `02_load_data.sql` to match your local CSV locations then run:
```sql
LOAD DATA INFILE 'your/path/to/medicare_part_b.csv'
INTO TABLE `P Table`
FIELDS TERMINATED BY ','
OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;
```

### Step 4 — Connect Power BI
1. Open Power BI Desktop
2. Get Data → MySQL Database
3. Server: `localhost` | Database: `cms`
4. Load the `fraud_risk_scores` view
5. Open `dashboard/fraud_detection_dashboard.pbix`

### Step 5 — Verify Results
```sql
SELECT risk_tier, COUNT(*) AS provider_count
FROM fraud_risk_scores
GROUP BY risk_tier
ORDER BY provider_count;
```

Expected output:
```
High    →   130
Medium  →  7,900
Low     → 683,473
```

---

## ⚠️ Important Disclaimer

This project is for **educational and portfolio purposes only.**

- All data used is **publicly available** from CMS Medicare Open Data
- All findings identify **statistical outliers only** — they do not constitute evidence of fraud or wrongdoing
- A high risk score means a provider bills differently from their peer group — it does not prove fraudulent intent
- Any real fraud investigation would require a formal review by qualified investigators
- No personally identifiable patient information is used or stored in this project

---

## 📄 License

This project uses publicly available government data from the Centers for Medicare & Medicaid Services (CMS). The code and analysis in this repository are available for educational use.

---

## 👤 Author

Built as a data analytics portfolio project demonstrating end-to-end skills in:
- Large-scale data ingestion and cleaning (13.6M+ rows)
- Advanced SQL (views, aggregations, statistical functions)
- Statistical analysis (Z-score, peer benchmarking)
- Business intelligence and data visualization (Power BI)
- Healthcare domain knowledge (Medicare billing, fraud detection)

---

*Data Source: Centers for Medicare & Medicaid Services (CMS) | 2023 Medicare Claims Data*
