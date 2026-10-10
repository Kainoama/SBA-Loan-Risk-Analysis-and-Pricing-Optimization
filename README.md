# SBA Loan Default Prediction and Risk Analysis

## Overview

This project uses historical U.S. Small Business Administration (SBA) 7(a) loan records to explore default prediction and loan-level profitability. It compares three classification models, examines borrower and loan risk patterns, estimates expected loss under stated assumptions, and exports risk-scored records for Tableau analysis.

The original proposal also envisioned an interest-rate optimization simulator that accounts for customer price sensitivity. The submitted notebook develops risk models and profitability calculations, but does not demonstrate a completed price-elasticity model or a validated optimal-rate search.

## Project Highlights

- Analyzed **357,866 original loan records** with 43 columns and removed **377 duplicate rows**.
- Restricted modeling to resolved loans (paid in full or charged off), then excluded loans approved in 2025 or later.
- Built a modeling dataset of **60,799 loans** with 12 features and a stratified 80/20 train-test split.
- Compared XGBoost, Random Forest, and Logistic Regression.
- XGBoost achieved an AUC of **0.963** and **86.0% recall** for defaulted loans on the test set.
- Created low-, medium-, and high-risk tiers and exported test-set records for Tableau exploration.
- Produced scenario-based financial estimates, including a **$71.6 million net model economic benefit** under the notebook's assumptions. This is not realized savings or a forecast of actual lender performance.

## Dataset

The project uses the U.S. Small Business Administration's public 7(a) and 504 FOIA loan dataset: [SBA 7(a) & 504 FOIA Dataset (FY2020–Present)](https://data.sba.gov/en/dataset/7-a-504-foia/resource/d67d3ccb-2002-4134-a288-481b51cd3479).

The source contains historical loan-level information, including loan characteristics and outcomes. The analysis in this project focuses on SBA 7(a) records available in the downloaded dataset.

## Data Preparation

The initial dataset contained 357,866 records and 43 columns. The workflow included:

1. Inspecting data types, missingness, and loan outcomes.
2. Removing 377 duplicate records.
3. Converting date and numeric fields to appropriate types.
4. Imputing missing numeric values with medians and assigning `"Unknown"` to missing categorical values.
5. Restricting the data to loans marked paid in full (PIF) or charged off (CHGOFF).
6. Excluding loans approved in 2025 or later from modeling because their outcomes may not have had sufficient time to mature.
7. Engineering `guarantee_ratio`, `loan_to_term_ratio`, and `is_real_estate`.
8. Encoding six categorical features and using 12 features for model training.

The resolved-loan subset contained 61,648 records before the removal of three rows with blank business type and the exclusion of loans approved in 2025 or later. The final modeling dataset contained **60,799 loans**, including 4,861 charged-off loans (approximately **8.0%** of the modeling data). This default rate describes the filtered modeling sample, not the original 357,866-record dataset.

## Modeling Approach

Three classification models were trained and evaluated using a stratified 80/20 train-test split:

- **XGBoost**, with class weighting to account for the minority default class
- **Random Forest**, using balanced class weights
- **Logistic Regression**, using balanced class weights and standardized features

### Test-set performance

| Metric | XGBoost | Random Forest | Logistic Regression |
|---|---:|---:|---:|
| AUC | 0.963 | 0.961 | 0.858 |
| Recall — Defaults | 0.860 | 0.637 | 0.817 |
| Precision — Defaults | 0.656 | 0.849 | 0.241 |
| Recall — Paid Back | 0.961 | 0.990 | 0.777 |
| Accuracy | 95% | 96% | 78% |

XGBoost had the highest AUC and identified the largest share of actual defaults among the three models. Random Forest had higher precision for flagged defaults but missed more actual defaults. Logistic Regression served as a baseline.

## Probability Calibration and Risk Tiers

The notebook experimented with isotonic calibration to adjust predicted probabilities of default (PD). However, the calibration model was fitted using the test set itself. Because the test labels were used during calibration, the calibrated probabilities and resulting risk-tier distribution should **not** be treated as an independent test of calibration quality. A stronger evaluation would fit calibration using a separate validation set or cross-validation, then evaluate once on an untouched test set.

The notebook's risk tiers were assigned using the calibrated probabilities:

- **Low risk:** PD below 5%
- **Medium risk:** PD from 5% to below 15%
- **High risk:** PD of 15% or higher

The exported test set contained:

| Risk tier | Loans | Share of test set |
|---|---:|---:|
| Low risk | 10,435 | 85.8% |
| Medium risk | 467 | 3.8% |
| High risk | 1,258 | 10.4% |

One implementation detail matters: the notebook uses the **raw XGBoost probabilities** in its expected-loss calculation, while the **calibrated probabilities** are used to assign risk tiers. The README therefore does not characterize expected-loss estimates as based on calibrated probabilities.

## Financial Analysis

### Expected loss and modeled net profit

The notebook calculates expected loss using:

`Expected Loss = Probability of Default × Loss Given Default × Unguaranteed Exposure`

where unguaranteed exposure is calculated as:

`Gross Approval − SBA Guaranteed Approval`

For its loan-level net-profit estimate, the notebook assumes:

- Loss Given Default (LGD): **50%**
- Cost of funds: **4%** of gross loan approval
- Operating expenses: **1%** of gross loan approval
- Interest income: gross loan approval multiplied by the recorded interest rate

The notebook reports approximately **$2,294 average net profit per test-set loan** and **$27.9 million total modeled net profit** across the test set. These are scenario-based calculations using the stated assumptions, not observed lender profits. The formula is a simplified estimate and does not include a demonstrated rate-optimization or customer-elasticity model.

### Estimated classification impact

A separate notebook calculation reports:

- **$78.7 million** in potential losses avoided for true positives
- **$7.1 million** in opportunity cost for false positives
- **$71.6 million** in net estimated benefit

This calculation assumes losses equal to 60% of the full loan amount for correctly flagged defaults and an opportunity cost equal to 10% of the full loan amount for false positives. It does **not** use the SBA-guarantee-adjusted expected-loss formula above. These figures are illustrative outputs of the project's assumptions and should not be interpreted as realized savings, validated causal impact, or guaranteed future results.

## Exploratory Findings

The exploratory analysis suggested the following patterns in the filtered loan data:

- **Business age:** Newer businesses had higher observed default rates. Records with unknown business age also showed elevated default rates.
- **Business type:** The notebook's descriptive analysis found differences in observed default rates across business types.
- **Loan size:** Smaller loans appeared to have a higher concentration of defaults in the dataset. This is an association and does not establish why the pattern occurs.
- **Interest rates:** Defaults appeared across rate ranges. Records with a 0% interest rate were flagged for data-quality review.

These findings are descriptive and should not be interpreted as causal effects.

## Tableau Export

The notebook exports a test-set CSV containing model scores, risk tiers, and loan-level financial calculations for further exploration in Tableau. The proposal envisioned dashboards for industry/geographic risk profiling and pricing scenarios. The notebook supports risk and profitability exploration, but a completed dynamic pricing optimizer and customer price-elasticity analysis are not demonstrated in the submitted notebook.

## Tools and Methods

- **Python:** pandas, scikit-learn, XGBoost, Matplotlib, seaborn
- **Modeling:** XGBoost, Random Forest, Logistic Regression
- **Evaluation:** AUC, recall, precision, accuracy, stratified train-test split
- **Data preparation:** Deduplication, missing-value handling, feature engineering, categorical encoding
- **Calibration:** Isotonic calibration experiment
- **Financial analysis:** Expected loss, unguaranteed exposure, simplified net-profit and classification-impact estimates
- **Visualization:** Tableau export and exploratory plots

## Limitations

- Modeling was restricted to resolved loans, so results may not generalize to loans without mature outcomes.
- Defaults were a minority class, requiring trade-offs between missed defaults and false alarms.
- The isotonic calibration procedure was fitted on the test set, so calibration performance was not independently evaluated.
- The financial impact figures depend on simplified assumptions and use a separate calculation from the SBA-adjusted expected-loss formula.
- The submitted notebook does not demonstrate a validated price-elasticity model or completed optimal-interest-rate search.
- Label encoding represents categories as integers; for nominal categories, this can introduce artificial ordering into some models.
- Observed associations do not establish causation, and historical model performance does not guarantee future results.

## Project Context and Team

This project was completed as part of **Project Labs with the UCL Data Science Society**.

### Team Members and Responsibilities

- **Lohla** — Data cleaning and preprocessing; model training
- **Sarah** — Exploratory data analysis (EDA); conclusions and findings
- **Jocelyn** — Feature engineering; model evaluation
- **Kainoa Martinez** — Model training; conclusions and findings

## Author

**Kainoa Martinez**
