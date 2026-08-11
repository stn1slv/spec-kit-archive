# Data Model: Shared Task Ownership

## Entities

### Delegation

A temporary transfer of a task from its owner to another user.

| Field | Type | Notes |
|---|---|---|
| id | UUID | primary key |
| task_id | UUID | the delegated task |
| from_user_id | UUID | the owner who delegated |
| to_user_id | UUID | the delegate |
| starts_at | timestamp | when the delegation takes effect |
| expires_at | timestamp | never more than 14 days after `starts_at` |

### Task (extension)

- `co_owner_ids` (UUID array, nullable) — additional owners sharing the task equally; empty for a singly-owned task.

## Relationships

- A `Task` has at most one active `Delegation` at a time.

## Validation Rules

- `expires_at` must be after `starts_at` and at most 14 days later.
