# Rapid CounterPoint API 1.08.06 Release Notes
**Release Date:** July 22, 2026

_Fixes to inventory adjustment serial validation, account number generation, and Documents pagination, plus a new date-filter alias._

## Endpoint Enhancements

### `GET /Documents`
Added `BeginDate` as an accepted alias for the `StartDate` query parameter.

**Supported fields**
- `BeginDate` — optional; binds to the same lower date-range bound as `StartDate`.

**Example**
```http
GET /Documents?BeginDate=2026-07-01&EndDate=2026-07-22&Page=1&PageSize=100
```
Response — total matching record count is returned in the `x-total-count` header:
```json
[
  { "DOC_ID": 10042, "STR_ID": "GLEN" },
  { "DOC_ID": 10042, "STR_ID": "MTJY" }
]
```
Returns **HTTP 200 OK**.

## Bug Fixes

### `GET /Documents` — pagination consistency
Pagination now returns a consistent, complete result set with no duplicate or missing records, regardless of page size. Results are ordered deterministically before being split into pages, and a query with no matching records now returns an empty array instead of a null response body.

**Example**
```http
GET /Documents?StartDate=2017-07-03&EndDate=2199-12-01&Page=1&PageSize=100
```
Returns **HTTP 200 OK**.

### `POST /InventoryAdjustments` — serial validation over-rejection
Negative adjustments on valid, available serialized items no longer return a **500** error. A serial's status also now updates to the correct next state after the adjustment posts, preventing the follow-on "1 transaction(s) had errors" failure.

**Example**
```http
POST /InventoryAdjustments
```
```json
{
  "BAT_ID": "DEFAULT",
  "ITEM_NO": "U242617",
  "LOC_ID": "ETWN",
  "TRX_DAT": "2026-07-08T00:00:00-05:00",
  "SEQ_NO": 2,
  "QTY": -1.0,
  "UNIT": "EACH",
  "DOC_NO": "TS11890-2AZ",
  "REF": "TS11890-2AZ",
  "REAS_COD": "TRANSFER",
  "IM_ADJ_TRX_SER": [
    {
      "BAT_ID": "DEFAULT",
      "ITEM_NO": "U242617",
      "LOC_ID": "ETWN",
      "TRX_DAT": "2026-07-08T00:00:00-05:00",
      "SEQ_NO": 2,
      "SER_NO": "AC238972",
      "SER_SEQ_NO": 0
    }
  ],
  "CustomFields": { "USER_VEND_NO": "5214-GLEN" }
}
```
Previous (now-fixed) error response:
```json
{
  "ErrorCode": "ERROR_INTERNAL_SERVER_ERROR",
  "ERROR_DESCRIPTION": "Serial number \"AC238972\" is not on file or is not valid."
}
```
Now returns **HTTP 200 OK**; adjustment accepted and posts without error.

### `POST /InventoryAdjustments` — invalid reason code
An adjustment submitted with a `REAS_COD` that doesn't match a configured reason code no longer throws an internal server error. It now falls back to the item's default adjustment account and profit-center setup instead of failing.

**Example**
```http
POST /InventoryAdjustments
```
```json
{
  "BAT_ID": "string",
  "ITEM_NO": "string",
  "LOC_ID": "string",
  "TRX_DAT": "datetime",
  "SEQ_NO": 0,
  "QTY": 0,
  "UNIT": "string",
  "DOC_NO": "string",
  "REF": "string",
  "REAS_COD": "string (optional; no longer required to match an existing reason code)",
  "CustomFields": { "USER_VEND_NO": "string" }
}
```
Returns **HTTP 200 OK**, and returns the created `IM_ADJ_TRX` record.

### `POST /InventoryAdjustments` — duplicate serial rejection
Submitting a serial number that's already present on a different unposted adjustment for the same item is now rejected, returning an error that identifies the conflicting batch, document number, and date, instead of silently accepting the duplicate.

**Example**
```http
POST /InventoryAdjustments
```
```json
{
  "BAT_ID": "string",
  "ITEM_NO": "string",
  "LOC_ID": "string",
  "TRX_DAT": "datetime",
  "SEQ_NO": 0,
  "IM_ADJ_TRX_SER": [
    {
      "BAT_ID": "string",
      "ITEM_NO": "string",
      "LOC_ID": "string",
      "TRX_DAT": "datetime",
      "SEQ_NO": 0,
      "SER_NO": "string",
      "SER_SEQ_NO": null
    }
  ]
}
```
New error response when the serial is already pending on an unposted adjustment:
```json
{
  "ErrorCode": "ERROR_INTERNAL_SERVER_ERROR",
  "ERROR_DESCRIPTION": "Serial number \"<SER_NO>\" already exists on an unposted transaction. Type: Inventory adjustments, Batch ID: \"<BAT_ID>\", Document number: \"<DOC_NO>\", Date: \"<TRX_DAT>\""
}
```
A valid, non-duplicate submission returns **HTTP 200 OK**.

### `POST /InventoryAdjustments` — G/L account number truncation
The posted G/L account number no longer drops characters.

- Accounts configured with no profit-center method are now used exactly as stored instead of being clipped to the company's main-account length.
- Example: an account code of `5003-PARTS -101` previously posted as `5003101`; it now posts as `5003-PARTS -101`.
- Example: an 11-character account like `51950-2-222` previously posted as `51950-2-22` (last character dropped); it now posts in full.
- Accounts that do use a profit-center method (location, category, sub-category, store, or document) no longer lose characters sitting between the main-account and profit-center segments — only the profit-center portion itself is replaced.

No request or response schema changes; both cases return **HTTP 200 OK**.
