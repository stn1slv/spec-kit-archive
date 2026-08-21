# Bug Assessment: Weekly report boundaries use server time

- **Slug**: report-timezone
- **Created**: 2026-08-20
- **Source**: pasted text
- **Verdict**: likely valid, needs reproduction
- **Severity**: low

## Report (verbatim or summarized)

Weekly completion reports are cut at server midnight rather than the team's own timezone, so tasks completed late in the evening land in the wrong week for teams west of the server.

## Symptom

Weekly report buckets are offset for teams not in the server's timezone.

## Root Cause Hypothesis

The aggregation query truncates timestamps with the database session timezone instead of the team's configured zone. Confidence: medium.
