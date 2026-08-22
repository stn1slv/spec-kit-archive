# Implementation Plan: Invoice Settings

**Branch**: `006-invoice-settings` | **Date**: 2026-08-16 | **Spec**: specs/billing/006-invoice-settings/spec.md

**Input**: Feature specification from `/specs/billing/006-invoice-settings/spec.md`

## Summary

Per-team invoice configuration: number prefix, starting number, and billing address. Number allocation is serialized in the database so concurrent issuance cannot produce a duplicate or a gap. Issued invoices snapshot the address and number they were rendered with.

## Technical Context

**Language/Version**: Python 3.12
**Primary Dependencies**: FastAPI 0.115, SQLAlchemy 2.0
**Storage**: PostgreSQL 16
**Testing**: pytest
**Target Platform**: Linux server
**Project Type**: web-service with background worker
**Performance Goals**: invoice number allocation under 50ms at the 99th percentile
**Constraints**: one invoice configuration per team
**Scale/Scope**: up to 100k invoices per team per year

## Constitution Check

Invoice settings include a billing address, which is user data. The spec's Assumptions state its retention rule: it lives as long as the team record, while issued invoices keep their rendered address for seven years.

## Project Structure

### Source Code (repository root)

```text
src/
├── services/
│   └── invoice_settings_service.py
└── models/
    └── invoice.py
```

**Structure Decision**: Invoice settings live beside the existing team services; no new deployable.

## Routing

- `GET /teams/{id}/invoice-settings` — read a team's invoice configuration
- `PUT /teams/{id}/invoice-settings` — update it

## Configuration

- `INVOICE_NUMBER_LOCK_TIMEOUT` — how long to wait for the number allocation lock

## Testing Strategy

- Unit tests for prefix formatting and starting-number validation
- Concurrency test issuing invoices in parallel to prove no duplicates
