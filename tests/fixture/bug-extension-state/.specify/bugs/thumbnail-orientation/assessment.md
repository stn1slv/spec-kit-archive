# thumbnail-orientation: Portrait images render sideways in previews

**Type**: Implementation drift
**Severity**: Medium
**Status**: Assessed
**Reported**: 2026-08-18

## Report

Photographs taken in portrait orientation are shown rotated 90 degrees in the attachment preview. The stored original is correct; only the generated thumbnail is wrong.

## Root Cause Analysis

**Issue:** Thumbnails of portrait photographs are displayed sideways.
**Root Cause:** The thumbnail generator reads raw pixel data and ignores the EXIF orientation tag, which the originating cameras rely on rather than physically rotating the image.
**Prevention Rule:** Any image transformation must apply the EXIF orientation tag before resizing, and strip the tag afterwards so downstream viewers do not rotate a second time.
