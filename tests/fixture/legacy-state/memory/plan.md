# Main Plan

> **Revision note (2026-08-09)**: Created from `specs/001-task-manager` after merge.

## Task Manager

### 001 Technical Context

**Language/Version**: Python 3.12 [Source: specs/001-task-manager]
**Primary Dependencies**: FastAPI 0.115, SQLAlchemy 2.0, Jinja2 3.1 [Source: specs/001-task-manager]
**Storage**: PostgreSQL 16 [Source: specs/001-task-manager]
**Constraints**: single region, no offline mode [Source: specs/001-task-manager]
**Scale/Scope**: up to 50 users per team, 10k tasks per team [Source: specs/001-task-manager]

### 001 Structure

```text
src/
├── api/
├── models/
├── services/
└── web/
```

[Source: specs/001-task-manager/plan.md -> "Project Structure"]

### 001 Routing

- `GET /tasks`, `POST /tasks`, `POST /tasks/{id}/complete`, `GET /users/{id}/tasks` [Source: specs/001-task-manager]

### 001 Configuration

- `DATABASE_URL`, `SSO_ISSUER_URL` [Source: specs/001-task-manager]

### 001 Testing

- Unit tests for task_service; API tests for all four routes [Source: specs/001-task-manager]
