# Rapid CounterPoint API 1.09.01 Release Notes
**Release Date:** August 16, 2026

_Adds the ability to create, fetch, and update saved (unposted) purchase requests with a companion purchase-entry lookup feed, and fixes a pagination-related regression in the DocumentHistory route._

## New Endpoints

### `POST /PurchaseRequests`
Creates one or more saved, unposted purchase requests. This is not a purchase order — it saves a document for review that gets posted separately, the same way a purchase request entered by hand in CounterPoint works.

**Highlights**
- Body is a JSON array of request objects — create one or many in a single call, up to 500 per request.
- `workgroupId` is required on the query string; it resolves the request-numbering counter and the fallback ship-to location for any request that omits its own `LOC_ID`.
- Records are processed independently. A record that fails validation is reported and skipped; it does not fail the other records in the same call.
- `PREQ_NO` is optional — omit it, or send the literal value `(AUTO-ASSIGN)`, to have one assigned automatically.
- Header totals, line extended cost, the single-cell flag, the cell description, all internal GUIDs, and line/cell sequence numbers are always computed server-side; any value sent for these is overridden.
- Supports site-specific custom fields via an optional `CustomFields` object on the header, each line, and each cell.
- Returns **HTTP 200 OK** for both a full success and a partial success — check the response's `ErrorCode`, not the HTTP status, to tell them apart.

**Example**
```http
POST /PurchaseRequests?workgroupId={id}
```
```json
[
  {
    "PREQ_NO": "string, optional — omit, or send \"(AUTO-ASSIGN)\", to auto-number",
    "VEND_NO": "string, required",
    "LOC_ID": "string, optional — falls back to the workgroup's default location",
    "ORD_DAT": "yyyy-mm-dd",
    "DELIV_DAT": "yyyy-mm-dd",
    "CANCEL_DAT": "yyyy-mm-dd",
    "BUYER": "string",
    "FOB": "string",
    "COMMNT_1": "string",
    "COMMNT_2": "string",
    "COMMNT_3": "string",
    "IS_ALLOC": "Y or N",
    "ALLOC_LOC_GRP": "string",
    "IS_DROPSHIP_PREQ": "Y or N",
    "ORD_MISC_AMT_1": 0,
    "ORD_MISC_AMT_2": 0,
    "ORD_MISC_AMT_3": 0,
    "ORD_MISC_AMT_4": 0,
    "ORD_MISC_AMT_5": 0,
    "SHIP_VIA_COD": "string",
    "TERMS_COD": "string, optional — falls back to the vendor's own terms",
    "VEND_NAM": "string, optional — backfilled from the vendor record",
    "VEND_FST_NAM": "string",
    "VEND_LST_NAM": "string",
    "VEND_ADRS_1": "string",
    "VEND_ADRS_2": "string",
    "VEND_ADRS_3": "string",
    "VEND_CITY": "string",
    "VEND_STATE": "string",
    "VEND_ZIP_COD": "string",
    "VEND_CNTRY": "string",
    "VEND_PHONE": "string",
    "VEND_FAX": "string",
    "VEND_CONTCT_1": "string",
    "VEND_CONTCT_2": "string",
    "SHIP_NAM": "string, optional — sending any value here skips the automatic ship-to backfill from LOC_ID",
    "SHIP_ADRS_1": "string",
    "SHIP_ADRS_2": "string",
    "SHIP_ADRS_3": "string",
    "SHIP_CITY": "string",
    "SHIP_STATE": "string",
    "SHIP_ZIP_COD": "string",
    "SHIP_CNTRY": "string",
    "SHIP_PHONE": "string",
    "SHIP_FAX": "string",
    "SHIP_CONTCT_1": "string",
    "SHIP_CONTCT_2": "string",
    "CURNCY_COD": "string, default HOME",
    "ORD_EXCH_RATE_NUMER": 1,
    "ORD_EXCH_RATE_DENOM": 1,
    "ORD_MISC_1_EXCH_RATE_NUMER": 1,
    "ORD_MISC_1_EXCH_RATE_DENOM": 1,
    "ORD_MISC_2_EXCH_RATE_NUMER": 1,
    "ORD_MISC_2_EXCH_RATE_DENOM": 1,
    "ORD_MISC_3_EXCH_RATE_NUMER": 1,
    "ORD_MISC_3_EXCH_RATE_DENOM": 1,
    "ORD_MISC_4_EXCH_RATE_NUMER": 1,
    "ORD_MISC_4_EXCH_RATE_DENOM": 1,
    "ORD_MISC_5_EXCH_RATE_NUMER": 1,
    "ORD_MISC_5_EXCH_RATE_DENOM": 1,
    "ORD_HDR_DISC_TYP": "A or P",
    "ORD_HDR_DISC_VALUE": 0,
    "ORD_HDR_DISC_AMT": 0,
    "TAX_COD_NORM": "string",
    "TAX_COD": "string",
    "ORD_TAX_AMT_NORM_HOME": 0,
    "ORD_TAX_AMT_HOME": 0,
    "CustomFields": { "site-specific column name": "value, optional" },
    "PO_PREQ_LIN": [
      {
        "ITEM_NO": "string, required",
        "ORD_QTY": 0,
        "ORD_COST": "decimal, optional — falls back to vendor cost on file, then the item's last cost",
        "ORD_UNIT": "string, optional — resolved from the item's own stocking unit",
        "ORD_QTY_UNIT": "string",
        "ORD_QTY_NUMER": 1,
        "ORD_QTY_DENOM": 1,
        "VEND_ITEM_NO": "string, optional — falls back to the vendor's item number on file, then ITEM_NO",
        "DESCR": "string, optional — falls back to the item's description",
        "ITEM_DESCR": "string",
        "DELIV_DAT": "yyyy-mm-dd",
        "CANCEL_DAT": "yyyy-mm-dd",
        "COMMNT_1": "string",
        "COMMNT_2": "string",
        "COMMNT_3": "string",
        "DIM_1_UPR": "string, default is the wildcard value",
        "DIM_2_UPR": "string, default is the wildcard value",
        "DIM_3_UPR": "string, default is the wildcard value",
        "ORD_LIN_DISC_TYP": "A or P",
        "ORD_LIN_DISC_VALUE": 0,
        "ORD_LIN_DISC_AMT": 0,
        "APPLY_DOC_DISC": "Y or N",
        "IS_TXBL_NORM": "Y or N",
        "IS_TXBL": "Y or N",
        "TAX_CATEG_NORM": "string",
        "TAX_CATEG": "string",
        "UNIT_WEIGHT": "decimal, optional — falls back to the item's own weight",
        "UNIT_CUBE": "decimal, optional — falls back to the item's own cube",
        "CustomFields": { "site-specific column name": "value, optional" },
        "PO_PREQ_CELL": [
          {
            "DIM_1_UPR": "string, default is the wildcard value",
            "DIM_2_UPR": "string, default is the wildcard value",
            "DIM_3_UPR": "string, default is the wildcard value",
            "ORD_QTY": "decimal, required if a cell is sent at all — the line's own ORD_QTY should equal the sum of its cells"
          }
        ]
      }
    ]
  }
]
```

Response — batch envelope wrapping the created request(s) in their full response shape (see `GET /PurchaseRequests/{preqNumber}` below for that shape):
```json
{
  "ErrorCode": "SUCCESS or ERROR_INVALID_DATA",
  "ERROR_DESCRIPTION": "list of per-record error messages, one per rejected record, identified by its position in the array; null on full success",
  "Data": "list of the created requests, each in the full response shape; a rejected record has no entry here"
}
```

Returns **HTTP 200 OK** (including on a partial success — inspect `ErrorCode`).

### `PUT /PurchaseRequests/{preqNumber}`
Replaces an existing, still-unposted purchase request in place.

**Highlights**
- Single request object, not an array.
- `{preqNumber}` in the URL always wins — any `PREQ_NO` sent in the body is overridden by the route value.
- Every line and cell is replaced wholesale from the request body; this is a full rewrite of the request's children, not a partial update — a line omitted from the body is removed.
- `DOC_GUID` is preserved automatically from the existing record.
- Same field contract, defaults, and validation as `POST /PurchaseRequests` above.
- Fails if the request no longer exists — most commonly because it has already been posted, at which point CounterPoint owns it and it can no longer be edited through this endpoint.
- Returns **HTTP 200 OK** with the updated request in the full response shape.

**Example**
```http
PUT /PurchaseRequests/{preqNumber}?workgroupId={id}
```
Request body: same object contract as `POST /PurchaseRequests` above, sent as a single object rather than an array.

Response: same full response shape as `GET /PurchaseRequests/{preqNumber}` below.

### `GET /PurchaseRequests/{preqNumber}`
Fetches one purchase request by number, with its lines and cells populated.

**Highlights**
- No request body and no query parameters.
- Returns **HTTP 200 OK** with a null body when `{preqNumber}` does not exist or has already been posted (and so no longer exists as a request) — check for a null/empty body rather than relying on the status code.

**Example**
```http
GET /PurchaseRequests/{preqNumber}
```
Response:
```json
{
  "...": "every field accepted on create/update, echoed back or defaulted",
  "PREQ_NO": "string, server-assigned if not supplied on create",
  "BAT_ID": "string, server-assigned",
  "DOC_GUID": "guid, server-assigned on create, preserved on update",
  "ORD_SUB_TOT": "decimal, recomputed from the lines on every save",
  "ORD_TOT_MISC": "decimal, recomputed",
  "ORD_TOT": "decimal, recomputed",
  "ORD_TOT_WEIGHT": "decimal, recomputed",
  "ORD_TOT_CUBE": "decimal, recomputed",
  "ORD_QTY_IN_STK_UNITS": "decimal, recomputed",
  "LST_MAINT_DT": "yyyy-mm-ddThh:mm:ss, server-assigned",
  "LST_MAINT_USR_ID": "string, server-assigned",
  "PO_PREQ_LIN": [
    {
      "...": "every line field accepted on create/update, echoed back or defaulted",
      "SEQ_NO": "integer, server-assigned in array order",
      "ORD_EXT_COST": "decimal, recomputed from cost times quantity unless sent explicitly",
      "IS_SINGLE_CELL": "Y or N, derived from the item's tracking method and cell count",
      "CELL_DESCR": "string, derived",
      "LIN_GUID": "guid, server-assigned",
      "PO_PREQ_CELL": [
        { "...": "every cell field accepted on create/update, echoed back", "SEQ_NO": "integer, server-assigned, matches the parent line" }
      ]
    }
  ]
}
```

Returns **HTTP 200 OK**.

### `GET /Purchasing/ReferenceData`
Returns every lookup set a purchase-entry client needs in one call: vendors, locations, vendor terms, item attributes, ship-via codes, and grid dimension values, alongside the existing item/category feed.

**Highlights**
- Read-only.
- Optional query params: a vendor number to scope the item list to one vendor, a zero-based page number, and a page size capped at 1000 — paging applies to the item list only; the lookup tables are always returned complete.
- Returns **HTTP 200 OK**.

**Example**
```http
GET /Purchasing/ReferenceData?vendorNo={vendorNo}&page={page}&pageSize={pageSize}
```
Response (new lookup sets shown; the pre-existing item/category feed is unchanged):
```json
{
  "Vendors": [
    { "VEND_NO": "string", "NAM": "string", "ADRS_1": "string", "CITY": "string", "STATE": "string", "PHONE_1": "string", "TERMS_COD": "string" }
  ],
  "Locations": [
    { "LOC_ID": "string", "DESCR": "string", "ADRS_1": "string", "CITY": "string", "STATE": "string", "PHONE_1": "string" }
  ],
  "VendorTerms": ["..."],
  "ItemAttributes": ["..."],
  "ShipViaCodes": ["..."],
  "GridDimensions1": ["..."],
  "GridDimensions2": ["..."],
  "GridDimensions3": ["..."]
}
```

Returns **HTTP 200 OK**.

## Bug Fixes

### DocumentHistory date-filter behavior
The pagination correction shipped in the August 10, 2026 RapidGO deployment had an unintended side effect on the `DocumentHistory` route's date-filter behavior, which impacted third-party API consumers who were not expected to see any change from that deployment. The date-filter behavior has been corrected.
