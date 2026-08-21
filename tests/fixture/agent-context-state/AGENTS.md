# Fixture Project — Agent Instructions

This preamble sits **above** the managed block and is freely writable.

## Active Technologies

- Python 3.12 + FastAPI 0.115, SQLAlchemy 2.0 (001-task-manager)

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

- specs/001-task-manager: task CRUD, single-owner assignment, deadlines
