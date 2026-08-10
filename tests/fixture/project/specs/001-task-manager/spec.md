# Feature Specification: Task Manager

**Feature Branch**: `001-task-manager`
**Created**: 2026-07-01
**Status**: Implemented
**Input**: User description: "Small team task manager with owners and deadlines"

## Clarifications

### Session 2026-07-02

- Q: Can a task have more than one owner? → A: No, exactly one owner per task.
- Q: What happens to tasks when their owner is deactivated? → A: They move to the team backlog with no owner.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Create and assign a task (Priority: P1)

A team member creates a task, gives it a title and a deadline, and assigns it to exactly one owner.

**Why this priority**: Without task creation nothing else in the product has meaning.

**Independent Test**: Can be tested by creating a task through the UI and checking it appears in the owner's list.

**Acceptance Scenarios**:

1. **Given** a logged-in user, **When** they create a task with a title, deadline, and owner, **Then** the task appears in the owner's task list within 2 seconds.
2. **Given** a task creation form, **When** the user submits it without a title, **Then** the form shows a validation error and no task is created.

### User Story 2 - Complete a task (Priority: P2)

An owner marks a task as completed, and the task moves out of the active list.

**Why this priority**: Completion is the core signal of progress, but it depends on creation existing first.

**Independent Test**: Mark an existing task completed and verify it leaves the active list.

**Acceptance Scenarios**:

1. **Given** an active task, **When** its owner marks it completed, **Then** it disappears from the active list and appears in the completed list.

### User Story 3 - Filter tasks by deadline (Priority: P3)

A user filters the task list to see only tasks due this week.

**Why this priority**: Useful for planning, but the product works without it.

**Independent Test**: Apply the deadline filter and verify only matching tasks are shown.

### Edge Cases

- What happens when a task's owner account is deactivated? The task moves to the team backlog with no owner.
- Deadline set in the past: the task is created but immediately flagged as overdue.
- Two users edit the same task at the same time: last write wins, and the earlier editor sees a notice that the task changed.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST allow users to create tasks with a title, an optional description, and a deadline.
- **FR-002**: Users MUST be able to assign a task to exactly one owner.
- **FR-003**: System MUST show a paginated task list of 25 tasks per page, ordered by deadline.
- **FR-004**: Completed tasks MUST be kept forever and MUST never be deleted from the system.
- **FR-005**: System MUST flag a task as overdue when its deadline has passed and it is not completed.
- **FR-006**: When an owner account is deactivated, the system MUST move that owner's tasks to the team backlog.

### Key Entities

- **Task**: A unit of work with title, description, deadline, status (active/completed), and exactly one owner.
- **User**: A team member with a name, an email address, and an active/deactivated state.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Users can create and assign a task in under 30 seconds.
- **SC-002**: 95% of task list page loads complete in under 1 second.
- **SC-003**: Zero completed tasks are lost or deleted over any 12-month period.

## Assumptions

- Teams are small: no team has more than 50 members.
- All users authenticate through the existing company single sign-on.
- Tasks do not need offline support in this version.
