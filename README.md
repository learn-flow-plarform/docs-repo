# MindFlow — Documentation Repository

Centralized documentation for the **MindFlow (Study Partner)** program. All
project documentation lives here, kept up to date in the same change as the
code it describes — this repository **is** the source of truth for docs.

Content lives under [`docs/`](docs/index.md).

## Quick navigation

| I need... | Go to |
|-----------|-------|
| Where to start, what this is | [Docs index](docs/index.md) |
| Setup, environment, running, testing | [Getting Started](docs/getting-started.md) |
| How the system is built (agents, services, bus) | [Architecture](docs/architecture.md) |
| How models are routed per agent | [LLM Configuration](docs/llm-config.md) |
| What we plan to build | [Sprint Backlog](docs/sprint-backlog.md) |
| Current progress story-by-story | [Implementation Tracker](docs/implementation-tracker.md) |
| Detailed runtime data flows | [Agents Data Flow Report](docs/reference/agents-dataflow-report.md) |

## Maintenance

- Update the docs in the same change/commit as the code they document.
- Keep [`docs/index.md`](docs/index.md) in sync when adding documents.
- The code repository is intentionally lightweight: only `README.md` + code —
  detailed docs stay here.