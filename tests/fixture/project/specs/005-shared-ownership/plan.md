# Implementation Plan: Shared Task Ownership

**Branch**: `005-shared-ownership` | **Date**: 2026-08-11 | **Spec**: specs/005-shared-ownership/spec.md

**Input**: Feature specification from `/specs/005-shared-ownership/spec.md`

## Summary

Temporary delegation of tasks, plus co-ownership for tasks that genuinely belong to more than one person. Adds a `Delegation` record and a nightly expiry job on the existing worker.

## Technical Context

**Language/Version**: Python 3.12
**Primary Dependencies**: FastAPI 0.115, SQLAlchemy 2.0
**Storage**: PostgreSQL 16
**Testing**: pytest
**Target Platform**: Linux server
**Project Type**: web-service with background worker
**Performance Goals**: delegation created and visible within 2 seconds
**Constraints**: delegation is team-internal only
**Scale/Scope**: up to 500 active delegations per team

## Constitution Check

No violations. Co-ownership extends the ownership model rather than removing it, since every co-owner is still an owner.

## Project Structure

### Source Code (repository root)

```text
src/
└── ownership/
    ├── delegation.py   # create, list and clear delegations
    └── expiry.py       # nightly delegation expiry job
```

**Structure Decision**: Delegation lives in a new `ownership/` package inside the existing deployable; the expiry job runs on the existing worker.

## Routing

- `POST /tasks/{id}/delegate` — delegate a task
- `DELETE /tasks/{id}/delegate` — clear a delegation

## Configuration

- `DELEGATION_MAX_DAYS` — upper bound on a delegation's length

## Testing Strategy

- Unit tests for expiry arithmetic
- API tests for both delegation routes
