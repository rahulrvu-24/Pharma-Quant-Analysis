# Pharma / Healthcare Sector — Quantitative Equity Analysis

**Quant Analyst Internship Task | Institute of Digital Risk | May 2026**  

---

## Overview

A end-to-end quantitative equity analysis of five large-cap US pharmaceutical stocks (JNJ, PFE, MRK, ABBV, LLY) covering January 2020 – December 2024. The project identifies three empirical patterns in pharma equity returns, proposes and backtests a buy-the-dip trading strategy, and evaluates its statistical validity using bootstrap resampling.
- JNJ - Johnson & Johnson
- PFE - Pfizer
- MRK - Merck & Co.
- ABBV - AbbVie Inc.
- LLY - Eli Lilly & Co.

---

## Sector Rationale

Pharma equities were chosen for three reasons:

- **Frequent dateable catalysts** — FDA decisions, clinical trial readouts, and earnings create sharp, identifiable price events
- **Defensive macro profile** — healthcare spending is non-cyclical, providing a stable baseline for identifying idiosyncratic signals
- **High intra-sector dispersion** — individual stock catalysts dominate over macro factors, making this a stock-picker's sector with genuine alpha opportunities

---

## Project Structure

```
QUANT_ANALYSIS_ASSIGNMENT/
├── pharma_quant_analysis.ipynb   ← Main analysis notebook (self-contained, runnable)
├── pharma_quant_summary.docx     ← Written summary report
├── charts & visualizations
│   ├── fig1_normalised_performance.png
│   ├── fig2_correlation.png
│   ├── fig3_mean_reversion.png
│   ├── fig4_volatility_clustering.png
│   ├── fig5_dispersion.png
│   ├── fig6_backtest_results.png
│   ├── fig7_bootstrap_sharpe.png
│   ├── fig8_sensitivity.png
├── requirements.txt
└── README.md                     
```

---

## Data

| Item | Detail |
|---|---|
| **Stocks** | JNJ, PFE, MRK, ABBV, LLY |
| **Period** | 2020-01-03 → 2024-12-31 |
| **Frequency** | Daily close prices |
| **Source (primary)** | `yfinance` with `auto_adjust=True` (splits + dividends adjusted) |
| **Source (fallback)** | Calibrated GBM synthetic data — activated automatically when yfinance is unavailable |
| **Trading days** | 1,256 (live) / 1,303 (synthetic) |

> **Important:** All reported results (events, statistics, backtest metrics) reflect the **live yfinance dataset**. The synthetic fallback generates only 14 events due to fewer injected shocks, making the sensitivity grid unreliable on that dataset. Run on a machine with internet access for full results.

---

## Patterns Identified

### Pattern 1 — Post-Large-Drop Mean Reversion
- **45 events** detected where a stock fell > 5% in a single trading day
- Mean cumulative return at **T+10: +3.47%** (t=2.96, p=0.005 ✱)
- Mean cumulative return at **T+15: +5.51%** (t=3.88, p<0.001 ✱✱)
- T+5 not significant (p=0.637) — recovery concentrates in week two, not week one
- Interpretation: institutional rebalancing (not immediate sentiment reversal) drives the recovery

### Pattern 2 — Volatility Clustering
- |Return| autocorrelation positive at lag 1 for all stocks (range: 0.18–0.28)
- COVID crash (March 2020) drove simultaneous 80–100% annualised vol across all five
- Calm and turbulent regimes persist for weeks — not random daily noise
- Implication: constant-volatility risk models underestimate exposure during turbulent periods

### Pattern 3 — High Intra-sector Dispersion
- Mean daily cross-sectional return dispersion: **1.01%**
- HIGH-vol regime: **1.22%** vs LOW-vol regime: **0.85%** — 45% gap
- Jun–Sep 2022 trough (0.55–0.70%): only period where macro (rate hikes) dominated over individual stories
- Implication: ETF exposure sacrifices alpha from idiosyncratic catalyst events

---

## Trading Idea: Buy-the-Dip Strategy

| Parameter | Value |
|---|---|
| **Signal** | Daily return < -5% |
| **Entry** | T+1 open (approximated as T+1 close) |
| **Exit** | T+10 close |
| **Universe** | JNJ, PFE, MRK, ABBV, LLY |
| **Position sizing** | Equal notional per trade |

### Backtest Results (live data, 45 trades)

| Metric | Value |
|---|---|
| Trades | 45 |
| Win Rate | 55.6% |
| Avg Return / Trade | +3.47% |
| Avg Win | +8.78% |
| Avg Loss | -3.16% |
| Profit Factor | 2.78 |
| Sharpe (approx.) | 2.10 |
| Bootstrap 95% CI | [0.85, 3.43] |
| Max Drawdown | -10.46% |

### Critical Finding — COVID / Non-COVID Split

| Group | n | Mean Return | Win Rate |
|---|---|---|---|
| COVID-era (2020) | 27 | +5.21% | 67% |
| Non-COVID (2021–2024) | 18 | +0.86% | 39% |

The strategy's headline metrics are dominated by the March 2020 market recovery. Non-COVID trades have a sub-50% win rate — the strategy has **no confirmed edge on idiosyncratic drops** without a drop-type filter. This is the most important finding.

---

## Validation Methodology

1. **In/out-of-sample split** — COVID/non-COVID split effectively serves as this test; significant degradation out-of-sample confirmed
2. **Parameter sensitivity grid** — 4 thresholds × 4 hold periods; 10d+ holds dominate across all thresholds
3. **Bootstrap significance test** — n=5,000 resamples; 95% CI [0.85, 3.43] excludes zero  
   *(Note: permutation test was found invalid for Sharpe — mean/std are order-independent; bootstrap was substituted)*
4. **Benchmarking** — proposed next step: compare vs XLV buy-and-hold and random-entry baseline

---

## Known Limitations

| Risk | Detail |
|---|---|
| COVID regime dependency | 60% of trades from 2020; non-COVID win rate = 39% |
| Drop-type ambiguity | Cannot distinguish panic selling from fundamental repricing |
| Small post-COVID sample | 18 non-COVID trades — insufficient for robust conclusions |
| Survivorship bias | Delisted firms excluded; their drops may not have recovered |
| Execution slippage | T+1 close entry overstates quality; spreads widen after large drops |
| Multiple testing | 16-cell sensitivity grid inflates false positive rate without correction |

---

## Dependencies

```bash
pip install numpy pandas matplotlib seaborn scipy yfinance jupyter
```

| Library | Version tested | Purpose |
|---|---|---|
| `numpy` | ≥ 1.24 | Array operations, GBM simulation |
| `pandas` | ≥ 2.0 | Data manipulation, rolling statistics |
| `matplotlib` | ≥ 3.7 | All charts |
| `seaborn` | ≥ 0.12 | Correlation heatmap, sensitivity heatmap |
| `scipy` | ≥ 1.10 | t-test (`ttest_1samp`) |
| `yfinance` | ≥ 0.2 | Live price data (optional — falls back to synthetic) |

---

## How to Run

```bash
# 1. Clone / download the repository
# 2. Install dependencies
pip install numpy pandas matplotlib seaborn scipy yfinance jupyter

# 3. Launch Jupyter
jupyter notebook pharma_quant_analysis.ipynb

# 4. Run all cells (Kernel → Restart & Run All)
```

The notebook will attempt to download live data via `yfinance`. If unavailable (firewall, no internet), it automatically falls back to the calibrated GBM synthetic dataset. The data source used is printed in Cell 3 output.

---

## Notebook Structure

| Cell | Section | Description |
|---|---|---|
| 0 | Title | Markdown header and overview |
| 1 | Setup | Initialization note |
| 2 | Imports | Libraries and global plot settings |
| 3 | Data | yfinance download + GBM synthetic fallback |
| 4 | EDA header | Markdown |
| 5 | Fig 1 | Normalised price performance (base=100) |
| 6 | Fig 2 | Summary statistics table |
| 7 | Fig 3 | Correlation heatmap |
| 8 | Pattern header | Markdown with pattern summary table |
| 9 | Pattern 1 | Event detection (>5% drop days) |
| 10 | Fig 4 | Mean cumulative return profile post-event |
| 11 | t-test | Statistical significance at T+5, T+10, T+15 |
| 12 | Pattern 2 | Rolling vol + autocorrelation |
| 13 | Pattern 3 | Cross-sectional dispersion analysis |
| 14 | Trading idea | Markdown — hypothesis and rules |
| 15 | Backtest | Trade simulation loop |
| 16 | Metrics | Win rate, Sharpe, drawdown, COVID split |
| 17 | Fig 6 | Equity curve + return distribution |
| 18 | Testing | Markdown — validation framework |
| 19 | Bootstrap | Bootstrap CI for Sharpe (n=5,000) |
| 20 | Sensitivity | Threshold × hold period grid |
| 21 | Risks | Markdown — risks and weaknesses table |

---

## AI Usage

Claude (Anthropic) was used as a coding and structuring assistant.

- **AI generated:** Notebook skeleton, matplotlib boilerplate, loop structure for bootstrap and sensitivity grid
- **Personally validated:** GBM parameters cross-checked against reported price histories; shock events based on documented real events; date range extended from 2022 to 2020 after understanding statistical power implications
- **Personal decisions:** Sector choice; three pattern hypotheses; trading idea design; COVID/non-COVID split analysis; discovery that permutation test is invalid for Sharpe (order-independent statistic) and substitution with bootstrap; debugging of bin-range ValueError and missing fig/ax error

All AI-generated code was reviewed, executed, and corrected before submission.