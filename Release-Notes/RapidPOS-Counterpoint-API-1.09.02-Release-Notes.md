# Rapid CounterPoint API v1.08.04 - Release Notes 
**Release Date:** June 17th 2026
_Enhancements to serial number validation for inventory adjustments and improved item description handling and total calculation accuracy for receiving documents._

## Breaking Changes

### `POST /InventoryAdjustments`
Negative quantity adjustments that reference a serial number not on file now return an error instead of succeeding silently.

Integrations performing serialized inventory reductions must ensure the serial number exists in inventory before submitting a negative adjustment. Submitting an invalid serial number now returns **HTTP 500** with the following response:

```json
{
  "ErrorCode": "ERROR_INTERNAL_SERVER_ERROR",
  "ERROR_DESCRIPTION": "Serial number \"xxxxx\" is not on file or is not valid."
}
```

## Endpoint Enhancements

### `POST /InventoryAdjustments`
Serial number existence is now validated when `QTY` is negative, aligning API behavior with native CounterPoint functionality.

**Supported fields**
- `QTY` — when negative, triggers serial number validation against on-file inventory
- `IM_ADJ_TRX_SER[].SER_NO` — must reference an existing serial number when `QTY < 0`

### `POST /Documents/Receivings`
Item descriptions and receiver totals are now calculated correctly at creation time, with no manual refresh required in CounterPoint.

**Supported fields**
- `DESCR` — now populated using the item's long description; falls back to the default item description when long description is null or empty
- `ITEM_DESCR` — follows the same fallback logic as `DESCR`
- `Total` — now correctly calculated as subtotal + total misc at creation time, based on active Purchasing Control configuration

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
      "RECVD_COST": 2.7500
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

> **Configuration note:** If your Purchasing Control setting (**Setup > Purchasing > Purchasing Control > Activity > Purchase Order > Default Item Description**) is set to `Long Description`, ensure item records contain valid long descriptions. Items with a blank long description will fall back to the default item description. RapidGo users should refresh item data from Settings after updating item descriptions.
