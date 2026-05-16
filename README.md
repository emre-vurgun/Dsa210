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

All four models were ranked across every metric. **Random Forest** is the clear winner on all measures:

| Model | RMSE (Vehicles) | MAE (Vehicles) | R² | MAPE (%) |
|:---|---:|---:|---:|---:|
| **Random Forest** | **26,912** | **20,463** | **0.10** | **19.53%** |
| Decision Tree | 28,217 | 21,859 | 0.01 | 20.93% |
| kNN | 29,432 | 23,247 | -0.08 | 22.45% |
| Linear Regression | 36,416 | 28,357 | -0.65 | 28.79% |

**RMSE & MAE** are in vehicle units. **R² > 0** means the model beats a mean-only baseline. **MAPE** gives an intuitive percentage error of the model.

---

## Results & Interpretation

- **The Time-Series Breakthrough:** Adding historical sales lags resolved the "extrapolation flatline" in tree models. Random Forest went from negative R² to **+0.10**, and MAPE dropped below 20%.

- **Best Model:** **Random Forest** — lowest error on all metrics. Its ensemble approach handles non-linear interactions between sales momentum, seasonality, and tariff effects better than a single tree.

- **Momentum Over Macroeconomics:** Models only achieved positive accuracy after being given lag features. In a rapidly shifting market, **recent consumer momentum is a stronger short-term predictor than GDP or oil prices**.

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
