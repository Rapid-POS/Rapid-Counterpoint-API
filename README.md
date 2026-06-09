# Rapid Counterpoint API

The Rapid POS Counterpoint API provides access to your NCR Counterpoint retail management system data through a REST API. This API enables integration with mobile applications, third-party systems, and custom solutions.

---

## System Requirements

This API requires Counterpoint version **8.5.7.2 or later**. For compatibility questions, contact Rapid support.

For additional connector requirements, see the [CI/CD Connector Requirements](https://github.com/Rapid-POS/Miscellaneous-Documents/blob/main/CICD-Connector-Requirements.md) documentation.

---

## Authentication

### API Key Authentication with Counterpoint Credentials

All API access requires both an API key and valid Counterpoint user credentials.

**Required Headers:**

| Header | Description |
|--------|-------------|
| `x-api-key` | Your API key (contact Rapid to obtain) |
| `Content-Type` | `application/json` |

### Getting an Access Token

**Endpoint:** `POST /Authentication/Token`

**Request Body:**

```json
{
  "username": "{CompanyAlias}.{UserId}",
  "password": "{UserPassword}"
}
```

**Example:**

```bash
curl -X POST "https://rapid.cpapi.com/Authentication/Token" \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "username": "RAPIDPOS.RAPID",
    "password": "your_password"
  }'
```

**Response:**

```json
{
  "user": {
    "USR_ID": "ADMIN",
    "NAM": "Administrator"
  },
  "expiresIn": 720,
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "base64_refresh_token"
}
```

### Using the Access Token

Include the access token in the `Authorization` header for all subsequent API calls:

```bash
curl -X GET "https://rapid.cpapi.com/Items" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "x-api-key: YOUR_API_KEY"
```

### Refreshing Tokens

**Endpoint:** `POST /Authentication/TokenRefresh`

**Request Body:**

```json
{
  "refreshToken": "your_refresh_token",
  "accessToken": "your_current_access_token"
}
```

### How to Obtain Your API Key

1. Contact Rapid support
2. Provide your company information and intended use case
3. Rapid will provision an API key for your environment
4. Use the provided API key in the `x-api-key` header

---

## Pagination

Most list endpoints support pagination to manage large result sets efficiently.

### Pagination Parameters

All pagination is handled through query parameters:

| Parameter | Type | Description |
|-----------|------|-------------|
| `Page` | integer (optional) | Page number (1-based indexing). Default: `1` |
| `PageSize` | integer (optional) | Records per page. Default: `50`, Maximum: typically `250–500` |
| `StartDate` | datetime (optional) | Filter from this date (uses `RS_UTC_DT`). Format: ISO 8601 or `YYYY-MM-DD` |
| `EndDate` | datetime (optional) | Filter to this date (uses `RS_UTC_DT`). Format: ISO 8601 or `YYYY-MM-DD` |
| `Include` | boolean (optional) | Include related data (e.g., barcodes, grid dimensions). Default: `false` |
| `Keyword` | string (optional) | Search term for filtering results using Counterpoint's Data Dictionary fields |

### Pagination Examples

#### Basic Pagination

```bash
# Get first page (default 50 records)
curl -X GET "https://rapid.cpapi.com/Items" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "x-api-key: YOUR_API_KEY"

# Get second page with 25 records per page
curl -X GET "https://rapid.cpapi.com/Items?Page=2&PageSize=25" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "x-api-key: YOUR_API_KEY"
```

#### Date Range Filtering

```bash
curl -X GET "https://rapid.cpapi.com/Items?StartDate=2024-01-01&EndDate=2024-01-31&Page=1&PageSize=100" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "x-api-key: YOUR_API_KEY"
```

#### Search with Pagination

```bash
curl -X GET "https://rapid.cpapi.com/Items?Keyword=shirt&Page=1&PageSize=50" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "x-api-key: YOUR_API_KEY"
```

### Iterating Through All Records

```javascript
let page = 1;
const pageSize = 50;
let hasMoreData = true;

while (hasMoreData) {
  const response = await fetch(
    `https://rapid.cpapi.com/Items?Page=${page}&PageSize=${pageSize}`,
    {
      headers: {
        Authorization: "Bearer YOUR_TOKEN",
        "x-api-key": "YOUR_API_KEY",
      },
    }
  );

  const data = await response.json();

  if (data.length === 0) {
    hasMoreData = false;
  } else {
    console.log(`Processing page ${page} with ${data.length} records`);
    page++;
  }
}
```

---

## Common Endpoints

### Items

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/Items` | Get all items with pagination |
| `GET` | `/Items/{itemNo}` | Get specific item |
| `POST` | `/Items` | Create new item |
| `PUT` | `/Items/{itemNo}` | Update item |

### Customers

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/Customers` | Get all customers with pagination |
| `GET` | `/Customers/{customerNo}` | Get specific customer |

### Documents (Orders, Tickets, etc.)

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/Documents` | Get documents with filtering and pagination |
| `GET` | `/Documents/{docId}` | Get specific document |
| `POST` | `/Documents` | Create new document |

### Inventory

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/ItemInventory` | Get inventory levels with pagination |
| `GET` | `/ItemPrices` | Get pricing information |

### Purchase Orders

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/PurchaseOrders` | Get purchase orders with pagination |
| `GET` | `/Receivings` | Get receiving documents |

---

## Error Handling

The API uses standard HTTP status codes and returns detailed error information:

```json
{
  "errorCode": "ERROR_INVALID_REQUEST_DATA",
  "message": "The request data is invalid",
  "details": "Additional error details here"
}
```

### HTTP Status Codes

| Code | Meaning |
|------|---------|
| `200` | OK – Success |
| `400` | Bad Request – Invalid request data |
| `401` | Unauthorized – Authentication required or invalid |
| `403` | Forbidden – Insufficient permissions |
| `404` | Not Found – Resource not found |
| `500` | Internal Server Error |

### Error Codes

#### Authentication & Authorization

| Code | Description |
|------|-------------|
| `ERROR_UNAUTHORIZED` | User is not authorized to access the resource |
| `ERROR_INVALID_APIKEY` | The provided API key is invalid |
| `ERROR_INVALID_LOGIN_CREDENTIALS` | Username or password is incorrect |
| `ERROR_INVALID_ROLE_PERMISSIONS` | User does not have sufficient permissions |

#### Data & Validation

| Code | Description |
|------|-------------|
| `ERROR_MISSING_REQUIRED_DATA` | Required fields are missing from the request |
| `ERROR_INVALID_DATA` | The provided data is invalid or malformed |
| `ERROR_INVALID_REQUEST_DATA` | The request structure or format is invalid |
| `ERROR_RECORD_NOT_FOUND` | The requested record does not exist |
| `ERROR_PROPERTY_NOT_SET` | A required property has not been set |

#### Document & Transaction

| Code | Description |
|------|-------------|
| `ERROR_DOCUMENT_NOT_FULLY_PAID` | Operation requires the document to be fully paid |
| `ERROR_NOT_PURE_ORDER` | Operation can only be performed on pure orders |
| `ERROR_DOCUMENT_HAS_SHIPPING_ADDRESS` | Document already has a shipping address |
| `ERROR_DOCUMENT_HAS_BILLING_ADDRESS` | Document already has a billing address |
| `ERROR_DOCUMENT_BILLING_ADDRESS_NOT_FOUND` | Document does not have a billing address |
| `ERROR_DOCUMENT_SHIPPING_ADDRESS_NOT_FOUND` | Document does not have a shipping address |
| `ERROR_INVALID_LIN_TYP` | Invalid line type specified |
| `ERROR_INVALID_CONTACT_ID` | The provided contact ID is invalid |

#### Payment

| Code | Description |
|------|-------------|
| `ERROR_INVALID_GIFTCARD_PAYMENT` | Gift card payment is invalid |
| `ERROR_AR_PAYMENT_FAILED` | Accounts receivable payment failed |

#### User Management

| Code | Description |
|------|-------------|
| `ERROR_PASSWORD_EXPIRED` | User password has expired |

#### System

| Code | Description |
|------|-------------|
| `ERROR_DATA_UNABLE_TO_POST` | Unable to post data to the system |
| `ERROR_INTERNAL_SERVER_ERROR` | An internal server error occurred |
| `ERROR_UNKNOWN` | An unknown error occurred |
| `SUCCESS` | Operation completed successfully |

---

## Examples

### Complete Authentication Flow

```javascript
// 1. Get access token
const tokenResponse = await fetch(
  "https://rapid.cpapi.com/Authentication/Token",
  {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "x-api-key": "YOUR_API_KEY",
    },
    body: JSON.stringify({
      username: "RAPID.ADMIN",
      password: "your_password",
    }),
  }
);

const tokenData = await tokenResponse.json();
const accessToken = tokenData.accessToken;

// 2. Use access token to get data
const itemsResponse = await fetch(
  "https://rapid.cpapi.com/Items?Page=1&PageSize=10",
  {
    headers: {
      Authorization: `Bearer ${accessToken}`,
      "x-api-key": "YOUR_API_KEY",
    },
  }
);

const items = await itemsResponse.json();
console.log(items);
```

### POST /Documents Example

```bash
curl -X POST "https://rapid.cpapi.com/Documents" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "STR_ID": "101",
    "STA_ID": "101-01",
    "DOC_TYP": "T",
    "DRW_ID": "1",
    "CUST_NO": "CASH",
    "TAX_COD": "LOCAL",
    "TKT_TYP": "T",
    "USR_ID": "RAPID",
    "SLS_REP": "RAPID",
    "TKT_DT": "2025-06-17T10:50:54",
    "PS_DOC_LIN": [
      {
        "LIN_SEQ_NO": 1,
        "LIN_TYP": "S",
        "PRC": 699.0,
        "ITEM_NO": "ADM-SCD",
        "DESCR": "Adams SC Driver, RH",
        "QTY_SOLD": 1.0,
        "QTY_UNIT": "EACH",
        "IS_TXBL": "Y",
        "UNIT_COST": 162.7644,
        "USR_ENTD_PRC": "Y"
      }
    ],
    "PS_DOC_PMT": [
      {
        "PAY_COD": "CASH",
        "AMT": 756.67,
        "FINAL_PMT": "N"
      }
    ],
    "PS_TAX": {
      "SAL_NORM_TAX_AMT": 0,
      "SAL_TAX_AMT": 0
    },
    "BILL_TO_CONTACT": {
      "NAM": "Walk-in Customer",
      "EMAIL_ADRS_1": "default@rapidpos.com"
    },
    "SHIP_TO_CONTACT": {
      "NAM": "Walk-in Customer",
      "FST_NAM": "Walk-in",
      "LST_NAM": "Customer",
      "ADRS_1": "123 Elm Street",
      "CITY": "San Diego",
      "STATE": "CA",
      "ZIP_COD": "99999",
      "CNTRY": "US"
    },
    "PS_DOC_HDR_MISC_CHRG": [
      {
        "MISC_CHRG_NO": 1,
        "MISC_TYP": "A",
        "MISC_AMT": 5
      }
    ],
    "PS_DOC_HDR_PROF": {
      "PROF_COD_1": "SAMPLE",
      "PROF_NO_1": 150,
      "PROF_ALPHA_1": "This is a sample",
      "PROF_DAT_1": "2025-06-25T17:49:24.367Z"
    }
  }'
```

---

### Creating a Sales Order Document

Sample working payload for creating a Counterpoint sales order document. Fields marked with `// MODIFY` are the most common values that should be changed for each order.

#### Sample Request

```bash
curl -X POST "https://rapid.cpapi.com/Documents" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "CUST_NO": "202-100000",
    "REF": "RAPID-TEST",
    "SHIP_DAT": "2025-08-01",
    "DOC_TYP": "O",
    "STR_ID": "202",
    "STA_ID": "202-01",
    "DRW_ID": "202-01",
    "TKT_TYP": "T",
    "USR_ID": "RAPID",
    "TAX_COD": "EC_WOOCOMM",
    "TAX_OVRD_REAS": "EC_WOOCOMM",
    "TKT_DT": null,
    "REQ_REPRICE": "Y",
    "PS_DOC_LIN": [
      {
        "ITEM_NO": "APL-UMB",
        "QTY_UNIT": "Each",
        "QTY_SOLD": 1,
        "LIN_TYP": "O",
        "PRC": 14.99,
        "PS_DOC_LIN_PRICE": [
          { "QTY_PRCD": 1, "UNIT_PRC": 14.99 }
        ]
      },
      {
        "ITEM_NO": "ADM-PLAT",
        "QTY_UNIT": "DOZ",
        "SELL_UNIT": "0",
        "QTY_SOLD": 1,
        "LIN_TYP": "O",
        "PRC": 36.95,
        "PS_DOC_LIN_PRICE": [
          { "QTY_PRCD": 1, "UNIT_PRC": 36.95 }
        ]
      }
    ],
    "PS_DOC_PMT": [
      {
        "PAY_COD": "EC_WOOCOMM",
        "AMT": 56.35,
        "GFC_AUTHED": "N",
        "PS_DOC_PMT_PROMPT": [
          { "PROMPT_TXT": "Enter Ref/Auth", "PROMPT_VAL": "81047671168" }
        ]
      }
    ],
    "PS_DOC_TAX": [
      {
        "AUTH_COD": "EC_WOOCOMM",
        "RUL_COD": "EC_WOOCOMM",
        "TAX_DOC_PART": "O",
        "TAX_AMT": 4.41
      }
    ],
    "PS_TAX": {
      "ORD_NORM_TAX_AMT": 0,
      "ORD_TAX_AMT": 4.41
    }
  }'
```

#### Annotated Payload

> **Note:** JSON does not support comments. This annotated version is for reference only.

```javascript
{
  "CUST_NO": "202-100000",        // MODIFY: Customer number for the order
  "REF": "RAPID-TEST",            // MODIFY: External reference / web order number
  "SHIP_DAT": "2025-08-01",       // MODIFY: Requested ship date
  "DOC_TYP": "O",                 // Sales order document type
  "STR_ID": "202",                // MODIFY: Store ID
  "STA_ID": "202-01",             // MODIFY: Station ID
  "DRW_ID": "202-01",             // MODIFY: Drawer ID
  "TKT_TYP": "T",                 // Ticket type
  "USR_ID": "RAPID",              // MODIFY: Counterpoint user ID
  "TAX_COD": "EC_WOOCOMM",        // MODIFY: Tax code
  "TAX_OVRD_REAS": "EC_WOOCOMM",  // MODIFY: Tax override reason
  "TKT_DT": null,                 // Leave null to let Counterpoint assign the ticket date
  "REQ_REPRICE": "Y",             // Recalculate prices based on price rules on hold-and-recall

  "PS_DOC_LIN": [
    {
      "ITEM_NO": "APL-UMB",       // MODIFY: Item number
      "QTY_UNIT": "Each",         // Required if using an alternate unit; optional otherwise
      "QTY_SOLD": 1,              // MODIFY: Quantity sold
      "LIN_TYP": "O",             // Order line type
      "PRC": 14.99,               // MODIFY: Selling price

      "PS_DOC_LIN_PRICE": [       // REQUIRED: Allows Counterpoint to calculate correct price fields
        {
          "QTY_PRCD": 1,          // Same value as QTY_SOLD
          "UNIT_PRC": 14.99       // Same value as PRC
        }
      ]
    }
  ],

  "PS_DOC_PMT": [
    {
      "PAY_COD": "EC_WOOCOMM",    // MODIFY: Payment code
      "AMT": 56.35,               // MODIFY: Payment amount
      "GFC_AUTHED": "N",          // Gift card authorized flag

      "PS_DOC_PMT_PROMPT": [      // Must be nested inside PS_DOC_PMT
        {
          "PROMPT_TXT": "Enter Ref/Auth",
          "PROMPT_VAL": "81047671168" // MODIFY: Payment reference / auth number
        }
      ]
    }
  ],

  "PS_DOC_TAX": [
    {
      "AUTH_COD": "EC_WOOCOMM",   // MODIFY: Tax authority code
      "RUL_COD": "EC_WOOCOMM",    // MODIFY: Tax rule code
      "TAX_DOC_PART": "O",        // Tax document part
      "TAX_AMT": 4.41             // MODIFY: Tax amount
    }
  ],

  "PS_TAX": {
    "ORD_NORM_TAX_AMT": 0,        // MODIFY: Normal order tax amount
    "ORD_TAX_AMT": 4.41           // MODIFY: Total order tax amount
  }
}
```
````
