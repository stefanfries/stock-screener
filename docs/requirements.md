# Product Requirements Document (PRD)

## Stock Screener & Evaluator

**Version:** 0.1 (Draft)  
**Last Updated:** 2026-03-09  
**Status:** In Discussion

---

## 1. Vision & Goals

### 1.1 Problem Statement

Identifying high-quality stocks from a large investment universe is time-consuming and subjective without a systematic, data-driven approach. Decisions based on gut feeling or incomplete data lead to suboptimal portfolio construction.

### 1.2 Vision

Build a repeatable, transparent, and data-driven stock screening and evaluation system that ranks stocks within a defined investment universe — serving as the foundation for direct stock investments and warrant analysis.

### 1.3 Goals

- [ ] Screen stocks from a user-defined investment universe
- [ ] Evaluate fundamental quality (profitability, growth, balance sheet health)
- [ ] Evaluate price performance and momentum
- [ ] Produce ranked shortlists suitable for portfolio construction
- [ ] Support warrant analysis (identifying underlying stocks with favorable option characteristics)

---

## 2. Investment Universe

### 2.1 Definition

The **investment universe** is the set of stocks eligible for screening. It can be defined by:

- A stock index (e.g., S&P 500, NASDAQ 100, DAX 40, STOXX 600)
- A custom list of ticker symbols (manually curated or imported from a file)
- Sector or geographic filters applied on top of an index

### 2.2 Open Questions

> *These are topics for discussion — to be decided before implementation.*

- Which index/indices form the initial universe?
- Should the universe be configurable (e.g., load from a CSV)?
- Geographic scope: US only, European markets, global?
- Minimum liquidity/market cap requirements to be in the universe?

---

## 3. Screening Criteria

Screening criteria act as **pass/fail filters** applied before scoring. Stocks that fail a screen are excluded from the ranked shortlist.

### 3.1 Fundamental Screens (Pass/Fail)

| Criterion | Suggested Threshold | Rationale |
| - | - | - |
| Market Capitalization | > €/$ 1B (Large/Mid Cap) | Liquidity, data availability |
| Average daily volume | > 500,000 shares | Ensures tradability |
| Positive EPS (trailing 12M) | > 0 | Excludes loss-making companies |
| Debt-to-Equity | < 3.0 | Avoids highly leveraged companies |
| Years of trading history | ≥ 5 years | Enough data for analysis |

### 3.2 Open Questions

- Should screens be configurable per run, or fixed?
- Should we allow screening on analyst rating data?

---

## 4. Evaluation & Scoring Model

Stocks that pass all screens are **scored and ranked** on multiple dimensions. The final score is a weighted composite.

### 4.1 Fundamental Quality Score

Measures the financial health and earnings quality of a company.

| Metric | Description | Why It Matters |
| - | - | - |
| **Revenue Growth (3Y CAGR)** | Compound annual revenue growth over 3 years | Growing revenues signal market demand |
| **EPS Growth (3Y CAGR)** | Earnings per share growth | Reflects profitability scaling |
| **Net Profit Margin** | Net income / Revenue | Operating efficiency |
| **Return on Equity (ROE)** | Net income / Shareholders' equity | How efficiently capital is used |
| **Return on Assets (ROA)** | Net income / Total assets | Asset efficiency |
| **Free Cash Flow Margin** | FCF / Revenue | Cash generation quality (more reliable than EPS) |
| **Debt-to-Equity Ratio** | Total debt / Equity | Financial leverage / risk |
| **Current Ratio** | Current assets / Current liabilities | Short-term liquidity |
| **Gross Margin Stability** | Variance of gross margin over 3 years | Pricing power consistency |

### 4.2 Performance & Momentum Score

Measures recent price behavior and relative strength.

| Metric | Description | Why It Matters |
| - | - | - |
| **1M Price Return** | % price change over last 1 month | Short-term momentum |
| **3M Price Return** | % price change over last 3 months | Medium-term trend |
| **6M Price Return** | % price change over last 6 months | Intermediate momentum (often most predictive) |
| **12M Price Return** | % price change over last 12 months | Long-term trend |
| **Relative Strength vs. Index** | Stock return minus index return (1Y) | Alpha generation potential |
| **52-Week High Proximity** | Current price / 52-week high | Breakout potential |
| **Volatility (90-day HV)** | Historical 90-day volatility | Risk-adjusted view |
| **Beta** | Sensitivity to market moves | Systematic risk |

### 4.3 Valuation Score (Optional / Contrarian Layer)

> **Note:** Valuation metrics are tricky — cheap stocks are often cheap for a reason. Use with care.

| Metric | Description |
| - | - |
| **P/E Ratio** | Price / Earnings |
| **Forward P/E** | Price / Next-year estimated earnings |
| **EV/EBITDA** | Enterprise value to EBITDA |
| **P/B Ratio** | Price / Book value |
| **PEG Ratio** | P/E divided by earnings growth rate |

### 4.4 Composite Scoring

Each dimension receives a **percentile rank** (0–100) within the universe, then weighted:

| Dimension | Suggested Weight | Notes |
| - | - | - |
| Fundamental Quality | 40% | Long-term foundation |
| Performance / Momentum | 40% | Timing and trend |
| Valuation | 20% | Avoid overpaying |

> **Open Question:** Are these weights right for your investment style? Growth investors might lower valuation weight; value investors would raise it.

### 4.5 Open Questions

- Weighted average or factor model?
- Should weights be configurable per strategy (e.g., "growth", "value", "momentum")?
- How to handle missing data? (e.g., no forward P/E estimate available)

---

## 5. Warrant-Specific Considerations

For stocks intended as warrant underlyings, additional filters are needed:

| Criterion | Requirement |
| - | - |
| Implied Volatility | Warrants are more attractive when IV is moderate |
| Options / Warrants Availability | Liquid derivatives must exist on the stock |
| Expected Holding Period | Relevant for warrant time value decay |
| Warrant Type | Call (bullish), Put (bearish), or Turbo/Knock-out |

> **Open Question:** Should warrant screening be a separate module, or integrated into the main scoring?

---

## 6. Output & Reporting

### 6.1 Desired Outputs

- **Ranked shortlist**: Top N stocks by composite score (configurable N)
- **Score breakdown**: Per-stock, per-dimension scores (transparency)
- **Screening report**: Which stocks were excluded and why
- **Watchlist export**: CSV or Excel for further analysis

### 6.2 Open Questions

- Should there be a visual dashboard (charts, tables)?
- Export to Excel / Google Sheets integration desired?
- Email/notification when scores change significantly?

---

## 7. Non-Functional Requirements

| Category | Requirement |
| - | - |
| **Data freshness** | Daily update frequency sufficient for weekly reviews |
| **Data sources** | Must use free or low-cost data APIs initially |
| **Reproducibility** | Same inputs must always produce same outputs |
| **Transparency** | Each score must be traceable back to source data |
| **Extensibility** | Easy to add new metrics or data sources |

---

## 8. Out of Scope (v1)

- Real-time intraday data
- Automated order execution / brokerage integration
- News sentiment analysis
- Machine learning models (potential future feature)
- Portfolio optimization (mean-variance, etc.) — phase 2

---

## 9. Open Discussion Items

This section tracks questions and ideas raised during discussions.

| # | Topic | Status | Notes |
| - | - | - | - |
| D-01 | Initial investment universe definition | Open | |
| D-02 | Weight configuration per strategy | Open | |
| D-03 | Warrant module: separate or integrated | Open | |
| D-04 | Output format preferences | Open | |
| D-05 | Data source selection | Open | See Architecture doc |
