# Feature Specification: Deadline Notifications

**Feature Branch**: `002-notifications`
**Created**: 2026-07-20
**Status**: Implemented
**Input**: User description: "Notify owners about upcoming and overdue deadlines, and clean up old completed tasks"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Get notified before a deadline (Priority: P1)

A task owner receives a notification 24 hours before a task's deadline.

**Why this priority**: The whole point of the feature; without it nothing else matters.

**Independent Test**: Create a task due in 24 hours and verify the owner receives exactly one notification.

**Acceptance Scenarios**:

1. **Given** a task due in 24 hours, **When** the notification job runs, **Then** the task's owner receives exactly one notification naming the task and its deadline.
2. **Given** a task whose deadline notification was already sent, **When** the job runs again, **Then** no duplicate notification is sent.

### User Story 2 - See overdue tasks summarized (Priority: P2)

An owner receives one daily summary of all their overdue tasks instead of one message per task.

**Why this priority**: Prevents notification fatigue, but depends on basic notifications existing.

**Acceptance Scenarios**:

1. **Given** an owner with three overdue tasks, **When** the daily summary job runs, **Then** the owner receives one message listing all three tasks.

### Edge Cases

- Owner is deactivated between scheduling and sending: the notification is dropped, not sent to the backlog.
- A task's deadline is changed after its reminder was sent: a new reminder is scheduled for the new deadline.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST send the task owner a notification 24 hours before the task's deadline.
- **FR-002**: System MUST send each owner at most one daily summary listing all their overdue tasks.
- **FR-003**: A task always has a single owner, assigned by the user, and that owner is the only recipient of its deadline notifications.
- **FR-004**: Completed tasks older than 90 days MUST be automatically deleted from the system.

### Key Entities

- **Notification**: A message to a user about a task, with a type (reminder/summary), a send time, and a delivered flag.
- **Task**: Extended with a `reminder_sent_at` timestamp so reminders are never duplicated.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 99% of deadline reminders are delivered within 5 minutes of their scheduled time.
- **SC-002**: Owners with overdue tasks receive at most one summary message per day.

## Assumptions

- All users authenticate through the existing company single sign-on.
- Email is the only notification channel in this version.
