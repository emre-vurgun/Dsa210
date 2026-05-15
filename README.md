# Analysis of EV Sales in the US

## Data Sources

This project integrates multiple heterogeneous datasets covering **2010–2026**, aligned temporally into a monthly format:

**1. US EV Sales (BEV + PHEV)**
- **Source:** Argonne National Laboratory
- **Frequency:** Monthly
- **Features:** Total EV Sales (Battery Electric + Plug-in Hybrid)

**2. Crude Oil Prices (WTI)**
- **Source:** Yahoo Finance API (`yfinance`)
- **Frequency:** Monthly
- **Usage:** Monthly closing prices for WTI Crude Oil.

**3. US GDP per Capita**
- **Source:** Federal Reserve Economic Data (FRED - A939RX0Q048SBEA)
- **Frequency:** Monthly (interpolated from quarterly)
- **Usage:** Represents consumer purchasing power and economic health.

**4. Public Interest (Google Trends)**
- **Source:** Google Trends (`multiTimeline.csv`)
- **Keyword:** "electric car" (US Region)
- **Usage:** Scaled search volume (0–100) acting as a proxy for consumer interest.

**5. US Tariff Policy**
- **Source:** Derived from US Policy Announcements
- **Usage:** Binary dummy variable — 1 after May 2024 (100% tariff on Chinese EV imports).

---

## Detailed Analysis & Code

All data collection, exploratory data analysis, hypothesis testing, and machine learning models are fully documented in the notebooks.

### Data Analysis Pipeline

**1. Data Preparation**
- Formatted various date strings into standardised `datetime` objects.
- Merged four disparate data sources into a single temporal DataFrame.
- Applied **Min-Max Normalisation** (0–1) to allow fair comparison across scales.
- Engineered a binary `Tariff_Active` feature for chronological policy tracking.

**2. Exploratory Data Analysis (EDA)**
- Overlaid time-series plots comparing normalised sales, oil prices, and search trends.
- Pearson correlation heatmaps across all features.
- Box-plots to evaluate EV sales distributions before and after the May 2024 tariffs.

---

## Hypothesis Testing

Evaluated using **Pearson Correlation** and **Welch's Independent T-Tests** (α = 0.05):

| Hypothesis | Description | Result |
|:---|:---|:---|
| **H1** | Higher GDP per Capita correlates with higher EV sales. | **Confirmed** (r = 0.916, p < 0.001) |
| **H2** | Increased Google Search volume correlates with higher sales. | **Confirmed** (r = 0.734, p < 0.001) |
| **H3** | High oil prices drive EV sales. | **Rejected** (p = 0.59) |
| **H4** | Mean EV sales differ significantly after the May 2024 tariffs. | **Confirmed** (Sales significantly higher post-tariff, p < 0.001) |

---

## Machine Learning Models

Four regression models were implemented to predict US EV sales: Linear Regression, k-Nearest Neighbors (kNN), Random Forest, and Decision Tree.

The dataset was enriched with **Feature Engineering** — adding historical sales momentum (`sales_lag1`, `sales_lag3`, `sales_lag12`) and temporal dynamics (`month`, `year`).

### Model Setup
- **Split:** Chronological — Training: Jan 2011 → May 2024 | Testing: Jun 2024 → 2026
- **Scaling:** `MinMaxScaler` fitted on training data only (no look-ahead bias)
- **Evaluation Metrics:** RMSE, MAE, R², and MAPE

---

## Results Summary

All four models ranked across every metric. **Random Forest** is the clear winner on all measures:

| Model | RMSE (Vehicles) | MAE (Vehicles) | R² | MAPE (%) |
|:---|---:|---:|---:|---:|
| **Random Forest** ✅ | **26,912** | **20,463** | **0.10** | **19.53%** |
| Decision Tree | 28,217 | 21,859 | 0.01 | 20.93% |
| kNN | 29,432 | 23,247 | -0.08 | 22.45% |
| Linear Regression | 36,416 | 28,357 | -0.65 | 28.79% |

> **RMSE & MAE** are in vehicle units. **R² > 0** means the model beats a mean-only baseline. **MAPE** gives an intuitive percentage error.

---

## Visualisations

The charts below are generated from the final model results. Run each block after training your models to reproduce them.

### 1. RMSE & MAE Comparison

```python
import matplotlib.pyplot as plt
import numpy as np

models = ['Random Forest', 'Decision Tree', 'kNN', 'Linear Regression']
rmse   = [26912.10, 28217.18, 29432.09, 36415.58]
mae    = [20462.68, 21858.94, 23247.01, 28356.84]
colors = ['#2E86AB', '#A23B72', '#F18F01', '#C73E1D']

x     = np.arange(len(models))
width = 0.35

fig, ax = plt.subplots(figsize=(11, 5.5))
bars1 = ax.bar(x - width/2, rmse, width, label='RMSE', color=colors, alpha=0.9)
bars2 = ax.bar(x + width/2, mae,  width, label='MAE',  color=colors, alpha=0.5)

ax.set_ylabel('Vehicles', fontsize=12)
ax.set_title('Model Comparison — RMSE & MAE (lower is better)', fontsize=13, fontweight='bold')
ax.set_xticks(x)
ax.set_xticklabels(models, fontsize=11)
ax.set_ylim(0, 43000)
ax.yaxis.set_major_formatter(plt.FuncFormatter(lambda v, _: f'{int(v):,}'))
ax.legend(fontsize=11)
ax.grid(axis='y', alpha=0.3)

for bar in bars1:
    ax.text(bar.get_x() + bar.get_width() / 2, bar.get_height() + 400,
            f'{int(bar.get_height()):,}', ha='center', va='bottom', fontsize=9, fontweight='bold')
for bar in bars2:
    ax.text(bar.get_x() + bar.get_width() / 2, bar.get_height() + 400,
            f'{int(bar.get_height()):,}', ha='center', va='bottom', fontsize=9)

plt.tight_layout()
plt.savefig('chart_rmse_mae.png', dpi=150, bbox_inches='tight')
plt.show()
```

---

### 2. MAPE Comparison

```python
import matplotlib.pyplot as plt

models = ['Random Forest', 'Decision Tree', 'kNN', 'Linear Regression']
mape   = [19.53, 20.93, 22.45, 28.79]
colors = ['#2E86AB', '#A23B72', '#F18F01', '#C73E1D']

fig, ax = plt.subplots(figsize=(9, 4.5))
bars = ax.barh(models, mape, color=colors, alpha=0.88)

ax.set_xlabel('MAPE (%)', fontsize=12)
ax.set_title('Mean Absolute Percentage Error — Lower is Better', fontsize=13, fontweight='bold')
ax.set_xlim(0, 35)
ax.axvline(20, color='gray', linestyle='--', linewidth=1.2, alpha=0.6, label='20% threshold')
ax.legend(fontsize=10)
ax.grid(axis='x', alpha=0.3)

for bar, val in zip(bars, mape):
    ax.text(bar.get_width() + 0.4, bar.get_y() + bar.get_height() / 2,
            f'{val:.1f}%', va='center', fontsize=11, fontweight='bold')

plt.tight_layout()
plt.savefig('chart_mape.png', dpi=150, bbox_inches='tight')
plt.show()
```

---

### 3. R² Score Comparison

```python
import matplotlib.pyplot as plt

models = ['Random Forest', 'Decision Tree', 'kNN', 'Linear Regression']
r2     = [0.10, 0.01, -0.08, -0.65]
colors = ['#2E86AB' if v >= 0 else '#C73E1D' for v in r2]

fig, ax = plt.subplots(figsize=(9, 4.5))
ax.bar(models, r2, color=colors, alpha=0.88)
ax.axhline(0, color='black', linewidth=1)

ax.set_ylabel('R² Score', fontsize=12)
ax.set_title('R² Score by Model (positive = beats mean-only baseline)', fontsize=12, fontweight='bold')
ax.set_ylim(-0.85, 0.25)
ax.grid(axis='y', alpha=0.3)

for i, val in enumerate(r2):
    ypos = val + 0.02 if val >= 0 else val - 0.05
    ax.text(i, ypos, f'{val:.2f}', ha='center', fontsize=11, fontweight='bold')

plt.tight_layout()
plt.savefig('chart_r2.png', dpi=150, bbox_inches='tight')
plt.show()
```

---

### 4. Random Forest — Feature Importance

```python
import matplotlib.pyplot as plt
import matplotlib.patches as mpatches
import pandas as pd

# Use the trained Random Forest model from your notebook
rf_model = models["Random Forest"]   # the fitted RandomForestRegressor

features    = ['Oil_Price', 'Search_Trend', 'GDP_per_Capita', 'Tariff_Active',
               'sales_lag1', 'sales_lag3', 'sales_lag12', 'month', 'year']
importances = pd.Series(rf_model.feature_importances_, index=features)
importances = importances.sort_values(ascending=True)

feat_colors = [
    '#2E86AB' if 'lag' in f else ('#F18F01' if f in ['year', 'month'] else '#A23B72')
    for f in importances.index
]

fig, ax = plt.subplots(figsize=(9, 5.5))
ax.barh(importances.index, importances.values, color=feat_colors, alpha=0.88)

ax.set_xlabel('Importance Score', fontsize=12)
ax.set_title('Random Forest — Feature Importance', fontsize=13, fontweight='bold')
ax.grid(axis='x', alpha=0.3)

lag_patch  = mpatches.Patch(color='#2E86AB', alpha=0.88, label='Lag Features (Momentum)')
time_patch = mpatches.Patch(color='#F18F01', alpha=0.88, label='Temporal Features')
eco_patch  = mpatches.Patch(color='#A23B72', alpha=0.88, label='Economic / External Features')
ax.legend(handles=[lag_patch, time_patch, eco_patch], fontsize=10, loc='lower right')

for val, feat in zip(importances.values, importances.index):
    ax.text(val + 0.003, list(importances.index).index(feat),
            f'{val:.3f}', va='center', fontsize=10, fontweight='bold')

plt.tight_layout()
plt.savefig('chart_feature_importance.png', dpi=150, bbox_inches='tight')
plt.show()
```

---

## Results & Interpretation

- **The Time-Series Breakthrough:** Adding historical sales lags resolved the "extrapolation flatline" in tree models. Random Forest went from negative R² to **+0.10**, and MAPE dropped below 20%.

- **Best Model:** **Random Forest** — lowest error on all metrics. Its ensemble approach handles non-linear interactions between sales momentum, seasonality, and tariff effects better than a single tree.

- **Momentum Over Macroeconomics:** Models only achieved positive accuracy *after* being given lag features. In a rapidly shifting market, **recent consumer momentum is a stronger short-term predictor than GDP or oil prices**.

- **The Post-Tariff Structural Break:** The remaining ~20% MAPE reflects market "X-factors" not yet in the model: localised policy incentives (Inflation Reduction Act), new model launches, and sudden tariff implementations.

---

## Limitations & Future Work

- GDP data is quarterly; linear interpolation introduces artificial smoothness.
- No hyperparameter tuning yet — Grid Search on Random Forest could reduce MAPE further.
- Native time-series algorithms (SARIMA, Prophet) have not been evaluated.
- Precise monthly policy data (e.g., federal tax credits claimed per month) could explain the residual error.

---

## Project Structure

```text
US_EV_Sales_Analysis/
├── Data collection, EDA and HypothesisTests/
│   ├── ev_sales.csv
│   ├── multiTimeline.csv
│   └── Data_collection_EDA_HypothesisTests.ipynb
├── Machine Learning Modeling/
│   ├── ev_sales.csv
│   ├── multiTimeline.csv
│   └── Machine_Learning_Modeling.ipynb
├── Raw data/
│   └── Total Sales for Website_February 2026.pdf
├── DSA210 Proposal Document.pdf
├── README.md
└── requirements.txt
```
