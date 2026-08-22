# Feature Specification: Invoice Settings

**Feature Branch**: `006-invoice-settings`
**Created**: 2026-08-16
**Status**: Draft
**Input**: User description: "Let a team configure how its invoices are numbered and addressed"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Configure invoice numbering (Priority: P1)

A billing administrator sets the prefix and starting number used for the team's invoices.

**Why this priority**: Invoices cannot be issued at all until their numbering is defined.

**Independent Test**: Set a prefix and starting number, issue an invoice, and verify the resulting invoice number.

**Acceptance Scenarios**:

1. **Given** a team with prefix `ACME-` and starting number 100, **When** the first invoice is issued, **Then** its number is `ACME-100`.
2. **Given** a team that has already issued invoices, **When** the administrator lowers the starting number, **Then** the change is rejected and the existing sequence is preserved.

### User Story 2 - Set the billing address shown on invoices (Priority: P2)

An administrator sets the billing address rendered on every invoice.

**Why this priority**: Required for real invoices, but numbering must exist first.

**Acceptance Scenarios**:

1. **Given** a team with a configured billing address, **When** an invoice is rendered, **Then** the address appears in the invoice header.

### Edge Cases

- Two invoices are issued at the same instant: numbers are allocated sequentially with no gap and no duplicate.
- The billing address is cleared after invoices exist: previously issued invoices keep the address they were rendered with.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST let a billing administrator configure an invoice number prefix and starting number.
- **FR-002**: The system MUST allocate invoice numbers sequentially with no duplicates.
- **FR-003**: The system MUST reject a change to the starting number once invoices have been issued.
- **FR-004**: The system MUST render the configured billing address on every issued invoice.
- **FR-005**: The system MUST preserve the address and number an invoice was issued with, regardless of later configuration changes.

### Key Entities

- **InvoiceSettings**: A team's invoice configuration — number prefix, starting number, and billing address.
- **Invoice**: An issued invoice — its allocated number, rendered address, and issue date.

## Success Criteria *(mandatory)*

- **SC-001**: Invoice numbers are unique within a team across 100,000 concurrent issuances.
- **SC-002**: An administrator can configure numbering and address in under two minutes.

## Assumptions

- One invoice configuration per team; per-project invoice settings are out of scope.
- Invoice numbering is per team, not global across teams.
- The billing address is retained for as long as the team record it belongs to, and is deleted with it; issued invoices keep the address they were rendered with for the statutory seven years.
