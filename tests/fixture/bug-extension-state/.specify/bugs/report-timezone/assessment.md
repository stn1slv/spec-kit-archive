# report-timezone: Weekly report boundaries use server time

**Type**: Implementation drift
**Severity**: Low
**Status**: Open
**Reported**: 2026-08-20

## Report

Weekly completion reports are cut at server midnight rather than the team's own timezone, so tasks completed late in the evening land in the wrong week for teams west of the server.

## Root Cause Analysis

**Issue:** Weekly report boundaries ignore the team timezone.
**Root Cause:** The aggregation query truncates timestamps with the database session timezone instead of the team's configured zone.
**Prevention Rule:** Every date-bucketing query must take the timezone as an explicit parameter.
