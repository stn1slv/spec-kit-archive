# Bug Assessment: Per-team attachment quota is not enforced on replace

- **Slug**: attachment-quota-drift
- **Created**: 2026-08-18
- **Source**: https://example.invalid/issues/412
- **Verdict**: valid
- **Severity**: high

## Report (verbatim or summarized)

Replacing an existing attachment with a larger file bypasses the per-team storage quota: the check runs on create but not on replace, so a team can exceed its quota by repeatedly replacing files.

## Symptom

A team can exceed its storage quota without limit through repeated replacement.

## Suspected Code Paths

- `src/services/attachment_service.py:88` — quota check attached to the upload handler

## Root Cause Hypothesis

The quota check sits on the upload handler rather than the storage-commit step, and the replace path commits storage without going through that handler. Confidence: high.

## Proposed Remediation

**Preferred**: the storage-commit step MUST become the sole gatekeeper for every per-team byte budget.
