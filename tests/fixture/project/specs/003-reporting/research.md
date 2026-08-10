# Research: Team Reporting

## Key Decisions

- **Nightly precomputed aggregates** instead of on-demand queries: weekly report generation reads one aggregates row per team-day, so report time is independent of task count.
- Reporting reads go to a **read replica** so a heavy report can never slow the primary.

## Known Issues & Gotchas

### Aggregation query timeout above 10k tasks

**Issue:** The per-team aggregation query timed out on teams with more than about 10k historical tasks during development.
**Root Cause:** The query scanned all of a team's tasks instead of only the reported period; there was no index on `(team_id, completed_at)`.
**Prevention Rule:** Every aggregate query must filter by the period index `(team_id, completed_at)`; never aggregate over a team's full task history.
