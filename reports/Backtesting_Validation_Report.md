# Customer Lifetime Value Predictive Analytics
## Back-Testing and Model Validation Report

**Case Study 2 — Everpure**  
**Evaluation:** Array-level churn prediction over 12 months

### 1. Executive Summary

This project evaluates whether machine learning can improve churn-risk estimates over an existing static 5% churn assumption.

Two supervised machine learning models, Random Forest and XGBoost, were developed and evaluated alongside the static assumption and a training-prevalence benchmark.

On the final holdout test set, the training-prevalence benchmark achieved the lowest Brier score of **0.1168**, compared with 0.1232 for the static 5% assumption, 0.1174 for Random Forest, and 0.1179 for XGBoost.

The results suggest that updating the overall churn assumption provides measurable improvement in probability accuracy, but the available features do not demonstrate additional predictive benefit from machine learning.

### 2. Dataset and Data Quality

The supplied dataset contains 2,000 storage-array records belonging to 1,000 customer accounts, with two arrays per account.

The initial data audit identified:

- 116 missing churn labels (5.8%).
- 862 utilization values exceeding 100%.
- 58 records with negative historical net profit.
- 111 records with an UNKNOWN expansion-potential signal.
- No duplicate rows or duplicate array identifiers.

Missing target labels were excluded from supervised training. Negative profits and utilization values exceeding 100% were preserved rather than automatically removed, because their business meaning could not be conclusively established.

### 3. Modeling and Validation Methodology

The primary target was `churn_event_next_12m`, indicating whether a storage array experienced churn within the following 12 months.

The modeling process used account-grouped training, validation, and test partitions to prevent arrays from the same account appearing in different partitions.

The final labeled partitions contained:

| Partition | Records |
|---|---:|
| Training | 1,328 |
| Validation | 280 |
| Test | 276 |

Four approaches were compared:

1. Static 5% churn probability.
2. Training-prevalence probability benchmark.
3. Random Forest classifier.
4. XGBoost classifier.

Regularized models were evaluated using account-grouped cross-validation. A conservative feature set was preferred after expanded features failed to improve performance.

Sigmoid and isotonic calibration were evaluated for Random Forest, but neither improved validation probability accuracy.

Brier score and log loss were prioritized because probability reliability is important for financial forecasting.

### 4. Final Holdout Performance

| Model | Brier Score ↓ | Log Loss ↓ | ROC-AUC ↑ |
|---|---:|---:|---:|
| Static 5% | 0.1232 | 0.4460 | 0.5000 |
| Training Prevalence | **0.1168** | **0.3969** | 0.5000 |
| Random Forest | 0.1174 | 0.3984 | **0.5199** |
| XGBoost | 0.1179 | 0.4003 | 0.5155 |

The training-prevalence benchmark reduced Brier score by approximately 5.2% relative to the static assumption.

However, ROC-AUC values close to 0.5 indicate weak individual-level discrimination.

The findings support improving the overall churn-rate estimate before introducing more complex individualized risk models.

### 5. Forecast Bias Analysis

Forecast bias was calculated as:

**Forecast Bias = Predicted Churn Rate − Observed Churn Rate**

A negative result indicates underestimated churn and therefore an overly optimistic retention assumption.

| Model | Predicted Churn | Observed Churn | Bias |
|---|---:|---:|---:|
| Static 5% | 5.00% | 13.41% | −8.41 pp |
| Training Prevalence | 16.11% | 13.41% | +2.71 pp |
| Random Forest | 16.53% | 13.41% | +3.12 pp |
| XGBoost | 16.39% | 13.41% | +2.99 pp |

The existing 5% baseline underestimated observed churn on the final test set.

Additional analysis across capacity, expansion-signal and contract-year groups revealed differences in forecast bias.

The largest static-baseline underestimation occurred in:

- Medium-capacity arrays: −10.05 percentage points.
- Medium expansion-potential arrays: −11.49 percentage points.
- Contracts ending in 2026: −17.73 percentage points.

These findings are exploratory because some groups contain relatively few observations and records within accounts are related.

### 6. Model Explainability

SHAP was applied to the Random Forest comparator to explain how the model's input features contributed to its churn probabilities.

Feature-level SHAP contributions were generated for 276 held-out arrays across seven encoded features.

A total of 1,932 SHAP records were successfully stored in Snowflake, with no missing contribution values.

The selected prevalence benchmark produces a constant probability and therefore does not provide individualized feature explanations.

SHAP explanations describe the Random Forest's predictions and should not be interpreted as causal relationships.

### 7. Limitations and Recommendations

**Historical back-testing:** The dataset does not contain historical prediction snapshots, so the analysis is a retrospective holdout comparison rather than a complete chronological forecast back-test.

**Long-horizon modeling:** The supplied 12-month churn labels do not provide the event times, censoring information, competing-event labels or financial trajectories needed to validate 15-year survival and CLV forecasts.

**Array versus account churn:** Discontinuation of an individual array cannot automatically be interpreted as complete customer loss.

**Capacity expansion:** The expansion-potential signal is an input feature, not an observed future expansion outcome.

**Recommendations:** The static churn assumption should be reviewed against representative observed outcomes. Future modeling should prioritize richer customer history, renewal events, contract trajectories and product-lifecycle information.

The present analysis does not establish that more complex ML models deliver additional value beyond the training-prevalence benchmark. Their operational use would require stronger validation and additional historical data.

### Conclusion

This case study demonstrates a complete analytical workflow covering data-quality assessment, machine learning, probability calibration, comparative validation, cohort-level bias analysis, SHAP explainability, and Snowflake integration.

The strongest observed improvement came from correcting the static churn assumption rather than from introducing model complexity.

This highlights the importance of establishing appropriate baselines and validating probability estimates before using predictive models in long-term financial calculations.