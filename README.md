# 💱 Currency Exchange ETL Pipeline — CBU (Central Bank of Uzbekistan)

An end-to-end **ETL pipeline** that pulls daily currency rates from the **CBU public API**, cleans/enriches them with **Python (pandas)**, loads them into **SQL Server**, and visualizes them in **Power BI**.

## Architecture

```
CBU API (JSON)
      │
      ▼
extract_to_raw.py  ──►  raw_currency_rates   (SQL Server)
                                │
                                ▼
transform_to_clean.py ──►  clean_currency_rates
                                │
                                ▼
                          Power BI Dashboard
```

Orchestrated by `run_pipeline.py`, logged by `logger.py`, configured centrally in `config.py`. Runs daily via Windows Task Scheduler.

## What each script does

- **`extract_to_raw.py`** — Calls the CBU API, inserts raw records (with original JSON) into `raw_currency_rates`.
- **`transform_to_clean.py`** — Cleans, dedupes, type-casts, and enriches the data (rate-per-unit, increase/decrease flags), loads into `clean_currency_rates` with duplicate protection.
- **`run_pipeline.py`** — Runs Extract → Transform, logs a run summary, returns exit codes for scheduling.
- **`demo_test.py`** — Runs the same logic offline on sample data — no DB or internet needed.
- **`config.py`** / **`logger.py`** — Central settings and console+file logging.

## Dashboard

Power BI connects live to `clean_currency_rates` and shows: KPI cards (total currencies, latest USD/EUR, increased/decreased counts), a rate trend chart, current rates by currency, daily change chart, a detail table, and the ETL flow diagram.

## Tech Stack

Python (`requests`, `pandas`, `pyodbc`) · SQL Server · Power BI · Windows Task Scheduler

## Setup

```bash
pip install -r requirements.txt
# install ODBC Driver 17 for SQL Server
# run sql/create_raw_table.sql and create_clean_table.sql in SSMS
# set your DB details in config.py

python demo_test.py      # optional: test offline
python run_pipeline.py   # run full pipeline
```

Then connected Power BI Desktop to `clean_currency_rates`. Scheduled `run_pipeline.py` in Task Scheduler for daily automation.

## Highlights

- Modular, production-style ETL (config/logging/extract/transform separated)
- Defensive error handling + full logging at each stage
- Idempotent loads (no duplicate rows on reruns)
- Offline demo mode for easy testing/demoing
