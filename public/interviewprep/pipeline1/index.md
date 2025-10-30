# Data Pipeline

# Databricks Medallion Playbook (ADLS CSV → Delta)

> End‑to‑end guide for **incremental loading**, **data validations/quality**, and **SCD Type 2** using **Databricks** + **ADLS** with **Medallion** layers: *landing → raw (bronze) → rawint (silver‑staging) → curated (silver) → enriched (gold).* Uses **Delta Lake** everywhere.

---

## 0) Principles & Conventions

**General**

* Use **Delta** for all managed tables; avoid plain CSV/Parquet beyond landing.
* Prefer **Auto Loader** for incremental files; fall back to `COPY INTO` for one‑off backfills.
* Treat *raw* as immutable; never update, only append/reprocess.
* Store **ingestion metadata** on every row: `ingest_id`, `ingest_ts`, `source_file`, `source_mod_ts`, `batch_id`.

**Naming**

* Workspaces & Catalogs: `uc_catalog = <org>_prod|stg`, `uc_schema = <domain>_<layer>` (e.g., `sales_raw`, `sales_curated`).
* Tables: `<entity>_<granularity>` (snake_case). Examples: `orders_header`, `orders_line`.
* Columns: `snake_case` (e.g., `order_id`, `valid_from`).
* Notebook files: `<layer>-<domain>-<entity>-<purpose>.nb` (kebab-case). Example: `raw-sales-orders-ingest.nb`.
* Jobs: `<layer>.<domain>.<entity>.<schedule>` (e.g., `raw.sales.orders.hourly`).
* Checkpoint & schema locations: `/checkpoints/<domain>/<entity>/<layer>/` and `/schemas/<domain>/<entity>/`.

**Source control & CI/CD**

* Store notebooks as **Databricks Repos** (files) with code‑review; use **bundle**/IaC to deploy.
* Parameterize via widgets or job parameters; keep **secrets** in Key Vault / Databricks secrets.

**Security & Governance**

* Use **Unity Catalog** (UC) for data access, lineage, tags, and ownership.
* Assign owners per schema/table; define data classifications via UC tags (e.g., `pii=true`).

---

## 1) ADLS Folder Taxonomy (Sources as CSV)

```
/adls/<container>/landing/
  └── <domain>/
       └── <source_system>/
            └── <entity>/
                 ├── year=YYYY/ month=MM/ day=DD/  # preferred partitioned landing
                 │    └── <entity>_<YYYYMMDD_HHMMSS>_<batchid>.csv
                 └── quarantine/  # failed/invalid files
```

Notes

* Keep **one entity per folder**; include date partitions or place date/batch in filename.
* Preserve file immutability; never delete sources, only move to `quarantine/` when necessary.

---

## 2) Medallion Table Layout in Unity Catalog

* **Catalog**: `<org>_prod` (and `<org>_stg` for non‑prod).
* **Schemas (per domain & layer)**:

  * `sales_raw` (bronze), `sales_rawint` (silver‑staging), `sales_curated` (silver), `sales_enriched` (gold).
* **Managed Storage**: Delta tables stored in UC‑managed locations; external volumes for checkpoints/schemas.

---

## 3) Incremental Ingestion (Landing → Raw/Bronze)

**Preferred: Auto Loader (cloudFiles)**

* Handles **new files only**, scalable file discovery, schema inference & evolution.

**PySpark pattern**

```python
from pyspark.sql.functions import input_file_name, current_timestamp

source_path = "/Volumes/<catalog>/<schema>/<volume>/landing/sales/erp/orders/"  # or abfss://
chkpt_path   = "/Volumes/<catalog>/<schema>/<volume>/checkpoints/sales/orders/raw/"
schema_loc   = "/Volumes/<catalog>/<schema>/<volume>/schemas/sales/orders/"
raw_tbl      = "<catalog>.sales_raw.orders"

(df := (spark.readStream
        .format("cloudFiles")
        .option("cloudFiles.format", "csv")
        .option("header", True)
        .option("inferSchema", True)
        .option("cloudFiles.schemaLocation", schema_loc)
        .load(source_path)
        .withColumn("source_file", input_file_name())
        .withColumn("ingest_ts", current_timestamp())
       )) \
  .writeStream \
  .format("delta") \
  .option("checkpointLocation", chkpt_path) \
  .trigger(availableNow=True) \
  .toTable(raw_tbl)
```

**Alternative: COPY INTO (batch/backfill)**

```sql
COPY INTO <catalog>.sales_raw.orders
FROM 'abfss://<container>@<account>.dfs.core.windows.net/landing/sales/erp/orders/'
FILEFORMAT = CSV
FORMAT_OPTIONS ('header'='true')
COPY_OPTIONS ('mergeSchema'='true');
```

**Raw Table Schema** (append‑only)

* Original columns as in source.
* * `ingest_ts TIMESTAMP`, `source_file STRING`, `batch_id STRING` (optional), `ingest_id STRING` (uuid).

---

## 4) Data Quality & Validations (Raw → RawInt)

**Approach**

* Apply **schema enforcement**, **type casting**, **conformance** (trim, null handling, defaulting), and **row‑level rules**.
* Route failed rows to **quarantine table** with error reasons.

**Delta Expectations (simple built‑in checks)**

```sql
CREATE TABLE IF NOT EXISTS <catalog>.sales_rawint.orders_expectations (
  rule_name STRING, rule_expr STRING, violated_count BIGINT, batch_ts TIMESTAMP
);
```

```python
from pyspark.sql.functions import col, length, when, current_timestamp

raw = spark.table("<catalog>.sales_raw.orders")

validated = (raw
  .withColumn("order_id", col("order_id").cast("string"))
  .withColumn("order_date", col("order_date").cast("date"))
  .withColumn("amount", col("amount").cast("decimal(18,2)"))
)

# Rule examples
rules = {
  "not_null_order_id": "order_id IS NOT NULL",
  "valid_amount": "amount >= 0",
  "date_present": "order_date IS NOT NULL"
}

ok_df    = validated.where(" AND ".join(rules.values()))
rejects  = validated.exceptAll(ok_df) \
  .withColumn("error_reason", when(col("order_id").isNull(), "order_id_null")
                           .when(col("amount") < 0, "amount_negative")
                           .otherwise("generic_failure")) \
  .withColumn("batch_ts", current_timestamp())

ok_df.write.mode("append").saveAsTable("<catalog>.sales_rawint.orders")
rejects.write.mode("append").saveAsTable("<catalog>.sales_rawint.orders_rejects")
```

**Best practices**

* Keep **idempotency**: filter by `ingest_ts`/`batch_id` when reprocessing.
* Log metrics per batch to an **observability** table and dashboard in **Lakeview/Grafana**.

---

## 5) Deduplication & Keys (RawInt → Curated/Silver)

* Define **business key** (e.g., `order_id` + `source_system`).
* Deduplicate by key & latest `source_mod_ts`/`ingest_ts`.

```sql
CREATE OR REPLACE TABLE <catalog>.sales_curated.orders_dedup AS
SELECT * EXCEPT (rn)
FROM (
  SELECT *, ROW_NUMBER() OVER (PARTITION BY order_id ORDER BY source_mod_ts DESC, ingest_ts DESC) rn
  FROM <catalog>.sales_rawint.orders
) t
WHERE rn = 1;
```

---

## 6) SCD Type 2 in Curated (Slowly Changing Dimensions)

**Target layout**

* Columns: business keys + attributes + `valid_from TIMESTAMP`, `valid_to TIMESTAMP`, `is_current BOOLEAN`, optional `record_hash`.

**Upsert logic (MERGE)**

```sql
-- Staging changes (latest per key)
CREATE OR REPLACE TEMP VIEW v_stg AS
SELECT * FROM <catalog>.sales_curated.customer_changes_latest; -- build from rawint

MERGE INTO <catalog>.sales_curated.dim_customer AS tgt
USING v_stg AS src
ON tgt.customer_id = src.customer_id AND tgt.is_current = true
WHEN MATCHED AND tgt.hash <> src.hash THEN
  -- close old record
  UPDATE SET tgt.valid_to = current_timestamp(), tgt.is_current = false
WHEN NOT MATCHED THEN
  -- insert new current version
  INSERT (customer_id, name, city, hash, valid_from, valid_to, is_current)
  VALUES (src.customer_id, src.name, src.city, src.hash, current_timestamp(), TIMESTAMP('9999-12-31'), true);
```

**Hash diff**

```sql
SELECT *, sha2(concat_ws('||', coalesce(name,''), coalesce(city,'')), 256) AS hash
FROM <catalog>.sales_rawint.customer_clean;
```

**Querying current/historical**

```sql
-- Current snapshot
SELECT * FROM <catalog>.sales_curated.dim_customer WHERE is_current = true;
-- As-of time travel
SELECT * FROM <catalog>.sales_curated.dim_customer WHERE valid_from <= ts AND valid_to > ts;
```

---

## 7) Enriched/Gold Layer

* Build **fact** tables (e.g., `fact_orders`) and **semantic marts** (aggregates, BI‑ready models).
* Maintain **surrogate keys** via identity columns or mapping tables.
* Enforce **Z‑ORDER** on common predicates and **OPTIMIZE** periodically.

```sql
OPTIMIZE <catalog>.sales_curated.dim_customer ZORDER BY (customer_id);
VACUUM <catalog>.sales_curated.dim_customer RETAIN 168 HOURS; -- 7 days
```

---

## 8) Orchestration & Scheduling

* **Databricks Jobs** per domain/entity with dependency graph: `raw → rawint → curated → enriched`.
* Triggers: time‑based (cron) or event‑based (file arrival) using Auto Loader.
* Pass parameters: `domain`, `entity`, `since_ts` for incremental windows.
* Implement **retry with backoff**, **timeouts**, and **alerts** to email/Slack.

---

## 9) Notebooks You’ll Need (by layer)

**Landing/Raw**

1. `raw-<domain>-<entity>-ingest.nb` – Auto Loader stream job.
2. `raw-<domain>-<entity>-backfill.nb` – `COPY INTO` for historical loads.

**RawInt (Validation/Conformance)**
3. `rawint-<domain>-<entity>-dq.nb` – type casting, rules, rejects, metrics.

**Curated (Modeling/SCD2)**
4. `curated-<domain>-<entity>-dedup.nb` – dedup/windowing.
5. `curated-<domain>-<entity>-scd2.nb` – hash diff + MERGE.

**Enriched**
6. `enriched-<domain>-<entity>-facts.nb` – facts/aggregations.
7. `enriched-<domain>-<entity>-optimize.nb` – OPTIMIZE/VACUUM.

**Ops/Utils**
8. `utils-params.nb` – common params & widgets.
9. `utils-common-fns.nb` – reusable functions (ingest metadata, logging, hashing).
10. `utils-observability.nb` – metrics tables & dashboards.

---

## 10) Parameters (shared fragment)

```python
# utils-params.nb
import uuid, datetime as dt

dbutils.widgets.text("domain", "sales")
dbutils.widgets.text("entity", "orders")
dbutils.widgets.text("source_path", "/Volumes/.../landing/${domain}/${entity}")
dbutils.widgets.text("catalog", "<org>_prod")

cfg = {k: dbutils.widgets.get(k) for k in ["domain","entity","source_path","catalog"]}
cfg["ingest_id"] = str(uuid.uuid4())
```

---

## 11) Copying from ADLS → Delta (summary)

* **Auto Loader** streaming job (preferred) from `abfss://` paths with checkpoints & schema evolution.
* **COPY INTO** for bulk loads/backfills.
* Always write to **Delta tables** in UC; never keep CSV beyond landing.

---

## 12) Performance & Cost Best Practices

* **Partition** large tables by low‑cardinality/time columns (e.g., `load_date`); avoid over‑partitioning.
* Periodic **OPTIMIZE** + **Z‑ORDER** on query keys.
* Use **availableNow** triggers for micro‑batch bulk catch‑up.
* Use **Auto Compaction** and **Optimize Write** (Workspace settings / table properties).

---

## 13) Observability & Lineage

* Create **metrics tables** (rows in/out, rejects, rule violations, late files).
* Build **Lakeview** dashboards.
* Use **Unity Catalog lineage** for end‑to‑end traceability.

---

## 14) What You Might Have Missed

* **Quarantine flows** with reason codes.
* **PII handling**: column‑level masking (views) + UC privileges; store hashes/salts if needed.
* **Idempotent reprocessing** by `batch_id` or `ingest_ts` windows.
* **Schema evolution** policy: allow additive only in raw/rawint; controlled evolution in curated.
* **Backfill strategy** separate from incremental.
* **Vacuum retention** policy aligned to recovery/RPO.
* **Dev/Stg/Prod** isolation via separate catalogs & volumes.

---

## 15) Minimal End‑to‑End Flow (Checklist)

1. **Create UC** catalog/schemas and ADLS volumes; set grants.
2. **Define ADLS landing taxonomy** per domain/source/entity.
3. **Implement Auto Loader** (landing→raw) with checkpoints & schema locations.
4. **Add validations** (raw→rawint) + rejects/quarantine + metrics.
5. **Deduplicate & standardize** (rawint→curated).
6. **Implement SCD2** dimensions and load **facts** (curated→enriched).
7. **Optimize** tables and set maintenance (OPTIMIZE, VACUUM).
8. **Schedule jobs**, retries, and alerts.
9. **Monitor** with metrics dashboards & UC lineage.

---

## 16) Quick SCD2 Example (Complete)

```sql
-- 1) Clean latest changes per key from rawint
CREATE OR REPLACE TEMP VIEW v_latest AS
SELECT * EXCEPT (rn)
FROM (
  SELECT *, ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY source_mod_ts DESC, ingest_ts DESC) rn
  FROM <catalog>.sales_rawint.customer_clean
) t WHERE rn=1;

-- 2) Stage with hash diffs
CREATE OR REPLACE TEMP VIEW v_stage AS
SELECT *, sha2(concat_ws('||', coalesce(name,''), coalesce(city,''), coalesce(status,'')), 256) AS hash
FROM v_latest;

-- 3) Merge into SCD2 dim
MERGE INTO <catalog>.sales_curated.dim_customer tgt
USING v_stage src
ON tgt.customer_id = src.customer_id AND tgt.is_current = true
WHEN MATCHED AND tgt.hash <> src.hash THEN
  UPDATE SET tgt.valid_to = current_timestamp(), tgt.is_current = false
WHEN NOT MATCHED THEN
  INSERT (customer_id, name, city, status, hash, valid_from, valid_to, is_current)
  VALUES (src.customer_id, src.name, src.city, src.status, src.hash, current_timestamp(), TIMESTAMP('9999-12-31'), true);
```

---

## 17) Housekeeping Snippets

```sql
-- Optimize & clean
OPTIMIZE <catalog>.sales_curated.dim_customer ZORDER BY (customer_id);
VACUUM <catalog>.sales_curated.dim_customer RETAIN 168 HOURS;

-- Table properties (enable auto tuning)
ALTER TABLE <catalog>.sales_curated.dim_customer SET TBLPROPERTIES (
  'delta.autoOptimize.optimizeWrite' = 'true',
  'delta.autoOptimize.autoCompact'  = 'true'
);
```

---

### Final Notes

* The correct term is **Medallion** (not *medtallion*). The `rawint` layer is optional; it’s useful to isolate validation from business modeling.
* Consider **DLT** for declarative pipelines (expectations + lineage) if you want fewer custom notebooks; patterns above map 1:1 to DLT.
