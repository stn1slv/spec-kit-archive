# Fixture Project — Agent Instructions

This preamble sits **above** the managed block and is freely writable.

## Active Technologies

- Celery 5.4, Redis 7.4 (broker only) (002-notifications)

<!-- TEAM CONTEXT START -->
## Managed by the agent-context extension

Active plan: specs/002-notifications/plan.md

Anything written inside this block is overwritten by `agent-context` on its next
run, so the archive command must never place its sections here.
<!-- TEAM CONTEXT END -->

## Project Structure

```text
src/
├── api/
├── models/
├── services/
└── web/
```

## Commands

- `make test` — run pytest

## Recent Changes

- 002-notifications: 24h deadline reminders, daily overdue summaries, 90-day completed-task retention
