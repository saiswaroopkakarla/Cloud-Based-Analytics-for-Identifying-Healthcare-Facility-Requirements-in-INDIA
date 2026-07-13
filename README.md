# Cloud-Based Analytics — Healthcare Facility Planning in India

![Python](https://img.shields.io/badge/Python-3.x-blue?logo=python&logoColor=white)
![AWS](https://img.shields.io/badge/Cloud-AWS%20EC2%20%7C%20S3%20%7C%20RDS-FF9900?logo=amazonaws&logoColor=white)
![MySQL](https://img.shields.io/badge/Database-MySQL%20%7C%20SQLAlchemy-4479A1?logo=mysql&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-Data%20Engineering-150458?logo=pandas&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-lightgrey)

A 27-problem end-to-end data engineering pipeline identifying **underserved regions in India** for new hospital facility placement — combining Census 2011, housing, and healthcare infrastructure data with MySQL stored procedures, triggers, and AWS cloud deployment.

---

## Problem Overview

India's healthcare infrastructure is critically uneven. This project answers:
- Which states have the fewest hospital beds per 10,000 people?
- Which state should receive the next government hospital?
- How many new hospitals are needed to meet WHO standards per state?
- Which districts lack in-premise toilets, affecting health outcomes?

**Key finding:** Bihar, Jharkhand, and Uttar Pradesh have the lowest hospital bed-to-population ratios. Jharkhand is recommended for the next government hospital (fewest total government hospitals among the three).

---

## Pipeline Architecture

```
Raw Data Sources (4 datasets)
        │
        ▼
Data Cleaning & Imputation (Problems 1–9)
Census 2011 → housing.csv → all_hospitals.csv → govt_hospitals.csv
        │
        ▼
Feature Engineering (Problems 10–17)
Beds per 10,000 · WHO standard gap · Facility disparity scoring
        │
        ▼
MySQL Database (Problems 18–19)
4 tables · Primary/Foreign key constraints · SQLAlchemy ORM
        │
        ▼
Stored Procedures & Triggers (Problems 20–27)
7 procedures · 2 triggers · Hospital log · Bed log
        │
        ▼
Cloud Deployment (AWS)
EC2 (Jupyter) · S3 (data lake) · RDS (MySQL) · Auto Scaling
```

---

## Datasets

| Dataset | Source | Records | Description |
|---|---|---|---|
| `census_2011.csv` | Census of India 2011 | 640 districts | Population, literacy, gender, age group, households |
| `housing.csv` | Census housing data | 640 districts | Rural/urban livable, dilapidated, latrine households |
| `hospitals.csv` | Ministry of Health | 37 states | PHC, CHC, SDH, DH, total hospitals, beds |
| `government_hospitals.csv` | Ministry of Health | 37 states | Rural/urban govt hospitals and beds, last updated |

---

## 27 Problem Statements Solved

### Data Cleaning (Census 2011)

| Problem | Description | Technique |
|---|---|---|
| 1 | Keep relevant columns | `usecols` selective load |
| 2 | Rename column headers | Dictionary mapping |
| 3 | Normalise State/UT name casing | `.str.title()` + regex |
| 4 | Telangana state formation (2014) | District-index lookup + reassignment |
| 5 | Impute missing values | Arithmetic imputation (Male + Female = Population) |
| 6 | Save cleaned census | `to_csv` → `census.csv` |

**Missing value reduction:** Population NaN 4.69% → 0.16%, Literate NaN 5.63% → 0.31%

### Housing Data Engineering

| Problem | Description | Output |
|---|---|---|
| 7 | Merge census + housing on District + State | Inner join across datasets |
| 8 | Visualise household statistics | State-wise toilet coverage bar charts |
| 9 | Compute rural/urban population from household ratios | `Number_of_ppl_per_house` feature |

### Hospital Data Processing

| Problem | Description |
|---|---|
| 10 | Expand hospital column acronyms (PHC, CHC, SDH, DH) using metadata.csv |
| 11 | Normalise State/UT names across all 4 datasets (reusable `filter_State_func`) |
| 12 | Identify 3 states with fewest beds per 10,000 people |

**Result:** Bihar · Jharkhand · Uttar Pradesh (lowest Beds_per_10000)

### Analysis & Visualisation

| Problem | Description |
|---|---|
| 15 | Recommend state for new government hospital |
| 16 | Visualise expected (WHO 3/1000) vs actual hospital beds per state |
| 17 | Calculate new govt hospitals needed to meet WHO standard per state |

**Result — hospitals needed to meet WHO standard (selected states):**
- Bihar: highest gap, most new hospitals required
- Jharkhand: recommended for next hospital (least govt hospitals among bottom 3)

---

## Database Design (MySQL via SQLAlchemy)

### Tables

```sql
all_hospital_table   (States PK) — hospital counts + beds per state
govt_hospital_table  (Id PK, States FK) — rural/urban govt hospitals + beds
census_table         (Id PK, States FK) — population demographics per district
housing_table        (Id PK, State_Name FK) — housing conditions per district
hospital_log         — audit log for hospital add/remove events
hospital_bed_log     — audit log for bed add/remove events
```

### Stored Procedures (7)

```sql
get_population_district(district, OUT population)
get_population(state, OUT population)
senior_citizen_population(state, OUT senior_pop)
get_hospital_beds(state, OUT beds)
get_govt_hospitals_beds(state, OUT beds)
beds_per_lakh(state, OUT beds_per_lakh)         -- calls 2 procedures internally
govt_hospital_beds_per_lakh(state, OUT rate)
```

### Triggers (2)

```sql
update_hospital_data_trigger   -- logs hospital add/remove to hospital_log
hospital_beds_log_trigger      -- logs bed add/remove to hospital_bed_log
```

### Key Queries

**Households without in-premise toilets in Bihar** (lowest bed-to-population state):
```sql
SELECT State_Name, District_Name,
  ROUND((Households_Rural + Households_Urban
       - Household_Rural_Latrine_premise
       - Household_Urban_Latrine_premise)) AS Household_without_toilet
FROM housing_table
WHERE State_Name = (
  SELECT c.States FROM census_table c
  INNER JOIN all_hospital_table ah ON c.States = ah.States
  GROUP BY c.States
  ORDER BY HospitalBeds / SUM(Population)
  LIMIT 1
)
```

**North-East states healthcare report** (Problem 22):
Queries population, senior citizen count, govt hospitals, beds per lakh for 7 NE states (Arunachal Pradesh, Mizoram, Manipur, Tripura, Assam, Meghalaya, Nagaland) using stored procedure calls.

---

## Key Results

| Metric | Value |
|---|---|
| State with fewest beds/10K | Bihar |
| Recommended for new hospital | Jharkhand |
| Beds per lakh — J&K | 90 |
| Govt beds per lakh — Tamil Nadu | 107 |
| Districts with worst toilet access | Bihar (Saharsa: 390,897 without toilets) |
| Cross-4-table JOIN verified | ✅ (Problem 19) |
| Trigger events logged | 4 hospital + 4 bed events (Andhra Pradesh test) |

---

## Cloud Deployment (AWS)

```
EC2 (t2.medium)    — Hosts Jupyter notebook server
S3                 — Raw CSV data lake storage
RDS (MySQL)        — Structured database layer (SQLAlchemy connection)
Auto Scaling       — Handles variable compute load during batch processing
```

For local execution — no AWS required:

```bash
git clone https://github.com/saiswaroopkakarla/Cloud-Based-Analytics-for-Identifying-Healthcare-Facility-Requirements-in-INDIA.git
pip install pandas numpy matplotlib seaborn sqlalchemy pymysql jupyter
jupyter notebook VCC_Analysis_Code.ipynb
```

Update file paths from `E:\\Data\\` to your local data directory before running.

---

## Tech Stack

| Component | Technology |
|---|---|
| Cloud | AWS EC2, S3, RDS, Auto Scaling |
| Database | MySQL + SQLAlchemy ORM |
| Data engineering | Pandas, NumPy |
| Visualisation | Matplotlib, Seaborn |
| Notebook | Jupyter (302 cells, 27 problems) |
| Data | Census 2011, Ministry of Health India |

---

## Course Details

**Course:** CSL7510 — Virtualisation and Cloud Computing
**Instructor:** Dr Sumit Kalra, IIT Jodhpur
**Semester:** II (2024-25)

---

## Author

**Kakarla Sai Swaroop** M25DE1023, IIT Jodhpur M.Tech Data Engineering
