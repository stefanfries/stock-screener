# Roadmap

## Stock Screener & Evaluator

**Version:** 0.1 (Draft)  
**Last Updated:** 2026-03-09  
**Status:** Planning

---

## 1. Guiding Principles

- **Ship working software early** — even a basic screener that works is more valuable than a perfect one that doesn't exist yet
- **Iterate in short cycles** — each sprint delivers something usable and testable
- **Requirements drive development** — no feature gets built without a clear requirement
- **Learning is part of the work** — time for research and experimentation is built in

---

## 2. High-Level Phases

| Phase | Name | Goal | Status |
| - | - | - | - |
| **Phase 0** | Foundation | Project setup, tooling, data exploration | 🟡 In Progress |
| **Phase 1** | MVP Screener | Working screener with fundamental + performance scores | ⬜ Not Started |
| **Phase 2** | Polish & Usability | Reports, CLI, configuration, reproducibility | ⬜ Not Started |
| **Phase 3** | Warrant Module | Identify warrant candidates from screener output | ⬜ Not Started |
| **Phase 4** | Advanced Features | Backtesting, alerts, web UI, paid data | ⬜ Not Started |

---

## 3. Phase 0 — Foundation

**Goal:** Everything needed before writing business logic.  
**Duration:** ~1–2 weeks

### Deliverables

- [x] Repository created and initialized
- [x] Virtual environment set up (uv + Python 3.13)
- [x] VS Code configured
- [ ] Requirements documented (`docs/requirements.md`) ← In discussion
- [ ] Architecture documented (`docs/architecture.md`) ← In discussion
- [ ] Core dependencies installed (`pandas`, `numpy`, `yfinance`)
- [ ] Jupyter notebook for data exploration
- [ ] Fetch data for 5–10 test stocks manually (proof of concept)
- [ ] Understand what data yfinance provides (prices, fundamentals, ratios)
- [ ] Define initial investment universe (answer D-01 from requirements)
- [ ] Agree on initial metric weights (answer from requirements D-02)

---

## 4. Phase 1 — MVP Screener

**Goal:** A working end-to-end screener that can screen a universe and output ranked results.  
**Duration:** ~3–4 sprints (6–8 weeks)

---

### Sprint 1 — Universe & Data Fetching

**Goal:** Load a universe of stocks and fetch raw data for all of them.

| Task | Description | Notes |
| - | - | - |
| Implement `UniverseManager` | Load universe from CSV or hardcoded index list | Start with S&P 500 or DAX |
| Implement `DataFetcher` | Wrap yfinance calls for price history + fundamentals | |
| Implement caching layer | Save/load Parquet files with TTL logic | Avoids hitting API on every run |
| Write basic tests | Test that fetched data has expected columns/types | |

**Definition of Done:** Running `python -m stock_screener fetch` fetches and caches data for all universe stocks.

---

### Sprint 2 — Metric Calculation

**Goal:** Compute all fundamental and performance metrics from raw data.

| Task | Description | Notes |
| - | - | - |
| Implement performance metrics | 1M, 3M, 6M, 12M return, relative strength, beta, volatility | |
| Implement fundamental metrics | ROE, ROA, margins, growth rates, D/E, current ratio | |
| Implement valuation metrics | P/E, EV/EBITDA, P/B, PEG | |
| Handle missing data | Define strategy: skip metric, use 0, use median | |
| Write tests | Unit tests for each metric calculation | |

**Definition of Done:** For any stock in the universe, all metrics can be computed and are returned as a clean dict/DataFrame.

---

### Sprint 3 — Screening & Scoring

**Goal:** Apply pass/fail screens and produce a ranked shortlist.

| Task | Description | Notes |
| - | - | - |
| Implement `Screener` | Apply configurable pass/fail filters | |
| Implement `Scorer` | Percentile ranking per metric, weighted composite | |
| Implement strategy config | YAML/JSON config for weights and thresholds | |
| Write tests | Test screening exclusions, test score ordering | |

**Definition of Done:** Running the pipeline produces a ranked DataFrame of stocks that passed screening, with per-dimension and composite scores visible.

---

### Sprint 4 — Basic Output

**Goal:** Make results readable and usable.

| Task | Description | Notes |
| - | - | - |
| Console output | Formatted table of top N stocks | |
| CSV export | Export full scored universe + shortlist to CSV | |
| Screening rejection log | Show which stocks were excluded and why | |
| Notebook visualization | Basic charts: score distributions, top 10 bar charts | |

**Definition of Done:** Someone can run the tool and understand the results without looking at code.

---

## 5. Phase 2 — Polish & Usability

**Goal:** Make the tool reliable and easy to use repeatedly.  
**Duration:** ~2–3 sprints

### Planned Work

- CLI interface (using `typer` or `click`) — run screener from the command line with options
- HTML report generation (Jinja2 template)
- Strategy profiles — quickly switch between "growth", "value", "momentum" weighting configs
- Improved error handling and logging
- Data quality checks and alerts (e.g., warn when metric data seems stale)
- Full test suite with >80% coverage

---

## 6. Phase 3 — Warrant Module

**Goal:** Extend the screener to identify warrant/warrant candidates.  
**Duration:** ~2 sprints

### Planned Work

- Add warrant availability check (does the stock have liquid derivatives?)
- Volatility analysis for warrant premium estimation
- Separate output section for warrant candidates
- Documentation of warrant evaluation criteria

---

## 7. Phase 4 — Advanced Features (Backlog)

*These are ideas for the future — not committed to any timeline.*

| Feature | Description |
| - | - |
| Backtesting | Test historical screener performance — did top-ranked stocks outperform? |
| Portfolio optimizer | Combine screener with mean-variance or risk-parity optimization |
| Alerting | Notify when a stock's score crosses a threshold |
| Web UI | Browser-based dashboard (FastAPI + React or Streamlit) |
| Paid data upgrade | Migrate to Polygon.io or FMP for higher reliability |
| Sentiment layer | Add news/earnings call sentiment as an additional factor |

---

## 8. Sprint Log

*Record of completed sprints — to be filled in as work progresses.*

| Sprint | Period | Goal | Status | Notes |
| - | - | - | - | - |
| Phase 0 | 2026-03-09 → ongoing | Foundation & setup | 🟡 In Progress | Docs being created |

---

## 9. Decisions Needed Before Sprint 1

Before writing the first line of business logic, we need answers to these questions from the requirements:

| # | Question |
| - | - |
| D-01 | Which index forms the initial investment universe? |
| D-02 | Initial metric weights (or start with equal weights and tune later)? |
| A-02 | Start with a CLI or notebook-first approach? |
