# System Overview

Study Partner AI is a multi-agent study-assistance platform. Agents listen for
signals/sessions and answer through a shared LLM client; long-running work is
moved onto an async job bus (RabbitMQ) with idempotent, retryable workers.

## Agents

| Agent | Role | Key modules |
|-------|------|-------------|
| **Planner** | Decomposes learning goals into atomic tasks, applies pacing rules, RAG over course material | `agents/planner/` |
| **Coach** | Real-time coaching decisions + nudges (rules first, LLM fallback) | `agents/coach/` |
| **Scheduler** | Time-tabling with constraints, pacing factor | `agents/scheduler/` |
| **Search** | Web search + extraction + LLM synthesis of answers | `agents/search/` |
| **Course Ingestion** | PDF → structured course JSON (parsing, OCR, normalization), enrichment, task generation | `agents/course_ingestion/` |
| **Evaluator** | Socratic question generation and answer grading (template fallback) | `agents/evaluator/` |
| **Reflection** | Weekly reflection journaling from focus/fatigue/XP trends | `agents/reflection/` |

`agents/orchestrator.py` wires course ingestion → planner for the study-plan
flow used by root-level integration scripts.

## Services

| Service | Purpose |
|---------|---------|
| `services/api` | FastAPI entry point (`/api/v1/session`, ingestion routes, etc.) |
| `services/ai_orchestrator` | Coordinates agent interactions (legacy direct HTTP routing) |
| `services/schedule_orchestrator` | Schedule management over the planner/scheduler output |
| `services/signal_processing_service` | EMA smoothing + trend computation for focus/fatigue signals |
| `services/vector_store` | SentenceTransformers embedder + FAISS/MongoDB adapter for RAG |
| `services/database.py` | Shared Mongo wiring |

## Job bus (RabbitMQ)

`messaging/` defines the envelope, topology, and failure handling;
`workers/` implements consumers:

- `workers/coach_worker.py` — consumes coaching jobs; rejects → `TerminalError`
  → dead-letter on validation failures (COACH-06 semantics)
- `workers/planner_worker.py` — planning jobs
- `workers/idempotency.py` — idempotent job completion

Coach/planner LLM failures on the bus surface as job retry events; per-agent
rule-based fallbacks keep the product working without a live LLM.

## LLM access

All agents call the LLM through `utils/llm_client.ask(agent, system, user, ...)`.
Model selection, temperature, and fallback chains live in
`litellm/config.yaml` (see [LLM Configuration](llm-config.md)).

Every user-derived string is wrapped in `UNTRUSTED` blocks by
`security/prompt_guard` (`build_system_block`, `wrap_untrusted`) so prompt
injection in user data cannot alter the system role.

## Cross-cutting

- **Logging** — structured JSON via `utils/logger.py`, `trace_id` propagated
  API → orchestrator → agent → LLM call.
- **Persistence** — MongoDB collections for coach history, pacing memory,
  reflections, embedding chunks; FAISS disk indices under `agents/planner/rag/`.
- **Tracking** — progress against the sprint backlog lives in
  [Implementation Tracker](implementation-tracker.md).

For a detailed runtime data-flow report (vector store pipeline, coach memory,
retrieval paths), see [Agents Data Flow & Structure Report](reference/agents-dataflow-report.md).