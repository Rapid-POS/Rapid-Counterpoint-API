# Rapid CounterPoint API v1.08.05 Release Notes

**Release Date:** July 6, 2026

_Bug fixes addressing alternate-unit stocking quantity conversion on receivings and item description population on receiver lines._

---

## Bug Fixes

### `POST /Receivings` — Alternate-unit stocking quantity miscalculated on receiving post

When posting a receiving with an alternate purchasing unit, the API was applying the conversion numerator from the item's base alternate-unit definition rather than from the unit actually received, causing stocking quantities to be calculated incorrectly. The API now resolves the conversion factor from the received purchasing unit at the time of posting, producing correct stocking quantities regardless of subsequent changes to alt-unit definitions on the item record.

**Example**

```http
POST /Receivings
```

```json
{
  "VendNo": "VENDOR-001",
  "Lines": [
    {
      "ItemNo": "128647",
      "QtyRcvd": 5,
      "PurchUnit": "FL03"
    }
  ]
}
```

Response:

```json
{
  "RecvrNo": "R-00123",
  "Lines": [
    {
      "ItemNo": "128647",
      "QtyRcvd": 5,
      "PurchUnit": "FL03",
      "QtyRcvdStkUnit": 15
    }
  ]
}
```

Returns **HTTP 200 OK** on success.

---

### `POST /Receivings` — `PO_RECVR_LIN.ITEM_DESCR` not populated on receiver lines

Receiver lines created via the API were leaving `ITEM_DESCR` and `ITEM_DESCR_UPR` null, inconsistent with CounterPoint's native receiving behavior. The API now populates `ITEM_DESCR` and `ITEM_DESCR_UPR` on all receiver lines at the time of creation.
