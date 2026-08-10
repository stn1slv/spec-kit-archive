# Implementation Plan: Task Manager

**Branch**: `001-task-manager` | **Date**: 2026-07-03 | **Spec**: specs/001-task-manager/spec.md

**Input**: Feature specification from `/specs/001-task-manager/spec.md`

## Summary

A small-team task manager: create tasks, assign a single owner, track deadlines and completion. Backend REST API with a server-rendered frontend.

## Technical Context

**Language/Version**: Python 3.12
**Primary Dependencies**: FastAPI 0.115, SQLAlchemy 2.0, Jinja2 3.1
**Storage**: PostgreSQL 16
**Testing**: pytest
**Target Platform**: Linux server
**Project Type**: web-service
**Performance Goals**: task list page under 1 second at p95
**Constraints**: single region, no offline mode
**Scale/Scope**: up to 50 users per team, 10k tasks per team

## Constitution Check

No violations. Data retention rule (completed tasks kept forever) matches Principle II.

## Project Structure

### Source Code (repository root)

```text
src/
├── api/
│   ├── tasks.py        # task CRUD endpoints
│   └── users.py        # user lookup endpoints
├── models/
│   ├── task.py
│   └── user.py
├── services/
│   └── task_service.py # assignment, completion, overdue flagging
└── web/
    └── templates/      # Jinja2 pages
```

**Structure Decision**: Single web-service project. API and server-rendered pages live in one deployable unit.

## Routing

- `GET /tasks` — paginated task list (25 per page)
- `POST /tasks` — create task
- `POST /tasks/{id}/complete` — mark completed
- `GET /users/{id}/tasks` — owner's task list

## Configuration

- `DATABASE_URL` — PostgreSQL connection string
- `SSO_ISSUER_URL` — company single sign-on issuer

## Testing Strategy

- Unit tests for `task_service` (assignment, completion, overdue rules)
- API tests for all four routes with an ephemeral PostgreSQL container
