# Everpure — Customer Lifetime Value Predictive Analytics

## Project Overview

This project investigates whether machine learning can improve array-level churn-risk estimates compared with an existing static 5% churn assumption.

The workflow covers data-quality assessment, feature engineering, Random Forest and XGBoost modeling, probability calibration, forecast-bias analysis, SHAP explainability, and Snowflake integration.

## Dataset

The supplied dataset contains 2,000 storage-array records belonging to 1,000 customer accounts.

The target, `churn_event_next_12m`, represents an observed array-level churn event within the next 12 months.

Key data-quality findings:

- 116 missing churn labels.
- 862 utilization values above 100%.
- 58 negative historical-profit records.
- 111 UNKNOWN expansion-potential signals.
- No duplicate array IDs or duplicate records.

Missing churn labels were excluded from supervised learning. Other potentially meaningful observations were retained and documented.

## Modeling Approach

Four approaches were evaluated:

1. Static 5% churn assumption.
2. Training-prevalence benchmark.
3. Random Forest classifier.
4. XGBoost classifier.

Account-grouped training, validation, and test partitions were used to prevent arrays belonging to the same account from appearing across partitions.

Regularized models, grouped cross-validation, and probability-calibration methods were evaluated. Model selection emphasized Brier score and log loss.

## Final Test Results

| Model | Brier Score | Log Loss | ROC-AUC |
|---|---:|---:|---:|
| Static 5% | 0.1232 | 0.4460 | 0.5000 |
| Training Prevalence | **0.1168** | **0.3969** | 0.5000 |
| Random Forest | 0.1174 | 0.3984 | 0.5199 |
| XGBoost | 0.1179 | 0.4003 | 0.5155 |

The training-prevalence benchmark achieved the best probability accuracy among the evaluated methods. Neither ML model demonstrated additional improvement on Brier score or log loss.

## Forecast Bias

The static 5% assumption underestimated observed churn on the holdout test set by 8.41 percentage points.

Cohort-level analyses examined differences across storage capacity, expansion-potential categories, and contract-end years.

These comparisons represent retrospective held-out evaluation rather than a complete historical forecasting back-test.

## Snowflake Integration

Model predictions were loaded into Snowflake using a staging table and SQL MERGE workflow.

**Warehouse tables:**

- `CLV_SCORED_STAGING` — Imported prediction records.
- `CLV_SCORED_OUTPUT` — Final predictions and risk buckets.
- `CLV_SHAP_EXPLANATIONS` — Random Forest feature contributions.

Verification results:

- 276 scored test arrays loaded successfully.
- 1,932 SHAP contribution records stored.
- No missing or invalid selected churn scores.
- MERGE rerun confirmed without duplicate output records.

## Project Structure

```text
Everpure-CLV-Analytics/
├── data/
│   ├── raw/
│   └── processed/
├── notebooks/
│   ├── 01_Data_Understanding.ipynb
│   ├── 02_EDA_Preprocessing.ipynb
│   ├── 03_Baseline_and_Model_Training.ipynb
│   ├── 04_Model_Improvement.ipynb
│   ├── 05_Calibration_and_Final_Validation.ipynb
│   └── 06_Cohort_Bias_and_SHAP.ipynb
├── snowflake/
│   ├── 01_create_tables.sql
│   ├── 02_merge_predictions.sql
│   ├── 03_validation_queries.sql
│   └── 04_shap_explanations.sql
├── reports/
│   ├── Backtesting_Validation_Report.md
│   ├── Technical_Handover.md
│   └── figures/
├── outputs/
├── requirements.txt
└── README.md
```

## Running the Project

1. Create and activate a Python virtual environment.
2. Install dependencies using `pip install -r requirements.txt`.
3. Place the supplied CSV in `data/raw/`.
4. Open the notebooks in numerical order using Jupyter.
5. Run the notebooks to reproduce data preparation, model evaluation and scored outputs.
6. Execute the Snowflake SQL scripts in numerical order, loading the generated CSV files at the indicated steps.

## Limitations

The dataset lacks historical prediction snapshots, full survival-event histories, censoring information, and observed expansion outcomes.

Consequently, the project does not claim to validate 15-year churn or CLV forecasts.

Random Forest SHAP explanations describe model predictions, not causal drivers or explanations of the selected constant-prevalence estimator.

## Conclusion

The results demonstrate the value of comparing machine learning against meaningful baselines and assessing probability quality before introducing additional model complexity.

The project provides a reproducible analytical assessment and a demonstrated Snowflake scoring pipeline.