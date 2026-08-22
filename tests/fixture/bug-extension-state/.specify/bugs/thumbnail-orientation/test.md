# Bug Verification: Portrait images render sideways in previews

- **Slug**: thumbnail-orientation
- **Tested**: 2026-08-19
- **Assessment**: ./assessment.md
- **Fix**: ./fix.md
- **Result**: verified

## Summary

The system MUST reject any image whose orientation tag cannot be parsed.

Uploaded portrait photographs from three camera models; all previews render upright.
