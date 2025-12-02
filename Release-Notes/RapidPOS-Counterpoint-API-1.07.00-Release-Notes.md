# RapidPOS CounterPoint API Connector 1.07.00 — Release Notes

**Release Date:**   December 3, 2025

---

## 🛠️ Patches & Performance Improvements

### Resolved Service Crash Issue on System Shutdown or Restart
- Resolved an issue where the **RapidPOS.API.CP service** would crash when the server was restarted, shut down, or powered off unexpectedly.
- The service now exits cleanly and restarts reliably without manual intervention.

---
 
### Multiple Fixes for Final Payment Handling

- Resolved an issue where the connector threw **"Object reference not set to an instance of an object"** when processing final payments in `DocumentExtensions`.
  - Orders originating from the API document endpoint can now be **released** or **cancelled** even when `FINAL_PMT` is set to `Y`.
  - Corrected logic so release and cancel operations no longer fail when the final payment flag is present.
- Addressed an issue where setting `FINAL_PMT` to `Y` caused errors related to non-nullable fields (e.g., `IS_EBT_FOOD`).
  - Updated logic to prevent null-update exceptions during final payment processing.

### Fix for Bound Book Local Copy Report Export

- The Rapid Custom Task Agent (RCTA) uses the Rapid CP API for bound book local copy operations.
- Corrected the command execution logic so that bound book local copy reports export properly.
- Affected API endpoints:
  - **`RunReport`**
  - **`RunExport`**

### Adjusted Document History Endpoint

- Added **Business Date**, **Store ID**, and **Document ID** fields to the `DocumentHistory` endpoint to ensure a consistent and predictable sequence of returned data.

