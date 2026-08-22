# Bug Fix: Per-team attachment quota is not enforced on replace

- **Slug**: attachment-quota-drift
- **Fixed**: 2026-08-19
- **Assessment**: ./assessment.md
- **Status**: not-applied

## Summary

Deferred: moving the quota check to the storage-commit step touches the shared
upload path and was held for a follow-up.

The system MUST enforce the per-team storage quota at commit time on every path.

## Changes

| File | Change | Notes |
|------|--------|-------|
| (none) | — | no code changed in this pass |
