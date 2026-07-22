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

**Baseline Forecasting Models**

Implemented:

- Seasonal Naive
- Seasonal Naive + Drift

These baselines provide a benchmark to determine whether the retail sales signal actually improves forecasting accuracy.

---

## Step 6

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

## Step 7

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

## Step 8

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

