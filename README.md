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

### Model Setup
* **Split:** Chronological 80/20 split (Training: 2010–2022 | Testing: 2023–2026).
* **Evaluation Metrics:** Root Mean Squared Error (RMSE) and Mean Absolute Error (MAE) were used to assess performance in real-world vehicle units.

### Results
| Model              | MSE       | MAE       | R²   |
|--------------------|-----------|-----------|-------|
| kNN                | 26558.70  | 20892.18  | -0.06 |
| Random Forest      | 32396.05  | 23559.83  | -0.58 |
| Linear Regression  | 30803.15  | 26704.05  | -0.43 |
| Decision Tree      | 41069.50  | 30568.27  | -1.54 |


### Results & Interpretation
* **The "Negative R²" Phenomenon:** All models yielded negative R² scores. This indicates that the models performed worse than a simple horizontal line based on training averages. This is a direct result of the unprecedented "exponential boom" in EV sales occurring entirely within the testing period.
* **Extrapolation Limits:** The analysis reveals that tree-based models (Decision Tree/Random Forest) cannot extrapolate; they are mathematically unable to predict values higher than those seen in the training data (pre-2023).
* **Paradigm Shift:** The failure of these models suggests a structural shift in the US market. The drivers of early adoption (GDP, Search Trends) are no longer sufficient to explain the current mass-market surge, which is likely driven by external "X-factors" like the Inflation Reduction Act subsidies and lower battery costs.

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
