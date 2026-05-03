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
All data collection, exploratory data analysis, hypothesis testing, and machine learning models are fully documented and reproducible in the main Jupyter Notebook.

## Data Analysis Pipeline
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
| **H1** | Higher GDP per Capita correlates with higher EV sales. | **Confirmed** (r = 0.916, p < 0.001) |
| **H2** | Increased Google Search volume correlates with higher sales. | **Confirmed** (r = 0.734, p < 0.001) |
| **H3** | High oil prices drive EV sales. | **Rejected** (No significant linear relationship, p = 0.59) |
| **H4** | Mean EV sales differ significantly after the May 2024 tariffs. | **Confirmed** (Sales significantly higher post-tariff, p < 0.001) |
