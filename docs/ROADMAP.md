# Project C — Crypto Firehose Roadmap

This roadmap tracks the staged build-out of the Binance → Lakehouse pipeline when running the project fully locally via Docker Compose ("Option A"). Each phase is scoped with a crisp definition of done (DoD) so that we know when to move forward.

> ✅ = exit criteria for the phase

## Phase 0 — Repo & Infrastructure Baseline

### Repo scaffold & policies
- GitHub repository initialized with README, LICENSE, `.gitignore`, and pre-commit / CI workflow.
- ✅ DoD: Repository checks (pre-commit & CI) pass.

### Local infrastructure (Docker Compose)
- Stand up local services: Redpanda + Console, MinIO, Iceberg REST Catalog, Spark (master/worker), n8n.
- ✅ DoD: Every UI reachable. MinIO bucket `lake` exists. Redpanda topic smoke test works.

### Project configs & `.env`
- Parameterize broker, object store, and Iceberg REST endpoints.
- ✅ DoD: `cp .env.example .env` filled with local overrides. `make up` starts stack.

## Phase 1 — Ingestion (Binance WS → Kafka)

### Binance WebSocket producer
- Python producer connects to `<symbol>@aggTrade` streams, handles reconnect/ping, normalizes payload, and produces to the `binance.aggtrade` topic.
- ✅ DoD: Messages visible in Redpanda Console with structured JSON (`symbol`, `agg_id`, `price`, `qty`, `trade_time`, ...).

## Phase 2 — Bronze (Kafka → Iceberg Append)

### Spark Structured Streaming to Bronze
- Consume from Kafka, parse JSON, add `event_ts` & `dt_hour`, and append into Iceberg `crypto_bronze_aggtrade` partitioned hourly (stored on MinIO).
- ✅ DoD: Iceberg table created, MinIO contains data files, and `spark-sql` queries return recent rows.

## Phase 3 — Silver (Clean + Dedup + Watermark)

### Watermark & deduplicate
- Stream from Bronze with `withWatermark(10m)` and `dropDuplicates(symbol, agg_id)` before merging into `crypto_silver_trades`.
- ✅ DoD: No duplicates per `(symbol, agg_id)` and late events (≤10m) land correctly.

### Data quality & quarantine
- Enforce `price > 0`, `qty > 0`, and non-null `event_ts`. Bad rows route to `crypto_quarantine`.
- ✅ DoD: Data quality logs report pass/fail counts and quarantine table captures forced errors.

## Phase 4 — Gold (Features & Signals)

### Feature engineering
- Calculate rolling VWAP (1m), volume (5m), price moving average (30m), and spike flags.
- ✅ DoD: `crypto_gold_minute` Iceberg table populated and returns sensible example values.

### Compaction & housekeeping
- Schedule Iceberg `rewrite_data_files` jobs to reduce small files and boost read performance.
- ✅ DoD: Average file size increases post-compaction and latency decreases on queries.

## Phase 5 — AI Layer (n8n)

### Market Pulse (cron)
- n8n workflow: Cron → query Gold layer → LLM summary (neutral tone) → Slack/Telegram notifications.
- ✅ DoD: Each run posts summary with `symbol`, `Δ%`, VWAP, volume, and spike status.

### Spike Explainer (event-driven)
- Spark emits webhooks on spike events; n8n generates short LLM narratives and posts to Slack/Jira.
- ✅ DoD: Induced spike triggers explainer message with narrative and supporting mini table.

## Phase 6 — Observability & Ops Polish

### Audit table & metrics
- Jobs emit `run_id`, `rows_in/out`, `max_lag_ms`, `dq_pass/fail` into `ops_audit`.
- ✅ DoD: `ops_audit` queryable for dashboards (Grafana or SQL-based charts).

### Lineage (optional)
- (Stretch) Emit OpenLineage events from orchestrator to Marquez for DAG visibility.
- ✅ DoD: Runs appear with upstream/downstream lineage links.
