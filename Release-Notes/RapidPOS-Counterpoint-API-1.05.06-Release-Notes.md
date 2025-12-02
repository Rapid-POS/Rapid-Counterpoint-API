# Rapid Counterpoint API v1.5.6 Release Notes

_Released June 10, 2025_

## Enhance Custom Field Handling with Data Type Support

- Custom fields now respect their underlying data types. You’ll receive numbers, booleans, and ISO-8601 dates instead of quoted strings in the JSON response.
- Nullable custom fields return `null` in the API response when unset `NULL` in the database.
- If a custom field has a database default (e.g. `GETDATE()`), and no value is provided, the API will return that default.

**Previous:**

```
GET /Items/ABC123
Host: cpapi.rapidpos.com

{
  "ITEM_NO": "ABC123",
  "DESCR": "Sample Item",
  "CustomFields": {
    "USER_BOOLEAN_EXAMPLE": "false",
    "USER_NUMBER_EXAMPLE": "1",
    "USER_DATE_EXAMPLE": "2025-05-15 14:23:00"
  }
}
```

**After:**

```
GET /Items
Host: cpapi.rapidpos.com

{
  "ITEM_NO": "ABC123",
  "DESCR": "Sample Item",
  "CustomFields": {
    "USER_BOOLEAN_EXAMPLE": false,
    "USER_NUMBER_EXAMPLE": 1,
    "USER_DATE_EXAMPLE": "2025-05-15T14:23:00Z",
    "USER_NULL_EXAMPLE": null
  }
}
```

## Refined Pagination Logic

- Requests with `?page=0` or `?page=-1` now return page 1.
- Previously, pagination applied only when both `page` & `pageSize` provided. Now, if you supply only `page`, it returns the first 50 records. Supplying only page no longer causes a skip without a limit.
  - No endpoint or parameter changes, continue using `?page` and `?pageSize` as before.
  - Note: You will always get the first page when parameters are out of range or incomplete. The max page size is 250 for every endpoint.
- Passing only page (without pageSize) no longer skips records unexpectedly.
