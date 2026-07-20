# Rapid CounterPoint API 1.08.07 Release Notes
**Release Date:** July 22, 2026

_Fixes to Inventory Adjustments serial and reason-code handling, and to Documents pagination._

## Endpoint Enhancements

### `GET /Documents`
You can now use `BeginDate` as an accepted alias for the `StartDate` query parameter.

**Supported fields**
- `BeginDate` — sets the lower bound of the date range; previously this parameter name was silently ignored. `StartDate` continues to work as before.

**Example**
```http
GET /Documents?BeginDate=7/3/2017&EndDate=12/1/2199&Page=1&PageSize=100
```

## Bug Fixes

### Documents Pagination
`GET /Documents` now returns a consistent, complete result set with no duplicate or missing records regardless of page size. Query results are sorted in a stable order before being split into pages, so paging through the full result set returns every record exactly once. When no records match, the endpoint now returns an empty array instead of a null response body.

**Example**
```http
GET /Documents?BeginDate=7/3/2017&EndDate=12/1/2199&Page=1&PageSize=100
```
Success: **HTTP 200 OK**

### Serial Status Validation for Inventory Adjustments
`POST /InventoryAdjustments` no longer incorrectly rejects valid, available serials on a negative-quantity adjustment for serialized items. A serial's status (committed/posted/available) now updates to the correct next state once an adjustment posts, matching Counterpoint's own behavior — this previously caused a subsequent posting failure ("Processing complete. 1 transaction(s) had errors and were not processed.").

**Example request**
```json
{
  "IM_ADJ_TRX_SER": [
    {
      "BAT_ID": "DEFAULT",
      "TRX_DAT": "2026-07-08T00:00:00-05:00",
      "LOC_ID": "ETWN",
      "SEQ_NO": 2,
      "SER_NO": "AC238972",
      "SER_SEQ_NO": 0
    }
  ],
  "BAT_ID": "DEFAULT",
  "ITEM_NO": "U242617",
  "LOC_ID": "ETWN",
  "TRX_DAT": "2026-07-08T00:00:00-05:00",
  "SEQ_NO": 2,
  "QTY": -1.0,
  "QTY_NUMER": 0.0,
  "QTY_DENOM": 0.0,
  "UNIT": "EACH",
  "DOC_NO": "TS11890-2AZ",
  "REF": "TS11890-2AZ",
  "REAS_COD": "TRANSFER",
  "CustomFields": {
    "USER_VEND_NO": "5214-GLEN"
  }
}
```
Previous (now-fixed) error response:
```json
{
  "ErrorCode": "ERROR_INTERNAL_SERVER_ERROR",
  "ERROR_DESCRIPTION": "Serial number \"AC238972\" is not on file or is not valid."
}
```
Success: **HTTP 200 OK**, adjustment accepted and posts without error.

### Reason Code Fallback for Inventory Adjustments
`POST /InventoryAdjustments` no longer returns an internal server error when `REAS_COD` is blank or doesn't match a configured reason code. The adjustment is now processed using the item's configured default adjustment account and profit center instead of failing.

**Example request body**
```json
{
  "BAT_ID": "string",
  "ITEM_NO": "string",
  "LOC_ID": "string",
  "TRX_DAT": "datetime",
  "SEQ_NO": "number",
  "QTY": "number",
  "UNIT": "string",
  "DOC_NO": "string",
  "REF": "string",
  "REAS_COD": "string (optional; no longer required to match an existing reason code)",
  "CustomFields": { "USER_VEND_NO": "string" }
}
```
Success: **HTTP 200 OK**, returns the created `IM_ADJ_TRX` record.

### Duplicate Serial Detection for Inventory Adjustments
`POST /InventoryAdjustments` now rejects a serial number (`IM_ADJ_TRX_SER.SER_NO`) that is already present on a different unposted adjustment for the same item, instead of silently accepting the duplicate.

**Example request body**
```json
{
  "BAT_ID": "string",
  "ITEM_NO": "string",
  "LOC_ID": "string",
  "TRX_DAT": "datetime",
  "SEQ_NO": "number",
  "IM_ADJ_TRX_SER": [
    {
      "BAT_ID": "string",
      "ITEM_NO": "string",
      "LOC_ID": "string",
      "TRX_DAT": "datetime",
      "SEQ_NO": "number",
      "SER_NO": "string",
      "SER_SEQ_NO": "number | null"
    }
  ]
}
```
Error response when the serial is already pending on an unposted adjustment:
```json
{
  "ErrorCode": "ERROR_INTERNAL_SERVER_ERROR",
  "ERROR_DESCRIPTION": "Serial number \"<SER_NO>\" already exists on an unposted transaction. Type: Inventory adjustments, Batch ID: \"<BAT_ID>\", Document number: \"<DOC_NO>\", Date: \"<TRX_DAT>\""
}
```
Success: **HTTP 200 OK** on a valid submission.

### Committed Serial Error for Inventory Adjustments
`POST /InventoryAdjustments` now distinguishes an already-in-stock serial (`STAT` `A`) from a committed serial (`STAT` `C`) on positive-quantity adjustments, returning a dedicated `"Serial number is committed."` error instead of a generic validation failure.

### Account Number Truncation in Profit Center Lookup
Account numbers are no longer clipped by the profit-center padding logic when the account's profit-center method is set to none; the full-length account number is now returned as configured.
