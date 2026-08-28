# Rapid CP API 1.09.05 Release Notes - Coming Soon
**Release Date:** 8/2/2026

_Fixes a date-range query failure on older SQL Server compatibility levels, a blank-transfer-number data integrity bug, a custom-field default being cleared on write, and startup reliability against a slow-to-mount network share._

## Bug Fixes

### `GET /Documents`
Document lookups using a date range in `customFilter` no longer fail on SQL Server instances running at a database compatibility level below 110 (SQL Server 2012); the date comparison is now a typed comparison against `TKT_DT` instead of a raw SQL fragment.

**Supported fields**
- `customFilter` — JSON-encoded filter object, accepted as a query string value.
  - `StartDate` (`DateTimeOffset`, optional) — minimum `TKT_DT`, inclusive.
  - `EndDate` (`DateTimeOffset`, optional) — maximum `TKT_DT`, inclusive.
  - `Keyword` (`string`, optional)
  - `Page` (`int`, optional, default `1`)
  - `PageSize` (`int`, optional, default `50`, capped at the server's configured maximum)

**Example**
```http
GET /Documents?customFilter=%7B%22StartDate%22%3A%222026-08-01T00%3A00%3A00Z%22%2C%22EndDate%22%3A%222026-08-31T23%3A59%3A59Z%22%7D
```
Response:
```json
[
  {
    "DOC_ID": 10234,
    "TKT_NO": "10045821",
    "TKT_DT": "2026-08-15T14:22:00"
  }
]
```
Returns **HTTP 200 OK**.

### `POST /TransferOut`
A transfer submitted with a blank or whitespace `XFER_NO` is now auto-assigned a transfer number, matching the existing behavior for an explicit auto-assign value; previously a blank `XFER_NO` was posted through as-is, producing a transfer with no number.

**Supported fields**
- `XFER_NO` — optional. Blank, whitespace, or omitted now auto-assigns the next transfer number. An explicit value is still used as given.

**Example**
```http
POST /TransferOut
```
```json
{
  "XFER_NO": "",
  "FM_LOC_ID": "01",
  "TO_LOC_ID": "02"
}
```
Response:
```json
{
  "XFER_NO": "000482",
  "FM_LOC_ID": "01",
  "TO_LOC_ID": "02"
}
```
Returns **HTTP 200 OK**.

### `POST /Customers`, `PATCH /Customers/{CustNo}`
Custom fields on a customer record that have a database-configured default value no longer get written as blank when omitted from a create or update; the field now resolves to its configured default. This applies to any custom field with a default value, not just customer-specific ones.

**Supported fields**
- Any `USER_*` custom field with a database default — omitting it from the request body now leaves it at that default instead of clearing it.

**Example**
```http
POST /Customers
```
```json
{
  "NAM": "Jane Doe",
  "FST_NAM": "Jane",
  "LST_NAM": "Doe"
}
```
Response (custom field with a configured default of `"N"`, omitted from the request):
```json
{
  "CustomFields": {
    "USER_MAILCHIMP_INVALID_EMAIL": "N"
  }
}
```
Returns **HTTP 201 Created** (`POST`) or **HTTP 200 OK** (`PATCH`).

### Service startup reliability against a slow-to-mount network share
The API no longer gives up too early when a configured network share isn't immediately reachable at startup — for example, right after the host machine reboots. Retry time against the share was raised from roughly 6 seconds to roughly 95 seconds, and startup failures now report whether the host was unreachable, the share wasn't shared, or the specific subdirectory was missing, instead of one generic message. No request or response contract changed; this affects service availability during startup only.
