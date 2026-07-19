# Note.md — Patient Readmission Risk Prediction

This file explains my steps and choices in this project.

## 1. Target Framing

The `readmitted` column has 3 values: `<30`, `>30`, `NO`. I made a new column `target`, where `<30` = 1 (early readmission) and everything else = 0. I chose this because 30 days is the common clinical cutoff for "early readmission", and it is the most important risk to catch.

## 2. Data Cleaning and Leakage

I found some problems in the data:

- **Missing values.** `weight`, `payer_code`, and `medical_specialty` were missing in most rows, so I removed these columns. `max_glu_serum` and `A1Cresult` were also missing a lot, but here missing means "test was not done", so I filled them with a new category, "Not Tested", instead of dropping them.
- **Death and hospice patients.** Some rows have `discharge_disposition_id` values that mean the patient died or went to hospice care. These patients cannot be "readmitted", so keeping them would confuse the model. I removed these rows (2,423 rows).
- **Patient-level leakage.** The data has 97,109 rows but only 68,167 unique patients. This means some patients appear more than once (28,942 duplicate rows). If I split the data randomly, the same patient could be in both train and test, and the model could "remember" the patient instead of learning general patterns. To fix this, I used `GroupShuffleSplit` on `patient_nbr`, so each patient is only in one side (train or test). I checked this after splitting, and the overlap was 0.

## 3. Class Imbalance

Only about 11% of cases are early readmissions, so the classes are imbalanced. I used `scale_pos_weight` in XGBoost (value ≈ 7.73) to give more weight to the minority class. For the baseline model, I used `class_weight='balanced'` in Logistic Regression. I did not use resampling (like SMOTE), because weighting is simpler and works directly with the real data.

## 4. Baseline Comparison

I compared a simple Logistic Regression baseline with XGBoost:

| Model | ROC-AUC | PR-AUC |
|---|---|---|
| Logistic Regression (baseline) | 0.645 | 0.208 |
| XGBoost | 0.662 | 0.217 |

XGBoost is a little better than the baseline on both metrics. The improvement is small, which is normal for this kind of real, messy clinical data.

## 5. Evaluation Metrics

I did not use accuracy, because the classes are imbalanced (~89% are 0, ~11% are 1). A model that always predicts "0" would get high accuracy but would be useless. Instead, I used:

- **ROC-AUC**: how well the model separates the two classes overall.
- **PR-AUC (average precision)**: more informative than ROC-AUC when the positive class is small.
- **Precision, Recall, F1**: at the default threshold (0.5), the model gets recall = 0.60 for the positive class. This means it finds 60% of real early-readmission cases, which is more useful in a clinical setting than a model with high accuracy but low recall.

## 6. Model Interpretability (SHAP)

I used SHAP to explain the model, both globally (summary plot) and locally (individual cases, using force plots and waterfall plots).

**Global result.** The top features were:
- `number_inpatient` (past inpatient visits) — the strongest signal by far
- `discharge_disposition_id`
- `number_diagnoses`
- `diabetesMed`
- `age`
- `diag_1_group` (diagnosis category)
- `number_emergency`, `num_lab_procedures`, `time_in_hospital`

**Local result.** For one high-risk patient, the biggest reason for the high prediction was `number_inpatient = 11` (11 past hospital stays), which alone added +1.71 to the score. Other diagnosis-related features added small amounts.

## 7. Critical Insight — Can I Trust This Model?

I checked where `race` and `gender` features ranked in feature importance:

- `race` features had very low importance (ranked around 33–79 out of 100 features).
- `gender_Male` had low but not zero importance (ranked around 20).

**Conclusion:** the model relies mainly on clinically meaningful signals — how many times a patient was hospitalized before, how many diagnoses they have, and their discharge type. It does not rely heavily on race, which is a good sign. Gender has some small effect, so this is worth watching if the model is used in practice, but it is not a top driver of the predictions.

## 8. Limitations and Next Steps

- The model's PR-AUC (0.217) is still low in absolute terms, showing this is a hard problem — many important factors (like social situation, medication adherence) are not in this dataset.
- A fairness check across age groups, not just race and gender, could add more confidence.
- Calibrating the probabilities would make the risk scores more usable for real clinical decisions.