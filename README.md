# Hindsight Setup for AI Agent 🧠

[Hindsight](https://github.com/vectorize-io/hindsight) — biomimetic agent memory system. Makes AI agents **learn**, not just remember.

## Architecture

```
┌──────────────┐    ┌──────────────────┐    ┌──────────────────────┐
│   Hindsight   │───▶│  PostgreSQL 18   │    │  Ollama bge-m3       │
│  latest-slim  │    │  + pgvector      │    │  (local embeddings)  │
│  API :8888    │    │  :5432           │    │  :11434              │
│  CP  :9999    │    └──────────────────┘    └──────────────────────┘
└──────┬───────┘
       │
       ├──▶ TEI cpu-1.9 (reranker) :8081
       │     cross-encoder/mmarco-mMiniLMv2-L12-H384-v1
       │
       └──▶ OpenAI-compatible LLM (9Router / OpenAI / Groq / etc.)
```

## Quick Start

```bash
# 1. Clone
git clone https://github.com/jefripunza/hindsight-setup-for-ai-agent.git
cd hindsight-setup-for-ai-agent

# 2. Copy & edit .env
cp .env.example .env
# Edit .env — set least:
#   HINDSIGHT_API_LLM_API_KEY=your-api-key
#   HINDSIGHT_API_LLM_BASE_URL=https://your-llm-provider.com/v1

# 3. Start
podman-compose up -d
# or: docker compose up -d

# 4. Verify
curl http://localhost:8888/health
# {"status":"healthy","database":"connected"}
```

## Access

| Service | URL |
|---------|-----|
| **Hindsight API** | `http://localhost:8888` |
| **Hindsight Control Plane** | `http://localhost:9999` |
| **TEI Reranker** | `http://localhost:8081` (internal) |

## Requirements

- **Podman** 5+ or **Docker** with compose plugin
- **Ollama** running locally with `bge-m3:latest` (for embeddings)
- ~4 GB free disk for images + model cache

## .env Configuration

| Variable | Required | Description |
|----------|----------|-------------|
| `HINDSIGHT_API_LLM_API_KEY` | **Yes** | API key for LLM provider |
| `HINDSIGHT_API_LLM_BASE_URL` | **Yes** | LLM API base URL |
| `HINDSIGHT_API_LLM_PROVIDER` | No | Default: `openai` |
| `HINDSIGHT_API_LLM_MODEL` | No | Default: `hermes` |

See `.env.example` for all available options.

## API Usage

```bash
# Create memory bank
curl -X PUT http://localhost:8888/v1/default/banks/my-bank \
  -H "Content-Type: application/json" \
  -d '{"bank_id":"my-bank"}'

# Store memories (Retain)
curl -X POST http://localhost:8888/v1/default/banks/my-bank/memories \
  -H "Content-Type: application/json" \
  -d '{"items":[{"content":"User prefers Python for data analysis"}]}'

# Search memories (Recall)
curl -X POST http://localhost:8888/v1/default/banks/my-bank/memories/recall \
  -H "Content-Type: application/json" \
  -d '{"query":"What programming language?"}'
```

## Data Persistence

All data lives in `./data/` bind mounts — safe from `docker compose down -v`.

| Directory | Contents |
|-----------|----------|
| `data/db/` | PostgreSQL + pgvector data |
| `data/tei/` | TEI reranker model cache |
| `data/hindsight/` | App state |

## Tear Down

```bash
podman-compose down
# Data survives in ./data/ — remove manually if needed:
# rm -rf data/
```
