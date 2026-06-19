# Inventra — API reference

**Audience:** Backend, mobile, and frontend developers
**Applies to:** Inventra v1.0 backend
**Base path:** `/api`
**Owner:** Backend lead

This reference documents the endpoints exposed by the Inventra v1.0 backend as currently deployed. Endpoints are grouped by resource. The offline background sync endpoints are listed under [Planned endpoints](#12-planned-endpoints), because they are designed but not yet built.

## 1. Conventions

### 1.1 Base uniform resource locator (URL)

| Environment | Base URL |
| --- | --- |
| Production | `https://stocksense-backend-devlopment.onrender.com/api` |
| Local | `http://localhost:5000/api` |

> The production host still uses the original `stocksense` slug because the repository has not been renamed. The base path on its own returns a 404; individual endpoints sit beneath it.

### 1.2 Authentication

Inventra uses JSON Web Tokens (JWT) issued by the `jsonwebtoken` library, with bcrypt for password hashing. Send the token in the authorization header:

```
Authorization: Bearer <token>
```

You receive a token from `POST /auth/login`.

### 1.3 Error format

```json
{
  "message": "Error description",
  "error": "Detailed error"
}
```

### 1.4 Status codes

| Code | Meaning |
| --- | --- |
| 200 | OK |
| 201 | Created |
| 400 | Bad request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not found |
| 500 | Server error |

## 2. Authentication

Base path: `/auth`

| Method | Path | Description |
| --- | --- | --- |
| POST | `/auth/register` | Create a new user account |
| POST | `/auth/login` | Authenticate and return a token |
| POST | `/auth/send-otp` | Send a one-time password (OTP) to a phone number |
| POST | `/auth/verify-otp` | Verify the OTP |
| POST | `/auth/create-pin` | Set a six-digit personal identification number (PIN) |
| POST | `/auth/login-pin` | Log in with the PIN |

`POST /auth/login` response:

```json
{ "token": "", "user": {} }
```

## 3. Users

Base path: `/users`

| Method | Path | Description |
| --- | --- | --- |
| GET | `/users` | List all users |
| GET | `/users/:id` | Get a single user |
| PUT | `/users/:id` | Update a user |
| DELETE | `/users/:id` | Delete a user |

## 4. Categories

Base path: `/categories`

| Method | Path | Description |
| --- | --- | --- |
| POST | `/categories` | Create a category |
| GET | `/categories` | List all categories |
| GET | `/categories/:id` | Get a single category |
| PUT | `/categories/:id` | Update a category |
| DELETE | `/categories/:id` | Delete a category |

## 5. Products

Base path: `/products`

| Method | Path | Description |
| --- | --- | --- |
| POST | `/products` | Create a product |
| GET | `/products` | List all products |
| GET | `/products/:id` | Get a single product |
| PUT | `/products/:id` | Update a product |
| DELETE | `/products/:id` | Delete a product |

## 6. Inventory management

Base path: `/inventory`

Stock movements are recorded through dedicated inventory endpoints and written to the `inventoryHistory` collection.

| Method | Path | Description |
| --- | --- | --- |
| POST | `/inventory/stock-in` | Record a stock arrival |
| POST | `/inventory/stock-out` | Record a manual stock removal |
| PUT | `/inventory/update` | Adjust inventory |
| POST | `/inventory/reconcile` | Reconcile a product to a physical count |
| GET | `/inventory/history` | List inventory movement history |

## 7. Inventory reconciliation

Base path: `/reconciliation`

Reconciliation runs a stock-count session that compares the system count against a physical count, values the variance in Naira, and resolves each discrepancy. Approving a discrepancy writes to stock and to the movement log.

| Method | Path | Description |
| --- | --- | --- |
| POST | `/reconciliation` | Start a new stock-count session |
| GET | `/reconciliation` | List sessions (supports date-range and status filters) |
| GET | `/reconciliation/:id` | Get a session with its items and summary |
| POST | `/reconciliation/:id/count` | Submit a physical count for a product |
| GET | `/reconciliation/:id/items/:itemId` | Get a single discrepancy detail |
| PUT | `/reconciliation/:id/items/:itemId/approve` | Approve the adjustment (updates stock) |
| PUT | `/reconciliation/:id/items/:itemId/recount` | Set the item status to Recount |
| PUT | `/reconciliation/:id/finalize` | Finalize the session |

Discrepancy item response shape:

```json
{
  "id": "",
  "productId": "",
  "systemCount": 7,
  "physicalCount": 5,
  "difference": -2,
  "result": "Loss",
  "valueImpact": "-600.00",
  "status": "Pending",
  "reason": "Broken bottles found on the shelf",
  "countedBy": "",
  "countedAt": ""
}
```

`result` is one of Match, Loss, or Surplus. `status` is one of Approved, Pending, or Recount.

## 8. Suppliers

Base path: `/suppliers`

| Method | Path | Description |
| --- | --- | --- |
| POST | `/suppliers` | Create a supplier |
| GET | `/suppliers` | List suppliers |
| GET | `/suppliers/:id` | Get a single supplier |
| PUT | `/suppliers/:id` | Update a supplier |
| DELETE | `/suppliers/:id` | Delete a supplier |

## 9. Purchase orders

Base path: `/purchase-orders`

| Method | Path | Description |
| --- | --- | --- |
| POST | `/purchase-orders` | Create a purchase order |
| GET | `/purchase-orders` | List purchase orders |
| PUT | `/purchase-orders/:id/status` | Update the order status |
| DELETE | `/purchase-orders/:id` | Delete a purchase order |

> A request for a purchase order that does not exist returns 404 with `{ "message": "Purchase order not found" }`.

## 10. Sales

Base path: `/sales`

| Method | Path | Description |
| --- | --- | --- |
| POST | `/sales` | Record a sale |
| GET | `/sales/history` | List sales history |
| GET | `/sales/attendant/:attendantId` | List sales for one attendant |
| PUT | `/sales/:id/void` | Void a sale |
| GET | `/sales/summary/daily` | Get the daily sales summary |

## 11. Business profile, alerts, and reports

**Business profile** (`/profile`): `GET /profile`, `PUT /profile`.

**Alerts** (`/alerts`): `POST /alerts/check`, `POST /alerts/check-expiry`, `GET /alerts`, `GET /alerts/summary`, `PUT /alerts/:id/read`, `PUT /alerts/read-all`, `DELETE /alerts/:id`.

**Reports** (`/reports`): `GET /reports/summary`, `GET /reports/low-stock`, `GET /reports/supplier-performance`, `GET /reports/order-status`, `GET /reports/recent-orders`.

## 12. Planned endpoints

The offline background sync endpoints appear in the product design but are not in the v1.0 backend. Do not integrate against them yet.

| Group | Planned path | Status |
| --- | --- | --- |
| Sync (upload queue) | `/sync` | Planned |
| Sync (delta pull) | `/sync/pull` | Planned |

## 13. Data types

| Field type | Stored as | Notes |
| --- | --- | --- |
| Identifier | MongoDB ObjectId | Auto-generated `_id` |
| Monetary value | Number | Serialised as a two-decimal string in JSON (for example, `"700.00"`) |
| Timestamp | Date (coordinated universal time) | `createdAt` and `updatedAt` |
| Reconciliation result | String | One of Match, Loss, Surplus |
| Reconciliation status | String | One of Approved, Pending, Recount |

---

*Inventra · API reference · v2.0 · June 2026*
