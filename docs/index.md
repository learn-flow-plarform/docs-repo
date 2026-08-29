# Docs

Welcome. MindFlow is a multi-agent AI study-assistance platform (planner,
coach, evaluator, search, ingestion, reflection) built on Python agents, a
RabbitMQ job bus, and one shared LLM client.

This index is the single entry point for all documentation.

## Contents

1. [Getting Started](getting-started.md) — prerequisites, install, `.env`,
   running, testing.
2. [Architecture](architecture.md) — agents, services, workers, messaging,
   LLM access, and cross-cutting concerns (logging, persistence).
3. [LLM Configuration](llm-config.md) — per-agent model routing via
   `litellm/config.yaml`, keys, mock/fallback behaviour, error handling.
4. [Sprint Backlog](sprint-backlog.md) — feature-based sprint backlog
   (F01–F14): user stories, story format, acceptance criteria.
5. [Implementation Tracker](implementation-tracker.md) — live progress
   (e.g. `F03 AI Coach 8/16`) against the backlog.
6. [Reference](reference/agents-dataflow-report.md) — detailed runtime data
   flows, persistence, observability (historical enhancement report).

## Reading order

- New to the project → **Getting Started**, then **Architecture**.
- Working on AI features → **LLM Configuration** + the tracker.
- Planning/tracking work → **Sprint Backlog** + **Implementation Tracker**.