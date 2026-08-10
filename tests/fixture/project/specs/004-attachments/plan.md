# Implementation Plan: File Attachments

**Branch**: `004-attachments` | **Date**: 2026-08-06 | **Spec**: specs/004-attachments/spec.md

**Input**: Feature specification from `/specs/004-attachments/spec.md`

## Summary

File attachments on tasks, stored in a shared object store, scanned before download.

## Technical Context

**Language/Version**: Python 3.12
**Primary Dependencies**: FastAPI 0.115, SQLAlchemy 2.0, MinIO client 8.x
**Storage**: PostgreSQL 16, shared object store (MinIO)
**Testing**: pytest
**Target Platform**: Linux server
**Project Type**: web-service

## Constitution Check

No violations. Attachment retention follows the owning task's retention rule.

## Project Structure

### Source Code (repository root)

```text
src/
└── attachments/
    ├── storage.py   # object-store client wrapper
    └── scan.py      # scan-before-download gate
```

**Structure Decision**: Attachments live in a new `attachments/` package inside the existing deployable.

## Routing

- `POST /tasks/{id}/attachments` — upload
- `GET /attachments/{id}` — download (scanned only)

## Configuration

- `OBJECT_STORE_URL` — shared object store endpoint

## Testing Strategy

- Unit tests for the scan gate; API tests for upload and download routes
