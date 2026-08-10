# Implementation Plan: Team Reporting

**Branch**: `003-reporting` | **Date**: 2026-08-03 | **Spec**: specs/003-reporting/spec.md

**Input**: Feature specification from `/specs/003-reporting/spec.md`

## Summary

Weekly per-team reports and per-task overdue reminders, computed from nightly aggregates on a read replica so reporting load never touches the primary database.

## Technical Context

**Language/Version**: Python 3.12
**Primary Dependencies**: FastAPI 0.115, SQLAlchemy 2.0, APScheduler 4.0
**Storage**: PostgreSQL 16 (primary), PostgreSQL read replica (reporting)
**Testing**: pytest
**Target Platform**: Linux server
**Project Type**: web-service with background worker
**Performance Goals**: weekly report generated in under 30 seconds per team
**Constraints**: reporting queries run only against a multi-region read replica, never the primary
**Scale/Scope**: up to 200 reports per week

## Constitution Check

No violations. Reports contain only task metadata already visible to team members.

## Project Structure

### Source Code (repository root)

```text
src/
└── reports/
    ├── aggregates.py   # nightly precomputed per-team aggregates
    ├── weekly.py       # weekly report generation
    └── reminders.py    # per-task overdue reminder job
```

**Structure Decision**: Reporting lives in a new `reports/` package inside the existing deployable; jobs run on the existing worker.

## Routing

- `GET /reports/weekly` — latest weekly report for the caller's team

## Configuration

- `REPORTS_REPLICA_URL` — read replica connection string

## Testing Strategy

- Unit tests for aggregate math and report rendering
- Integration test for `GET /reports/weekly` against seeded aggregates
