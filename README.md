# Analysis of EV Sales in the US

## Data Sources
This project integrates multiple heterogeneous datasets covering **2010–2026**, aligned temporally into a monthly format:

**1. US EV Sales (BEV + PHEV)**
* **Source:** Argonne National Laboratory
* **Frequency:** Monthly
* **Features:** Total EV Sales (Battery Electric + Plug-in Hybrid)

**2. Crude Oil Prices (WTI)**
* **Source:** Yahoo Finance API (`yfinance`)
* **Frequency:** Monthly
* **Usage:** Monthly closing prices for WTI Crude Oil.

**3. US GDP per Capita**
* **Source:** Federal Reserve Economic Data (FRED - A939RX0Q048SBEA)
* **Frequency:** Monthly
* **Usage:** Represents consumer purchasing power and economic health.

**4. Public Interest (Google Trends)**
* **Source:** Google Trends (`multiTimeline.csv`)
* **Keyword:** "electric car" (US Region)
* **Usage:** Scaled search volume (0-100) acting as a proxy for consumer interest.

**5. US Tariff Policy**
* **Source:** Derived from US Policy Announcements
* **Usage:** A binary dummy variable representing the 100% tariff placed on Chinese EV imports (Active = 1 after May 2024).

## Detailed Analysis & Code
All data collection, exploratory data analysis, hypothesis testing, and machine learning models are fully documented.

### Data Analysis Pipeline
**1. Data Preparation**
* Formatted various date strings into standardized `datetime` objects.
* Merged four disparate data sources into a single temporal DataFrame.
* Applied **Min-Max Normalization** (0 to 1 scaling) to allow fair comparison between massive numbers (like GDP) and small indexes (like Search Trends).
* Engineered a binary feature (`Tariff_Active`) for chronological policy tracking.

**2. Exploratory Data Analysis (EDA)**
EDA focused on understanding temporal trends and visual correlations:
* Overlaid time-series plots to compare normalized sales, oil prices, and searches.
* Generated Pearson correlation heatmaps.
* Constructed boxplots to visually evaluate EV sales distributions before and after the May 2024 tariffs.

## Hypothesis Testing
The following hypotheses were evaluated using **Pearson Correlation** and **Welch's Independent T-Tests** (α = 0.05):

| Hypothesis | Description | Result |
| :--- | :--- | :--- |
| **H1** | Higher GDP per Capita correlates with higher EV sales. | **Confirmed** (Reject Null, r = 0.916, p < 0.001) |
| **H2** | Increased Google Search volume correlates with higher sales. | **Confirmed** (Reject Null, r = 0.734, p < 0.001) |
| **H3** | High oil prices drive EV sales. | **Rejected** (Fail to Reject Null, p = 0.59) |
| **H4** | Mean EV sales differ significantly after the May 2024 tariffs. | **Confirmed** (Reject Null, Sales significantly higher post-tariff, p < 0.001) |

## Machine Learning Models
Four regression models were implemented to predict US EV sales: Linear Regression, k-Nearest Neighbors (kNN), Random Forest, and Decision Tree. 

To address the volatile time-series nature of the market, the dataset was enriched with **Feature Engineering**, adding historical sales momentum (`sales_lag1`, `sales_lag3`, `sales_lag12`) and temporal dynamics (`month`, `year`).

### Model Setup
* **Split:** A strategic chronological split was used to test the impact of the May 2024 Chinese EV tariffs (Training: 2010 to May 2024 | Testing: June 2024 to 2026).
* **Evaluation Metrics:** Root Mean Squared Error (RMSE), Mean Absolute Error (MAE), R², and Mean Absolute Percentage Error (MAPE) were used to assess performance in real-world, interpretable units.

### Results
| Model | RMSE (Vehicles) | MAE (Vehicles) | R² | MAPE (%) |
| :--- | :--- | :--- | :--- | :--- |
| **Random Forest** | 26,912.10 | 20,462.68 | 0.10 | 19.53% |
| **Decision Tree** | 28,217.18 | 21,858.94 | 0.01 | 20.93% |
| **kNN** | 29,432.09 | 23,247.01 | -0.08 | 22.45% |
| **Linear Regression**| 36,415.58 | 28,356.84 | -0.65 | 28.79% |

### Results & Interpretation
* **The Time-Series Breakthrough:** Adding historical sales lags resulted in a major predictive breakthrough. Previously, tree-based models suffered from an "extrapolation flatline," but giving them immediate historical context allowed them to finally learn the temporal momentum of the market.
* **Best Model:** **Random Forest** emerged as the clear winner, achieving the lowest error, a positive R² score (0.10), and the best MAPE (19.53%). Its ensemble approach successfully handled the complex, non-linear interactions between recent sales trends, seasonality, and the May 2024 tariffs.
* **Momentum Over Macroeconomics:** The fact that the models only achieved positive accuracy *after* being fed historical sales lags provides a crucial real-world insight: In a rapidly shifting market, historical consumer momentum is a stronger short-term predictor than broad macroeconomic indicators like GDP or oil prices.
* **The Paradigm Shift (Post-Tariff Market):** While keeping the error under 20% on highly volatile post-tariff data is a massive success, the remaining variance highlights a structural break in the EV market. The rapid growth observed in the test period proves the market is currently being driven by external "X-factors," such as localized policy incentives (e.g., Inflation Reduction Act), technological advancements, and sudden tariff implementations.

## Project Structure
```text
US_EV_Sales_Analysis/
├── Data collection, EDA and HypothesisTests/          # EDA & Stats
│   ├── ev_sales.csv                                   # Base EV sales dataset
│   ├── multiTimeline.csv                              # Base Google Trends dataset
│   └── Data_collection_EDA_HypothesisTests.ipynb      # Main Analysis
├── Machine Learning Modeling/                         # ML Phase Folder
│   ├── ev_sales.csv                                   # Base EV sales dataset
│   ├── multiTimeline.csv                              # Base Google Trends dataset
│   └── Machine_Learning_Modeling.ipynb                # ML Regression Models
├── Raw data/                                          
│   └── Total Sales for Website_February 2026.pdf      # EV sales data in US
├── DSA210 Proposal Document.pdf                       # Project proposal
├── README.md                                          
└── requirements.txt                                   # Project dependencies                                         
