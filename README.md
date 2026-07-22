# walmart-revenue-forecasting
**Time-series forecasting project analyzing whether FRED retail sales can predict Walmart quarterly revenue using Python.
Project Overview**

This project investigates whether publicly available U.S. retail sales data from the Federal Reserve Economic Data (FRED) database can be used as a leading indicator to forecast Walmart's quarterly revenue growth.

Rather than simply fitting a regression model, the project focuses on evaluating whether the retail sales signal actually improves forecasting performance compared to strong baseline forecasting methods.

The analysis follows best practices for time-series forecasting, including proper fiscal-quarter alignment, expanding-window backtesting, baseline comparisons, and look-ahead bias prevention.

---

# Business Problem

Investors and portfolio managers often seek early indicators of a company's financial performance before official earnings are released.

This project addresses the following question:

**Can monthly U.S. retail sales (FRED RSXFS) be used to predict Walmart's quarterly revenue growth?**

If successful, this signal could provide an early indication of Walmart's future revenue performance.

---

# Objectives

The primary objectives of this project are:

- Analyze the relationship between U.S. retail sales and Walmart revenue.
- Convert monthly retail sales into Walmart fiscal quarters.
- Calculate Year-over-Year (YoY) growth rates.
- Develop forecasting models using retail sales signals.
- Compare forecasting performance against naive baselines.
- Determine whether retail sales provide meaningful predictive value.

---

# Dataset

## 1. FRED Retail Sales

Source:
Federal Reserve Economic Data (FRED)

Series:
RSXFS (Retail Trade and Food Services)

Frequency:
Monthly

Features:
- Monthly retail sales
- Converted into Walmart fiscal quarters

---

## 2. Walmart Revenue

Source:
Public Walmart quarterly financial reports (SEC filings)

Frequency:
Quarterly

Features:
- Quarterly revenue
- Converted to Year-over-Year growth

---

# Project Workflow

## Step 1

**Load the datasets**
- Retail Sales (Monthly)
- Walmart Revenue (Quarterly)

---

## Step 2

**Align monthly retail sales to Walmart fiscal quarters.**

Since Walmart follows a fiscal calendar rather than calendar quarters, the monthly retail sales data was aggregated accordingly.

---

## Step 3

**Calculate Year-over-Year (YoY) Growth**

Instead of predicting raw revenue, both datasets were transformed into YoY growth rates to remove long-term trends.

---

## Step 4

**Exploratory Data Analysis**

Visualized:

- Retail sales trends
- Walmart revenue trends
- YoY growth comparison

---

## Step 5
## How to Run the Project

### a. Clone the Repository

Open a terminal or command prompt and run:

```bash
git clone https://github.com/Nandinikovuru/walmart-revenue-forecasting.git
```

Move into the project folder:

```bash
cd walmart-revenue-forecasting
```

### b. Create a Virtual Environment

Creating a virtual environment is recommended so the project dependencies remain separate from other Python projects.

On Windows:

```bash
python -m venv venv
venv\Scripts\activate
```

On macOS or Linux:

```bash
python3 -m venv venv
source venv/bin/activate
```

### c. Install the Required Libraries

Run:

```bash
pip install -r requirements.txt
```

The project uses:

- pandas
- NumPy
- Matplotlib
- scikit-learn
- statsmodels

### d. Start Jupyter Notebook

Run:

```bash
jupyter notebook
```

When Jupyter opens in your browser, select:

```text
analysis.ipynb
```

### e. Run the Notebook

Inside Jupyter Notebook, select:

```text
Kernel → Restart & Run All
```

The notebook will:

1. Load the Walmart revenue and FRED retail-sales datasets.
2. Align monthly retail sales with Walmart's fiscal quarters.
3. Calculate year-over-year growth.
4. Build naive baseline forecasts.
5. Fit concurrent and lagged OLS regression models.
6. Run an expanding-window backtest.
7. Calculate forecasting errors.
8. Generate the project figures.

## Expected Input Files

The following files must be available in the repository:

```text
retail_sales_fred.csv
walmart_revenue.csv
```

## Expected Outputs

Running the notebook generates the following visualizations:

```text
fig1_yoy_overlay.png
fig2_rolling_mae.png
fig3_rolling_beta.png
```

## Step 6
**Baseline Forecasting Models**

Implemented:

- Seasonal Naive
- Seasonal Naive + Drift

These baselines provide a benchmark to determine whether the retail sales signal actually improves forecasting accuracy.

---

## Step 7

**Regression Models**

Two forecasting models were evaluated.

### Concurrent Model

Uses retail sales from the same quarter.

Purpose:
Evaluate the relationship after the quarter ends.

---

### Lag-1 Model

Uses retail sales from the previous quarter.

Purpose:
Acts as a true leading indicator.

---

## Step 8

**Expanding Window Backtesting**

Instead of using a random train/test split, the project uses an expanding-window evaluation.

Example:

Train:
2011–2017

Predict:
2018

Then

Train:
2011–2018

Predict:
2019

This avoids look-ahead bias and better reflects real-world forecasting.

---

## Step 9

**Evaluation Metrics**

Models were evaluated using:

- Mean Absolute Error (MAE)
- Rolling 8-quarter MAE
- Rolling regression coefficients (beta)

---

# Results

Key findings:

- Retail sales and Walmart revenue exhibit some correlation.
- The relationship weakens significantly after COVID.
- Neither the concurrent model nor the lag-1 model consistently outperformed the Seasonal Naive + Drift baseline.
- Aggregate retail sales alone is not a reliable leading indicator for Walmart's quarterly revenue growth.

## Key Visualizations

### Walmart Revenue Growth vs. FRED Retail-Sales Growth

This chart compares Walmart's year-over-year revenue growth with the FRED retail-sales growth series aligned to Walmart's fiscal quarters.

[Walmart Revenue vs FRED Retail Sales]
<img width="1200" height="540" alt="fig1_yoy_overlay" src="https://github.com/user-attachments/assets/3ed6e4a6-48af-41c3-919b-cd5855d98f8f" />


### Rolling Forecast Error

This chart compares the rolling 8-quarter Mean Absolute Error of the seasonal-naive-plus-drift baseline and the retail-sales regression model.

A lower MAE indicates better forecasting performance.

[Rolling Forecast MAE]
<img width="1200" height="840" alt="fig2_rolling_mae" src="https://github.com/user-attachments/assets/300954b7-9eef-4a13-9011-6ed8c7a3cec9" />


### Rolling Regression Coefficient

This chart shows how the relationship between lagged retail-sales growth and Walmart revenue growth changed over time.

The coefficient weakened sharply after 2020, suggesting that the relationship was unstable across different economic periods.

[Rolling Regression Beta] <img width="1200" height="540" alt="fig3_rolling_beta" src="https://github.com/user-attachments/assets/d257c55f-91fd-42f9-8d6b-b43adbf95160" />

# Conclusion

This project demonstrates a complete time-series forecasting workflow, from data collection and preprocessing to model evaluation and interpretation.

Although FRED retail sales showed some relationship with Walmart revenue, the analysis found that this signal did not consistently outperform strong baseline forecasting methods. The project highlights the importance of rigorous validation and benchmarking before adopting economic indicators for predictive modeling.

