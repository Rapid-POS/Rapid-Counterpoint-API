# Rapid Counterpoint API v1.07.14 Release Notes  
**Release Date:** April 14, 2026  

---

## Fixes and Improvements

### POST Customer Endpoint – CustomField Handling
- Updated the **POST Customer** endpoint to exclude CustomFields from inserts and updates when they are not defined in the payload. Previously, if you didn't include a field in the CustomFields array, it would be set to NULL in the database.
- Prevents unintended default values from being inherited from the user template.
- Aligns API behavior with expected database defaults and reduces unintended data propagation.
---

### Database Error Handling Improvements
- Enhanced handling of malformed database errors.
- Added a **retry mechanism** to improve resilience during transient failures.

---

### Consistency and Error Handling
- Improved logic surrounding field definitions.
- Increased consistency and reduced edge-case errors.

---

### Release pipeline improvements
- Improved reliability and consistency in deployment pipeline. This ensures faster and error free releases of the Rapid CP API.
