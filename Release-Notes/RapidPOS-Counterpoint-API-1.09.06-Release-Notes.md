# Rapid CP API 1.09.06 Release Notes

**Release Date:** September 16, 2026

_Fixes to the item-changes delta sync endpoint: bounded batch size and DB-anchored sync bookmark._

## Bug Fixes

### `GET /DataFile/Changes/{TrackingTable}/{ChangeDate}`

For `TrackingTable=USER_RAPID_IM_ITEM_CHANGE_TRK`, the item-changes delta sync response is now capped at 1000 changed items per call and the sync bookmark is anchored to SQL Server's own clock instead of the API server's clock.

**Highlights**

- Each call now returns at most 1000 changed items, ordered by change date; a client whose backlog exceeds that count must call again with the updated bookmark to continue.
- The `x-server-time-utc` response header for this table is now taken from SQL Server's `GETUTCDATE()` — the same clock that writes each item's change timestamp — rather than the API server's own clock, and reflects the newest change actually returned in a capped batch rather than "now."
- Closes a clock-skew edge case between the API server and SQL Server that could previously cause the next sync call to miss or re-fetch item changes.
- Returns **HTTP 200 OK** with a zip-compressed JSON payload, unchanged from prior versions.
