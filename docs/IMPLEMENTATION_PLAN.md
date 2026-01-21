# Crypto Market Intelligence Platform: Phase 1 Implementation Plan

This plan translates the PRD and the Implementation & Build Guide into a concrete, phased execution checklist for building the local-first, agentic crypto market intelligence platform. It emphasizes deterministic signal detection, explainable AI evaluation, and Slack/terminal delivery, while keeping the architecture extensible for future phases.

## 1. Guiding Principles

- **Decision support, not trading automation.**
- **Deterministic logic before AI.**
- **Plain-English outputs only.**
- **Local-first operation with optional cloud LLMs.**
- **Extensible architecture for Phase 2+ features.**

## 2. Target Architecture (Phase 1)

- **Services (Python/FastAPI):**
  - Data Ingestion
  - Signal Detection
  - AI Evaluation
  - Output Service
- **Orchestration:** n8n workflows (5-minute polling cadence)
- **Data Store:** Postgres
- **AI Models:** Ollama (local), optional Claude (cloud)
- **Outputs:** Terminal + Slack

## 3. Repository Structure (Initial Scaffold)

Create the following top-level structure to keep boundaries clear:

```
docker-compose.yml
.env.example
docs/ (PRD.md, ARCHITECTURE.md, IMPLEMENTATION_PLAN.md)
config/ (assets.yaml, signals.yaml, sources.yaml, outputs.yaml)
services/
  data-ingestion/
  signal-detection/
  ai-evaluation/
  output-service/
  shared/
n8n/workflows/
database/init.sql
scripts/ (setup.sh, health-check.sh, reset-db.sh)
```

## 4. Phase Plan & Milestones

### Phase 1: Infrastructure & Repo Bootstrap (Days 1–2)

**Goal:** Services can start locally with Docker Compose.

- Create repository structure and docs.
- Add `docker-compose.yml` with Postgres, n8n, Ollama, and four service containers.
- Add `.env.example` with required keys.
- Add Postgres schema (`database/init.sql`).
- Add initial config files under `config/`.

**Success criteria:**
- `docker-compose up -d --build` brings up Postgres, n8n, Ollama.
- DB schema initializes successfully.

### Phase 2: Shared Module + Data Ingestion (Days 3–5)

**Goal:** Fetch and store raw asset + social data.

- Build shared models (`AssetData`, `SocialData`, `Signal`, `Evaluation`).
- Add shared logging, config loader, DB connection pool.
- Implement CoinGecko + Reddit fetchers.
- Implement validation rules (no negative price/volume, timestamps fresh).
- Save validated data into `raw_asset_data` and `social_data`.

**Success criteria:**
- `/fetch/price-data` and `/fetch/social-data` endpoints return data.
- Postgres tables show fresh rows for ≥3 assets.

### Phase 3: Signal Detection (Days 6–8)

**Goal:** Deterministic signal detection for four signal types.

- Implement detectors:
  - Rapid price movement
  - Volume spike
  - Liquidity thinning
  - Social surge
- Enforce:
  - ≥24h history requirement
  - 15-minute dedupe window
  - Skip invalid/stale data
- Store signals in `signals` table with evidence + baselines.

**Success criteria:**
- Manual test triggers at least one signal type.
- Signals are stored with correct evidence/baseline structure.

### Phase 4: AI Evaluation (Days 9–11)

**Goal:** Multi-evaluator consensus and explainable outputs.

- Implement evaluators:
  - Significance Assessor (local Ollama)
  - Explanation Generator (local or Claude)
  - Risk Highlighter (local Ollama)
- Add structured prompts with few-shot examples.
- Implement arbitrator (≥2 evaluators must flag for output).
- Log all AI I/O in `evaluations`.

**Success criteria:**
- AI evaluations complete within 15 seconds per signal.
- Outputs are plain English (manual review).

### Phase 5: Output Service (Days 12–13)

**Goal:** Deliver insights via terminal + Slack.

- Terminal formatter with color coding.
- Slack formatter using Block Kit.
- Senders: stdout + Slack webhook.
- Support filtering by asset, signal type, confidence.

**Success criteria:**
- Slack messages are formatted per spec.
- Terminal log shows color-coded summaries.

### Phase 6: Orchestration (Days 14–15)

**Goal:** Automated 5-minute pipeline.

- Build n8n workflow:
  - Fetch data → Detect signals → Evaluate → Output
- Add error handling and retry nodes.
- Save workflows as JSON in `n8n/workflows/`.

**Success criteria:**
- Pipeline runs for 1 hour without failure.
- Signals flow end-to-end without manual intervention.

### Phase 7: Testing & Hardening (Days 16–20)

**Goal:** Reliability, logs, and coverage for deterministic logic.

- Unit tests for detectors and validators.
- Integration tests for data flow and DB writes.
- Health check script for all services.
- Resource sanity checks (CPU/RAM within spec).

**Success criteria:**
- ≥80% coverage for deterministic logic.
- System runs 24–48 hours without crash.

## 5. Required Deliverables (Phase 1)

- `docker-compose.yml`, `.env.example`
- Config YAMLs (assets, signals, sources, outputs)
- Postgres schema (`init.sql`)
- Service code (ingestion, signals, AI eval, output)
- Shared module
- n8n workflow JSON exports
- Scripts: setup, health-check, reset DB
- Updated README with setup + run instructions

## 6. Acceptance Checklist

- ✅ Polling interval configurable 1–5 minutes.
- ✅ 4 signal types implemented with evidence + baselines.
- ✅ Multi-evaluator consensus logic in AI layer.
- ✅ Plain-English outputs with risk flags.
- ✅ Terminal + Slack delivery working.
- ✅ Local-first operation with optional cloud LLM.

## 7. Risks & Mitigations

- **API rate limits** → exponential backoff + caching.
- **Data outages** → log and continue per-asset.
- **AI failures** → deterministic fallback (magnitude rules).
- **Alert fatigue** → conservative thresholds + user tuning in config.

## 8. Immediate Next Actions

1. Bootstrap the repo structure + docs.
2. Add Docker Compose and Postgres schema.
3. Implement shared module.
4. Build data ingestion service and verify DB writes.

