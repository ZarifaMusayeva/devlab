# Patient Readmission Risk Prediction

This project predicts early hospital readmission risk (within 30 days) for diabetic patients, and explains the model's decisions using SHAP.

## Dataset

[Diabetes 130-US Hospitals Dataset (UCI)](https://archive.ics.uci.edu/dataset/296/diabetes+130-us+hospitals+for+years+1999-2008) — around 101,766 inpatient encounters, 1999-2008.

## How to Run

1. Install dependencies:
   ```
   pip install -r requirements.txt
   ```
2. Place `diabetic_data.csv` and `IDS_mapping.csv` inside `data/raw/`.
3. Open and run `notebooks/patient_readmission_risk.ipynb` from top to bottom.

## What This Project Does

1. Cleans real, messy clinical data (missing values, death/hospice records, patient-level duplication).
2. Frames the problem as binary classification: readmitted within 30 days or not.
3. Splits data by patient (not by row) to avoid data leakage.
4. Handles class imbalance with weighted models instead of resampling.
5. Compares a Logistic Regression baseline against XGBoost.
6. Explains predictions with SHAP, both globally and for individual patients.
7. Checks whether the model relies on clinically meaningful signals or on sensitive attributes like race and gender.

## Results

| Model | ROC-AUC | PR-AUC |
|---|---|---|
| Logistic Regression (baseline) | 0.645 | 0.208 |
| XGBoost | 0.662 | 0.217 |

The strongest predictor is `number_inpatient` (how many times the patient was hospitalized before). Race has very low importance in the model; gender has a small but non-negligible effect.

See [`note.md`](./note.md) for detailed methodology and reasoning, and [`DOCUMENTATION.md`](./DOCUMENTATION.md) for a full code walkthrough.

## Tech Stack

pandas, numpy, scikit-learn, xgboost, shap, matplotlib, seaborn