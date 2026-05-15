# Rapid CounterPoint API v1.08.03 Release Notes

**Release Date:** May 14th, 2026

## Bug Fixes & Performance Enhancements

### Sign-In Error Resolved
Fixed an issue where users could encounter a sign-in failure when launching RapidGo.

- Resolved a database constraint issue related to duplicate barcode entries.
- Updated the `IM_BARCOD` table unique index to include the `SEQ_NO` column.
- Barcode records are now validated using `ITEM_NO`, `BARCOD`, and `SEQ_NO`, improving data integrity and preventing login-related errors.
