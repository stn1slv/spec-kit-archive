# Fix report: thumbnail-orientation

**Status**: Fixed
**Applied**: 2026-08-19

## What changed

The thumbnail pipeline now applies the EXIF orientation tag before resizing.

The system MUST normalize EXIF orientation before generating a thumbnail, and
MUST strip the orientation tag from the generated thumbnail.

## Files touched

- `src/services/attachment_service.py`
