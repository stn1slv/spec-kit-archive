# Bug Fix: Portrait images render sideways in previews

- **Slug**: thumbnail-orientation
- **Fixed**: 2026-08-19
- **Assessment**: ./assessment.md
- **Status**: applied

## Summary

The thumbnail pipeline now applies the EXIF orientation tag before resizing.

The system MUST normalize EXIF orientation before generating a thumbnail.

## Changes

| File | Change | Notes |
|------|--------|-------|
| `src/services/attachment_service.py` | modified | orientation applied before resize |

## Tests Added or Updated

- `tests/test_attachments.py::test_portrait_thumbnail` — pins upright output
