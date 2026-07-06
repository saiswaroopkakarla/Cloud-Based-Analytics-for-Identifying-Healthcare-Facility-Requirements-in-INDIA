# Cloud-Based Analytics — Healthcare Facility Planning in India

![Python](https://img.shields.io/badge/Python-3.x-blue?logo=python&logoColor=white)
![AWS](https://img.shields.io/badge/Cloud-AWS%20EC2%20%7C%20S3%20%7C%20RDS-FF9900?logo=amazonaws&logoColor=white)
![Dataset](https://img.shields.io/badge/Dataset-Census%202011%20%7C%20Health%20Infrastructure-green)

A cloud-native analytics project identifying **underserved regions in India** for new hospital facility placement — using Census 2011, housing, and healthcare infrastructure data processed on AWS.

---

## Problem Statement

India's healthcare infrastructure is unevenly distributed. This project uses data-driven analysis to identify districts that are **underserved relative to their population density and demographics** — providing actionable insights for healthcare facility planning.

---

## Approach

```
Data Sources                  Cloud Infrastructure         Analysis
────────────                  ────────────────────         ────────
Census 2011 (district-level)  AWS EC2 (compute)       →   Column pruning & cleaning
Housing data                  AWS S3 (data lake)       →   Population density scoring
Health infrastructure data    AWS RDS (structured DB)  →   Facility-to-population ratio
                              AWS Auto Scaling          →   Underserved region ranking
```

---

## Repository Structure

```
.
├── VCC_Analysis_Code.ipynb    # Full analysis notebook
└── README.md
```

---

## Key Analysis Steps (Notebook)

1. **Data cleaning** — prune Census 2011 to relevant columns (population, literacy, gender, district)
2. **Feature engineering** — compute per-district healthcare facility ratios
3. **Scoring** — rank districts by need index (population density vs. existing facilities)
4. **Visualisation** — identify and map top underserved districts

---

## Dataset

| Source | Description |
|---|---|
| Census 2011 | District-level population, literacy, gender demographics |
| Housing data | Household counts, housing conditions per district |
| Health infrastructure | Hospital/clinic counts per district |

---

## Cloud Setup (AWS)

The project was designed to run on AWS:

```
EC2 (t2.medium)  — Jupyter notebook server
S3               — Raw CSV storage and processed output
RDS (PostgreSQL) — Structured query layer over cleaned data
Auto Scaling     — Handle variable compute load
```

For local execution, clone the repo and run the notebook directly — no AWS account required for the analysis itself.

---

## Run Locally

```bash
git clone https://github.com/saiswaroopkakarla/Cloud-Based-Analytics-for-Identifying-Healthcare-Facility-Requirements-in-INDIA.git
pip install pandas numpy matplotlib seaborn jupyter
jupyter notebook VCC_Analysis_Code.ipynb
```

---

## Author

**Kakarla Sai Swaroop**, M25DE1023, IIT Jodhpur M.Tech Data Engineering  
Course: VCC (Virtualisation & Cloud Computing)
