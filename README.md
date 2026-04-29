# Multi-Factor Equity Alpha Research: Accruals, Momentum, and Size

## Overview

This project studies whether **accrual mispricing** can generate economically meaningful and tradable alpha in a systematic long/short equity strategy.

The core hypothesis comes from the **accrual anomaly**:

* Firms with **high accruals** (earnings less supported by cash flow) may have overstated earnings quality and underperform.
* Firms with **low or negative accruals** (earnings strongly backed by cash flow) may be underpriced and outperform.

Rather than testing a simple academic anomaly in isolation, this project extends the idea into a **buy-side style alpha research pipeline**, combining:

* Signal construction
* Cross-sectional return forecasting
* Portfolio construction
* Robustness testing
* Tradability analysis

---

## Research Questions

This project asks:

1. Does the accrual anomaly persist in U.S. equities from 1990–2024?
2. Does combining accruals with momentum and size improve predictive power?
3. Are rank-weighted portfolios more robust than bucket sorting or regression-proportional weighting for noisy signals?
4. Does the signal survive factor controls, turnover costs, and regime shifts?

---

# Data

## CRSP Monthly Stock File (WRDS)

Universe:

* Common stocks only (`shrcd = 10,11`)
* NYSE / AMEX / NASDAQ (`exchcd = 1,2,3`)
* Sample: 1990–2024
* Price filter:

```python
abs(prc) >= 5
```

Variables:

* `permno`
* `date`
* `ret` (total return)
* `dlret` (delisting return)
* `prc`
* `shrout`

### Delisting Bias Adjustment

Returns are adjusted for delistings:

```python
ret_adj = (1+ret)*(1+dlret)-1
```

This avoids overstating performance by ignoring bankruptcies and other delistings.

---

## Compustat Quarterly Fundamentals

Variables:

* `niq` — net income
* `oancfq` — operating cash flow
* `atq` — total assets

Signal:

## Accruals

Defined as:

$$
Accruals_t = \frac{NI_t - OANCF_t}{Assets_{t-1}}
$$

Interpretation:

* Lower accruals → higher earnings quality
* Higher accruals → potentially lower future returns

---

## Reporting Lag Control

To avoid look-ahead bias:

```python
signal_date = datadate + 3 months
```

Signals become tradable only after assumed market availability.

---

## CRSP-Compustat Merge

Merged using CCM link table.

Used as-of alignment logic:

* Each monthly return uses only the most recent available signal
* Never uses future accounting information

This simulates realistic information availability.

---

# Additional Predictors

## 12–1 Momentum

Standard equity momentum signal:

$$
MOM_{i,t} = \prod_{s=t-12}^{t-2}(1+r_{i,s}) -1
$$

Uses months t-12 through t-2.
Skips month t-1 to avoid short-term reversal effects.

---

## Size

Market capitalization:

$$
ME = |PRC| \times SHROUT
$$

Feature used:

$$
log(ME)
$$

---

## Feature Standardization

Signals standardized cross-sectionally each month:

```python
z = (x - mean(x))/std(x)
```

Also winsorized at tails to reduce outlier sensitivity.

---

# Methodology

## Approach 1 — Portfolio Sorts

Baseline anomaly test:

* Sort stocks into quantiles based on accruals
* Form value-weighted portfolios
* Test long low-accrual / short high-accrual spread

This establishes whether the anomaly exists.

---

## Approach 2 — Cross-Sectional Return Forecasting

Estimate:

$$
R_{i,t+1} = \alpha_t + \beta_1 Accrual_{i,t} + \beta_2 Momentum_{i,t} + \beta_3 Size_{i,t} + \epsilon_{i,t}
$$

Run cross-sectional regressions monthly.

Use rolling windows:

* Train on prior 10 years
* Trade following year out of sample

Average coefficient estimates generate:

$$
\hat R_{i,t+1} = \hat\beta X_{i,t}
$$

Predicted returns become the alpha signal.

---

# Portfolio Construction

## Rank-Weighted Portfolio

Rather than weighting proportional to noisy predicted returns:

$$
w_i \propto \hat R_i
$$

I use rank-weighting.

### Step 1: Rank predicted returns

Take:

* Top 10% → long book
* Bottom 10% → short book

---

### Step 2: Convert rank to percentile

For stock rank (r_i):

$$
p_i = \frac{r_i-1}{N-1}
$$

---

### Step 3: Smoothed rank weights

Use:

$$
score_i = p_i + c
$$

with:

$$
c = 0.5
$$

Then:

$$
w_i = \frac{score_i}{\sum_j score_j}
$$

Properties:

* Long weights sum to +1
* Short weights sum to -1
* Dollar neutral portfolio
* Avoids overconcentration
* Maximum/minimum weight ratio:

$$
\frac{1+c}{c}=3
$$

This strikes a balance between:

* Equal weighting robustness
* Regression-weighted efficiency

---

## Rebalancing

Monthly:

* Recompute signals
* Rerank stocks
* Rebalance portfolio

---

# Performance Evaluation

Metrics tracked:

* Annualized Return
* Volatility
* Sharpe Ratio
* Maximum Drawdown
* Turnover

---

## Factor Attribution

Regress excess returns on Fama-French factors:

* Market
* SMB
* HML
* RMW
* CMA

Estimate:

* Factor exposures
* Residual alpha
* Statistical significance

---

# Robustness Tests

## Subsample / Regime Tests

Compare:

* Pre-2008
* Post-2008

Test whether signal decays across regimes.

---

## Monotonicity Test

Check whether returns deteriorate as accruals rise.

Helps validate economic structure of signal.

---

## Feature Sensitivity

Alternative accrual definitions tested.

Example:

```python
oancfy/4
```

Results should not depend on one exact accounting definition.

---

## Noise / Placebo Test

Randomize signal:

```python
random_signal = np.random.randn(len(data))
```

Random signal should produce no alpha.

Sanity check against data mining.

---

## Transaction Cost Sensitivity

Stress test:

```python
strategy_tc = strategy_returns - costs
```

Tests whether alpha survives implementation frictions.

---

# Key Pitfalls Addressed

This project explicitly addresses:

## Look-Ahead Bias

Fixed via reporting lag.

---

## Survivorship Bias

Uses full CRSP universe each period.

---

## Delisting Bias

Includes `dlret`.

---

## Outlier Fragility

Winsorization and clipping.

---

## Capacity / Tradability

Examines:

* Turnover
* Market-cap exposure
* Concentration control

---

# Preliminary Insights

(Results section to be updated as research finalizes.)

Questions being evaluated:

* Does low-accrual alpha survive FF5 controls?
* Does rank-weighting outperform equal-weighting?
* Does signal weaken after 2008?
* Are returns robust after costs?

---

# Technologies

* Python
* Pandas
* NumPy
* statsmodels
* WRDS
* CRSP
* Compustat

---

## Future Extensions

Potential next steps:

* Information coefficient (IC) analysis
* Additional signals (value, profitability)
* Signal decay horizons (t+1, t+3, t+6)
* Regularized models (Lasso / Ridge)
* Transaction-cost-aware optimization
* Compare rank weighting vs regression-proportional weighting

---

## References

* Sloan (1996), *Do Stock Prices Fully Reflect Information in Accruals and Cash Flows?*
* Fama and French (1993, 2015)
* Jegadeesh and Titman (1993)

---

## Author

Richard Guang
