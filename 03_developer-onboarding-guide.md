# Inventra — Developer onboarding guide

**Audience:** Backend, frontend, and mobile developers
**Applies to:** Inventra v1.0 MVP — all engineering tracks
**Prerequisites:** Git, Node.js 18+, MongoDB (Atlas or local), Android Studio (mobile track only)
**Owner:** Engineering Lead

This guide gets you from a clean machine to a running Inventra backend. It also explains the architecture, the API surface, and the standards you follow on this project.

> **Status note**
> This guide describes the **current build**. Where a feature is designed but not yet implemented (Redis, OTP, offline sync engine, supplier management, purchase orders), it's marked **Planned**. **Inventory reconciliation has shipped and is part of the current build.** Don't assume planned items exist when you write code against the backend.

---

## 1. Product overview

Inventra is a mobile-first inventory and sales management system for SMEs across Nigeria and Sub-Saharan Africa. It's designed to work on 2G/3G connections, with full offline operation as a product goal.

Your work honours three product constraints:

- **Offline-first (client goal):** Core write operations should work without a network connection and sync when connectivity returns. The client-side queue is part of the v1.0 design; the server sync endpoints are **planned**.
- **Performance on low-end devices:** Target Android 8.0+, minimum 2 GB RAM. Keep the APK under 30 MB.
- **Role-based access control (RBAC):** Every endpoint and screen enforces role permissions (Admin, Manager, Attendant).

---

## 2. System architecture

### 2.1 Three-tier architecture

| Layer | Technology | Responsibility |
|---|---|---|
| Client | React PWA / Android (Kotlin or Flutter) | UI, local storage, sync queue |
| API | Node.js + Express.js | REST API, JWT auth, RBAC, business logic |
| Data | MongoDB (Atlas/local) | Persistent storage via Mongoose |

> **Note:** Redis (caching, queues) and S3 (file exports) appear in the target architecture but **aren't implemented** in the v1.0 backend.

### 2.2 Authentication

The current backend uses email/password login with JWT:

- **Tokens:** JWT issued by the `jsonwebtoken` library, signed with a shared secret (`JWT_SECRET`).
- **Endpoints:** `POST /auth/register` and `POST /auth/login`. Login returns `{ token, user }`.
- **RBAC roles:** `admin`, `manager`, `attendant`.

> **Planned:** Phone OTP registration, 4-digit PIN login, RS256 signing, and refresh-token rotation. None are in the current build.

---

## 3. Repository structure

```
/inventra
  /backend                → Node.js + Express API
    /src
      /routes             → Express route definitions
      /controllers        → Route handler logic
      /services           → Business logic (alerts, reports, reconciliation)
      /middleware         → RBAC, JWT verification, validation
      /models             → Mongoose schemas
      /utils              → Helpers (error handlers, validators)
    /tests
    .env.example
  /frontend               → React + Next.js PWA
    /src
      /components         → Reusable UI components
      /pages              → Next.js page routes
      /store              → State management
      /services           → API client, offline sync engine
      /db                 → Dexie.js (IndexedDB) schema and access
    /public               → Static assets, manifest.json, service worker
  /mobile                 → Android app (Kotlin or Flutter)
  /docs                   → Technical documentation
  /scripts                → Seed data and tooling
```

> **Note:** The backend currently lives in its own repository: [`Triaaz/StockSense-Backend-Devlopment`](https://github.com/Triaaz/StockSense-Backend-Devlopment). The repo name predates the Inventra rename.

---

## 4. Environment setup

### 4.1 Prerequisites

- Node.js 18 LTS or higher
- MongoDB 6.0+ (local) or a free MongoDB Atlas account
- Git 2.40+
- Android Studio (Flamingo or newer) — mobile track only

### 4.2 Set up the backend

1. **Clone the repository:**

   ```bash
   git clone https://github.com/Triaaz/StockSense-Backend-Devlopment.git
   cd StockSense-Backend-Devlopment
   ```

2. **Install dependencies:**

   ```bash
   npm install
   ```

3. **Configure environment variables.** Copy the example file, then edit it:

   ```bash
   cp .env.example .env
   ```

   Set the following values:

   ```
   PORT=5000
   MONGO_URI=mongodb://localhost:27017/inventra
   # For MongoDB Atlas, use LIVE_URL instead:
   # LIVE_URL=mongodb+srv://<username>:<password>@cluster0.mongodb.net/inventra
   JWT_SECRET=<your secret>
   ```

4. **Seed the database (optional):**

   ```bash
   npm run seed
   ```

5. **Start the development server:**

   ```bash
   npm run dev
   ```

   The API is available at `http://localhost:5000/api`.

> **Important:** Never commit your `.env` file. Confirm it's in `.gitignore` before you push.

### 4.3 Set up the frontend

```bash
cd ../frontend
npm install
cp .env.local.example .env.local
# Set: NEXT_PUBLIC_API_URL=http://localhost:5000/api
npm run dev
```

The PWA is available at `http://localhost:3000`.

### 4.4 Set up the mobile app

See `/mobile/README.md`. Key points:

- Minimum Android SDK target: API 26 (Android 8.0 Oreo)
- Local database: SQLite via Room
- Background sync: WorkManager
- Push notifications: Firebase Cloud Messaging — add `google-services.json` to `/mobile/app/`

---

## 5. API reference

### 5.1 Base URL

| Environment | URL |
|---|---|
| Production | `https://stocksense-backend-devlopment.onrender.com/api` |
| Development | `http://localhost:5000/api` |

### 5.2 Authentication header

```
Authorization: Bearer <token>
```

RBAC middleware checks the `role` field in the JWT payload on every protected request. Roles in ascending permission level: `attendant` < `manager` < `admin`.

### 5.3 Error format

```json
{
  "message": "Error description",
  "error": "Detailed error"
}
```

### 5.4 Endpoint groups

For full request and response details, see the [API reference](./07_complete-api-reference.md).

| Group | Base path | Key endpoints |
|---|---|---|
| Authentication | `/auth` | `POST /register`, `POST /login` |
| Users | `/users` | `GET /`, `GET /:id`, `PUT /:id`, `DELETE /:id` |
| Categories | `/categories` | `POST /`, `GET /`, `GET /:id`, `PUT /:id`, `DELETE /:id` |
| Products | `/products` | `POST /`, `GET /`, `GET /:id`, `PUT /:id`, `DELETE /:id` |
| Inventory | `/inventory` | `POST /stock-in`, `POST /stock-out`, `PUT /update`, `GET /history` |
| Sales | `/sales` | `POST /`, `GET /history`, `GET /attendant/:attendantId`, `PUT /:id/void`, `GET /summary/daily` |
| Reconciliation | `/reconciliation` | `POST /`, `GET /`, `GET /:id`, count submit, approve, recount, finalize |
| Business profile | `/profile` | `GET /`, `PUT /` |
| Alerts | `/alerts` | `POST /check`, `POST /check-expiry`, `GET /`, `GET /summary`, `PUT /:id/read`, `PUT /read-all`, `DELETE /:id` |
| Reports | `/reports` | `GET /summary`, `GET /low-stock` |

> **Planned (not yet built):** `/suppliers` and `/purchase-orders` endpoint groups, the supplier-dependent reports (`/reports/order-status`, `/reports/recent-orders`, `/reports/supplier-performance`), and the `/sync` endpoint group are designed but not yet implemented. Don't integrate against them.

> **Reconciliation:** The reconciliation endpoints are documented in the [API reference](./07_complete-api-reference.md#7-inventory-reconciliation).

---

## 6. Offline-first architecture (design)

The offline-first pattern writes every operation to local storage first, then syncs to the server in the background. The UI never waits for a server response.

> **Status:** The client-side local store and queue are part of the v1.0 design. The **server sync endpoints (`/sync`) aren't built yet.** Treat this section as the target design, not a wiring diagram for the current API.

For the full design — queue schema, conflict rules, and platform implementation — see the [Offline architecture and sync guide](./06_offline-architecture-sync-guide.md).

---

## 7. Database schema overview

**Conventions:**

- Primary keys are MongoDB `ObjectId` (auto-generated `_id`).
- Monetary values are stored as `Number` in Nigerian Naira (NGN). Format with `toFixed(2)` for display.
- Timestamps are ISO 8601 UTC via the Mongoose `Date` type.
- Collections include `createdAt` and `updatedAt` (Mongoose `timestamps: true`).

**Collections in the current build:**

| Collection | References |
|---|---|
| `user` | — |
| `product` | `categoryId` (`supplierId` reserved for the planned supplier feature) |
| `category` | — |
| `sale` | `attendantId` |
| `saleItem` | `saleId`, `productId` |
| `inventoryHistory` | `productId`, `userId` — append-only movement log |
| `reconciliation` | `userId` (count session) |
| `reconciliationItem` | `reconciliationId`, `productId`, `countedBy` — per-product system/physical counts, variance, status |
| `alert` | `productId` |
| `businessProfile` | — |

> **Note:** The earlier `stockEntry` model was removed and replaced by the inventory tracking system (`inventoryHistory`). Treat `inventoryHistory` as the immutable movement log — don't update or delete its records.

> **Planned collections:** `supplier` and `purchaseOrder` are designed but **not yet built**. The `product.supplierId` reference is reserved for when supplier management ships.

---

## 8. Security requirements

### 8.1 Required for all endpoints

- HTTPS only in production (TLS 1.2+; TLS 1.3 preferred)
- Valid JWT Bearer token on all non-public endpoints
- RBAC middleware runs before any business logic
- Input validation on all request parameters
- Mongoose query methods only — never raw string concatenation

### 8.2 Token rules

- Tokens are signed with `JWT_SECRET` using the `jsonwebtoken` library.
- Store the secret in an environment variable. Never hardcode it.

> **Note:** Reconciliation approvals write to stock and to the movement log. Enforce RBAC on the approve/finalize routes and log the approving `userId` with every adjustment.

> **Planned:** RS256 signing, refresh-token rotation, OTP rate limiting, and account lockout are designed but not yet implemented in the backend.

---

## 9. CI/CD pipeline

| Trigger | Actions |
|---|---|
| Every pull request | Run tests (`npm test`), lint (`npm run lint`), build check |
| Merge to `main` | Deploy to staging, run smoke tests |
| Release tag (for example, `v1.0.0`) | Deploy to production, notify the team |

The backend deploys to [Render](https://stocksense-backend-devlopment.onrender.com/).

> **Important:** Production deployments require Engineering Lead approval. Never deploy to production from a local machine.

---

## 10. Development standards

### Git workflow

- Branch naming: `feature/name`, `fix/name`, `chore/name`
- Commit messages: Conventional Commits (`feat:`, `fix:`, `docs:`, `chore:`)
- Merge via pull request with at least one reviewer

### Error codes

All errors return the standard error envelope. Use the status codes in the [API reference](./07_complete-api-reference.md#14-status-codes):

| Code | Meaning |
|---|---|
| `400` | Bad request — malformed or missing input |
| `401` | Unauthorized — missing or invalid token |
| `404` | Not found |
| `500` | Server error |

### Testing requirements

- Unit tests for all service-layer functions
- Integration tests for all endpoints (happy path and error cases)
- RBAC tests: verify each role can and can't reach the right endpoints
- Reconciliation tests: variance calculation, approve-updates-stock, and recount paths

---

*Inventra · Developer onboarding guide · v1.2 · June 2026 · Internal / Confidential*
