# StockSense — Complete API Reference Manual

**Base URL (Production):** `https://api.stocksense.africa/v1/`
**Base URL (Development):** `http://localhost:3001/v1/`
**API Style:** RESTful JSON
**Authentication:** JWT Bearer Token (RS256) — all endpoints except those marked `PUBLIC`
**Content-Type:** `application/json`
**Swagger/OpenAPI Docs:** Available at `/api-docs` when running locally
**Version:** v1.0 · May 2026 · Internal / Confidential

---

## 1. Conventions & Global Rules

### 1.1 Authentication

All protected requests must include:
```
Authorization: Bearer <access_token>
```

Tokens are obtained from `POST /auth/login` or `POST /auth/verify-otp`. Access tokens expire after **24 hours**. Use `POST /auth/refresh` to renew.

> ℹ️ The `role` field in the JWT payload determines endpoint access. RBAC middleware validates this on every request before business logic runs.

### 1.2 Standard Response Envelope

```json
// Success (2xx)
{
  "success": true,
  "data": { },
  "message": "Operation completed successfully.",
  "meta": {
    "page": 1,
    "per_page": 20,
    "total": 150,
    "total_pages": 8,
    "timestamp": "2026-05-01T09:00:00Z"
  }
}

// Error (4xx / 5xx)
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Product name is required.",
    "field": "name"
  },
  "timestamp": "2026-05-01T09:00:00Z"
}
```

### 1.3 Common Error Codes

| Code | HTTP | Meaning |
|---|---|---|
| `UNAUTHORIZED` | 401 | Invalid or expired JWT token. Client must refresh and retry. |
| `FORBIDDEN` | 403 | User's role does not have permission for this action. |
| `NOT_FOUND` | 404 | Resource does not exist or has been soft-deleted. |
| `INSUFFICIENT_STOCK` | 422 | Sale quantity exceeds available stock. |
| `VALIDATION_ERROR` | 422 | Required field missing or fails format validation. `field` names the offending field. |
| `CONFLICT` | 409 | Sync conflict detected, or uniqueness constraint violated (e.g., duplicate SKU). |
| `RATE_LIMITED` | 429 | Rate limit exceeded. Check `Retry-After` header. |
| `SERVER_ERROR` | 500 | Unexpected internal error. Log the timestamp and report to engineering. |

### 1.4 Pagination

```
GET /products?page=2&per_page=20
```

Default: `page=1`, `per_page=20`. Maximum: `per_page=100`. Metadata returned in `meta`.

### 1.5 Rate Limiting

- **Authenticated users:** 100 requests/minute
- **Public endpoints:** 20 requests/minute per IP
- **OTP requests:** 3 per phone number per 15 minutes

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 87
X-RateLimit-Reset: 1746003600
```

---

## 2. Authentication Endpoints

| Method | Endpoint | Description | Auth |
|---|---|---|---|
| `POST` | `/auth/register` | Request OTP to begin phone registration | PUBLIC |
| `POST` | `/auth/verify-otp` | Verify OTP and create account | PUBLIC |
| `POST` | `/auth/login` | Login with phone/email + password | PUBLIC |
| `POST` | `/auth/login/pin` | Login with 4-digit PIN (attendant) | PUBLIC |
| `POST` | `/auth/logout` | Invalidate current refresh token | JWT |
| `POST` | `/auth/refresh` | Exchange refresh token for new access token | JWT |
| `POST` | `/auth/forgot-password` | Request password reset OTP | PUBLIC |
| `POST` | `/auth/reset-password` | Reset password using OTP | PUBLIC |
| `POST` | `/auth/invite` | Generate staff invitation code | Admin only |

---

### POST /auth/register

Initiates registration by sending an OTP SMS to the provided number.

**Request Body:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `phone` | string | Yes | E.164 format, e.g. `+2348012345678` |

**Success Response — 200:**
```json
{
  "success": true,
  "message": "OTP sent to +2348012345678. Valid for 10 minutes.",
  "data": { "expires_in": 600 }
}
```

---

### POST /auth/verify-otp

Verifies the OTP and creates the user account.

**Request Body:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `phone` | string | Yes | The phone number the OTP was sent to |
| `otp` | string | Yes | 6-digit OTP received via SMS |
| `business_name` | string | Yes | Business name used across all reports and receipts |
| `password` | string | Yes | Minimum 8 characters with at least one number |

**Success Response — 201:**
```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbGci...",
    "refresh_token": "eyJhbGci...",
    "user": { "id": "uuid", "role": "admin", "business_id": "uuid" },
    "business": { "id": "uuid", "name": "Mama Ngozi Stores" }
  }
}
```

---

### POST /auth/invite

Generates a staff invitation code. Admin role required.

**Request Body:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `phone` | string | Yes | Phone number of the staff member to invite |
| `role` | string | Yes | `manager` or `attendant` |

**Success Response — 201:**
```json
{
  "success": true,
  "data": {
    "invite_code": "ABC123",
    "invite_link": "https://app.stocksense.africa/join/ABC123",
    "expires_at": "2026-05-08T09:00:00Z"
  }
}
```

---

## 3. Product (Inventory) Endpoints

| Method | Endpoint | Description | Auth |
|---|---|---|---|
| `GET` | `/products` | List all products (paginated, filterable) | JWT |
| `POST` | `/products` | Create a new product | Admin / Manager |
| `GET` | `/products/:id` | Get single product with current stock | JWT |
| `PUT` | `/products/:id` | Update product fields | Admin / Manager |
| `DELETE` | `/products/:id` | Soft-delete (archive) a product | Admin only |
| `POST` | `/products/:id/stockin` | Record new stock received | Admin / Manager |
| `POST` | `/products/:id/stockout` | Record manual stock removal | Admin / Manager |
| `GET` | `/products/:id/history` | Full stock movement history for a product | JWT |
| `GET` | `/products/low-stock` | All products at or below reorder level | JWT |
| `GET` | `/categories` | All product categories | JWT |

---

### GET /products

**Query Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `page` | integer | No | Page number. Default: 1 |
| `per_page` | integer | No | Items per page. Default: 20. Max: 100 |
| `category_id` | uuid | No | Filter by category UUID |
| `low_stock` | boolean | No | If `true`, returns only products at/below reorder level |
| `search` | string | No | Partial name match, case-insensitive |
| `sort_by` | string | No | `name`, `current_stock`, `selling_price`, `created_at` |
| `sort_order` | string | No | `asc` or `desc`. Default: `asc` |

**Success Response — 200:**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "name": "Indomie Chicken 70g",
      "sku": "IND-CHK-70",
      "category": { "id": "uuid", "name": "Grocery" },
      "cost_price": "250.00",
      "selling_price": "350.00",
      "current_stock": 48,
      "reorder_threshold": 10,
      "unit_of_measure": "packs",
      "expiry_date": null,
      "supplier": { "id": "uuid", "name": "Best Foods Ltd" },
      "is_low_stock": false,
      "created_at": "2026-04-15T08:00:00Z"
    }
  ],
  "meta": { "page": 1, "per_page": 20, "total": 87, "total_pages": 5 }
}
```

---

### POST /products

**Request Body:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `name` | string | Yes | Max 200 characters |
| `category_id` | uuid | Yes | UUID of the product category |
| `cost_price` | decimal | Yes | Price paid to supplier in NGN. Must be ≥ 0 |
| `selling_price` | decimal | Yes | Price charged to customer in NGN. Must be > 0 |
| `opening_stock` | integer | Yes | Initial stock quantity |
| `reorder_threshold` | integer | Yes | Alert level. Default: 5. Set to 0 to disable alerts. |
| `sku` | string | No | Unique per business if provided. Max 50 characters. |
| `unit_of_measure` | string | No | e.g., `pieces`, `packs`, `cartons` |
| `expiry_date` | date | No | YYYY-MM-DD. Required for pharmacy products. |
| `supplier_id` | uuid | No | UUID of the primary supplier |

**Validation Rules:**
- `sku` must be unique within the business — returns `CONFLICT` if duplicate
- `selling_price` must be greater than 0
- `opening_stock` must be a non-negative integer
- `reorder_threshold` of 0 triggers a warning but is allowed

**Success Response — 201:**
```json
{ "success": true, "data": { "<full product object>" }, "message": "Product created successfully." }
```

---

### POST /products/:id/stockin

Records new stock received for a product.

**Request Body:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `quantity` | integer | Yes | Units received. Must be > 0. |
| `supplier_id` | uuid | No | UUID of the delivering supplier |
| `notes` | string | No | Free text, e.g., invoice reference |
| `device_time` | timestamp | Yes | ISO 8601 device timestamp |

**Success Response — 200:**
```json
{
  "success": true,
  "data": {
    "product_id": "uuid",
    "quantity_added": 50,
    "previous_stock": 12,
    "new_stock": 62,
    "movement_id": "uuid"
  }
}
```

---

### POST /products/:id/stockout

Records manual stock removal. Reason is mandatory.

**Request Body:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `quantity` | integer | Yes | Units removed. Must be > 0. |
| `reason` | string | Yes | Minimum 10 characters. e.g., "Damaged goods" |
| `notes` | string | No | Additional context |
| `device_time` | timestamp | Yes | ISO 8601 device timestamp |

---

## 4. Sales Endpoints

| Method | Endpoint | Description | Auth |
|---|---|---|---|
| `GET` | `/sales` | List sales (filterable) | JWT |
| `POST` | `/sales` | Record a new sale (single or multi-item) | JWT |
| `GET` | `/sales/:id` | Get single sale with full line items | JWT |
| `POST` | `/sales/:id/void` | Void a sale | Manager / Admin |
| `GET` | `/sales/summary/today` | Today's revenue and transaction count | JWT |
| `GET` | `/sales/summary` | Sales summary for a date range | JWT |
| `GET` | `/sales/by-attendant` | Sales grouped by attendant | Admin / Manager |

---

### POST /sales

Records a new sale. All stock deductions and the sale record are **a single database transaction** — all succeed or all roll back.

**Request Body:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `items` | array | Yes | Array of: `{ product_id, quantity, unit_price }` |
| `items[].product_id` | uuid | Yes | UUID of the product being sold |
| `items[].quantity` | integer | Yes | Units sold. Must be > 0. |
| `items[].unit_price` | decimal | Yes | Price at time of sale (snapshot for historical accuracy) |
| `payment_method` | string | No | `cash`, `transfer`, `pos`, `card`, `other` |
| `notes` | string | No | Optional free text |
| `device_time` | timestamp | Yes | ISO 8601 device timestamp |
| `sync_queue_id` | string | No | Queue UUID for idempotency (offline sales) |

> ℹ️ The `sync_queue_id` field prevents duplicate sales if the same operation is submitted twice after a retry.

**Success Response — 201:**
```json
{
  "success": true,
  "data": {
    "sale_id": "uuid",
    "total_amount": "1050.00",
    "attendant_id": "uuid",
    "items": [
      {
        "product_id": "uuid",
        "product_name": "Indomie Chicken 70g",
        "quantity": 3,
        "unit_price": "350.00",
        "subtotal": "1050.00"
      }
    ],
    "created_at": "2026-05-01T09:15:00Z"
  }
}
```

**Error — INSUFFICIENT_STOCK (422):**
```json
{
  "success": false,
  "error": {
    "code": "INSUFFICIENT_STOCK",
    "message": "Only 2 units of 'Indomie Chicken 70g' are available. Requested: 3.",
    "field": "items[0].quantity"
  }
}
```

---

### POST /sales/:id/void

Voids an existing sale. The original record is **never deleted**. Stock quantities are reversed.

**Request Body:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `reason` | string | Yes | Minimum 10 characters |

**Success Response — 200:**
```json
{ "success": true, "data": { "voided_sale_id": "uuid", "void_record_id": "uuid", "stock_reversed": true } }
```

---

### GET /sales/by-attendant

**Query Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `from` | date | Yes | Start date YYYY-MM-DD |
| `to` | date | Yes | End date YYYY-MM-DD |
| `attendant_id` | uuid | No | Filter to a single attendant |

**Response:**
```json
"data": [
  {
    "attendant_id": "uuid",
    "attendant_name": "Amaka",
    "total_sales": "45200.00",
    "transaction_count": 87,
    "top_product": "Indomie Chicken 70g"
  }
]
```

---

## 5. Stock Adjustment Endpoints

| Method | Endpoint | Description | Auth |
|---|---|---|---|
| `POST` | `/stock/adjust` | Create a stock adjustment with reason | Admin / Manager |
| `GET` | `/stock/adjustments` | List all adjustments (filterable) | Admin / Manager |

### POST /stock/adjust

**Request Body:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `product_id` | uuid | Yes | UUID of the product to adjust |
| `adjustment_type` | string | Yes | `increase` or `decrease` |
| `quantity_delta` | integer | Yes | Absolute number of units. Must be > 0. |
| `reason` | string | Yes | Minimum 10 characters |
| `notes` | string | No | Additional context |

---

## 6. Supplier & Purchase Order Endpoints

| Method | Endpoint | Description | Auth |
|---|---|---|---|
| `GET` | `/suppliers` | List all suppliers | JWT |
| `POST` | `/suppliers` | Add a new supplier | Admin / Manager |
| `PUT` | `/suppliers/:id` | Update supplier details | Admin / Manager |
| `DELETE` | `/suppliers/:id` | Remove supplier (preserves historical PO data) | Admin only |
| `GET` | `/purchase-orders` | List purchase orders | Admin / Manager |
| `POST` | `/purchase-orders` | Create a purchase order | Admin / Manager |
| `PUT` | `/purchase-orders/:id` | Update PO status | Admin / Manager |

### POST /suppliers

**Request Body:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `name` | string | Yes | Max 200 characters |
| `phone` | string | Yes | Must be unique per business |
| `contact_person` | string | No | Name of main contact |
| `email` | string | No | Supplier email |
| `payment_terms` | string | No | e.g., "Net 30", "Cash on delivery" |
| `bank_details` | string | No | Stored AES-256 encrypted at rest |

---

### POST /purchase-orders

**Request Body:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `supplier_id` | uuid | Yes | UUID of the supplier |
| `items` | array | Yes | `[{ product_id, quantity_ordered, unit_cost }]` |
| `expected_delivery_date` | date | No | YYYY-MM-DD |
| `notes` | string | No | e.g., special delivery instructions |
| `alert_id` | uuid | No | Pass alert UUID to auto-resolve the triggering alert |

---

### PUT /purchase-orders/:id

**Request Body:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `status` | string | Yes | `draft`, `sent`, `confirmed`, `delivered`, `cancelled` |
| `delivery_notes` | string | No | Notes on the delivery |
| `received_items` | array | No | Required when `status = delivered`. `[{ product_id, quantity_received }]` |

> ℹ️ When `status = delivered`, the server creates a `stock_in` movement for each item using `quantity_received`. This is a database transaction — all stock updates succeed or none do.

---

## 7. Alerts Endpoints

| Method | Endpoint | Description | Auth |
|---|---|---|---|
| `GET` | `/alerts` | List all active alerts | JWT |
| `PUT` | `/alerts/:id/dismiss` | Dismiss an alert | Admin / Manager |
| `PUT` | `/alerts/:id/resolve` | Mark an alert as resolved | Admin / Manager |

### GET /alerts

**Query Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `type` | string | No | `low_stock` or `expiry` |
| `status` | string | No | `active`, `dismissed`, `resolved`. Default: `active` |

**Response:**
```json
"data": [
  {
    "id": "uuid",
    "alert_type": "low_stock",
    "status": "active",
    "product": { "id": "uuid", "name": "Dettol Soap", "current_stock": 4 },
    "threshold_value": 10,
    "created_at": "2026-05-01T07:00:00Z"
  }
]
```

---

## 8. Reconciliation Endpoints

| Method | Endpoint | Description | Auth |
|---|---|---|---|
| `POST` | `/reconciliation` | Start a new reconciliation session | Admin / Manager |
| `GET` | `/reconciliation/:id` | Get session with expected quantities per product | Admin / Manager |
| `PUT` | `/reconciliation/:id` | Submit physical counts | Admin / Manager |
| `POST` | `/reconciliation/:id/finalize` | Finalize session and apply adjustments | Admin only |
| `GET` | `/reconciliation/history` | List past sessions | Admin / Manager |

### POST /reconciliation

Creates a new session capturing a snapshot of expected stock quantities for all active products.

**Success Response — 201:**
```json
"data": {
  "session_id": "uuid",
  "status": "in_progress",
  "products": [{ "product_id": "uuid", "name": "...", "expected_qty": 48 }],
  "created_at": "..."
}
```

---

### PUT /reconciliation/:id

**Request Body:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `counts` | array | Yes | `[{ product_id, actual_qty }]` — physically counted quantities |

**Response:**
```json
"data": {
  "items": [
    {
      "product_id": "uuid",
      "expected_qty": 48,
      "actual_qty": 45,
      "variance": -3,
      "variance_value": "-900.00"
    }
  ],
  "total_variance_value": "-900.00"
}
```

---

### POST /reconciliation/:id/finalize

**Request Body:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `decisions` | array | Yes | `[{ product_id, action }]` where `action` = `adjusted`, `flagged`, or `no_action` |

> ℹ️ `adjusted` items create an immutable `stock_movements` record and update `current_stock` to `actual_qty`. `flagged` items are marked for investigation without changing stock.

---

## 9. Reports Endpoints

| Method | Endpoint | Description | Auth |
|---|---|---|---|
| `GET` | `/reports/sales` | Sales report (filterable by date range) | Admin / Manager |
| `GET` | `/reports/inventory` | Inventory valuation report | Admin / Manager |
| `GET` | `/reports/audit-log` | Full immutable audit log export | Admin only |
| `GET` | `/reports/export` | Export as PDF or CSV (background job) | Admin / Manager |

### GET /reports/sales

**Query Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `from` | date | Yes | Start date YYYY-MM-DD |
| `to` | date | Yes | End date YYYY-MM-DD |
| `attendant_id` | uuid | No | Filter to a specific attendant |
| `product_id` | uuid | No | Filter to a specific product |

**Response:**
```json
"data": {
  "total_revenue": "285400.00",
  "total_transactions": 412,
  "top_products": [],
  "daily_breakdown": [],
  "by_attendant": []
}
```

---

### GET /reports/export

Triggers a background export job.

**Query Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `report_type` | string | Yes | `sales`, `inventory`, `audit_log`, `attendant_performance`, `supplier` |
| `format` | string | Yes | `pdf` or `csv` |
| `from` | date | No | Start date |
| `to` | date | No | End date |

**Success Response — 202 Accepted:**
```json
{
  "success": true,
  "data": {
    "job_id": "uuid",
    "status": "queued",
    "poll_url": "/reports/export/uuid"
  }
}
```

---

## 10. Sync Endpoints

| Method | Endpoint | Description | Auth |
|---|---|---|---|
| `POST` | `/sync` | Batch upload of offline operation queue | JWT |
| `GET` | `/sync/pull` | Pull delta of all changes since last sync | JWT |

### POST /sync

Processes a batch of queued offline operations in the order they were created.

**Request Body:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `device_id` | string | Yes | Unique device identifier (generated on first install) |
| `last_sync_at` | timestamp | Yes | ISO 8601 timestamp of last successful sync |
| `operations` | array | Yes | Ordered array from `sync_queue`: `[{ id, type, entity, payload, created_at }]` |

**Success Response — 200:**
```json
{
  "success": true,
  "data": {
    "applied": ["queue-id-1", "queue-id-2"],
    "conflicts": [{ "id": "queue-id-3", "reason": "STOCK_NEGATIVE", "details": {} }],
    "failed": [],
    "server_time": "2026-05-01T09:30:00Z"
  }
}
```

---

### GET /sync/pull

**Query Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `since` | timestamp | Yes | Returns all changes after this ISO 8601 timestamp |
| `device_id` | string | Yes | Changes from this device are excluded |

**Response:**
```json
"data": {
  "products": [],
  "sales": [],
  "alerts": [],
  "suppliers": [],
  "server_time": "2026-05-01T09:30:00Z"
}
```

---

## 11. User Management Endpoints

| Method | Endpoint | Description | Auth |
|---|---|---|---|
| `GET` | `/users` | List all users in the business | Admin only |
| `PUT` | `/users/:id/role` | Update a user's role | Admin only |
| `DELETE` | `/users/:id` | Deactivate a user account | Admin only |

### PUT /users/:id/role

Changes a user's role. **Immediately invalidates their current session.**

**Request Body:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `role` | string | Yes | `manager` or `attendant`. Cannot change Admin role via this endpoint. |

---

## Appendix: Data Types & Field Formats

| Type | Format | Example |
|---|---|---|
| UUID | UUID v4 string | `a1b2c3d4-e5f6-7890-abcd-ef1234567890` |
| Monetary amount | `DECIMAL(12,2)` as string in JSON | `"1250.00"` |
| Currency | Nigerian Naira (NGN). All amounts in NGN. | `"5500.00"` = ₦5,500.00 |
| Timestamp | ISO 8601 UTC | `"2026-05-01T09:00:00Z"` |
| Date only | ISO 8601 YYYY-MM-DD | `"2026-12-31"` |
| Phone number | E.164 format | `"+2348012345678"` |
| Boolean | JSON boolean (not string) | `true` or `false` |
| Soft delete | `deleted_at` field. Null = active. Timestamp = deleted. | `"2026-05-01T09:00:00Z"` |
| Pagination | `page` (1-indexed) + `per_page` in query params | `?page=2&per_page=20` |
| Role enum | `admin`, `manager`, `attendant` | `"attendant"` |

---

*StockSense · Complete API Reference Manual v1.0 · May 2026 · Internal / Confidential*
