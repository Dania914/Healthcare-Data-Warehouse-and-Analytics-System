
# 🏥 Healthcare Data Warehouse & Analytics System

### *Transforming fragmented healthcare data into actionable intelligence*

<br/>

> An end-to-end Data Warehousing and Business Intelligence solution integrating **ETL pipelines**, **Star Schema modeling**, **Machine Learning analytics**, and **interactive Power BI dashboards** — built to power data-driven decision-making in healthcare institutions.

<br/>

---

</div>

## 📋 Table of Contents

- [Overview](#-overview)
- [System Architecture](#-system-architecture)
- [Data Sources](#-data-sources--integration)
- [Star Schema Design](#-star-schema-design)
- [ETL Pipeline](#-etl-pipeline)
- [Data Warehouse](#-data-warehouse--postgresql)
- [Machine Learning](#-machine-learning--analytics)
- [Power BI Dashboards](#-power-bi-dashboards)
- [Key Results](#-key-results)
- [Project Structure](#-project-structure)
- [Getting Started](#-getting-started)
- [Challenges & Solutions](#-challenges--solutions)
- [Future Enhancements](#-future-enhancements)

---

## 🔭 Overview

Healthcare organizations generate massive volumes of operational and financial data daily — yet this data often resides in **siloed, disconnected systems**, making meaningful analysis nearly impossible.

This project solves that by building a **centralized, intelligent, and scalable Healthcare Data Warehouse** that:

- Consolidates data from 5 heterogeneous sources into a unified PostgreSQL warehouse
- Applies rigorous ETL processes with a comprehensive validation framework
- Leverages Machine Learning for patient segmentation, anomaly detection, and cost prediction
- Delivers executive-ready insights through interactive Power BI dashboards

<br/>

<div align="center">

| Dimension | Detail |
|:---:|:---:|
| 🗄️ **Warehouse** | PostgreSQL — Star Schema |
| 🔄 **ETL** | Python (Pandas) |
| 🤖 **ML Models** | K-Means · Isolation Forest · Random Forest · GAN |
| 📊 **BI Tool** | Microsoft Power BI |
| 📁 **Data Volume** | 55,500 records · 15 attributes (post-cleaning & validation)|

</div>

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          DATA SOURCES                                   │
│                                                                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                  │
│  │  OLTP (DB 1) │  │  OLTP (DB 2) │  │  OLTP (DB 3) │                  │
│  │   Patients   │  │  Treatments  │  │    Billing   │                  │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘                  │
│         │                 │                  │                          │
│  ┌──────┴───────┐  ┌──────┴────────────────────────┐                   │
│  │  WHO API     │  │     Insurance CSV (Flat File)  │                   │
│  │  (External)  │  │                               │                   │
│  └──────┬───────┘  └──────┬────────────────────────┘                   │
└─────────┼─────────────────┼───────────────────────────────────────────-┘
          │                 │
          ▼                 ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                       ETL PIPELINE (Python)                             │
│                                                                         │
│   Extract → Clean → Validate → Transform → Build Dimensions → Load      │
└─────────────────────────┬───────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────────────┐
│               DATA WAREHOUSE (PostgreSQL — Star Schema)                 │
│                                                                         │
│     FactPatientTreatment ←→ DimPatient · DimDoctor · DimHospital        │
│                          ←→ DimTreatment · DimDate · DimHealthIndicator  │
└───────────────────┬─────────────────────────────────────────────────────┘
                    │
          ┌─────────┴──────────┐
          ▼                    ▼
┌──────────────────┐  ┌──────────────────────────────┐
│  ML / ANALYTICS  │  │      POWER BI DASHBOARDS     │
│                  │  │                              │
│ • K-Means        │  │  • Summary KPIs              │
│ • Isolation Forest│  │  • Patient Demographics      │
│ • Random Forest  │  │  • Financial Performance     │
│ • GAN            │  └──────────────────────────────┘
└──────────────────┘
```

---

## 📡 Data Sources & Integration

The system pulls from **five distinct source types**, unified through the ETL pipeline:

| # | Source | Type | Description |
|---|--------|------|-------------|
| 1 | `patients_data.csv` | OLTP System | Patient demographics, conditions, BMI, insurance |
| 2 | `treatments_data.csv` | OLTP System | Treatment types, dates, descriptions, costs |
| 3 | `billing_data.csv` | OLTP System | Billing amounts, payments, discharge info |
| 4 | WHO API | External REST API | Global health indicators by country and year |
| 5 | `insurance_clean.csv` | CSV Flat File | Insurance provider and coverage data |

---

## ⭐ Star Schema Design

The warehouse follows a **Star Schema** — optimized for fast analytical queries and simplified reporting.

```
                        ┌─────────────────┐
                        │   DimHospital   │
                        │─────────────────│
                        │ hospital_key PK │
                        │ provider_street │
                        │ provider_city   │
                        │ provider_state  │
                        │ referral_region │
                        └────────┬────────┘
                                 │
   ┌─────────────────┐           │           ┌──────────────────┐
   │   DimPatient    │           │           │   DimTreatment   │
   │─────────────────│           │           │──────────────────│
   │ patient_key  PK │           │           │ treatment_key PK │
   │ patient_id      │           │           │ treatment_id     │
   │ name, age       │           │           │ treatment_type   │
   │ gender          ├───────────┤           │ description      │
   │ blood_type      │           │           │ cost             │
   │ medical_cond.   │    ┌──────┴──────┐    │ treatment_date   │
   │ insurance_prov  │    │             │    └──────────────────┘
   │ region, smoker  ├────┤  FACT TABLE ├────┐
   │ bmi, charges    │    │─────────────│    │  ┌───────────────────────┐
   └─────────────────┘    │fact_id   PK │    │  │  DimHealthIndicator   │
                          │─────────────│    │  │───────────────────────│
   ┌─────────────────┐    │patient_key  │    │  │ indicator_key      PK │
   │   DimDoctor     │    │doctor_key   │    └──┤ country_code          │
   │─────────────────│    │hospital_key │       │ year                  │
   │ doctor_key   PK ├────┤treatment_key│       │ indicator_name        │
   │ doctor_name     │    │date_key     │       │ value_numeric         │
   │ hospital        │    │indicator_key│       └───────────────────────┘
   │ room_number     │    │─────────────│
   │ admission_type  │    │billing_amt  │   ┌──────────────────┐
   └─────────────────┘    │treatment_cst│   │     DimDate      │
                          │total_disch. ├───│──────────────────│
                          │avg_covered  │   │ date_key      PK │
                          │avg_payments │   │ date             │
                          │avg_medicare │   │ day, month       │
                          │length_stay  │   │ quarter, year    │
                          └─────────────┘   └──────────────────┘
```

---

## 🔄 ETL Pipeline

> 📂 `etl/ETL_ipynb.ipynb`

The ETL pipeline is implemented in **Python** using modular functions for each stage:

### 1️⃣ Extract
Each data source is extracted via a dedicated function (`extract_patient_data()`, `extract_billing_data()`, etc.) with `try/except` blocks for fault-tolerant I/O handling.

### 2️⃣ Transform

| Step | Operation |
|------|-----------|
| **Standardization** | Column names normalized (lowercase, underscores); conflicting field names mapped to canonical schema |
| **Cleaning** | `drop_duplicates()` applied; string fields stripped and lowercased; dates normalized to `YYYY-MM-DD` |
| **Missing Values** | Numerical → median imputation · Categorical → mode assignment · Critical FKs → flagged for review |
| **Dimension Building** | Six dimension DataFrames constructed with unique synthetic surrogate keys |
| **Fact Table** | Built by joining dimension keys with financial and clinical measures |
| **Encoding** | One-hot and label encoding applied per model requirements |

### 3️⃣ Load
Transformed fact and dimension tables are exported as CSVs and imported into **PostgreSQL**, preserving the full Star Schema structure.

---

## 🐘 Data Warehouse — PostgreSQL

> 📂 `warehouse/Data Warehouse.sql`

PostgreSQL was selected for its robustness, scalability, and compatibility with analytical workloads.

**Implementation highlights:**
- Indexed primary and foreign keys for optimized query performance
- Partitioned tables for faster aggregations on large datasets
- Enforced referential integrity between all fact and dimension tables
- Schema designed to support both operational reporting and ML feature extraction

---

## 🤖 Machine Learning & Analytics

> 📂 `ml_models/ML Analytics.ipynb`

Four ML techniques were applied to extract intelligence from the warehouse data:

### 🔵 K-Means Clustering — Patient Segmentation

Three distinct patient segments were identified based on billing, treatment cost, length of stay, and payment attributes:

| Cluster | Avg. Billing | Avg. Treatment Cost | Avg. Stay | Avg. Payments | Profile |
|---------|-------------|---------------------|-----------|---------------|---------|
| **0** | $6,982 | $2,821 | 5.4 days | $1,297 | Moderate cost, longer stay |
| **1** | $7,908 | $2,551 | 3.3 days | $3,121 | Low cost, short stay |
| **2** | $25,271 | $3,039 | 6.5 days | $2,813 | High cost, long stay |

### 🔴 Isolation Forest — Anomaly Detection

Detected **4 billing anomalies** with unusually high billing amounts paired with inconsistent treatment costs — indicative of billing errors or insurance discrepancies.

```
⚠️  Anomaly Example: Billing $48,017  ←→  Disproportionately low treatment cost
⚠️  Anomaly Example: Billing $37,511  ←→  Disproportionately low treatment cost
```

### 🟢 Predictive Modeling — Cost Forecasting

Two regression models evaluated on engineered features (age, BMI, stay duration, discharge count, cost ratios, treatment type):

| Model | Train R² | Test R² | RMSE | MAPE | CV R² |
|-------|----------|---------|------|------|-------|
| **Random Forest** | 0.9631 | **0.8502** | $529 | 16.9% | 0.78 ± 0.14 |
| Linear Regression | 0.6645 | 0.6975 | $752 | 38.4% | 0.54 ± 0.07 |

> 🏆 **Random Forest** is the recommended model for production cost forecasting.

**Top predictive features:** `payment_coverage_ratio` · `medicare_ratio` · `average_total_payments` · `age_bmi_interaction` · `billing_amount`

### 🟣 GAN — Synthetic Data Generation

A Generative Adversarial Network was trained to generate privacy-preserving synthetic healthcare records for augmenting model training data.

| Epoch | D_real | D_fake | Generator Loss |
|-------|--------|--------|----------------|
| 0 | 0.574 | 0.672 | 0.625 |
| 200 | 0.728 | 0.729 | 0.539 |
| 400 | 0.748 | 0.749 | 0.511 |

> Convergence of D_real ≈ D_fake confirms the GAN successfully learned the underlying data distribution.

---

## 📊 Power BI Dashboards

> 📂 `dashboards/HealthCare dashboard + Report.pbix`

Three purpose-built dashboard pages, each targeting a different stakeholder:

### 📄 Page 1 — SUMMARY
Executive snapshot of clinical and financial KPIs.

- Total Patients · Active Admissions · Total Billing · Total Treatment Cost
- Average Length of Stay (ALOS) · Total Payments
- Time-series billing and cost trend charts
- Top departments by billing contribution

### 👥 Page 2 — PATIENT DEMOGRAPHICS
Population analysis for clinical resource planning.

- Age group and gender distribution
- Medical condition breakdown by blood type
- BMI vs Age trend analysis
- ML cluster visualization (spending patterns per segment)
- Smoking vs. disease prevalence analysis
- Insurance provider distribution

### 💰 Page 3 — FINANCE
Detailed financial analytics and anomaly visibility.

- Revenue vs. cost waterfall chart
- Top billing services/treatment types
- Accounts receivable aging matrix
- Anomaly-flagged billing records (from Isolation Forest)
- Payment source breakdown (Medicare · Private · Self-pay)

---

## 📈 Key Results

<div align="center">

| Metric | Result |
|--------|--------|
| 🎯 Prediction Accuracy (Random Forest) | **~85% (R² = 0.85)** |
| 🔍 Billing Anomalies Detected | **4 records flagged** |
| 👥 Patient Clusters Identified | **3 distinct segments** |
| 🧬 GAN Convergence | **Successfully achieved** |
| 📦 Data Sources Integrated | **5 heterogeneous sources** |
| 🏗️ Warehouse Tables | **1 Fact + 6 Dimension tables** |

</div>

---

## 📂 Project Structure

```
Healthcare-Data-Warehouse-and-Analytics-System/
│
├── 📁 data/
│   └── Healthcare Data.xlsx              # Source data (multi-sheet)
│
├── 📁 etl/
│   └── ETL_ipynb.ipynb                   # Full ETL pipeline (Extract → Transform → Load)
│
├── 📁 warehouse/
│   └── Data Warehouse.sql                # PostgreSQL Star Schema DDL & load scripts
│
├── 📁 ml_models/
│   └── ML Analytics.ipynb                # Clustering · Anomaly · Regression · GAN
│
├── 📁 dashboards/
│   └── HealthCare dashboard + Report.pbix # Power BI interactive dashboard
│
├── 📁 reports/
│   ├── Healthcare_Data_Warehouse_Project_Report.pdf
│   └── Healthcare-Data-Warehouse-and-Analytics-System.pptx
│
├── requirements.txt
└── README.md
```

---

## 🚀 Getting Started

### Prerequisites

- Python 3.10+
- PostgreSQL 14+
- Power BI Desktop
- Jupyter Notebook

### Installation & Setup

```bash
# 1. Clone the repository
git clone https://github.com/Dania914/Data-Warehouse-and-Analytics-System.git
cd Data-Warehouse-and-Analytics-System

# 2. Install Python dependencies
pip install -r requirements.txt
```

### Running the Pipeline

```bash
# Step 1 — Run the ETL Pipeline
# Open Jupyter and execute all cells in:
etl/ETL_ipynb.ipynb

# Step 2 — Load the Data Warehouse into PostgreSQL
psql -U <your_user> -d <your_database> -f "warehouse/Data Warehouse.sql"

# Step 3 — Run Machine Learning Models
# Open Jupyter and execute all cells in:
ml_models/ML Analytics.ipynb

# Step 4 — Open Power BI Dashboard
# Launch Power BI Desktop and open:
dashboards/HealthCare dashboard + Report.pbix
```

---

## ⚠️ Challenges & Solutions

| Challenge | Solution Applied |
|-----------|-----------------|
| Data inconsistency across 5 sources | Schema standardization & canonical field mapping in ETL |
| Missing values in clinical fields | Evidence-based imputation (median for numerical, mode for categorical) |
| API rate limits (WHO API) | Retry handling with fallback to pre-extracted CSV |
| Referential integrity violations | Multi-layer FK validation framework (9 rule categories) |
| Outliers in billing data | Z-score + IQR detection with clinical domain review |
| Data privacy for ML training | GAN-generated synthetic data as privacy-preserving augmentation |

---

## 🔮 Future Enhancements

- **⚡ Real-Time Streaming** — Integrate IoT/live monitoring data via Apache Kafka
- **☁️ Cloud Deployment** — Migrate to AWS Redshift / Azure Synapse / GCP BigQuery
- **🧠 Advanced ML** — XGBoost, Deep Neural Networks, early disease detection models
- **🔍 Explainable AI** — SHAP and LIME for cost driver transparency
- **📱 Role-Based Dashboards** — Tailored views for clinicians, finance officers, and executives
- **🔔 Automated Alerting** — Email/in-app notifications for anomaly detection triggers

---

## 📄 License

This project is developed for academic purposes under the Data Warehousing and Business Intelligence course (CT-472) — NED University, Department of Computer Science & IT.

---

<div align="center">

*Built with ❤️ for data-driven healthcare transformation*

⭐ **Star this repository if you found it useful!** ⭐

</div>
