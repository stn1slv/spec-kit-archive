# Implementation Plan: Scheduled Data Export

**Branch**: `20260814-101500-timestamped-export` | **Date**: 2026-08-14 | **Spec**: specs/20260814-101500-timestamped-export/spec.md

**Input**: Feature specification from `/specs/20260814-101500-timestamped-export/spec.md`

## Summary

Recurring, per-team exports of task data to an S3-compatible bucket. Reuses the existing Celery worker introduced by the notifications feature and adds a scheduler entry plus an export writer. Runs are recorded so administrators can audit outcomes.

## Technical Context

**Language/Version**: Python 3.12
**Primary Dependencies**: FastAPI 0.115, SQLAlchemy 2.0, Celery 5.4, boto3 1.35
**Storage**: PostgreSQL 16, S3-compatible object storage
**Testing**: pytest
**Target Platform**: Linux server
**Project Type**: web-service with background worker
**Performance Goals**: a 50k-task export completes within 10 minutes
**Constraints**: S3-compatible destinations only
**Scale/Scope**: up to 200 scheduled exports per day

## Constitution Check

Export files carry task data, so the retention rule is the destination bucket's lifecycle policy rather than this system's; this is stated in the spec's Assumptions.

## Project Structure

### Source Code (repository root)

```text
src/
├── workers/
│   └── exports.py          # scheduled export job
└── services/
    └── export_service.py   # schedule CRUD and run recording
```

**Structure Decision**: The export job joins the existing `workers/` package rather than adding a second worker deployment.

## Routing

- `GET /export-schedules` — a team's export schedules
- `POST /export-schedules` — create a schedule
- `GET /export-schedules/{id}/runs` — run history for a schedule

## Configuration

- `EXPORT_BUCKET_URL` — default destination bucket
- `EXPORT_MAX_CONCURRENCY` — how many export jobs may run at once

## Testing Strategy

- Unit tests for cadence calculation and skip-on-overlap behavior
- Integration test writing to a local S3-compatible server
