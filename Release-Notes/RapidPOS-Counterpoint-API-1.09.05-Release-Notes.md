# Rapid CP API 1.09.05 Release Notes
**Release Date:** September 7, 2026

_Fixes a date-range query failure on older SQL Server compatibility levels, a blank-transfer-number data integrity bug, a custom-field default being cleared on write, a duplicate-receiver bug on retried receiving submissions, and startup reliability against a slow-to-mount network share._

## Endpoint Enhancements

### `POST /Receivings`
You can now pass an idempotency key to safely retry a receiving submission after a network failure without creating a duplicate receiver.

**Supported fields**
- `X-Idempotency-Key` (request header, optional) — client-generated, unique per logical receiving attempt. A key not seen before behaves as before (a new receiver is created when auto-assigning). A key matching one seen within the last 15 minutes returns the receiver created on the first attempt instead of creating a new one.
- `X-Device-Id` (request header, optional) — identifies the submitting device for diagnostic logging only; does not affect routing, authorization, or the response.
- Existing query parameters (`WorkgroupId`, `poNumber`, `unreceivedHandling`) and the request/response body shape are unchanged.

**Example**
```http
POST /Receivings?WorkgroupId=27&poNumber=27-11059&unreceivedHandling=B
X-Idempotency-Key: 3f29a9d2-6b7a-4e11-9c2a-1d9a7a9b2e10
X-Device-Id: 8566612702a7d208
```
```json
{
  "RECVR_NO": "(AUTO-ASSIGN)",
  "BAT_ID": "27",
  "VEND_NO": "14210",
  "VEND_NAM": "Phillips Pet Supply",
  "RECVR_DAT": "2026-08-31T00:00:00",
  "RECVR_LOC_ID": "27",
  "USR_ID": "RAPID27",
  "IS_ALLOC": "N",
  "ALLOW_OVERRIDE_FROM_RAPIDGO": true,
  "PO_RECVR_LIN": [
    {
      "ITEM_NO": "121506",
      "PO_SEQ_NO": 1,
      "QTY_RECVD": 4.0000,
      "RECVD_COST": 12.50
    }
  ]
}
```
Response (same shape whether the receiver was newly created or returned via a matched idempotency key):
```json
{
  "RECVR_NO": "27-11205",
  "DOC_GUID": "12bc16fc-73bb-4265-9606-7c1c8d216f32",
  "BAT_ID": "27",
  "VEND_NO": "14210",
  "VEND_NAM": "Phillips Pet Supply",
  "RECVR_DAT": "2026-08-31T00:00:00",
  "RECVR_LOC_ID": "27",
  "USR_ID": "RAPID27",
  "PO_RECVR_LIN": [
    {
      "RECVR_NO": "27-11205",
      "ITEM_NO": "121506",
      "PO_SEQ_NO": 1,
      "QTY_RECVD": 4.0000,
      "RECVD_COST": 12.50
    }
  ]
}
```
Returns **HTTP 200 OK**.

## Bug Fixes

### `GET /Documents`
Document lookups using a date range in `customFilter` no longer fail on SQL Server instances running at a database compatibility level below 110 (SQL Server 2012). The date comparison is now a typed comparison against `TKT_DT` instead of a raw SQL fragment that relied on `TRY_CONVERT`, a function unavailable below that compatibility level.

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
A transfer submitted with a blank or whitespace `XFER_NO` is now auto-assigned a transfer number, matching the existing behavior for an explicit auto-assign value. Previously, only an exact auto-assign sentinel triggered numbering — a blank `XFER_NO` was posted through as-is, producing a transfer with no number.

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
Custom fields on a customer record that have a database-configured default value no longer get written as blank when omitted from a create or update. The field now resolves to its configured default, matching behavior prior to a recent custom-field handling change. This applies to any custom field with a default value, not just customer-specific ones.

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
The API no longer gives up too early when a configured network share isn't immediately reachable at startup — for example, right after the host machine reboots. Retry time against the share was raised from roughly 6 seconds to roughly 95 seconds, and the startup failure now reports whether the host was unreachable, the share wasn't shared, or the specific subdirectory was missing, instead of one generic message. No request or response contract changed; this affects service availability during startup only.
