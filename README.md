# fed_interest_rate_analysis_2026
# Predicting Federal Reserve Rate Cuts Using Macroeconomic Indicators  
**Created:** 2025-12-19 — **Last updated:** 2026-02-05  

---

## 1. Problem Statement
Anticipating U.S. monetary policy decisions—especially **interest rate cuts**—is challenging because:

- Macroeconomic indicators (inflation, unemployment, yields) evolve at different speeds  
- Federal Reserve decisions are **rare events** and highly regime-dependent  
- Binary “cut / no cut” forecasts often hide uncertainty and are misleading  

**This project addresses the following problem:**  
> How can we estimate the **probability** of a Federal Reserve **rate cut** at the next FOMC meeting using publicly available macroeconomic data, in an interpretable and transparent way?

Instead of predicting a deterministic outcome, the project focuses on producing a **probabilistic early-warning signal**.

> ⚠️ This project is for educational and analytical purposes only and does not constitute financial advice.

---

## 2. Project Objective
The main objectives of this project are to:

- Build a **complete end-to-end macroeconomic modeling pipeline**
- Transform heterogeneous macro data into meaningful features
- Model **rate cut probability** rather than a hard classification
- Use an **interpretable model** suitable for small and imbalanced datasets
- Evaluate performance using metrics adapted to rare-event prediction

---

## 3. Tools & Technologies Used

### Data Source
- **Federal Reserve Economic Data (FRED)** via the `fredapi` Python library

### Programming Language
- **Python**

### Python Libraries
- **pandas** – data manipulation, resampling, dataset construction  
- **numpy** – numerical operations and time ordering  
- **fredapi** – access to FRED macroeconomic series  
- **scikit-learn**  
  - `StandardScaler` (feature normalization)  
  - `LogisticRegression` (core model)  
  - `Pipeline` (scaler + model)  
  - `TimeSeriesSplit` (walk-forward validation)  
  - Evaluation metrics: ROC-AUC, Precision/Recall, PR-AUC  
- **matplotlib** – histograms and basic plots  
- **seaborn** – boxplots and correlation heatmaps  
- **statsmodels** – installed for potential econometric extensions  
- **os** – secure handling of the FRED API key via environment variables  

---

## 4. Data Sources (FRED Series)

### Macroeconomic Indicators
- Effective Federal Funds Rate (EFFR): `DFF`  
- Consumer Price Index (CPI): `CPIAUCSL`  
- Core CPI: `CPILFESL`  
- Unemployment Rate: `UNRATE`  
- Retail Sales: `RSAFS`  
- 10-Year Treasury Yield: `DGS10`  
- 2-Year Treasury Yield: `DGS2`  

### Target Variable Source
- Federal Funds Target Rate: `DFEDTARU`

---

## 5. Data Preparation
Macroeconomic series are reported at different frequencies (daily, monthly).  
To ensure temporal consistency:

- All series are **resampled to monthly frequency** (`MS`)
- Aggregation uses the **monthly mean**
- Missing values are handled using **forward-fill**

This produces a clean, aligned monthly macroeconomic dataset.

**Limitation:**  
FOMC decisions occur on specific dates, but this project uses a **monthly proxy**, which is a practical simplification.

---

## 6. Feature Engineering
The focus is on capturing **economic dynamics**, not just levels.

### Engineered Features
- **Inflation (YoY)**  
  - `inflation_yoy` (CPI, 12-month % change)  
  - `core_inflation_yoy` (Core CPI, 12-month % change)  
- **Monetary policy momentum**  
  - `effr_change_3m` (3-month change in EFFR)  
- **Labor market dynamics**  
  - `unrate_change_3m` (3-month change in unemployment)  
- **Yield curve slope**  
  - `yield_curve_10y_2y` = 10Y − 2Y Treasury yields  

These features are designed to detect:
- Economic slowdowns
- Shifts in monetary policy regimes
- Financial stress signals

---

## 7. Target Variable Construction

### Original Encoding (3 classes)
From monthly changes in the target rate:
- `1` = rate hike  
- `0` = hold  
- `-1` = rate cut  

### Binary Reformulation
To focus on easing risk:
- `1` = **Rate cut**
- `0` = **No cut** (hold or hike)

This creates a **highly imbalanced dataset**, reflecting real-world conditions.

---

## 8. Exploratory Data Analysis (EDA)
EDA is used to validate economic intuition and feature relevance.

Analyses include:
- Boxplots of key indicators by Fed decision regime  
- Correlation heatmap of macroeconomic variables  

Key observations:
- Inflation measures are strongly correlated with short-term rates  
- Unemployment moves inversely with tightening cycles  
- The yield curve slope behaves as a leading cycle indicator  

---

## 9. Modeling Strategy

### Why Logistic Regression?
- Simple and **highly interpretable**
- Robust for **small datasets**
- Naturally outputs **probabilities**
- Well-suited for imbalanced classification

### Model Pipeline
- Feature scaling with `StandardScaler`
- `LogisticRegression` with `class_weight="balanced"`
- Solver: `lbfgs`
- High iteration limit to ensure convergence

### Final Feature Set
- `inflation_yoy`
- `unrate`
- `unrate_change_3m`
- `yield_curve_10y_2y`
- `effr_change_3m`

---

## 10. Train / Test Split
To avoid look-ahead bias, a **time-based split** is applied:

- **Training set:** data before `2021-01-01`
- **Test set:** data from `2021-01-01` onward

---

## 11. Model Evaluation

### Why Accuracy Is Not Enough
Rate cuts are rare → a model predicting “no cut” always would appear accurate but be useless.

### Metrics Used
- **ROC-AUC** (ranking quality)
- **Precision / Recall / F1-score**
- **PR-AUC** (especially relevant for rare events)

### Test Set Results (post-2021)
- ROC-AUC ≈ **0.81**
- Recall for cuts is relatively high
- Precision is low (expected with class imbalance)

👉 The model is best interpreted as an **early-warning probability signal**, not a strict classifier.

---

## 12. Walk-Forward Validation
A **walk-forward (time-series) validation** is implemented using `TimeSeriesSplit`.

- Some folds contain no cut events → metrics undefined
- On the evaluable fold:
  - ROC-AUC ≈ **0.67**
  - PR-AUC ≈ **0.52**

Results are noisy due to the scarcity of rate cuts.

---

## 13. Baseline Comparisons

1. **Always “No Cut”**
   - High accuracy
   - Zero detection capability

2. **Yield Curve Rule (10Y–2Y < 0)**
   - Captures cycle stress
   - Poor timing for actual policy decisions

The model outperforms both baselines in ranking cut risk.

---

## 14. Latest Signal
At the most recent observation:

- **Estimated probability of a rate cut:** ~ **32%**

### Regime Classification

➡️ Current regime: **Watch**  
Moderate easing risk, not a high-confidence signal.

---
