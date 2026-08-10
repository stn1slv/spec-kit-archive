# Fixture Project — Agent Instructions

## Active Technologies

- Python 3.12 + FastAPI 0.115, SQLAlchemy 2.0, Jinja2 3.1 (001-task-manager)
- PostgreSQL 16 (001-task-manager)
- Celery 5.4, Redis 7.4 (broker only) (002-notifications)

## Project Structure

```text
src/
├── api/
├── models/
├── services/
├── workers/
└── web/
```

## Commands

- `make test` — run pytest

## Recent Changes

- 002-notifications: 24h deadline reminders, daily overdue summaries, 90-day completed-task retention
- 001-task-manager: task CRUD, single-owner assignment, deadlines

<!-- Revision: 2026-08-10 — updated Active Technologies while archiving 001-task-manager -->
<!-- Revision: 2026-08-10 — updated Active Technologies, Project Structure, Recent Changes while archiving 002-notifications -->
