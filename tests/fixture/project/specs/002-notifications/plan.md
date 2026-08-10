# Implementation Plan: Deadline Notifications

**Branch**: `002-notifications` | **Date**: 2026-07-22 | **Spec**: specs/002-notifications/spec.md

**Input**: Feature specification from `/specs/002-notifications/spec.md`

## Summary

Scheduled notification jobs on top of the task manager: 24-hour deadline reminders and a daily overdue summary, delivered by email. Adds a background worker and a retention job that deletes completed tasks older than 90 days.

## Technical Context

**Language/Version**: Python 3.12
**Primary Dependencies**: FastAPI 0.115, SQLAlchemy 2.0, Celery 5.4, Redis 7.4
**Storage**: PostgreSQL 16, Redis (broker only)
**Testing**: pytest
**Target Platform**: Linux server
**Project Type**: web-service with background worker
**Performance Goals**: reminders delivered within 5 minutes of schedule
**Constraints**: email only, no push or SMS
**Scale/Scope**: up to 10k notifications per team per month

## Constitution Check

The 90-day retention job conflicts with the original keep-forever retention wording; resolved in the spec by superseding the old rule.

## Project Structure

### Source Code (repository root)

```text
src/
├── workers/
│   ├── reminders.py    # 24h deadline reminder job
│   ├── summaries.py    # daily overdue summary job
│   └── retention.py    # 90-day completed-task cleanup
└── services/
    └── notification_service.py
```

**Structure Decision**: Background jobs live in a new `workers/` package inside the existing single deployable; Celery worker runs as a second process.

## Routing

- `GET /notifications` — a user's recent notifications

## Configuration

- `REDIS_URL` — Celery broker connection string
- `SMTP_URL` — outgoing email server

## Testing Strategy

- Unit tests for reminder scheduling and summary grouping
- Worker integration test with a fake SMTP server
