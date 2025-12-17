## Purpose

This file guides AI coding agents working on the Stock-Market-Crash-Predictor repo. Focus on making safe, minimal, and reproducible edits: prefer changing code/config over mutating large data files or notebooks unless asked.

## Big Picture
- **Primary pipeline:** `pipeline.ipynb` drives data acquisition, cleaning, and merging (prices, macro, extra FRED series). Agents should read it top-to-bottom to understand data flow.
- **Data flow:** `data/raw/*` (downloaded) → `data/cleaned/*` (per-series cleaned CSVs) → `data/merged/merged_dataset_cleaned.csv` (final daily merged dataset) → `output/` (predictions, models, plots).
- **Why notebooks:** Notebooks contain exploratory logic and reusable helper functions (download/clean/save). Changes to core logic should be moved into scripts under `src/` if modifications will be reused or tested.

## Important files & dirs
- `pipeline.ipynb` — canonical pipeline: download (yfinance, FRED, pytrends), clean, merge. Read variable names: `TICKERS`, `FRED_SERIES`, `EXTRA_FRED_DATA`, `START_DATE`, `TODAY`.
- `data/raw/`, `data/cleaned/`, `data/merged/` — naming conventions: raw price files use `*_prices_raw.csv`, cleaned price files use `*_prices_clean.csv`; macro raw files use `*_fred_raw.csv`, cleaned files use `*_fred_clean.csv`.
- `output/` — contains `crash_predictions.csv`, `crash_predictions_improved.csv`, `models/`, `plots/`.
- `src/config.py` — project-level configuration (check this before duplicating constants).

## Dependencies & environment
- Key Python packages used in notebooks: `pandas`, `yfinance`, `fredapi`, `pytrends`, `duckdb`, `python-dotenv`.
- The pipeline expects a `.env` with `FRED_API_KEY`. Do NOT commit API keys. If a run fails, verify `FRED_API_KEY` via `os.getenv("FRED_API_KEY")`.

Example quick install:
```
python -m pip install pandas yfinance fredapi pytrends duckdb python-dotenv
```

## Reproducible runs & developer workflow
- Preferred interactive flow: open and run `pipeline.ipynb` in order. Notebook cells contain helper functions (e.g., `download_prices`, `download_fred_series`, `clean_price_file`, `clean_macro_file`, merging logic).
- To convert to a runnable script for automation:
  - `jupyter nbconvert --to script pipeline.ipynb --output pipeline.py`
  - Inspect `pipeline.py`, refactor noisy notebook-specific prints into functions, then run `python pipeline.py`.
- Always ensure data directories exist before writing; do not commit large CSVs. Use gitignore for `data/` if not already ignored.

## Project-specific patterns & conventions
- Naming: friendly names (keys) map to external IDs; e.g., in `pipeline.ipynb` `TICKERS = {"sp500": "^GSPC", "vix": "^VIX"}` and FRED series are kept in `FRED_SERIES` / `EXTRA_FRED_DATA`.
- File suffixes: `_raw.csv` → download output, `_clean.csv` or `_fred_clean.csv` → cleaned series, `merged_dataset_cleaned.csv` → final merged dataset.
- Merging semantics: daily price data (S&P500+VIX) is merged with lower-frequency macro series using `how='left'`, then macro columns are forward-filled. Pay attention to date alignment (some macro series start on the 1st or 2nd of month).
- Logging: notebook uses `print()` statements liberally for progress/debug — preserve or convert to proper logging when moving logic into `src/`.

## Integration notes & gotchas
- Google Trends (`pytrends`) is rate-limited (429 errors); the pipeline currently documents this and disabled automatic bulk downloads. If implementing trends automation:
  - create a fresh `TrendReq()` per keyword, add long waits, or throttle requests via an external rate-limiter.
  - Prefer manual runs or gated automation to avoid transient 429 failures.
- FRED downloads require `FRED_API_KEY` set in environment; `fredapi` client is created inside the notebook. Handle client None-checks gracefully.
- Two macro series sometimes fail to download (noted in the notebook). If adding more series, update both `EXTRA_FRED_DATA` and the raw filename mapping in `FILES_FOR_EXTRA_MACRO_DATA`.

## How AI agents should make changes
- Read `pipeline.ipynb` fully before making edits. If a change affects reuse or testing, refactor notebook logic into `src/` modules and add unit tests.
- Avoid committing big dataset files; prefer editing code and small config files. If you must add generated artifacts for debugging, place them in a new `tmp/` directory and flag them in PR description.
- Provide short, focused PRs: one logical change per PR (e.g., add a new FRED series, fix date parsing, or refactor a helper into `src/`).

## Examples (common tasks)
- Add a new FRED series:
  1. Add mapping to `EXTRA_FRED_DATA` in `pipeline.ipynb`.
  2. Add corresponding filename to `FILES_FOR_EXTRA_MACRO_DATA`.
  3. Re-run the download + clean cells.
- Fix date alignment: inspect the specific cleaned file in `data/cleaned/macro/<name>_fred_clean.csv`, align to trading days, then merge.

## Questions for the maintainer (please confirm)
- Should `src/config.py` become the single source of truth for constants (TICKERS, START_DATE)?
- Is `data/` intended to be gitignored (recommended) or are some cleaned demos checked in?

If anything above is incomplete or unclear, tell me what you'd like expanded (examples, commands, or file references) and I will iterate.
