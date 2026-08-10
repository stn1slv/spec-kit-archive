# Main Implementation Plan

> **Revision**: 2026-08-10 — Archived feature `001-task-manager` (Task Manager); first population of this document.
> **Revision**: 2026-08-10 — Archived feature `002-notifications` (Deadline Notifications); added background worker, notification jobs, retention job, and related dependencies, routing, and configuration.

## Summary

A small-team task manager: create tasks, assign a single owner, track deadlines and completion. Backend REST API with a server-rendered frontend. [Source: specs/001-task-manager/plan.md -> "A small-team task manager"] Scheduled notification jobs run on top of it: 24-hour deadline reminders and a daily overdue summary delivered by email, plus a background worker and a retention job that deletes completed tasks older than 90 days. [Source: specs/002-notifications/plan.md -> "Scheduled notification jobs"]

## Technical Context

**Language/Version**: Python 3.12 [Source: specs/001-task-manager/plan.md -> "Language/Version"]

**Primary Dependencies**: FastAPI 0.115, SQLAlchemy 2.0, Jinja2 3.1, Celery 5.4, Redis 7.4 [Source: specs/001-task-manager/plan.md -> "Primary Dependencies"] [Source: specs/002-notifications/plan.md -> "Primary Dependencies"]

**Storage**: PostgreSQL 16, Redis (broker only) [Source: specs/001-task-manager/plan.md -> "Storage"] [Source: specs/002-notifications/plan.md -> "Storage"]

**Testing**: pytest [Source: specs/001-task-manager/plan.md -> "Testing"]

**Target Platform**: Linux server [Source: specs/001-task-manager/plan.md -> "Target Platform"]

**Project Type**: web-service with background worker [Source: specs/001-task-manager/plan.md -> "Project Type"] [Source: specs/002-notifications/plan.md -> "Project Type"]

**Performance Goals**: task list page under 1 second at p95; reminders delivered within 5 minutes of schedule [Source: specs/001-task-manager/plan.md -> "Performance Goals"] [Source: specs/002-notifications/plan.md -> "Performance Goals"]

**Constraints**: single region, no offline mode; notifications by email only, no push or SMS [Source: specs/001-task-manager/plan.md -> "Constraints"] [Source: specs/002-notifications/plan.md -> "Constraints"]

**Scale/Scope**: up to 50 users per team, 10k tasks per team, up to 10k notifications per team per month [Source: specs/001-task-manager/plan.md -> "Scale/Scope"] [Source: specs/002-notifications/plan.md -> "Scale/Scope"]

## Project Structure

```text
src/
├── api/
│   ├── tasks.py        # task CRUD endpoints
│   └── users.py        # user lookup endpoints
├── models/
│   ├── task.py
│   └── user.py
├── services/
│   ├── task_service.py # assignment, completion, overdue flagging
│   └── notification_service.py
├── workers/
│   ├── reminders.py    # 24h deadline reminder job
│   ├── summaries.py    # daily overdue summary job
│   └── retention.py    # 90-day completed-task cleanup
└── web/
    └── templates/      # Jinja2 pages
```

**Structure Decision**: Single web-service project. API and server-rendered pages live in one deployable unit. [Source: specs/001-task-manager/plan.md -> "Structure Decision"] Background jobs live in a `workers/` package inside the same single deployable; the Celery worker runs as a second process. [Source: specs/002-notifications/plan.md -> "Structure Decision"]

## Routing

- `GET /tasks` — paginated task list (25 per page) [Source: specs/001-task-manager/plan.md -> "GET /tasks"]
- `POST /tasks` — create task [Source: specs/001-task-manager/plan.md -> "POST /tasks"]
- `POST /tasks/{id}/complete` — mark completed [Source: specs/001-task-manager/plan.md -> "POST /tasks/{id}/complete"]
- `GET /users/{id}/tasks` — owner's task list [Source: specs/001-task-manager/plan.md -> "GET /users/{id}/tasks"]
- `GET /notifications` — a user's recent notifications [Source: specs/002-notifications/plan.md -> "GET /notifications"]

## Configuration

- `DATABASE_URL` — PostgreSQL connection string [Source: specs/001-task-manager/plan.md -> "DATABASE_URL"]
- `SSO_ISSUER_URL` — company single sign-on issuer [Source: specs/001-task-manager/plan.md -> "SSO_ISSUER_URL"]
- `REDIS_URL` — Celery broker connection string [Source: specs/002-notifications/plan.md -> "REDIS_URL"]
- `SMTP_URL` — outgoing email server [Source: specs/002-notifications/plan.md -> "SMTP_URL"]

## Testing Strategy

- Unit tests for `task_service` (assignment, completion, overdue rules) [Source: specs/001-task-manager/plan.md -> "Unit tests for task_service"]
- API tests for all four routes with an ephemeral PostgreSQL container [Source: specs/001-task-manager/plan.md -> "API tests for all four routes"]
- Unit tests for reminder scheduling and summary grouping [Source: specs/002-notifications/plan.md -> "Unit tests for reminder scheduling"]
- Worker integration test with a fake SMTP server [Source: specs/002-notifications/plan.md -> "Worker integration test"]
