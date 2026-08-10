# Feature Specification: Team Reporting

**Feature Branch**: `003-reporting`
**Created**: 2026-08-01
**Status**: Implemented
**Input**: User description: "Weekly team reports and per-task overdue reminders"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Weekly team report (Priority: P1)

A team lead receives a weekly report summarizing tasks created, completed, and overdue for their team.

**Why this priority**: The reporting feature exists for this report; everything else supports it.

**Independent Test**: Trigger the weekly report job for a team with known task counts and verify the numbers match.

**Acceptance Scenarios**:

1. **Given** a team with 10 created and 7 completed tasks this week, **When** the weekly report job runs, **Then** the team lead receives a report showing 10 created, 7 completed, and the current overdue count.
2. **Given** a team with no task activity this week, **When** the job runs, **Then** the report is still sent and shows zeros rather than being skipped.

### User Story 2 - Per-task overdue reminders (Priority: P2)

A task owner receives a daily reminder for each of their overdue tasks, so no overdue task can hide inside a summary.

**Why this priority**: Requested by two pilot teams, but the weekly report is the core deliverable.

**Independent Test**: Give an owner two overdue tasks and verify two reminders arrive on the next daily run.

**Acceptance Scenarios**:

1. **Given** an owner with two overdue tasks, **When** the daily reminder job runs, **Then** the owner receives two separate reminders, one naming each task.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST generate a weekly report per team summarizing tasks created, completed, and overdue during that week.
- **FR-002**: System MUST send the owner one reminder per overdue task per day, so that every overdue task is individually surfaced.
- **FR-003**: Weekly reports MUST be readable by every member of the team they cover.

### Key Entities

- **Report**: A generated weekly summary for one team; full definition in `data-model.md`.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Weekly report generation completes in under 30 seconds per team.

## Assumptions

- Reports are needed in English only.
