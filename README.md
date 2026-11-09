# U.S. Treasury 10-Year Yield Curve Analysis Pipeline
## End-to-End Databricks Medallion Architecture (PySpark & SQL)

### Executive Overview
This project implements a multi-stage **Medallion Data Lakehouse Architecture** on Databricks to ingest, cleanse, and model daily U.S. Treasury 10-Year Yield Curve data. It combines Python-based REST API extraction with advanced Databricks SQL transformations, window functions, and time-series analytics tailored for downstream reporting and BI integration.

---

### Architecture & Data Flow

```text
  [ U.S. Treasury API ] 
            │
            ▼ (Python / Requests / PySpark)
   ┌─────────────────┐
   │  BRONZE LAYER   │  Raw string array ingestion & audit metadata appending
   │ (bronze_nasdaq) │
   └────────┬────────┘
            │
            ▼ (Databricks SQL / Regex CTE / Quality Filters)
   ┌─────────────────┐
   │  SILVER LAYER   │  Schema enforcement, array tokenization, date coercion,
   │ (silver_nasdaq) │  and holiday 'N/A' value cleansing
   └────────┬────────┘
            │
            ▼ (Databricks SQL / Window Functions)
   ┌─────────────────┐
   │   GOLD LAYER    │  Time-series analytics: Day-over-day deltas (LAG), 
   │  (gold_nasdaq)  │  30-day sliding moving average (AVG), & yearly ranks (DENSE_RANK)
   └─────────────────┘
```
---

### Technical Highlights & Concepts Demonstrated

* **Bronze (Ingestion):** Custom HTTP header configuration to bypass bot blocking; raw payload preservation using append-only Delta Lake tables with audit metadata (`ingested_at`, `source_url`).
* **Silver (Cleansing & Structuring):** CTE-driven string sanitization (`regexp_replace`, `split`), multi-format date parsing (`COALESCE` handling `MM/dd/yyyy` and `yyyy-MM-dd`), and explicit type casting (`DOUBLE`) combined with data quality filters.
* **Gold (Analytics & Windowing):**
  * **Navigation Offset (`LAG`):** Time-series relative lookup for day-over-day yield change without expensive self-joins.
  * **Physical Frame Aggregation (`AVG OVER ROWS BETWEEN`):** True 30-day physical sliding window moving average (`29 PRECEDING AND CURRENT ROW`).
  * **Partitioned Ranking (`DENSE_RANK`):** Annual reset (`PARTITION BY YEAR`) identifying top yield spikes per calendar year with consecutive ranking tie-handling.
