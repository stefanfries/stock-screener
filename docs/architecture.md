# Architecture Document

## Stock Screener & Evaluator

**Version:** 0.1 (Draft)  
**Last Updated:** 2026-03-09  
**Status:** In Discussion

---

## 1. System Overview

The stock screener is a **data pipeline and scoring engine** composed of four logical layers:

```text
┌─────────────────────────────────────────────────────────┐
│                   1. Data Ingestion Layer                │
│   (Fetch raw price, fundamentals, metadata from APIs)   │
└───────────────────────┬─────────────────────────────────┘
                        │
┌───────────────────────▼─────────────────────────────────┐
│                   2. Storage Layer                       │
│         (Local cache: files or lightweight DB)           │
└───────────────────────┬─────────────────────────────────┘
                        │
┌───────────────────────▼─────────────────────────────────┐
│                3. Processing / Scoring Engine            │
│    (Screening filters → metric calculation → ranking)    │
└───────────────────────┬─────────────────────────────────┘
                        │
┌───────────────────────▼─────────────────────────────────┐
│                  4. Output / Reporting Layer             │
│          (Console, CSV export, notebooks, HTML)          │
└─────────────────────────────────────────────────────────┘
```

---

## 2. Technology Stack

### 2.1 Language

**Python 3.13** — industry standard for data analysis and finance. Excellent ecosystem (pandas, yfinance, etc.).

### 2.2 Package Management

**uv** — fast, modern Python package manager (replaces pip + venv). Already in use in this project.

### 2.3 Core Libraries (Candidates)

| Purpose | Library | Notes |
| - | - | - |
| Data manipulation | `pandas` | Industry standard for tabular data |
| Numerical computation | `numpy` | Required by pandas, used for scoring math |
| Financial data | `yfinance` | Free, Yahoo Finance wrapper — good starting point |
| Alternative data | `financedatabase` | Curated universe databases |
| Visualization | `matplotlib` / `plotly` | plotly for interactive charts |
| Configuration | `pydantic` | Type-safe config models |
| Report generation | `jinja2` | HTML report templating |
| Testing | `pytest` | Standard Python testing framework |
| Notebooks | `jupyter` | For exploratory analysis and visualization |

### 2.4 Open Questions

- Should we upgrade to a paid data provider (Polygon.io, Tiingo) once free tier limits are hit?
- Do we need a database (SQLite, DuckDB) or are Parquet files sufficient for caching?

---

## 3. Data Sources

### 3.1 Primary Source: Yahoo Finance (via yfinance)

**Pros:** Free, no API key needed, covers global markets, provides fundamentals + price history  
**Cons:** Rate limits, occasionally unreliable, no professional SLA

### 3.2 Alternative / Supplementary Sources (Future)

| Source | Type | Cost | Notes |
| - | - | - | - |
| Polygon.io | Price + fundamentals | Freemium | More reliable, institutional quality |
| Alpha Vantage | Price + fundamentals | Freemium | Good for fundamentals |
| Financial Modeling Prep (FMP) | Fundamentals | Freemium | Excellent financials API |
| EODHD | Global price data | Paid | Good European coverage |
| Tiingo | Price + news | Freemium | Clean API |

### 3.3 Data Caching Strategy

To avoid hitting API rate limits and to enable reproducible runs:

- Raw API responses are **cached locally** (Parquet or JSON)
- Cache is timestamped and has a **TTL (Time-To-Live)** — e.g., 24h for daily data
- A force-refresh flag can bypass the cache

---

## 4. Component Design

### 4.1 Universe Manager

**Responsibility:** Define and load the investment universe.

```text
UniverseManager
 ├── load_from_index(index_name: str) → list[Ticker]
 ├── load_from_file(path: str) → list[Ticker]
 └── apply_filters(market_cap_min, volume_min, ...) → list[Ticker]
```

### 4.2 Data Fetcher

**Responsibility:** Fetch and cache raw data for each ticker.

```text
DataFetcher
 ├── get_price_history(ticker, period) → DataFrame
 ├── get_fundamentals(ticker) → dict
 └── get_info(ticker) → dict
```

### 4.3 Metric Calculator

**Responsibility:** Compute derived metrics from raw data.

```text
MetricCalculator
 ├── calculate_performance_metrics(prices) → dict
 ├── calculate_fundamental_metrics(financials) → dict
 └── calculate_valuation_metrics(info, prices) → dict
```

### 4.4 Screener

**Responsibility:** Apply pass/fail filters to exclude stocks.

```text
Screener
 └── apply_screens(universe, screen_config) → (passed: list, rejected: list)
```

### 4.5 Scorer

**Responsibility:** Rank stocks by percentile and compute composite scores.

```text
Scorer
 ├── rank_by_dimension(universe_metrics) → DataFrame (with percentile ranks)
 └── compute_composite_score(ranks, weights) → DataFrame (sorted by score)
```

### 4.6 Reporter

**Responsibility:** Generate output artifacts from scoring results.

```text
Reporter
 ├── print_summary(results)
 ├── export_csv(results, path)
 └── generate_html_report(results, path)
```

---

## 5. Architecture Decision Records (ADRs)

ADRs document *why* key decisions were made. This creates an audit trail and helps when revisiting decisions later.

---

### ADR-001: Use Python as the primary language

**Date:** 2026-03-09  
**Status:** Accepted

**Context:**  
We need a language with strong data analysis libraries, financial ecosystem support, and beginner accessibility.

**Decision:**  
Use Python.

**Consequences:**  

+ Excellent library ecosystem (pandas, numpy, yfinance)  
+ Large community and learning resources  
+ Native Jupyter notebook support for exploration  
- Not ideal for real-time high-frequency applications (out of scope for this project)

---

### ADR-002: Use yfinance as the initial data source

**Date:** 2026-03-09  
**Status:** Accepted (subject to review)

**Context:**  
We need financial data (prices, fundamentals) without upfront cost while the project is in early development.

**Decision:**  
Use `yfinance` as the primary data source for v1.

**Consequences:**  

+ Zero cost, no API key required  
+ Covers fundamentals, price history, and metadata  
- Rate limits may be a problem with large universes  
- No professional SLA; data quality issues possible  
- Migration to a paid provider may be needed later

---

### ADR-003: Use percentile ranking for scoring

**Date:** 2026-03-09  
**Status:** Proposed

**Context:**  
Raw metric values (e.g., ROE of 15% vs 30%) are not directly comparable across metrics with different scales and distributions.

**Decision:**  
Convert all metrics to **percentile ranks** within the universe before scoring. A stock at the 80th percentile for ROE scores 80 on that metric regardless of the raw value.

**Consequences:**  

+ Metrics become comparable regardless of scale  
+ Handles outliers gracefully  
+ Score is always relative to the current universe (not absolute)  
- A "good" company in a bad universe still gets a high score (universe selection matters)

---

### ADR-004: Local file-based storage (Parquet) for caching

**Date:** 2026-03-09  
**Status:** Proposed

**Context:**  
We need to cache API responses to avoid rate limits and enable reproducible runs without always calling external APIs.

**Decision:**  
Store cached data as **Parquet files** in a local `data/` directory.

**Consequences:**  

+ No database setup required  
+ Parquet is compact, fast, and pandas-native  
+ Easy to inspect and debug  
- No concurrent multi-user support (acceptable for personal use)  
- Manual cleanup needed if data grows large

---

## 6. Folder Structure (Proposed)

```text
stock-screener/
├── docs/
│   ├── requirements.md       ← PRD
│   ├── architecture.md       ← This file
│   └── roadmap.md            ← Roadmap & sprint plans
├── src/
│   └── stock_screener/
│       ├── __init__.py
│       ├── universe.py       ← Universe Manager
│       ├── fetcher.py        ← Data Fetcher
│       ├── metrics.py        ← Metric Calculator
│       ├── screener.py       ← Screener (pass/fail)
│       ├── scorer.py         ← Scorer (ranking)
│       ├── reporter.py       ← Reporter (output)
│       └── config.py         ← Configuration models
├── data/
│   ├── raw/                  ← Cached raw API responses
│   └── universes/            ← Universe definition files (CSV)
├── notebooks/
│   └── exploration.ipynb     ← Jupyter for analysis & visualization
├── tests/
│   └── test_metrics.py
├── pyproject.toml
├── .venv/
└── .vscode/
```

---

## 7. Open Architecture Questions

| # | Question | Status |
| - | - | - |
| A-01 | SQLite or DuckDB vs. Parquet for caching? | Open |
| A-02 | CLI interface (click/typer) vs. notebook-first approach? | Open |
| A-03 | Should we add a web UI (Flask/FastAPI + React) in a later phase? | Open |
| A-04 | Which paid data provider to target for v2? | Open |
| A-05 | How to handle multiple simultaneous strategy configurations? | Open |
