# Changelog

> **Revision**: 2026-08-10 — Created with first archived feature `001-task-manager` (Task Manager).
> **Revision**: 2026-08-10 — Added entry for archived feature `002-notifications` (Deadline Notifications), retiring FR-004 and SC-003.

## Merged Features Log

### Deadline Notifications — archived 2026-08-10
**Branch:** `002-notifications`
**Spec:** [specs/002-notifications/spec.md](../../specs/002-notifications/spec.md)

**What was added:**
- 24-hour deadline reminder notifications to task owners (User Story 1)
- One daily summary per owner listing all overdue tasks (User Story 2)
- Automatic deletion of completed tasks older than 90 days
- `GET /notifications` endpoint for a user's recent notifications

**New Components:**
- `src/workers/` (reminders.py, summaries.py, retention.py)
- `src/services/notification_service.py`

**Superseded:**
- RETIRED: FR-004 (from specs/001-task-manager/spec.md) → replaced by FR-009. Reason: retention rule changed from keep-forever to automatic deletion of completed tasks after 90 days.
- RETIRED: SC-003 (from specs/001-task-manager/spec.md) → no replacement. Reason: measured the keep-forever retention rule that this feature retires outright.

**Tasks Completed:** 7/8 tasks

### Task Manager — archived 2026-08-10
**Branch:** `001-task-manager`
**Spec:** [specs/001-task-manager/spec.md](../../specs/001-task-manager/spec.md)

**What was added:**
- Create and assign a task with a title, deadline, and exactly one owner (User Story 1)
- Complete a task and move it out of the active list (User Story 2)
- Filter tasks by deadline to see tasks due this week (User Story 3)
- Paginated task list, overdue flagging, and owner-deactivation move to the team backlog

**New Components:**
- `src/api/` (tasks.py, users.py)
- `src/models/` (task.py, user.py)
- `src/services/task_service.py`
- `src/web/templates/`

**Tasks Completed:** 10/10 tasks
