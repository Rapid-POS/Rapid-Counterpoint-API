# Rapid CounterPoint API v1.07.00 Release Notes

_Release Date: December 3, 2025_

---

## Bug Fixes & Performance Enhancements

### Resolved Service Crash Issue on System Shutdown or Restart
- Resolved an issue where the **RapidPOS.API.CP service** would crash when the server was restarted, shut down, or powered off unexpectedly.
- The service now exits cleanly and restarts reliably without manual intervention.

### Multiple Fixes for Final Payment Handling

- Resolved an issue where the connector threw **"Object reference not set to an instance of an object"** when processing final payments in `DocumentExtensions`.
  - Orders originating from the API document endpoint can now be **released** or **cancelled** even when `FINAL_PMT` is set to `Y`.
  - Corrected logic so release and cancel operations no longer fail when the final payment flag is present.
- Addressed an issue where setting `FINAL_PMT` to `Y` caused errors related to non-nullable fields (e.g., `IS_EBT_FOOD`).
  - Updated logic to prevent null-update exceptions during final payment processing.

**Example Payload:**
```json
"PS_DOC_PMT": [
  {
    "AMT": 212.52,
    "FINAL_PMT": "Y",
    "GFC_AUTHED": "N",
    "PAY_COD": "A/R"
  }
]
```
 
### Fix for Bound Book Local Copy Report Export

- The Rapid Custom Task Agent (RCTA) uses the Rapid CP API for bound book local copy operations.
- Corrected the command execution logic so that bound book local copy reports export properly.
  
### Adjusted Document History Endpoint

- Added BUS_DAT, STR_ID and DOC_ID as the ordering for the DocumentHistory endpoint to ensure consistent ordering when calling the endpoint with StartDate and EndDate parameters.

