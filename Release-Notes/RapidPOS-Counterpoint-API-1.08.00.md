# Rapid CP API v1.08.00 Release Notes
**Release Date:** May 12th, 2026

## New Endpoint: Create or Update Item Barcodes

A new endpoint, `POST /Items/Barcode`, has been added to allow users and integrations to create or update individual barcode records associated with item units.

### Highlights

* Create new barcode records for item units.
* Update existing barcode records using `IM_BARCOD.BARCODE`.
* Returns the created or updated barcode object upon success.
* Successful requests return **HTTP 201 Created**.

### Example

```http
POST /Items/Barcode
Host: cpapi.rapidpos.com
```

```json
{
  "ITEM_NO": "ABC123",
  "BARCODE": "0123456789012",
  "UNIT": "EA"
}
```

Response:

```json
{
  "ITEM_NO": "ABC123",
  "BARCODE": "0123456789012",
  "UNIT": "EA"
}
```

---

## Receiving Endpoint Enhancements

The `POST /Receivings` endpoint now supports updating receiver header and line comment data as part of the receiving workflow.

### Supported Updates

#### Receiver Header Fields

* `REF`
* `COMMNT_1`
* `COMMNT_2`
* `COMMNT_3`

#### Receiver Line Fields

* `COMMNT_1`
* `COMMNT_2`
* `COMMNT_3`

These enhancements allow receiving metadata and line-level comments to be modified directly within RapidGO and synchronized during the receiving workflow, eliminating the need for manual updates in CounterPoint.

