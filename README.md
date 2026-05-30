# risk-analysis
# Risk Analysis: VaR and Expected Shortfall Estimation and Backtesting

**MSc Quantitative Finance — SMM272 Risk Analysis**
Bayes Business School, City St George's, University of London
Submitted: March 2026 | Grade: 90%



---

## Overview

This project investigates whether standard parametric and non-parametric VaR/ES methods can adequately capture the tail risk of a concentrated equity portfolio exhibiting fat tails, negative skewness, and volatility clustering, as well as the interest rate risk of a coupon bond. The work is structured in four sections.

The full write-up is in [`report/SMM272_Group9_report.pdf`](report/SMM272_Group9_report.pdf).

---

## Data

- Daily adjusted closing prices for six large-cap US technology stocks (Apple, Microsoft, IBM, NVIDIA, Alphabet, Amazon), January 2014 to December 2025 — sourced from Bloomberg Terminal
- UniCredit 5Y CDS spread daily history (Dec 2015–Nov 2025) — Investing.com

Raw data is not included in this repository due to Bloomberg licence restrictions.

---

## Key Results

| Method | Violation Rate | UC Test | IND Test | CC Test |
|---|---|---|---|---|
| Gaussian Parametric | 2.89% | Fail | Fail | Fail |
| Historical Simulation | 1.11% | Pass | Pass | Pass |
| Student-t MoM | 2.37% | Fail | Fail | Fail |
| GARCH(1,1) Rolling | 2.65% | Fail | Pass | Fail |
| GARCH(1,1) Expanding | 2.37% | Fail | Pass | Fail |

HS is the only method not formally rejected, but its low violation count reflects the ghost effect rather than accurate calibration. All parametric models produce violation rates of 2.37–2.89% against the 1% target.

**Kupiec test power** at the relevant 99% confidence level and T = 2,997: approximately 38%, meaning roughly 62% of misspecified GARCH alternatives would not trigger rejection.

**Bond VaR (Section 4):** MC+Full Revaluation is the only reliable method across all horizons (errors below 0.5%). Delta-Normal overestimates VaR by up to 77% at 90 days; Delta-Gamma underestimates by up to 33% due to Taylor approximation breakdown when yield shocks exceed 30% of the initial yield.

---

## Structure

### Section 1 — VaR and ES Estimation and Backtesting

An equally weighted portfolio of six large-cap US technology stocks (Apple, Microsoft, IBM, NVIDIA, Alphabet, Amazon) over January 2014 to December 2025 (2,997 log-return observations). Five estimation methods compared.

**Statistical analysis** (`q1_portfolio_statistical_analysis.ipynb`): annualised mean 24.66%, volatility 23.99%, excess kurtosis 6.34, skewness −0.337. Empirical 1st percentile of −4.37% is 24% more extreme than the Gaussian prediction. All four normality tests reject at p ≈ 0. Ljung-Box on squared returns: 2,160.45 (p ≈ 0), confirming strong volatility clustering.

**Estimation and backtesting** (`q1_var_es_estimation_backtesting.ipynb`): VaR and ES estimated at 99% confidence using a rolling 125-day window. Five methods: Gaussian parametric, Historical Simulation, Student-t Method of Moments, GARCH(1,1) rolling window, GARCH(1,1) expanding window. Five backtests applied: Kupiec UC, Christoffersen IND, Conditional Coverage, KS distributional, Berkowitz. Only HS passes all applicable tests, due to the ghost effect rather than accurate calibration. Both GARCH variants uniquely pass the independence test.

---

### Section 2 — Power of the Kupiec Test

(`q2_kupiec_test_power_simulation.ipynb`)

Monte Carlo simulation (M = 10,000) estimating the power of the Kupiec unconditional coverage test when the null is Gaussian constant-volatility and the true process is GARCH(1,1) fitted to the portfolio. Three dimensions varied: sample size T, confidence level, and GARCH persistence α + β.

Key findings: power at T = 2,997 and 99% confidence is only ~38%, meaning the test has substantial Type 2 error. Power follows a U-shape in confidence level (minimum near 97.5–99%) and increases monotonically with GARCH persistence. Under H₁, violation count mean rises 23% above H₀ but standard deviation more than doubles, causing substantial distributional overlap.

---

### Section 3 — Risk Parity Portfolio

(`q3_risk_parity_portfolio.ipynb`)

Three portfolio allocation strategies compared: Risk Parity (parametric and non-parametric), Maximum Diversification, and Equal Weighting. In-sample period: first 1,498 observations. Out-of-sample: remaining 1,499 observations.

Key findings: RP successfully equalises risk contributions but converges toward equal-weighting when pairwise correlations are high (0.32–0.63 across this tech-stock universe). Maximum Diversification achieves the highest Sharpe ratio (1.000) and lowest drawdown (−30.88%). Equal Weighting achieves the highest cumulative return (392.9%) driven by NVIDIA's AI-rally, but also the largest drawdown (−36.35%). VaR violation rates are nearly identical across strategies (5.9–6.3% vs 5% target) at the 95% level, reflecting the high pairwise correlations.

---

### Section 4 — VaR of a Bond

(`q4_bond_var_delta_gamma_mc.ipynb`)

Interest rate VaR for a 10-year annual coupon bond (F = 100, c = 5%, P₀ = 99, σ_y = 60 bp/day). Six methods compared across ten horizons (1 to 90 days): Exact Full Revaluation, Delta-Normal, Delta-Gamma, MC+Δ, MC+ΔΓ, MC+Full.

Key findings: Delta-Normal overestimates VaR by up to 77% at 90 days (ignores convexity). Delta-Gamma underestimates by up to 33% because the Taylor expansion overstates the convexity benefit when yield shocks are large relative to the initial yield — breakdown occurs when Δy/y₀ > ~30%, exceeded beyond 10–20 days for this bond. Only MC+Full Revaluation maintains errors below 0.5% across all horizons. The 32.86% probability of a 10% price decline within 30 days illustrates the substantial interest rate risk in long-duration bonds under elevated yield volatility.

---

## Repository Structure

```
├── README.md
├── report/
│   └── SMM272_Group9_report.pdf
├── code/
│   ├── q1_portfolio_statistical_analysis.ipynb
│   ├── q1_var_es_estimation_backtesting.ipynb
│   ├── q2_kupiec_test_power_simulation.ipynb
│   ├── q3_risk_parity_portfolio.ipynb
│   ├── q4_bond_var_delta_gamma_mc.ipynb
│   └── requirements.txt
└── data/
    └── README.md
```

---

## Requirements

```bash
pip install -r code/requirements.txt
```

---

## Key References

- Berkowitz, J. (2001). Testing density forecasts with applications to risk management. *Journal of Business and Economic Statistics*, 19(4).
- Christoffersen, P. (1998). Evaluating interval forecasts. *International Economic Review*, 39(4).
- Kupiec, P. (1995). Techniques for verifying the accuracy of risk measurement models. *Journal of Derivatives*, 3(2).
- Jorion, P. (2007). *Value at Risk*. McGraw-Hill, 3rd edition.
- Engle, R. (1982). Autoregressive conditional heteroscedasticity with estimates of the variance of UK inflation. *Econometrica*, 50(4).
