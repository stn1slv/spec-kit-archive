# Main Plan

> **Revision note (2026-08-09)**: Created from `specs/001-task-manager` after merge.

## Task Manager

### 001 Dependencies

- Python 3.12, FastAPI 0.115, SQLAlchemy 2.0, Jinja2 3.1
- PostgreSQL 16

### 001 Structure

```text
src/
├── api/
├── models/
├── services/
└── web/
```

### 001 Routing

- `GET /tasks`, `POST /tasks`, `POST /tasks/{id}/complete`, `GET /users/{id}/tasks`

### 001 Configuration

- `DATABASE_URL`, `SSO_ISSUER_URL`

### 001 Testing

- Unit tests for task_service; API tests for all four routes
