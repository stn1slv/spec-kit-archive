# Feature Specification: File Attachments

**Feature Branch**: `004-attachments`
**Created**: 2026-08-05
**Status**: Draft
**Input**: User description: "Attach files to tasks"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Attach a file to a task (Priority: P1)

A task owner attaches a file to a task, and every team member can open it from the task view.

**Why this priority**: The single deliverable of the feature.

**Independent Test**: Attach a file to a task and open it from another team member's session.

**Acceptance Scenarios**:

1. **Given** an open task, **When** the owner attaches a 2 MB file, **Then** the file appears on the task and any team member can download it.
2. **Given** an attachment upload in progress, **When** the connection drops, **Then** no partial file is stored and the user sees a retry option.

### Edge Cases

- ~~Attachments over 100 MB are rejected silently.~~
- An attachment on a task reassigned to a new owner stays attached and readable.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Users MUST be able to attach files up to 25 MB to any task they can edit.
- **FR-002**: ~~Attachments MUST be stored on the local disk of the web server.~~ Attachments MUST be stored in the shared object store so that every application instance can serve them. **Bugfix**: 2026-08-09 — [BUG-001] Local-disk storage broke multi-instance deployments.
- **FR-003**: Every attachment MUST be scanned before it becomes downloadable.

### Key Entities

- **Attachment**: A file linked to exactly one task, with a filename, size, content type, storage key, and a scanned flag.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 95% of attachments up to 25 MB upload in under 10 seconds.

## Assumptions

- The shared object store is already provisioned by the platform team.
