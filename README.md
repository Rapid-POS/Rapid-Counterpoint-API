# Rapid CounterPoint API

**Updated November 26th 2026**

The Rapid POS CounterPoint API provides access to your NCR CounterPoint retail management system data through a REST API. This API enables integration with mobile applications, third-party systems, and custom solutions.

> **Note:** This API requires CounterPoint version 8.5.6.2 or later. For compatibility questions, contact Rapid Support.

## Overview

The Rapid CounterPoint API exposes your NCR CounterPoint data through a standard REST interface. External systems can read items, customers, documents, inventory, and purchasing data, and can create and update items and documents. See [What the API Can and Can't Do](#what-the-api-can-and-cant-do) for the full list. All endpoints require both an API key and valid CounterPoint user credentials.

**Minimum System Requirements:**

- Minimum Counterpoint version: 8.5.6.2
- Minimum SQL Server version: 2016
- Minimum Supported Operating System version: Windows Server 2016 or Windows 11 Pro
- Minimum PowerShell version: 5.1

> [!WARNING]
> Your environment must meet our [CI/CD Connector Requirements](https://github.com/Rapid-POS/Miscellaneous-Documents/blob/main/CICD-Connector-Requirements.md) (server access, firewall rules, etc.) before any install or upgrade. Troubleshooting, manual installs, or follow-up work resulting from unmet requirements will be billed at standard T&M rates.

## Table of Contents

- [Section 1: Authentication](#section-1-authentication)
- [Section 2: Connection URLs and Environments](#section-2-connection-urls-and-environments)
- [Section 3: Pagination and Filtering](#section-3-pagination-and-filtering)
- [Section 4: Common Endpoints](#section-4-common-endpoints)
- [Section 5: Working with Orders](#section-5-working-with-orders)
- [Section 6: Error Handling](#section-6-error-handling)
- [Section 7: Error Codes](#section-7-error-codes)
- [Section 8: Examples](#section-8-examples)
- [Section 9: FAQ](#section-9-faq)

---

## Section 1: Authentication

All API access requires both an API key and valid CounterPoint user credentials.

### Required Headers

| Header | Description |
|---|---|
| `x-api-key` | Your API key (contact Rapid to obtain) |
| `Content-Type` | `application/json` |

### Getting an Access Token

**Endpoint:** `POST /Authentication/Token`

The username is your CounterPoint company alias and user ID, separated by a period. The company alias decides which CounterPoint company (database) every call made with that token uses. See [Practice vs. Live Companies](#practice-vs-live-companies).

**Request body:**

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
    "USR_ID": "RAPID",
    "NAM": "Rapid User"
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

**Request body:**

```json
{
  "refreshToken": "your_refresh_token",
  "accessToken": "your_current_access_token"
}
```

### How to Obtain Your API Key

1. Contact Rapid Support
2. Provide your company information and intended use case
3. Rapid will provision an API key for your environment
4. Use the provided API key in the `x-api-key` header

[Click here to start a support ticket to obtain your API key](mailto:support@rapidpos.com)

---

## Section 2: Connection URLs and Environments

### Private vs. Public URL

When the API is installed, Rapid provides one of two URL types:

| URL type | Format | Use it when |
|---|---|---|
| Private | `https://[server-name]:[port]/` | The integration runs on your own network. This address only works from inside your network. |
| Public | `https://[name]-cpapi.rapidextras.com` | The integration is a cloud service or runs outside your network. |

If a cloud-based integration can't connect using a `server-name:port` address, it needs the public URL. Contact Rapid Support to have one set up.

### Practice vs. Live Companies

Your practice and live companies use the **same API URL and the same API key**. The company is chosen when you request your token:

- Practice: `{PracticeCompanyAlias}.{UserId}`
- Live: `{LiveCompanyAlias}.{UserId}`

Every call made with that token runs against that company. There's no separate address or parameter on individual calls. To switch companies, request a new token with the other company alias.

> **Tip:** Keep a separate token or configuration for each environment so test traffic never reaches your live company by mistake.

### Multi-Site

For Multi-Site customers, the API is normally installed on the **hub server only**. The hub holds data from every store, so one install covers most integrations. Store data reaches the hub when replication runs, and anything created at the hub reaches the stores the same way.

An install at each store server may be needed if the integration needs live data from a store, needs transactions to appear at a store immediately, has apps or devices at the store that connect to the store server directly, or must keep working when the hub or the connection between sites is down. Contact Rapid Support to discuss.

---

## Section 3: Pagination and Filtering

Most list endpoints support pagination to manage large result sets efficiently.

### Pagination Parameters

| Parameter | Type | Description |
|---|---|---|
| `Page` | integer (optional) | Page number (1-based indexing). Default: 1 |
| `PageSize` | integer (optional) | Records per page. Default: 50, Maximum: 250–500 |
| `StartDate` | datetime (optional) | Filter from this date/time (uses `RS_UTC_DT`). Format: ISO 8601 or YYYY-MM-DD |
| `EndDate` | datetime (optional) | Filter to this date/time (uses `RS_UTC_DT`). Format: ISO 8601 or YYYY-MM-DD |
| `Include` | boolean (optional) | Include related data (e.g. barcodes, grid dimensions, document status). Default: false |
| `Keyword` | string (optional) | Search term for filtering results using CounterPoint Data Dictionary fields |

### About Date Filters

`StartDate` and `EndDate` filter on `RS_UTC_DT`, which is the date and time the record was **last modified**, in **UTC**. They don't filter on the document date or ticket date.

- Use them to pull records that changed since your last sync.
- Pass times in UTC.
- If you get no results, try a wider range. An empty result can simply mean nothing changed in that window.

### Basic Pagination

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

### Date Range Filtering

```bash
curl -X GET "https://rapid.cpapi.com/Items?StartDate=2024-01-01&EndDate=2024-01-31&Page=1&PageSize=100" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "x-api-key: YOUR_API_KEY"
```

### Search with Pagination

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

## Section 4: Common Endpoints

### What the API Can and Can't Do

| Area | Read | Create / Update |
|---|---|---|
| Items | Yes | Yes |
| Customers | Yes | No |
| Documents (tickets, sales orders) | Yes | Create |
| Completed tickets (history) | Yes | No |
| Inventory and pricing | Yes | No |
| Purchase requests | — | Create |
| Purchase orders | Yes | No |
| Receivings | Yes | No |
| Inventory transfers (history) | Yes | No |
| Adding a payment to an existing open document | — | Not supported |
| Vouchers / accounts payable | — | Not supported (CounterPoint has no AP module) |
| Landed cost | — | Not supported |
| Custom `USER_` tables | Not supported | Not supported |

Need something that isn't supported? Contact Rapid Support to request an enhancement quote.

### Items

| Method | Endpoint | Description |
|---|---|---|
| GET | `/Items` | Get all items with pagination |
| GET | `/Items/{itemNo}` | Get specific item |
| POST | `/Items` | Create new item |
| PUT | `/Items/{itemNo}` | Update item |

### Customers

| Method | Endpoint | Description |
|---|---|---|
| GET | `/Customers` | Get all customers with pagination |
| GET | `/Customers/{customerNo}` | Get specific customer |

### Documents (Orders, Tickets, etc.)

| Method | Endpoint | Description |
|---|---|---|
| GET | `/Documents` | Get open documents with filtering and pagination |
| GET | `/Documents/{docId}` | Get specific open document |
| POST | `/Documents` | Create new document |
| GET | `/DocumentHistory` | Get completed tickets. Requires `StartDate` and `EndDate` |

### Inventory

| Method | Endpoint | Description |
|---|---|---|
| GET | `/ItemInventory` | Get inventory levels with pagination |
| GET | `/ItemPrices` | Get pricing information |
| GET | `/TransferHistory` | Get posted inventory transfers |

### Purchasing

| Method | Endpoint | Description |
|---|---|---|
| GET | `/PurchaseOrders` | Get purchase orders with pagination |
| GET | `/Receivings` | Get receiving documents |

---

## Section 5: Working with Orders

### Open vs. Completed Orders

- **Open documents** (orders, layaways, holds, quotes) are returned by `GET /Documents`.
- **Completed tickets** move to history and are no longer returned by `GET /Documents`. Use `GET /DocumentHistory` instead.
- `GET /DocumentHistory` requires a `StartDate` and `EndDate`. Without them, only the current day is searched.

### Document Status

Send `include=true` to get the document's current status in `PS_DOC_HDR_DOC_STAT` (for example, `DOC_STAT` = `O` for open):

```bash
curl -X GET "https://rapid.cpapi.com/Documents/{docId}?include=true" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "x-api-key: YOUR_API_KEY"
```

CounterPoint only creates a status row for **orders and layaways**. Quotes, holds, and other document types have no status, so `PS_DOC_HDR_DOC_STAT` won't appear in their responses. This is expected.

### Custom Fields

- Custom fields added to the standard CounterPoint document tables come back in the `CustomFields` section of the `_EXT` objects (for example, `PS_DOC_HDR_EXT.CustomFields` and `PS_DOC_LIN_EXT.CustomFields`).
- Separate custom tables (tables starting with `USER_`) are **not** returned by the API.

### Avoiding Duplicate Documents

The API doesn't support idempotency keys, so a retried `POST /Documents` can create a duplicate. To prevent this:

1. Put a unique ID from your system in the `REF` field of every document you create.
2. Log the `REF`, the request time, and the response for every call.
3. If a request times out, **don't resend it right away.** Query `GET /Documents` for a short window around the send time and check for your `REF` value. Only retry if it isn't there.

### Payments

Payments can be included on a new document in `PS_DOC_PMT`. Adding a payment to an existing open order or layaway isn't currently supported.

---

## Section 6: Error Handling

The API uses standard HTTP status codes and returns detailed error information.

**Error response format:**

```json
{
  "errorCode": "ERROR_INVALID_REQUEST_DATA",
  "message": "The request data is invalid",
  "details": "Additional error details here"
}
```

### HTTP Status Codes

| Code | Meaning |
|---|---|
| 200 | OK — Success |
| 400 | Bad Request — Invalid request data |
| 401 | Unauthorized — Authentication required or invalid |
| 403 | Forbidden — Insufficient permissions |
| 404 | Not Found — Resource not found |
| 500 | Internal Server Error |

---

## Section 7: Error Codes

### Authentication & Authorization

| Code | Description |
|---|---|
| `ERROR_UNAUTHORIZED` | User is not authorized to access the resource |
| `ERROR_INVALID_APIKEY` | The provided API key is invalid |
| `ERROR_INVALID_LOGIN_CREDENTIALS` | Username or password is incorrect |
| `ERROR_INVALID_ROLE_PERMISSIONS` | User does not have sufficient permissions |

### Data & Validation

| Code | Description |
|---|---|
| `ERROR_MISSING_REQUIRED_DATA` | Required fields are missing from the request |
| `ERROR_INVALID_DATA` | The provided data is invalid or malformed |
| `ERROR_INVALID_REQUEST_DATA` | The request structure or format is invalid |
| `ERROR_RECORD_NOT_FOUND` | The requested record does not exist |
| `ERROR_PROPERTY_NOT_SET` | A required property has not been set |

### Document & Transaction

| Code | Description |
|---|---|
| `ERROR_DOCUMENT_NOT_FULLY_PAID` | Operation requires the document to be fully paid |
| `ERROR_NOT_PURE_ORDER` | Operation can only be performed on pure orders |
| `ERROR_DOCUMENT_HAS_SHIPPING_ADDRESS` | Document already has a shipping address |
| `ERROR_DOCUMENT_HAS_BILLING_ADDRESS` | Document already has a billing address |
| `ERROR_DOCUMENT_BILLING_ADDRESS_NOT_FOUND` | Document does not have a billing address |
| `ERROR_DOCUMENT_SHIPPING_ADDRESS_NOT_FOUND` | Document does not have a shipping address |
| `ERROR_INVALID_LIN_TYP` | Invalid line type specified |
| `ERROR_INVALID_CONTACT_ID` | The provided contact ID is invalid |

### Payment

| Code | Description |
|---|---|
| `ERROR_INVALID_GIFTCARD_PAYMENT` | Gift card payment is invalid |
| `ERROR_AR_PAYMENT_FAILED` | Accounts receivable payment failed |

### User Management

| Code | Description |
|---|---|
| `ERROR_PASSWORD_EXPIRED` | User password has expired |

### System

| Code | Description |
|---|---|
| `ERROR_DATA_UNABLE_TO_POST` | Unable to post data to the system |
| `ERROR_INTERNAL_SERVER_ERROR` | An internal server error occurred |
| `ERROR_UNKNOWN` | An unknown error occurred |
| `SUCCESS` | Operation completed successfully |

---

## Section 8: Examples

### Complete Authentication Flow

```javascript
const tokenResponse = await fetch(
  "https://rapid.cpapi.com/Authentication/Token",
  {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "x-api-key": "YOUR_API_KEY",
    },
    body: JSON.stringify({
      username: "RAPIDPOS.RAPID",
      password: "your_password",
    }),
  }
);

const tokenData = await tokenResponse.json();
const accessToken = tokenData.accessToken;

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

### GET /DocumentHistory — Completed Tickets for a Date Range

```bash
curl -X GET "https://rapid.cpapi.com/DocumentHistory?StartDate=2026-11-01T00:00:00&EndDate=2026-11-30T23:59:59&Page=1&PageSize=100" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "x-api-key: YOUR_API_KEY"
```

### POST /Documents — Create a Ticket

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
        "DESCR": "Adams
