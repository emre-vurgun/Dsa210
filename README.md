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
| **H1** | Higher GDP per Capita correlates with higher EV sales. | **Fail to Reject** (r = 0.916, p < 0.001) |
| **H2** | Increased Google Search volume correlates with higher sales. | **Fail to Reject** (r = 0.734, p < 0.001) |
| **H3** | High oil prices drive EV sales. | **Rejected** (No significant linear relationship, p = 0.59) |
| **H4** | Mean EV sales differ significantly after the May 2024 tariffs. | **Fail to Reject** (Sales significantly higher post-tariff, p < 0.001) |

## Machine Learning Models
Three regression models were implemented to predict normalized US EV sales:

### Model Setup
* **Training period:** First 80% of the timeline (2010 to mid-2022)
* **Testing period:** Last 20% of the timeline (mid-2022 to 2026, time-aware chronological split to prevent data leakage)

### Results
| Model | MSE | R² |
| :--- | :--- | :--- |
| **Linear Regression** | 0.1082 | -5.1147 |
| **k-Nearest Neighbors (k=5)** | 0.0996 | -4.6290 |
| **Decision Tree (Depth=4)** | **0.0386** | **-1.1818** |

### Results & Interpretation
* **Finding:** The **Decision Tree** significantly outperformed the other models, achieving the lowest Mean Squared Error (0.0386). 
* **The "Negative R²" Phenomenon:** All models yielded a negative R². Because we used a chronological train/test split, the models were trained on pre-2022 data. The negative R² perfectly illustrates that the EV market experienced a massive, unprecedented structural shift in the testing period (post-2022) that could not be linearly predicted using only historical pre-2022 trends. The past no longer perfectly predicts the future in the rapidly evolving EV sector!

## Project Structure

US_EV_Sales_Analysis/
├── Data collection, EDA and HypothesisTests/          # EDA & Stats
│   ├── ev_sales.csv                                   # Base EV sales dataset
│   ├── multiTimeline.csv                              # Base Google Trends dataset
│   └── Data_collection_EDA_HypothesisTests.ipynb      # Main Analysis Notebook
├── Machine Learning Modeling/                         # ML Phase Folder
│   ├── ev_sales.csv                                   # Base EV sales dataset
│   ├── multiTimeline.csv                              # Base Google Trends dataset
│   └── Machine_Learning_Modeling.ipynb                # ML Regression Models Notebook
├── Raw data/                                          
│   └── Total Sales for Website_February 2026.pdf      # EV sales data in US
├── DSA210 Proposal Document.pdf                       # Project proposal
├── requirements.txt                                   # Project dependencies
└── README.md                                          # Project documentation
