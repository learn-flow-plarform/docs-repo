# Backend API — study-partner-api

Express.js microservices backend. One Node process per concern behind an API
Gateway, shared code in the `@study-partner/shared` workspace package.

**Repo:** `study-partner-api/` (Node.js 20+, Express, MongoDB 7 / Mongoose,
JWT, Winston)

## Services & ports

| Service | Port | Role |
|---------|------|------|
| API Gateway | 3000 | Request routing, rate limiting, monitoring, healthcheck |
| Auth Service | 3001 | JWT auth & RBAC (register, login, refresh, OTP, email verify) |
| User Profile Service | 3002 | Profiles, availability, gamification, goals |
| Study Service | 3003 | Tasks, topics, sessions, courses, plans |
| AI Orchestrator | 3004 | Proxies to Python AI (ingest, plan, coach, signals) |
| Signal Processing | 3005 | Focus session tracking |
| Analytics Service | 3006 | Event tracking & insights |
| Notification Service | 3007 | In-app notifications |
| (Python AI) | 8000 | External — `study-partner-ai` FastAPI service |

## Shared package (`shared/` → `@study-partner/shared`)

Shared utilities reused by every service:

- `auth.js` / `middleware.js` — JWT verification + RBAC middleware
- `database.js` — MongoDB/Mongoose connection helper
- `logger.js` — Winston structured JSON (request IDs via UUID)
- `cache.js` — Redis-backed caching
- `tierGate.js` — plan-tier feature gates
- `rate limit / CORS` — express-rate-limit + cors wiring
- `ai-messaging/` — envelope helpers for AI-bound messages
- `processHandlers.js` — graceful shutdown

## Docker Compose (`docker-compose.yml`)

12 containers on `study-partner-network`: `api-gateway`, `auth-service`,
`user-profile-service`, `study-service`, `ai-orchestrator-service`,
`signal-processing-service`, `analytics-service`, `notification-service`,
`redis`, `rabbitmq`, `ai-service`, `frontend`. The gateway healthcheck hits
`/api/v1/health`.

## Key endpoints

- **Auth** — `POST /api/v1/auth/register|login`, `GET /api/v1/auth/me`,
  `POST /api/v1/auth/refresh|verify-otp|forgot-password|reset-password`
- **User** — `GET/PUT /api/v1/users/profile`, `GET/POST
  /api/v1/users/profile/goals`
- **Study** — `CRUD /api/v1/study/tasks|topics`, `POST/GET
  /api/v1/study/sessions`
- **AI Orchestrator** — `POST /api/v1/ai/ingest`, `POST /api/v1/ai/plan/create`
  (fetches availability, calls Python AI), `POST /api/v1/ai/schedule`,
  `POST /api/v1/ai/coach`, `GET /api/v1/ai/coach/history/:userId`,
  `POST /api/v1/ai/signals/analyze-frame`, signal snapshot/history endpoints,
  `GET /api/v1/ai/status`
- **Signals** — focus start/data/end + stats (`/api/v1/signals/focus/...`)
- **Analytics** — `POST /api/v1/analytics/track`, timeline/summary/insights

## Setup

### Docker (recommended)

```bash
npm run docker:up          # docker-compose up -d
npm run docker:logs        # follow logs
```

### Local development

```bash
cd shared && npm install && cd ..
# per service, e.g.:
cd services/api-gateway && npm install && npm run dev
# repeat for auth, user-profile, study, ai-orchestrator,
#       signal-processing, analytics, notification
# start MongoDB + Redis: docker-compose up mongo redis
```

Per-service `.env` (common):

```env
PORT=300X
MONGODB_URI=mongodb://admin:admin123@localhost:27017/study_partner
JWT_SECRET=change-in-production
NODE_ENV=development
# ai-orchestrator only:
AI_SERVICE_URL=http://ai-service:8000
```

## Scripts

`npm run dev:*` (per workspace), `npm run start`, `npm test`
(`test:unit` / `test:integration` via Jest, `test:security` via `npm audit`),
`npm run lint`, `npm run format( check)`, `npm run db:migrate(:undo)`,
`npm run db:seed`, `npm run seed:characters`, `npm run seed:rank-badges`.

## AI boundary

The AI Orchestrator (`:3004`) is the only service that talks to
`study-partner-ai`. See [AI Service](ai-service.md) and
[LLM Configuration](llm-config.md) for the Python side (agents, job bus,
per-agent model routing).