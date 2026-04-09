# Rapid Counterpoint API v1.07.14 Release Notes  
**Release Date:** April 14, 2026  

---

## Fixes and Improvements

### Post Customer Endpoint – USER_ Field Handling
- Updated the **Post Customer** endpoint to exclude `USER_` fields from the model definition when they are not included in the request body.
- Prevents unintended default values from being inherited from the user template.
- Ensures **database schema defaults** are applied correctly.

---

### PO_VEND_ITEM Field Definition Fix
- Fixed a field definition issue affecting the `PO_VEND_ITEM` table.
- Ensures proper handling of schema and data integrity.

---

### Purchase Order (PO) Improvements
- **PO Line Comments**
  - Comments now properly carry over to RapidGO

- **Quantity Window Enhancements**
  - Removed non-selectable units from the quantity selection interface
  - Improves usability and prevents invalid selections

---

## Internal Connector Handling Improvements

### Database Error Handling Improvements
- Enhanced handling of malformed database errors
- Added a **retry mechanism** to improve resilience during transient failures

---

### API and Mobile Field Definition Improvements
- Improved logic surrounding field definitions across both:
  - API layer
  - Mobile application
- Increases consistency and reduces edge-case errors

---

### CI/CD Release Template Configuration Fix
- Resolved an issue in the **RapidPOS.API.CP release template** where application settings were retrieved incorrectly when multiple servers shared the same agent machine name.
- Improves reliability and consistency in deployment pipelines.

---

### Customer Service Model Cleanup
- Updated customer service logic to remove `USER_` fields from the model during insert operations when they are not present in the request payload.
- Aligns API behavior with expected database defaults and reduces unintended data propagation.

---

### Bulk Insert and Model Initialization Enhancements
- Refactored `BulkInsertAsync`:
  - Added **NLog error logging**
  - Improved exception handling
- Ensured `modelDef.AfterInit` is always executed
- Cleared **OrmLite cache** after table recreation to prevent stale schema issues
- Extended `PO_VEND_ITEM`:
  - Implemented `ICustomFieldHolder`
  - Added support for a `CustomFields` dictionary (excluded from ORM mapping)

