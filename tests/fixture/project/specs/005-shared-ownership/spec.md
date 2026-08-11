# Feature Specification: Shared Task Ownership

**Status**: Implemented

## User Scenarios

### User Story 1 - Delegate a task while away (Priority: P1)

A task owner going on leave hands a task to a colleague for a fixed period, so the work does not stall while they are away.

**Why this priority**: Tasks stalling during leave was the single most common complaint in the last support review.

**Independent Test**: Delegate a task, confirm the delegate can complete it, confirm the original owner regains it when the delegation expires.

**Acceptance Scenarios**:

1. **Given** a task owned by A, **When** A delegates it to B for 7 days, **Then** B can complete the task and A remains the recorded owner.
2. **Given** a delegation that has expired, **When** the nightly job runs, **Then** the delegation is cleared and only A can act on the task.

### User Story 2 - See who is covering a task (Priority: P2)

A team lead looks at the task list and sees which tasks are currently delegated and to whom.

**Why this priority**: Without it, delegation is invisible and the lead cannot tell why someone else is answering for a task.

**Independent Test**: Delegate a task, open the task list, confirm the delegate is shown alongside the owner.

**Acceptance Scenarios**:

1. **Given** a delegated task, **When** the lead opens the task list, **Then** the task shows both its owner and its current delegate.

## Requirements

### Functional Requirements

- **FR-001**: A task MUST support up to three co-owners who share responsibility for it equally, with no single owner among them.
- **FR-002**: The system MUST record who delegated a task, to whom, and when.
- **FR-003**: A delegation MUST expire automatically after at most 14 days.

### Key Entities

- **Delegation**: A temporary transfer of a task from its owner to another user; full definition in `data-model.md`.

## Edge Cases

- A delegate is deactivated while a delegation is active: the delegation is cleared immediately and the task returns to its owner.

## Success Criteria

- **SC-001**: 95% of delegations are created in under 10 seconds.

## Assumptions

- Delegation targets are always members of the same team.
