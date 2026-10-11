# Technical Handover — CLV Predictive Analytics

## 1. System Overview

The solution comprises three components:

- Python-based data validation, preprocessing and machine learning.
- Retrospective holdout evaluation and forecast-bias reporting.
- Snowflake warehouse integration for churn scores and SHAP contributions.

The current implementation is an assessment prototype using a supplied 2,000-record dataset.

## 2. Data Inputs

Primary input: `Case_Study_2_CLV_Predictive_Analytics.csv`

Primary target: `churn_event_next_12m`

Identifier columns: `account_id`, `array_id`

The dataset includes missing churn labels, out-of-bound reported utilization, negative historical profits and UNKNOWN expansion signals.

Missing outcome labels are not treated as negative outcomes.

## 3. Feature Pipeline

The core model uses:

- Current capacity in TB.
- Reported utilization.
- Indicator for utilization above 100%.
- Expansion-potential category.

Numeric inputs undergo median imputation where necessary. Expansion categories are encoded using one-hot encoding.

Preprocessing is fitted within training pipelines to prevent validation or test data from influencing fitted transformations.

Historical net profit and contract date features were investigated separately but excluded from the final conservative configuration due to weak improvement and unresolved prediction-time availability.

## 4. Model Training and Validation

The labeled data is separated into account-grouped training, validation and test partitions.

Models:

- Random Forest classifier.
- XGBoost classifier.
- Constant 5% baseline.
- Training-prevalence benchmark.

Model selection prioritizes Brier score and log loss. ROC-AUC and average precision provide supplementary ranking information.

Grouped cross-validation is used during model development, and the held-out test data is evaluated after model selection.

## 5. Final Model Decision

The training-prevalence benchmark was selected because it achieved the lowest validation Brier score and log loss.

The final test Brier score was 0.1168, compared with 0.1232 for the existing static 5% assumption.

Random Forest and XGBoost were retained as comparison models. Neither demonstrated improved probability accuracy over the training-prevalence benchmark.

The selected estimator provides an overall churn probability, not individualized array-risk discrimination.

## 6. Snowflake Warehouse

Database: `EVERPURE_CLV`

Schema: `ANALYTICS`

Tables:

- `CLV_SCORED_STAGING`
- `CLV_SCORED_OUTPUT`
- `CLV_SHAP_EXPLANATIONS`

The scoring CSV is uploaded into staging. A SQL MERGE updates or inserts records into the final output table using array ID and model version as matching keys.

SHAP explanations are currently loaded separately. Repeated SHAP loads require deduplication or an equivalent staging-and-MERGE process.

## 7. Validation Checks

Before using scored results, verify:

- Expected row counts.
- Unique array/model-version keys.
- Non-null array identifiers.
- Churn probabilities between 0 and 1.
- Valid timestamps and risk buckets.
- Completeness of SHAP feature contributions.
- Successful reruns without duplicate prediction rows.

The current demonstration stores 276 holdout predictions and 1,932 SHAP feature-contribution rows.

## 8. Retraining and Monitoring

For a future production implementation, monitoring should include:

- Changes in observed churn prevalence.
- Brier score and log loss on newly matured outcome cohorts.
- Predicted-versus-observed churn by segment.
- Calibration deterioration.
- Feature distribution drift.
- Changes in renewal and product-lifecycle behavior.

A retraining decision should be triggered by sustained deterioration against a predefined business tolerance, not by a single noisy metric.

Before deployment, all input features must be checked for availability at their intended scoring timestamps.

## 9. Long-Horizon Survival Modeling

A 15-year survival model requires longitudinal observations with event dates, censoring information and clear definitions of churn.

Array retirement, hardware refresh, renewal and genuine account loss may represent different outcomes.

With suitable historical data, time-to-event methods such as Cox models or discrete-time hazard models could estimate churn probability over time. Competing-risks methods could distinguish array retirement from customer loss.

Extrapolation beyond the observed data horizon would require separately documented assumptions and validation.

## 10. Production Limitations

This implementation is not a complete production deployment.

The assessment dataset does not provide:

- Historical feature snapshots.
- Event times and censoring labels.
- A confirmed definition of account-level churn.
- Verified 15-year future financial outcomes.
- Observed capacity-expansion labels.

The existing outputs should therefore be interpreted as a validated assessment demonstration rather than certified 15-year CLV predictions.