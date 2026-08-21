# attachment-quota-drift: Per-team attachment quota is not enforced on replace

**Type**: Implementation drift
**Severity**: High
**Status**: Fixed
**Reported**: 2026-08-18
**Feature**: specs/004-attachments

## Report

Replacing an existing attachment with a larger file bypasses the per-team storage quota: the check runs on create but not on replace, so a team can exceed its quota by repeatedly replacing files.

## Root Cause Analysis

**Issue:** The per-team storage quota can be exceeded through repeated replacement.
**Root Cause:** The quota check was attached to the upload handler rather than to the storage-commit step, and the replace path commits storage without going through the upload handler.
**Prevention Rule:** Enforce quota at the single point where bytes are committed to storage, never on an individual entry path.
