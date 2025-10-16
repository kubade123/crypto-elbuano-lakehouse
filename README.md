# Crypto Firehose Lakehouse

Binance → Kafka/Redpanda → Spark Structured Streaming → Iceberg on S3/MinIO → n8n alerts.

## Project Roadmap

The full phase-by-phase build plan lives in [`docs/ROADMAP.md`](docs/ROADMAP.md). It captures the staged definitions of done for infrastructure, streaming jobs, gold features, and the n8n AI layer.

## Getting Started

1. Clone the repository and review `.env.example` (to be provided in Phase 0 configs work).
2. Install Docker (Desktop) and ensure Compose V2 is enabled.
3. Run `docker compose up -d` to boot the local stack once environment files are available.
4. Visit the service UIs:
   - Redpanda Console → http://localhost:8080
   - MinIO → http://localhost:9001 (default `minio` / `minio123`)
   - Iceberg REST Catalog → http://localhost:8181/v1/config
   - Spark Master UI → http://localhost:8081
   - n8n → http://localhost:5678
5. Tail logs with `docker compose logs -f <service>` as needed.

## Repository Layout

```
├── configs/                 # Environment templates, Spark configs, etc.
├── docker-compose.yml       # Local infra stack (Redpanda, MinIO, Iceberg, Spark, n8n)
├── docs/                    # Roadmap and additional design notes
├── scripts/                 # Helper scripts (e.g., data smoke tests)
├── services/                # Service-specific code (producers, Spark jobs)
└── src/                     # Shared Python modules & utilities
```

## Status

Phase 0 is underway. See the roadmap for granular progress and upcoming deliverables.
