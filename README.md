# MindFlow — Documentation Repository

Centralized documentation for the **MindFlow (Study Partner)** platform: a
multi-agent AI study-assistance system built during the PIDEV – 3rd Year
Engineering Program (Esprit, 2025–2026).

All project documentation lives here in one place. It is the source of truth
for docs across the three code repositories.

## The platform in one line

```
study-partner-web (React) → study-partner-api (Node microservices) → study-partner-ai (Python agents + LiteLLM)
```

## Quick navigation

| I need... | Go to |
|-----------|-------|
| Where to start | [Docs index](docs/index.md) |
| How the whole platform is built | [Architecture](docs/architecture.md) |
| **AI service** (Python agents, RabbitMQ job bus) | [AI Service](docs/ai-service.md) · [LLM Config](docs/llm-config.md) |
| **Backend API** (Node microservices, Gateway :3000) | [Backend API](docs/backend-api.md) |
| **Frontend Web** (React + Vite) | [Frontend Web](docs/frontend-web.md) |
| What we plan to build | [Sprint Backlog](docs/sprint-backlog.md) |
| Current progress story-by-story | [Implementation Tracker](docs/implementation-tracker.md) |
| Detailed runtime data flows | [Agents Data Flow Report](docs/reference/agents-dataflow-report.md) |

## Maintenance

- Update the docs in the same change/commit as the code they document.
- Keep [`docs/index.md`](docs/index.md) in sync whenever a document is added.
- Code repositories stay lightweight (`README.md` + code) — detailed docs live
  here only.