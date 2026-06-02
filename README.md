# AI-For-Finance---Corporate-Bankruptcy-Prediction
# 🏆 Corporate Bankruptcy Prediction

### 1st Place — Padova × Vilnius International AI & Finance Challenge | AY 2025/2026

> **Predict which Italian and Lithuanian SMEs will go bankrupt within 12 months using three years of financial statement data.**

-----

## Results

|Metric           |Score                          |
|-----------------|-------------------------------|
|🥇 Prediction rank|**1st place**                  |
|ROC-AUC          |**0.786**                      |
|Accuracy         |**94.6%**                      |
|Precision        |**70.4%**                      |
|Recall           |**54.3%**                      |
|F1 Score         |**0.545**                      |

-----

## The Problem

Binary classification: given financial statement data for 1,780 Italian and Lithuanian SMEs observed across 2016–2018, predict which firms enter a state of insolvency within the following 12 months.

**Class imbalance:** only 7.9% of firms are bankrupt. A naive model predicting zero defaults scores 92.1% accuracy — and catches nothing. This motivates evaluation on AUC, precision, and recall rather than accuracy alone.

-----

## Methodology

### 1. Feature Engineering

Starting from 90 raw financial variables, we constructed **40 features** across 8 economic families:

|Family           |Proxy                              |Direction       |
|-----------------|-----------------------------------|----------------|
|Liquidity        |Working Capital / Total Assets     |↓ reduces risk  |
|Profitability    |EBIT / Total Assets                |↓ reduces risk  |
|Solvency         |Equity / Total Assets              |↓ reduces risk  |
|Leverage         |Debt / Total Assets                |↑ increases risk|
|Financial Burden |Financial Expenses / Revenue       |↑ increases risk|
|Momentum         |2-year trend per ratio             |directional     |
|Sector Benchmarks|Ratio minus industry median        |relative        |
|Crash Flags      |EBIT crash, solvency drop, WC drain|↑ increases risk|

Each feature maps to a distinct economic mechanism of distress — not just a statistical signal, but a story about how a firm deteriorates.

> Sector benchmarks computed on training data only — no data leakage.

### 2. Model

**L1-penalised Logistic Regression (Lasso)** — chosen for interpretability: each coefficient is a direct, economically readable signal.

### 3. Hyperparameter Tuning

GridSearch over 25 configurations via stratified 5-fold cross-validation, optimising ROC-AUC:

```
Penalty:       L1 (Lasso)
C:             0.05
Class weight:  {0: 1, 1: 2}  ← missing a failure costs 2× more than a false alarm
CV folds:      5 (stratified — preserves 7.9% default rate)
```

**Lasso retained 13 of 40 features**, dropping 27 noisy or redundant variables.

### 4. Threshold Optimisation

Classification threshold of **0.461** selected at the F1 peak using out-of-fold cross-validated probabilities — no data leakage into threshold selection.

### 5. Robustness Test

We tested 4 additional financial ratios to try to improve recall:

|Added ratio        |AUC  |Recall|F1   |Decision                     |
|-------------------|-----|------|-----|-----------------------------|
|Baseline (6 ratios)|0.786|0.429 |0.546|✅ KEPT                       |
|+ cash_ratio       |0.775|0.429 |0.511|❌ worse                      |
|+ stock/assets     |0.770|0.429 |0.500|❌ worse                      |
|+ debtors/sales    |0.770|0.429 |0.500|❌ worse                      |
|+ net_income/assets|0.765|0.429 |0.480|❌ worse (corr=0.99 with EBIT)|

**Parsimony is a result, not a limitation.** The Lasso had already found the signal ceiling.

-----

## Confusion Matrix

```
                Predicted Healthy   Predicted Bankrupt
Actual Healthy       324                   4
Actual Bankrupt       16                  12
```

- **12 True Positives** — bankruptcies correctly flagged
- **324 True Negatives** — healthy firms correctly cleared
- **4 False Positives** — manageable false alarms
- **16 False Negatives** — missed failures (most costly error)

-----

## Repository Structure

```
├── Log_Rev_V11.ipynb          # Full model pipeline (final version)
├── AI_and_Finance_Challenge_PPTX.pptx  # Presentation (12 slides)
├── data/
│   ├── challenge_train.xlsx           # Training set (1,780 firms + target)
│   ├── challenge_test_without_target_.xlsx   # Test set (446 firms)
│   └── challenge_submission_file.xlsx        # Submission template
└── README.md
```

> **Note:** the dataset is not publicly available — it was provided by the challenge organisers (University of Padova & Vilnius University). The notebook runs as-is if you have access to the data files.

-----

## How to Run

```bash
# Install dependencies
pip install pandas numpy scikit-learn category_encoders openpyxl matplotlib seaborn

# Open the notebook
jupyter notebook Log_Rev_V11.ipynb
```

Run all cells top to bottom. The final cell exports `final_submission.xlsx` with predictions for the 446 test firms.

-----

## References

- Altman, E. I. (1968). *Financial Ratios, Discriminant Analysis and the Prediction of Corporate Bankruptcy.* Journal of Finance, 23(4), 589–609.
- Altman, E. I. (1993, 2000). *Corporate Financial Distress and Bankruptcy* (2nd & 3rd ed.). Wiley Finance.
- Pedregosa et al. (2011). *Scikit-learn: Machine Learning in Python.* JMLR, 12, 2825–2830.
- Ambrosini, F. (2025). *Lecture 5: Prediction Models — Altman Z-Score and Scoring Models.* Financial Reporting & Financial Statement Analysis, University of Padova.