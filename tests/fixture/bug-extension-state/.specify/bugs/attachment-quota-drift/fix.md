# Fix report: attachment-quota-drift

**Status**: Fixed
**Applied**: 2026-08-19

## What changed

The quota check moved to the storage-commit step.

The system MUST enforce the per-team storage quota at commit time on every path
that writes attachment bytes, including replacement.

## Files touched

- `src/services/attachment_service.py`
