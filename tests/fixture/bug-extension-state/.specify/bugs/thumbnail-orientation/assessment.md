# Bug Assessment: Portrait images render sideways in previews

- **Slug**: thumbnail-orientation
- **Created**: 2026-08-18
- **Source**: pasted text
- **Verdict**: valid
- **Severity**: medium

## Report (verbatim or summarized)

Photographs taken in portrait orientation are shown rotated 90 degrees in the attachment preview. The stored original is correct; only the generated thumbnail is wrong.

## Symptom

Portrait photographs preview sideways while the downloaded original is upright.

## Suspected Code Paths

- `src/services/attachment_service.py:210` — thumbnail generation reads raw pixel data

## Root Cause Hypothesis

The thumbnail generator ignores the EXIF orientation tag, which the originating cameras rely on rather than physically rotating the image. Confidence: high.

## Proposed Remediation

**Preferred**: apply the EXIF orientation tag before resizing, then strip the tag so downstream viewers do not rotate a second time.
