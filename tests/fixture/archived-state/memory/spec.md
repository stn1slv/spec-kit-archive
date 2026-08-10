# Main Project Specification

> **Revision**: 2026-08-10 — Archived feature `001-task-manager` (Task Manager); first population of this document.
> **Revision**: 2026-08-10 — Archived feature `002-notifications` (Deadline Notifications); retired FR-004 and SC-003 (superseded retention rule), added notification stories, requirements, and outcomes.

## User Scenarios & Testing

### User Story 1 - Create and assign a task (Priority: P1)

A team member creates a task, gives it a title and a deadline, and assigns it to exactly one owner.

**Why this priority**: Without task creation nothing else in the product has meaning.

**Independent Test**: Can be tested by creating a task through the UI and checking it appears in the owner's list.

**Acceptance Scenarios**:

1. **Given** a logged-in user, **When** they create a task with a title, deadline, and owner, **Then** the task appears in the owner's task list within 2 seconds.
2. **Given** a task creation form, **When** the user submits it without a title, **Then** the form shows a validation error and no task is created.

[Source: specs/001-task-manager/spec.md -> User Story 1]

### User Story 2 - Complete a task (Priority: P2)

An owner marks a task as completed, and the task moves out of the active list.

**Why this priority**: Completion is the core signal of progress, but it depends on creation existing first.

**Independent Test**: Mark an existing task completed and verify it leaves the active list.

**Acceptance Scenarios**:

1. **Given** an active task, **When** its owner marks it completed, **Then** it disappears from the active list and appears in the completed list.

[Source: specs/001-task-manager/spec.md -> User Story 2]

### User Story 3 - Filter tasks by deadline (Priority: P3)

A user filters the task list to see only tasks due this week.

**Why this priority**: Useful for planning, but the product works without it.

**Independent Test**: Apply the deadline filter and verify only matching tasks are shown.

[Source: specs/001-task-manager/spec.md -> User Story 3]

### User Story 4 - Get notified before a deadline (Priority: P1)

A task owner receives a notification 24 hours before a task's deadline.

**Why this priority**: The whole point of the feature; without it nothing else matters.

**Independent Test**: Create a task due in 24 hours and verify the owner receives exactly one notification.

**Acceptance Scenarios**:

1. **Given** a task due in 24 hours, **When** the notification job runs, **Then** the task's owner receives exactly one notification naming the task and its deadline.
2. **Given** a task whose deadline notification was already sent, **When** the job runs again, **Then** no duplicate notification is sent.

[Source: specs/002-notifications/spec.md -> User Story 1]

### User Story 5 - See overdue tasks summarized (Priority: P2)

An owner receives one daily summary of all their overdue tasks instead of one message per task.

**Why this priority**: Prevents notification fatigue, but depends on basic notifications existing.

**Acceptance Scenarios**:

1. **Given** an owner with three overdue tasks, **When** the daily summary job runs, **Then** the owner receives one message listing all three tasks.

[Source: specs/002-notifications/spec.md -> User Story 2]

### Edge Cases

- What happens when a task's owner account is deactivated? The task moves to the team backlog with no owner; a notification pending for that owner is dropped, not sent to the backlog. [Source: specs/001-task-manager/spec.md -> "What happens when a task's owner account is deactivated?"] [Source: specs/002-notifications/spec.md -> "Owner is deactivated between scheduling and sending"]
- Deadline set in the past: the task is created but immediately flagged as overdue. [Source: specs/001-task-manager/spec.md -> "Deadline set in the past"]
- Two users edit the same task at the same time: last write wins, and the earlier editor sees a notice that the task changed. [Source: specs/001-task-manager/spec.md -> "Two users edit the same task at the same time"]
- A task's deadline is changed after its reminder was sent: a new reminder is scheduled for the new deadline. [Source: specs/002-notifications/spec.md -> "A task's deadline is changed after its reminder was sent"]

## Requirements

### Functional Requirements

- **FR-001**: System MUST allow users to create tasks with a title, an optional description, and a deadline. [Source: specs/001-task-manager/spec.md -> FR-001]
- **FR-002**: Users MUST be able to assign a task to exactly one owner; a task always has a single owner, and that owner is the only recipient of its deadline notifications. [Source: specs/001-task-manager/spec.md -> FR-002] [Source: specs/002-notifications/spec.md -> FR-003]
- **FR-003**: System MUST show a paginated task list of 25 tasks per page, ordered by deadline. [Source: specs/001-task-manager/spec.md -> FR-003]
- **FR-005**: System MUST flag a task as overdue when its deadline has passed and it is not completed. [Source: specs/001-task-manager/spec.md -> FR-005]
- **FR-006**: When an owner account is deactivated, the system MUST move that owner's tasks to the team backlog. [Source: specs/001-task-manager/spec.md -> FR-006]
- **FR-007**: System MUST send the task owner a notification 24 hours before the task's deadline. [Source: specs/002-notifications/spec.md -> FR-001]
- **FR-008**: System MUST send each owner at most one daily summary listing all their overdue tasks. [Source: specs/002-notifications/spec.md -> FR-002]
- **FR-009**: Completed tasks older than 90 days MUST be automatically deleted from the system. [Source: specs/002-notifications/spec.md -> FR-004]

### Key Entities

- **Task**: A unit of work with title, description, deadline, status (active/completed), exactly one owner, and a `reminder_sent_at` timestamp so reminders are never duplicated. [Source: specs/001-task-manager/spec.md -> "Task"] [Source: specs/002-notifications/spec.md -> "Task"]
- **User**: A team member with a name, an email address, and an active/deactivated state. [Source: specs/001-task-manager/spec.md -> "User"]
- **Notification**: A message to a user about a task, with a type (reminder/summary), a send time, and a delivered flag. [Source: specs/002-notifications/spec.md -> "Notification"]

## Success Criteria

### Measurable Outcomes

- **SC-001**: Users can create and assign a task in under 30 seconds. [Source: specs/001-task-manager/spec.md -> SC-001]
- **SC-002**: 95% of task list page loads complete in under 1 second. [Source: specs/001-task-manager/spec.md -> SC-002]
- **SC-004**: 99% of deadline reminders are delivered within 5 minutes of their scheduled time. [Source: specs/002-notifications/spec.md -> SC-001]
- **SC-005**: Owners with overdue tasks receive at most one summary message per day. [Source: specs/002-notifications/spec.md -> SC-002]

## Assumptions

- **AS-001**: Teams are small: no team has more than 50 members. [Source: specs/001-task-manager/spec.md -> "Teams are small"]
- **AS-002**: All users authenticate through the existing company single sign-on. [Source: specs/001-task-manager/spec.md -> "All users authenticate through the existing company single sign-on"]
- **AS-003**: Tasks do not need offline support in this version. [Source: specs/001-task-manager/spec.md -> "Tasks do not need offline support in this version"]
- **AS-004**: Email is the only notification channel in this version. [Source: specs/002-notifications/spec.md -> "Email is the only notification channel in this version"]
