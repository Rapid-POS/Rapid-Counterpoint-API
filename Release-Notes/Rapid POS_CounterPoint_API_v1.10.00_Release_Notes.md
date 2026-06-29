# Rapid CP API v1.10.00 Release Notes

**Release Date:** July 1, 2026

_Adds a new item-field patch endpoint for label code updates from RapidGO, and fixes two receiving bugs: incorrect stocking-unit conversion and an ambiguous-column error on paginated receiver queries._

---

## New Endpoints

### `PATCH /Items/{itemNo}/Fields`

Updates selected fields on an `IM_ITEM` record by item number. Initial supported field: `IM_LBL_COD` (Label Code). Triggers a sync after a successful update.

**Highlights**

- Accepts a `ItemFieldPatchRequest` body specifying the field(s) to update.
- Field-level validation is applied before the write; invalid values are rejected before any change is persisted.
- Label Code values must already exist in CounterPoint — this endpoint selects from existing codes, it does not create new ones.
- Returns **HTTP 200 OK** on success.

---

## Bug Fixes

### `POST /Receivings` — Incorrect stocking-unit conversion when received unit differs from PO unit

Receiving a quantity in an alternate unit was computing the stocking quantity against the PO's purchasing unit and its stored conversion factor instead of the unit actually received. For example, receiving 5 of a unit defined as `1 = 3 EACH` was stocking an incorrect quantity instead of the expected 15 EACH. The quantity unit is now refreshed from the received unit before the stocking calculation runs, so the conversion is always driven by what was physically received. This regression was introduced in the 6/19 release.

### `POST /Receivings` — Ambiguous column error on paginated receiver queries

Paginated receiver lookups that combined a `JOIN` with a `GROUP BY` failed with an ambiguous-column error on `RECVR_NO` / `VEND_NO`. The receiver query now uses a `.Contains()` filter, which resolves the column ambiguity and also allows multiple receiver numbers to be queried in a single request.

### Service startup — `DirectoryNotFoundException` causing Windows Service crash

On startup, the service threw `System.IO.DirectoryNotFoundException` when the configured network share (e.g. `\\SERVER\CPSQL.1\Toplevel`) was temporarily unavailable, which caused the Windows Service to stop without restarting. Retry logic with logging and delays has been added to the directory-existence check, and the service now exits with a non-zero code on unrecoverable failure so the Windows Service Manager triggers an automatic restart.

