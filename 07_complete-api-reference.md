# Inventra — API reference

**Audience:** Backend, mobile, and frontend developers; QA engineers
**Applies to:** Inventra v1.0 backend
**Base path:** `/api`
**Owner:** Backend Lead

This reference documents every endpoint exposed by the Inventra v1.0 backend, as currently deployed. Endpoints are grouped by resource. Each group lists its routes, the expected request, and the response shape.

> **Status note**
> This reference reflects the **live backend** at the time of writing. **Inventory reconciliation has shipped** and is documented in [section 7](#7-inventory-reconciliation). Some capabilities described in earlier design docs — phone OTP, PIN login, token refresh, batch sync, supplier management, and purchase orders — aren't in the current build. They're listed under [Planned endpoints](#12-planned-endpoints-not-yet-implemented) so you can plan integration work without assuming they exist.

---

## 1. Conventions

### 1.1 Base URL

| Environment | Base URL |
|---|---|
| Production | `https://stocksense-backend-devlopment.onrender.com/api` |
| Local | `http://localhost:5000/api` |

> **Note:** The production host still uses the original `stocksense` project slug because the repository hasn't been renamed. The product is Inventra; the deployment URL is unchanged.

### 1.2 Authentication

Inventra uses JSON Web Tokens (JWT) issued by the `jsonwebtoken` library and signed with a shared secret (`JWT_SECRET`).

To call a protected endpoint, send the token in the `Authorization` header:

```
Authorization: Bearer <token>
```

You receive a token from `POST /auth/login`. See [Authentication](#2-authentication).

### 1.3 Error format

Every error returns the same JSON shape:

```json
{
  "message": "Error description",
  "error": "Detailed error"
}
```

### 1.4 Status codes

| Code | Meaning |
|---|---|
| `200` | OK — request succeeded |
| `201` | Created — resource created |
| `400` | Bad request — malformed or missing input |
| `401` | Unauthorized — missing or invalid token |
| `404` | Not found — resource doesn't exist |
| `500` | Server error — unexpected failure |

---

## 2. Authentication

Base path: `/auth`

### Register a user

```
POST /auth/register
```

Creates a new user account. Returns `201 Created` on success.

### Log in

```
POST /auth/login
```

Authenticates a user and returns a token.

**Response:**

```json
{
  "token": "",
  "user": {}
}
```

Use the returned `token` as a Bearer token on all protected requests.

---

## 3. Users

Base path: `/users`

| Method | Path | Description |
|---|---|---|
| `GET` | `/users` | List all users |
| `GET` | `/users/:id` | Get a single user |
| `PUT` | `/users/:id` | Update a user |
| `DELETE` | `/users/:id` | Delete a user |

---

## 4. Categories

Base path: `/categories`

| Method | Path | Description |
|---|---|---|
| `POST` | `/categories` | Create a category |
| `GET` | `/categories` | List all categories |
| `GET` | `/categories/:id` | Get a single category |
| `PUT` | `/categories/:id` | Update a category |
| `DELETE` | `/categories/:id` | Delete a category |

**Create category — request body:**

```json
{
  "name": ""
}
```

---

## 5. Products

Base path: `/products`

| Method | Path | Description |
|---|---|---|
| `POST` | `/products` | Create a product |
| `GET` | `/products` | List all products |
| `GET` | `/products/:id` | Get a single product |
| `PUT` | `/products/:id` | Update a product |
| `DELETE` | `/products/:id` | Delete a product |

---

## 6. Inventory management

Base path: `/inventory`

Stock movements are recorded through dedicated inventory endpoints rather than as sub-resources of a product. Each movement is written to the `inventoryHistory` collection.

| Method | Path | Description |
|---|---|---|
| `POST` | `/inventory/stock-in` | Record a stock arrival |
| `POST` | `/inventory/stock-out` | Record a manual stock removal |
| `PUT` | `/inventory/update` | Adjust inventory |
| `GET` | `/inventory/history` | List inventory movement history |

---

## 7. Inventory reconciliation

Base path: `/reconciliation`

Reconciliation runs a stock-count session that compares the system count against a physical count, values the variance in Naira, and resolves each discrepancy. Approving a discrepancy writes to stock and to the `inventoryHistory` movement log.

| Method | Path | Description |
|---|---|---|
| `POST` | `/reconciliation` | Start a new stock-count session |
| `GET` | `/reconciliation` | List stock-count sessions (supports date-range and status filters) |
| `GET` | `/reconciliation/:id` | Get a session with its items and summary (counted/total, pending approval, discrepancies, variance value) |
| `POST` | `/reconciliation/:id/count` | Submit a physical count for a product in the session |
| `GET` | `/reconciliation/:id/items/:itemId` | Get a single discrepancy detail |
| `PUT` | `/reconciliation/:id/items/:itemId/approve` | Approve the adjustment — updates stock to the physical count |
| `PUT` | `/reconciliation/:id/items/:itemId/recount` | Set the item status to Recount |
| `PUT` | `/reconciliation/:id/finalize` | Finalize the session |

**Discrepancy item — response shape (proposed):**

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

`result` is one of `Match`, `Loss`, or `Surplus`. `status` is one of `Approved`, `Pending`, or `Recount`.

---

## 8. Sales

Base path: `/sales`

| Method | Path | Description |
|---|---|---|
| `POST` | `/sales` | Record a sale |
| `GET` | `/sales/history` | List sales history |
| `GET` | `/sales/attendant/:attendantId` | List sales for one attendant |
| `PUT` | `/sales/:id/void` | Void a sale |
| `GET` | `/sales/summary/daily` | Get the daily sales summary |

---

## 9. Business profile

Base path: `/profile`

| Method | Path | Description |
|---|---|---|
| `GET` | `/profile` | Get the business profile |
| `PUT` | `/profile` | Update the business profile |

**Update profile — request body:**

```json
{
  "businessName": "",
  "email": "",
  "phone": "",
  "address": "",
  "logo": "",
  "currency": "",
  "timezone": "",
  "lowStockAlert": 0
}
```

---

## 10. Alerts

Base path: `/alerts`

| Method | Path | Description |
|---|---|---|
| `POST` | `/alerts/check` | Run a low-stock check |
| `POST` | `/alerts/check-expiry` | Run an expiry check |
| `GET` | `/alerts` | List all alerts |
| `GET` | `/alerts?isRead=false` | List unread alerts |
| `GET` | `/alerts/summary` | Get an alert summary |
| `PUT` | `/alerts/:id/read` | Mark one alert as read |
| `PUT` | `/alerts/read-all` | Mark all alerts as read |
| `DELETE` | `/alerts/:id` | Delete an alert |

---

## 11. Reports and dashboard

Base path: `/reports`

| Method | Path | Description |
|---|---|---|
| `GET` | `/reports/summary` | Get the dashboard summary |
| `GET` | `/reports/low-stock` | List low-stock products |

> **Planned:** `/reports/order-status`, `/reports/recent-orders`, and `/reports/supplier-performance` depend on purchase orders and suppliers, which aren't built yet. See [Planned endpoints](#12-planned-endpoints-not-yet-implemented).

---

## 12. Planned endpoints (not yet implemented)

The following endpoint groups appear in the product design but **aren't in the v1.0 backend**. Don't integrate against them yet. This table tracks them so the roadmap stays visible.

| Group | Planned path | Status | Tracking |
|---|---|---|---|
| Suppliers | `/suppliers` | Planned | Phase 2 |
| Purchase orders | `/purchase-orders` | Planned | Phase 2 |
| Order-status report | `/reports/order-status` | Planned | Depends on purchase orders |
| Recent-orders report | `/reports/recent-orders` | Planned | Depends on purchase orders |
| Supplier-performance report | `/reports/supplier-performance` | Planned | Depends on suppliers |
| OTP verification | `/auth/verify-otp` | Planned | Phase 2 |
| PIN login | `/auth/login/pin` | Planned | — |
| Token refresh | `/auth/refresh` | Planned | — |
| Sync (upload queue) | `/sync` | Planned | Phase 2 |
| Sync (delta pull) | `/sync/pull` | Planned | Phase 2 |

---

## 13. Data types appendix

| Field type | Stored as | Notes |
|---|---|---|
| Identifier | MongoDB `ObjectId` | Auto-generated `_id` |
| Monetary value | `Number` | Serialised as a 2-decimal string in JSON (for example, `"700.00"`) |
| Timestamp | `Date` (ISO 8601, UTC) | `createdAt` / `updatedAt` via Mongoose `timestamps: true` |
| Boolean flags | `Boolean` | For example, `isRead` on alerts |
| Reconciliation result | `String` | One of `Match`, `Loss`, `Surplus` |
| Reconciliation status | `String` | One of `Approved`, `Pending`, `Recount` |

---

*Inventra · API reference · v1.2 · June 2026 · Internal / Confidential*
