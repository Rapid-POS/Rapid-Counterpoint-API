# Rapid CounterPoint API v1.08.04 Release Notes

**Release Date:** June 17, 2026

_Bug fixes for serial number validation on inventory adjustments, and for item description handling, null-safety, and total calculation on receiving documents._

---

## Bug Fixes

### `POST /InventoryAdjustments`

Negative quantity adjustments now validate the serial number against native CounterPoint rules before being accepted. A serial that is not on file, or is on file but not in a valid state for reduction (for example, already committed or posted), is now rejected rather than accepted.

**Integrator action required:** Ensure the serial exists and is valid for reduction before submitting a negative adjustment. Submitting an invalid serial number now returns **HTTP 500**.

**Supported fields**

- `QTY` — when negative, triggers serial validation against on-file inventory
- `IM_ADJ_TRX_SER[].SER_NO` — must reference a serial number that is on file and valid for reduction when `QTY < 0`

**Example**

```http
POST /InventoryAdjustments
```

```json
{
  "IM_ADJ_TRX_SER": [
    {
      "BAT_ID": "101",
      "ITEM_NO": "TEST-ITEM",
      "LOC_ID": "101",
      "TRX_DAT": "9999-01-01T00:00:00-00:00",
      "SEQ_NO": 1,
      "SER_NO": "xxxxx"
    }
  ],
  "BAT_ID": "101",
  "ITEM_NO": "TEST-ITEM",
  "LOC_ID": "101",
  "TRX_DAT": "9999-01-01T00:00:00-00:00",
  "SEQ_NO": 1,
  "QTY": -1.0,
  "QTY_NUMER": 0,
  "QTY_DENOM": 0,
  "UNIT": "EACH",
  "DOC_NO": "null",
  "REF": "null",
  "REAS_COD": "TRANSFER",
  "CustomFields": {
    "USER_VEND_NO": "xxxxx"
  }
}
```

Response:

```json
{
  "ErrorCode": "ERROR_INTERNAL_SERVER_ERROR",
  "ERROR_DESCRIPTION": "Serial number \"xxxxx\" is not on file or is not valid."
}
```

---

### `POST /Documents/Receivings`

Item descriptions are now populated correctly. `DESCR` and `ITEM_DESCR` previously returned blank when the configured description source was null or empty; both fields now fall back to the item's default description. `QTY_UNIT` handling is now null-safe during receiving creation.

**Supported fields**

- `DESCR` — now populated using the item's long description; falls back to the default item description when long description is null or empty
- `ITEM_DESCR` — follows the same fallback logic as `DESCR`
- `QTY_UNIT` — quantity-unit handling is now null-safe during receiving creation

**Example**

```http
POST /Documents/Receivings?WorkgroupId=27&poNumber=&unreceivedHandling=
```

```json
{
  "RECVR_NO": "(AUTO-ASSIGN)",
  "BAT_ID": "27",
  "VEND_NO": "1000",
  "VEND_NAM": "21st Century - PRIMETIME",
  "RECVR_LOC_ID": "27",
  "PO_RECVR_LIN": [
    {
      "PO_SEQ_NO": 1,
      "ITEM_NO": "10003",
      "QTY_RECVD": 0,
      "QTY_UNIT": "PACK",
      "RECVD_COST": 2.75
    }
  ]
}
```

Response:

```json
{
  "ITEM_NO": "10003",
  "DESCR": "NV QUIET MMNTS CLMNG TAB",
  "ITEM_DESCR": "NV QUIET MMNTS CLMNG TAB"
}
```

> **Configuration note:** If your Purchasing Control setting (_Setup > Purchasing > Purchasing Control > Activity > Purchase Order > Default Item Description_) is set to `Long Description`, ensure item records contain valid long descriptions. Items with a blank long description will fall back to the default item description.

---

### `POST /Documents/Receivings` — Receiver total calculation

The `Total` section is now computed as subtotal + total misc at creation time. Previously, totals required a manual refresh in CounterPoint to reflect the correct value. No request or response fields change; the correct total is now returned on the initial create response.
````
