# Donor Pledge Prediction & Segmentation

## Overview

This project predicts annual donor pledge amounts and segments donors into actionable fundraising groups for outreach prioritization.

The project was independently developed using approximately 10 years of historical donation data obtained through collaboration with the BYU-Idaho Data Science Society. The dataset originated from an anonymous organization and combined records from multiple internal relational database tables.

The primary business objective was to improve fundraising efficiency by replacing manual donor prioritization with a predictive workflow grounded in historical giving behavior, reliability, and donor engagement patterns.

**Link to Notebook and Analysis: https://manolovm21.github.io/donor_project/**
---

## Business Problem

Fundraising agents manage large donor portfolios with limited analytical guidance.

Without prioritization, high-value donors can be overlooked while agents spend time on low-probability outreach.

This project addresses two operational questions:

1. **How much is each donor likely to pledge next year?**
2. **Which donors deserve the highest outreach priority?**

The final system produces:

* Predicted annual pledge amounts
* Donor reliability scores
* Operational donor segments for fundraising strategy
* Tableau-ready exports for downstream reporting

---

## Dataset

### Data Sources

The modeling table was constructed from six internal MySQL tables joined through `FanID`.

Included data:

* Historical pledges and payments
* Demographic attributes
* Membership tier information
* Education history
* Wealth indicators (iWave)
* Board membership and engagement data

### Scope

* ~45,000 active donors with pledge history
* ~706,000 total records existed in the broader donor system
* Time range: 2014–2025

The model only targeted donors with historical giving behavior. Prospecting for entirely new donors would require a separate modeling framework.

---

## Exploratory Data Analysis

### Regression vs Classification

An early design decision was whether to frame the problem as:

* Classification → predicting whether a donor would fail to fulfill a pledge
* Regression → predicting donation amount

Analysis showed only ~3% of donors were non-compliant with pledges, making classification highly imbalanced and operationally less useful.

The project therefore focused on regression.

### Target Distribution

Donation amounts were heavily right-skewed:

* Median donation ≈ $650
* Maximum donation > $790,000

To stabilize variance and reduce sensitivity to extreme donors, the target variable used:

genui{"math_block_widget_always_prefetch_v2":{"content":"y = \log(1 + \text{Annual Pledge Amount})"}}

Predictions were transformed back to dollar scale using `expm1()` for interpretation.

---

## Feature Engineering

Several preprocessing decisions were designed to reduce noise, preserve signal, and avoid leakage.

### Membership Tier Encoding

Organization donor membership levels were ordinal by nature.

Instead of one-hot encoding, tiers were mapped numerically:

| Tier         | Encoding |
| ------------ | -------- |
| Legacy Chief | 10       |
| Silver Chief | 9        |
| Golden Chief | 8        |
| ...          | ...      |
| Spirit       | 1        |

This preserved the hierarchical relationship between donor tiers.

### Degree Consolidation

More than 90 unique degree labels were consolidated into five broader categories using custom logic.

This reduced dimensionality while preserving educational signal.

### Geographic Encoding

Rather than one-hot encoding all 50 states, donors were grouped into:

* Florida
* Southeast
* Northeast
* Midwest
* West

Florida received special treatment because approximately 75% of donors were located there.

### Missing Data Strategy

Missingness itself contained predictive information.

Instead of simply imputing values, the pipeline created explicit indicators such as:

* `Age_null`
* `Properties_null`

Median age imputation was computed strictly on the training set to avoid temporal leakage.

---

## Modeling Strategy

### Temporal Validation

The project intentionally avoided random train/test splits.

A random split would leak future donor behavior into the training process.

Instead, the pipeline used chronological separation:

| Split      | Years     |
| ---------- | --------- |
| Training   | 2014–2022 |
| Validation | 2023      |
| Holdout    | 2024      |

This structure better simulated real deployment conditions.

---

## Baseline Model

A Ridge Regression model served as the baseline.

Ridge was selected because temporal donation features were highly correlated and required regularization.

The baseline established a reference point before introducing gradient boosting methods.

---

## Final Model — XGBoost

### Why XGBoost?

XGBoost was selected because it:

* Handles nonlinear relationships
* Captures interaction effects
* Performs well on tabular structured data
* Is robust to mixed feature types
* Handles skewed distributions effectively

### Hyperparameter Tuning

The final model used:

* `RandomizedSearchCV`
* 25 hyperparameter combinations
* 3-fold cross-validation

Optimization focused primarily on minimizing MAE.

---

## Evaluation

### Primary Metrics

Model performance was evaluated using:

* MAE (Mean Absolute Error)
* RMSE (Root Mean Squared Error)

MAE was prioritized because RMSE became dominated by mega-donors with unusually large pledges.

### Validation Performance

| Metric | Result                       |
| ------ | ---------------------------- |
| MAE    | ≈ $1,145                     |
| R^2    | ≈ 0.750                      |

The model outperformed a naïve baseline that simply predicted each donor would repeat the previous year's donation.

### Key Observation

A small number of ultra-high-value donors disproportionately increased RMSE.

Segmented evaluation showed substantially stronger performance across the majority of the donor population once those outliers were isolated.

---

## Feature Importance

The most influential feature was:

* `TotalPledgeAnnual`

This indicated that current-year giving behavior was the strongest predictor of future commitments.

Additional important signals included:

* Membership tier
* Wealth indicators
* Historical payment behavior
* Donor engagement variables

---

## Model Explainability

SHAP values were used to interpret both:

* Global model behavior
* Individual donor predictions

This improved transparency for fundraising stakeholders and allowed analysts to explain:

* Why certain donors received high predictions
* Which variables most influenced each estimate
* How behavioral patterns differed across donor types

Two contrasting donor profiles were analyzed to demonstrate how the model differentiated between high-value and low-value contributors.

---

## Donor Segmentation

After prediction, donors were segmented using:

* Predicted pledge amount
* Historical payment reliability

Reliability was defined using commitment fulfillment history.

The operational framework produced four outreach quadrants:

| Segment        | Predicted Amount | Reliability | Recommended Strategy                           |
| -------------- | ---------------- | ----------- | ---------------------------------------------- |
| VIP            | High             | 100%        | Relationship maintenance and priority outreach |
| High Potential | High             | <100%       | High upside but requires closer follow-up      |
| Stable Base    | Low              | 100%        | Consistent low-maintenance donors              |
| Low Priority   | Low              | <100%       | Lower outreach investment                      |

The segmentation output was exported to CSV and integrated into Tableau dashboards used by fundraising staff.

---

## Key Findings

* Donation behavior is highly persistent over time
* Temporal validation is critical in fundraising prediction problems
* Extreme donors distort RMSE and can hide strong operational performance
* Reliability metrics add important context beyond pure donation amount
* Feature engineering decisions had major impact on downstream performance
* Explainability tools increased stakeholder trust in model outputs

---

## Limitations

* The model does not address donor acquisition or prospecting
* Performance on the 2025 holdout set was affected by atypical pledge distributions
* Historical pledge amount dominated model importance, limiting sensitivity to sudden behavioral changes
* The dataset represented active donors only, not the full donor ecosystem

---

## Tools & Technologies

### Languages & Libraries

* Python
* pandas
* NumPy
* scikit-learn
* XGBoost
* SHAP
* Matplotlib
* MySQL

### Techniques

* Regression Modeling
* Feature Engineering
* Temporal Cross-Validation
* Hyperparameter Optimization
* Model Explainability
* Donor Segmentation
* Data Leakage Prevention

---

## Collaboration Context

The data was obtained in colaboration with BYUI Data Science Society.

### My Contributions

* SQL feature engineering
* Data cleaning and preprocessing
* Exploratory data analysis
* Regression model development
* Hyperparameter tuning
* Model evaluation
* SHAP explainability analysis
* Donor segmentation framework
* Tableau export pipeline


