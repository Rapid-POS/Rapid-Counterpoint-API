# Rapid CounterPoint API 1.09.08 Release Notes

**Release Date:** October 5, 2026

_Fixes two order-submission defects: a crash on split documents with custom field data, and an incorrect default on electronic-check order lines._

## Bug Fixes

### `POST /Documents`: custom field crash on split documents

Orders with custom field data no longer fail when CounterPoint has to save them as more than one document, for example when part of the order ships immediately and the rest goes on backorder.

Before this fix, sending custom field values on a document or document line in this situation returned **HTTP 400**. Custom field values now process correctly in all cases. The request and response schema are unchanged.

### `POST /Documents`: electronic-check sequence offset

When an order includes an electronic-check (`PS_DOC_HDR_EC`) block and `NXT_EC_SEQ_NO_OFFSET` is zero or negative, the API now sets it to `1`. Before this fix, the invalid value was saved as submitted.
