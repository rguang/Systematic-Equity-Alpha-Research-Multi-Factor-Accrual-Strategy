# Multi-Factor Equity Alpha Research: Accruals, Momentum, and Volatility

## Overview

This project investigates whether accounting quality, price trends, and risk characteristics can be combined into a profitable systematic equity strategy.

The core hypothesis is based on the **accrual anomaly**:

- High accrual firms report earnings less supported by cash flow and may underperform.
- Low accrual firms generate stronger cash-backed earnings and may outperform.

I combine:

- Accruals
- 12–1 Momentum
- Volatility

into a cross-sectional alpha model and test whether the signal generates economically significant returns after controlling for common risk factors.

---

# Research Pipeline

```text
Raw Data
    ↓
Signal Construction
    ↓
Cross-Sectional Regression
    ↓
Expected Return Forecasts
    ↓
Portfolio Construction
    ↓
Performance Evaluation
    ↓
Robustness Testing
```

---

# Data

## CRSP Monthly Stock File

**Source:** WRDS

**Universe:**

- Common stocks (SHRCD 10, 11)
- NYSE / AMEX / NASDAQ
- 1990–2024

**Variables:**

- Returns (`ret`)
- Delisting returns (`dlret`)
- Price (`prc`)
- Shares outstanding (`shrout`)

---

## Compustat Quarterly Fundamentals

**Source:** WRDS

**Variables:**

- Net Income (`niq`)
- Operating Cash Flow (`oancfq`)
- Total Assets (`atq`)

---

# Signals

## 1. Accruals

```text
Accruals = (Net Income - Operating Cash Flow) / Lagged Total Assets
```

Lower accruals indicate higher earnings quality.

---

## 2. Momentum

Standard 12–1 momentum:

```text
Past 12-month return excluding most recent month
```

This follows Jegadeesh & Titman (1993).

---

## 3. Volatility

Realized trailing volatility:

```text
12-month standard deviation of returns
```

Used to capture the low-volatility anomaly.

---

# Data Quality Controls

This project explicitly addresses common backtesting pitfalls.

## Look-Ahead Bias

Accounting data becomes tradable only after:

```text
signal_date = datadate + 3 months
```

to simulate realistic reporting delays.

---

## Delisting Bias

Returns adjusted using:

```python
ret_adj = (1 + ret) * (1 + dlret) - 1
```

to capture bankruptcy and acquisition outcomes.

---

## Survivorship Bias

Uses the full historical CRSP universe rather than surviving firms only.

---

## Outlier Robustness

Signals are winsorized at extreme percentiles to reduce sensitivity to accounting outliers.

---

## Microcap Distortions

Implemented:

```text
Price >= $5
Market Cap >= $500M
```

using lagged market capitalization to avoid including stocks whose extreme returns temporarily pushed them above the threshold.

---

# Alpha Model

Each month, estimate:

```text
Future Return =
β1·Accruals +
β2·Momentum +
β3·Volatility +
ε
```

using rolling cross-sectional regressions.

**Training framework:**

```text
Train: Previous 10 years
Test: Following year
```

This creates a realistic out-of-sample forecasting process.

---

# Portfolio Construction

## Rank-Weighted Long/Short Portfolio

Instead of using raw predicted returns directly:

1. Rank stocks by expected return
2. Long top decile
3. Short bottom decile

Convert ranks into percentile scores:

```text
score = percentile + c
```

where:

```text
c = 0.5
```

This:

- Avoids excessive concentration
- Maintains signal strength
- Produces stable weights through time

The resulting portfolio is:

```text
Dollar Neutral
Long Weights = +1
Short Weights = -1
```

with monthly rebalancing.

---

# Performance Metrics

Evaluated using:

- Annualized Return
- Volatility
- Sharpe Ratio
- Maximum Drawdown
- Turnover
- Information Coefficient (IC)

---

# Results

## Cross-Sectional Alpha Strategy

| Metric | Result |
|----------|----------|
| FF5 Alpha | 1.38% Monthly |
| FF5 Alpha t-stat | 8.4 |
| Sharpe Ratio | 1.28 |

---

## Information Coefficient (IC)

Measures whether stocks with stronger predicted returns subsequently outperform.

```text
IC = Spearman Rank Correlation
(predicted returns, realized returns)
```

Used to evaluate signal quality independently of portfolio construction.

---

# Robustness Tests

## Regime Analysis

Evaluated:

- Pre-2008
- Post-2008

to determine whether signal effectiveness changed through time.

---

## Feature Sensitivity

Tested alternative accrual definitions.

Results remained broadly consistent across specifications.

---

## Transaction Costs

Applied turnover-based trading cost assumptions and evaluated net performance.

---

## Placebo Test

Randomized signals produced no statistically significant alpha.

This helps validate that observed performance is not the result of chance.

---

# Key Challenges Encountered

During development several implementation issues materially affected results:

- Incorrect portfolio formation period alignment
- Monthly vs. annual portfolio reweighting mistakes
- Market capitalization scaling errors (`SHROUT` measured in thousands)
- Microcap-driven returns dominating backtests
- Incorrect contemporaneous market-cap filtering

Fixing these issues substantially improved realism and reduced overstated performance.

---

# Technologies

- Python
- Pandas
- NumPy
- Statsmodels
- WRDS
- CRSP
- Compustat

---

# Key Takeaways

This project evolved from a simple accrual anomaly replication into a complete systematic research workflow:

- Alpha signal generation
- Data engineering
- Bias mitigation
- Portfolio construction
- Factor attribution
- Robustness testing

The primary lesson was that most of the work in quantitative research is not finding a signal—it is ensuring that the signal survives realistic implementation constraints.

---

# References

- Sloan (1996), *Do Stock Prices Fully Reflect Information in Accruals and Cash Flows?*
- Fama & French (1993, 2015)
- Jegadeesh & Titman (1993)

---

## Author

Richard Guang  
MIT MFin | Quantitative Research | Systematic Investing
