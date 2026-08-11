# Main Specification

> **Revision note (2026-08-09)**: Bootstrapped from `specs/001-task-manager` after merge.

## Feature 001: Task Manager

**Feature Branch**: `001-task-manager`
**Created**: 2026-07-01
**Status**: Implemented
**Input**: User description: "Small team task manager with owners and deadlines"

## User Scenarios & Testing

### User Story 1 - Create and assign a task (Priority: P1)

A team member creates a task, gives it a title and a deadline, and assigns it to exactly one owner. [Source: specs/001-task-manager/spec.md -> User Story 1]

**Why this priority**: Without task creation nothing else in the product has meaning.

**Independent Test**: Can be tested by creating a task through the UI and checking it appears in the owner's list.

**Acceptance Scenarios**:

1. **Given** a logged-in user, **When** they create a task with a title, deadline, and owner, **Then** the task appears in the owner's task list within 2 seconds.
2. **Given** a task creation form, **When** the user submits it without a title, **Then** the form shows a validation error and no task is created.

### User Story 2 - Complete a task (Priority: P2)

An owner marks a task as completed, and the task moves out of the active list. [Source: specs/001-task-manager/spec.md -> User Story 2]

**Why this priority**: Completion is the core signal of progress, but it depends on creation existing first.

**Independent Test**: Mark an existing task completed and verify it leaves the active list.

**Acceptance Scenarios**:

1. **Given** an active task, **When** its owner marks it completed, **Then** it disappears from the active list and appears in the completed list.

### User Story 3 - Filter tasks by deadline (Priority: P3)

A user filters the task list to see only tasks due this week. [Source: specs/001-task-manager/spec.md -> User Story 3]

**Why this priority**: Useful for planning, but the product works without it.

**Independent Test**: Apply the deadline filter and verify only matching tasks are shown.

### Edge Cases

- What happens when a task's owner account is deactivated? The task is reassigned to the team lead, who becomes its owner. [Source: specs/001-task-manager]
- Deadline set in the past: the task is created but immediately flagged as overdue. [Source: specs/001-task-manager]
- Two users edit the same task at the same time: last write wins, and the earlier editor sees a notice that the task changed. [Source: specs/001-task-manager]

## Requirements

### Functional Requirements

- **FR-001**: System MUST allow users to create tasks with a title, an optional description, and a deadline. [Source: specs/001-task-manager/spec.md -> FR-001]
- **FR-002**: Users MUST be able to assign a task to exactly one owner. [Source: specs/001-task-manager]
- **FR-003**: System MUST show a paginated task list of 25 tasks per page, ordered by deadline. [Source: specs/001-task-manager/spec.md -> FR-003]
- **FR-004**: Completed tasks MUST be kept forever and MUST never be deleted from the system. [Source: specs/001-task-manager/spec.md -> FR-004]
- **FR-005**: System MUST flag a task as overdue when its deadline has passed and it is not completed. [Source: specs/001-task-manager/spec.md -> FR-005]
- **FR-006**: When an owner account is deactivated, the system MUST reassign that owner's tasks to the team lead, who becomes their owner. [Source: specs/001-task-manager/spec.md -> FR-006]

### Key Entities

- **Task**: A unit of work with title, description, deadline, status (active/completed), and exactly one owner. [Source: specs/001-task-manager/spec.md -> "Task"]
- **User**: A team member with a name, an email address, and an active/deactivated state. [Source: specs/001-task-manager/spec.md -> "User"]

## Success Criteria

### Measurable Outcomes

- **SC-001**: Users can create and assign a task in under 30 seconds. [Source: specs/001-task-manager/spec.md -> SC-001]
- **SC-002**: 95% of task list page loads complete in under 1 second. [Source: specs/001-task-manager/spec.md -> SC-002]
- **SC-003**: Zero completed tasks are lost or deleted over any 12-month period. [Source: specs/001-task-manager/spec.md -> SC-003]

## Assumptions

- Teams are small: no team has more than 50 members. [Source: specs/001-task-manager]
- All users authenticate through the existing company single sign-on. [Source: specs/001-task-manager]
- Tasks do not need offline support in this version. [Source: specs/001-task-manager]
