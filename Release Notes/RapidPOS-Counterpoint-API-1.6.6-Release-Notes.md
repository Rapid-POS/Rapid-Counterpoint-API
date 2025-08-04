# Rapid Counterpoint API v1.6.6 Release Notes

_Released August 4, 2025_

## Configurable `MaxRows` Setting Overrides Default Page Size

The Rapid CP API now supports configuring a custom maximum number of rows to return per page for `GET` endpoints via the `MaxRows` setting in `appsettings.json`.

This enhancement allows greater control over how many rows are returned in a single page of API results, which is especially useful for high-volume data retrieval scenarios.

- **Default Behavior**: `PageSize` is set to a default of **250 rows** per page on `GET` endpoints.
<img width="908" height="528" alt="2025-08-04 14_20_43-PageSize Example" src="https://github.com/user-attachments/assets/f5f6f3e5-ebca-4b49-89e8-d1d9bc24851b" />

- **Override Mechanism**: If the `MaxRows` value is present in the `appsettings.json` file, it will **override** the default `PageSize` limit.

- **Applicability**: This override applies to **all `GET` endpoints** in the Rapid CP API that support pagination.

### Configuration Example

Below is a sample configuration in `appsettings.json`:

```json
{
  ...
  "ApiSettings": {
    ...
    "ApiKey": "9f67e3d9-0139-4181-b388-45735477deb0",
    "MaxRows": 1000,
    ...
  }
}
```

If `MaxRows` is configured as:
  ```json
  "MaxRows": 1000
  ```
Then the API will return **up to 1000 rows per page** on `GET` requests, instead of the default 250.
