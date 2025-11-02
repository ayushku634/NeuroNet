# 🏥 Patient Data Integration and Feature Engineering Pipeline

## 📘 Overview
This project builds a **comprehensive patient-level dataset** by integrating and aggregating multiple healthcare data sources — such as patient demographics, diagnoses, care records, visits, and risk assessments.  
The goal is to create a **clean, feature-rich dataset** ready for training predictive models (e.g., patient readmission risk, critical condition prediction, etc.).

---

## ⚙️ Pipeline Summary

### **Step 0 — Load Cleaned Data**
The notebook starts by loading several pre-cleaned CSVs:
- `patient_cleaned.csv` — core patient demographic and baseline data  
- `diagnosis_cleaned.csv` — patient-level diagnosis information  
- `care_cleaned.csv` — medical care and measurements  
- `risk_cleaned.csv` — risk assessment features  
- `visit_cleaned.csv` — hospital and outpatient visit records  

These are merged later using the `patient_id` key.

---

### **Step 1 — Diagnosis Data Aggregation**
Aggregates multiple diagnosis records per patient to extract features such as:
- `num_diagnoses` — total diagnosis count  
- `num_chronic_conditions` — count of chronic diagnoses  
- `unique_conditions` — number of unique conditions per patient  
- `has_cancer`, `has_diabetes`, `has_hypertension` — binary indicators for key conditions  

This creates one diagnosis summary row per patient.

---

### **Step 2 — Care Data Processing**
Handles temporal data from medical care records:
- Converts `last_care_dt` and `next_care_dt` to proper datetime objects.  
- Removes unrealistic dates (e.g., >2100 or <1950).  
- Computes care interval metrics:  
  - `days_to_next_care` — time gap between visits  
- Aggregates care statistics:  
  - `num_care_records`, `avg_days_since_last_care`, `avg_days_to_next_care`  
  - `has_care_gap`, `avg_msrmnt_value`, `unique_msrmnt_types`, etc.  

All missing values are filled with zeros for consistency.

---

### **Step 3 — Visit Type Aggregation**
Counts how many times each patient visited different facilities (e.g., ER, Urgent Care, Outpatient).
- Each visit type becomes a column (e.g., `num_ER_visits`, `num_URGENT_CARE_visits`).
- Ensures no missing types by filling with `0`.

---

### **Step 4 — Visit Duration and Follow-up Metrics**
Processes visit timelines:
- Converts `visit_start_dt`, `visit_end_dt`, and `follow_up_dt` into datetimes.  
- Computes:
  - `visit_duration_days` — length of stay  
  - `days_to_followup` — days until follow-up appointment  
  - `has_followup` — binary indicator (1 if follow-up scheduled)
- Handles extreme or missing values by clipping to `[0, 365]` days.

Derived metrics include:
- `num_visits`, `num_readmissions`, `unique_visit_types`  
- `avg_visit_duration`, `max_visit_duration`, `avg_days_to_followup`  
- `days_since_last_visit` (relative to the latest date in the dataset)

---

### **Step 5 — Merge All Data Sources**
Combines all aggregates into a single **patient-level dataset**:

```python
merged_df = (
    patient_df
    .merge(diagnosis_agg, on='patient_id', how='left')
    .merge(care_agg, on='patient_id', how='left')
    .merge(visit_agg, on='patient_id', how='left')
    .merge(risk_df, on='patient_id', how='left')
)
