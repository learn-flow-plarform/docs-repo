# Architecture — Platform

The platform is three collaborating repositories:

```
study-partner-web        React frontend (users interact here)
        │  HTTPS /api/*, /ws
        ▼
study-partner-api        Express microservices behind an API Gateway
        │  AI-bound requests → ai-orchestrator-service (:3004)
        ▼
study-partner-ai         Python agents + job bus + shared LLM client
```

## Request flow (end-to-end)

1. The **web app** calls `/api/*` (same-origin in dev via the Vite proxy,
   `VITE_API_URL` in production). Websocket traffic uses `/ws` for real-time
   features.
2. The **API Gateway** (`:3000`) authenticates (JWT in httpOnly cookies),
   rate-limits, and routes to the backing microservices.
3. The **AI Orchestrator** (`:3004`) converts study-planning, coaching,
   ingestion, and signal requests into calls to the **Python AI** service.
4. The **Python AI** service runs agents synchronously (FastAPI) or pushes
   long-running work (planning, coaching) onto the **RabbitMQ job bus**
   consumed by workers, with per-agent LLM calls through one **LiteLLM**
   client.

## Backend (study-partner-api)

Express microservices, one process per concern, orchestrated by Docker
Compose on the `study-partner-network`.

| Service | Port | Role |
|---------|------|------|
| API Gateway | 3000 | Request routing, rate limiting, monitoring |
| Auth | 3001 | JWT auth (register/login/me, refresh, OTP, email verify) |
| User Profile | 3002 | Profiles, availability, gamification, goals |
| Study | 3003 | Tasks, topics, sessions, courses, plans |
| AI Orchestrator | 3004 | Proxy to Python AI; coach history; signal snapshots |
| Signal Processing | 3005 | Focus session tracking |
| Analytics | 3006 | Event tracking & insights |
| Notification | 3007 | In-app notifications |

Cross-cutting code lives in the `@study-partner/shared` package (rate limit,
CORS, Winston logging, auth/tier gates, DB connection). Redis backs rate
limiting/caching; RabbitMQ is reserved for the AI job bus.

Full details: [Backend API](backend-api.md).

## Frontend (study-partner-web)

React single-page app (Vite). Key areas: onboarding/auth, dashboard, study
planner + sessions, subjects, characters & store (Stripe checkout), social
(friends, leaderboard, teams), AI search, voice/WebRTC chat, and admin panels.

It talks to the API through one axios instance (relative `/api/*`, cookies
`withCredentials`) and a shared `authStore` that guards the refresh-token
flow against races.

Full details: [Frontend Web](frontend-web.md).

## AI layer (study-partner-ai)

Python multi-agent service. The **planner**, **coach**, **scheduler**,
**search**, **course ingestion**, **evaluator**, and **reflection** agents
answer through a shared LLM client (`utils/llm_client.ask`) with per-agent
models and fallback chains declared in `litellm/config.yaml`.

Long-running AI work travels over a RabbitMQ job bus (`messaging/` topology,
`workers/` consumers) with idempotency and retry/fallback semantics. User
content is always wrapped in `UNTRUSTED` blocks by `security/prompt_guard`.

Full details: [AI Service](ai-service.md), [LLM Configuration](llm-config.md),
and the [Agents Data Flow Report](reference/agents-dataflow-report.md).

## Cross-cutting concerns

- **Authentication** — JWT; access + refresh tokens in httpOnly cookies;
  single-flight refresh on the web side.
- **Observability** — structured JSON logging everywhere (Winston in Node,
  `utils/logger.py` in Python), request/trace IDs propagated end-to-end.
- **Data** — MongoDB (7) with Mongoose for API state; Mongo + FAISS disk
  indices for AI RAG/embeddings; Redis for rate limits/cache.
- **Deployment** — Docker Compose (12 containers: 8 services + redis,
  rabbitmq, ai-service, frontend); the AI repo is also valid as a LiteLLM
  proxy setup.