# Data Model: Team Reporting

## Entities

### Report

A generated weekly summary for one team.

| Field | Type | Notes |
|---|---|---|
| id | UUID | primary key |
| team_id | UUID | the team the report covers |
| period_start | date | Monday of the reported week |
| created_count | int | tasks created in the period |
| completed_count | int | tasks completed in the period |
| overdue_count | int | overdue tasks at generation time |
| owner_breakdown | jsonb | per-owner completed counts, keyed by user id, with the owner's display name |
| generated_at | timestamp | when the report job ran |

### Task (extension)

- `completed_at` (timestamp, nullable) — set when the task is marked completed; the weekly aggregates group by this field.

## Relationships

- A `Report` summarizes many `Task` rows through the team relationship; tasks are not linked to reports individually.

## Validation Rules

- `period_start` must be a Monday; one report per team per period (unique on `team_id, period_start`).
