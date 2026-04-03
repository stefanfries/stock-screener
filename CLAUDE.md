# Stock Screener

(In design/early development) Data pipeline and scoring engine for systematic stock screening.
Combines data ingestion, metric calculation, multi-dimensional scoring, and ranking to identify
high-quality stocks within a defined investment universe.

## How to run

```bash
uv sync
uv run python -m app.main
```

## Planned architecture (4 layers)

```
1. Data Ingestion    → fetch from yfinance (free, no API key needed)
2. Storage           → local Parquet files with TTL-based caching
3. Processing        → screening filters → metric calculation → ranking
4. Output            → CSV, HTML report, console, Jupyter notebooks
```

Key modules (proposed):
- `src/stock_screener/universe.py` — define investment universe (index, file, or filters)
- `src/stock_screener/fetcher.py` — fetch and cache price + fundamental data
- `src/stock_screener/metrics.py` — compute performance, fundamental, valuation metrics
- `src/stock_screener/screener.py` — apply pass/fail filters
- `src/stock_screener/scorer.py` — percentile ranking + weighted composite score
- `src/stock_screener/reporter.py` — output generation

## Screening criteria (pass/fail)

- Market cap > €1B
- Daily volume > 500k shares
- Positive EPS
- Debt-to-Equity < 3.0
- 5+ years trading history

## Scoring dimensions

| Dimension | Weight |
|-----------|--------|
| Fundamental quality (revenue/EPS growth, margins, ROE, FCF, leverage) | 40% |
| Performance / momentum (1M–12M returns, relative strength, volatility) | 40% |
| Valuation (P/E, EV/EBITDA, P/B, PEG) | 20% |

Scoring uses percentile ranking so metrics are comparable across different scales.

## Open design decisions

- SQLite/DuckDB vs. Parquet for caching?
- CLI (click/typer) vs. notebook-first interface?
- Which paid data provider for v2 (after yfinance)?
- Web UI in a later phase?

See `docs/requirements.md` and `docs/architecture.md` for full PRD and ADRs.
