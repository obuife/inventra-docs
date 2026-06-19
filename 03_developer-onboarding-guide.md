# Inventra — Developer onboarding guide

**Audience:** Backend, frontend, and mobile developers
**Applies to:** Inventra v1.0
**Prerequisites:** Git, Node.js 18 or higher, MongoDB (Atlas or local)
**Owner:** Engineering lead

This guide takes you from a clean machine to a running Inventra backend. It also explains the architecture, the application programming interface (API) surface, and the project standards. Where a feature is designed but not yet implemented (the offline background sync engine, Redis), it is marked **Planned**.

## 1. Product overview

Inventra is an inventory and sales management system for SMEs across Nigeria and Sub-Saharan Africa, designed to work on poor connections. Two product constraints shape the work:

- **Performance on low-end devices.** Keep the client light and responsive on entry-level phones.
- **Role-based access control (RBAC).** Every endpoint and screen enforces role permissions (Admin, Manager, Attendant).

## 2. System architecture

The system uses a three-tier, monolithic representational state transfer (REST) architecture in the model-view-controller (MVC) pattern.

| Layer | Technology | Responsibility |
| --- | --- | --- |
| Client | React Native mobile application (in development); web interface | User interface |
| API | Node.js and Express.js | REST API, JWT auth, RBAC, business logic |
| Data | MongoDB (Atlas or local) | Persistent storage through Mongoose |

> Redis (caching and queues) appears in the target architecture but is not implemented in the v1.0 backend.

## 3. Authentication

The backend uses JSON Web Token (JWT) authentication:

- **Tokens:** issued by the `jsonwebtoken` library and signed with a secret held in `JWT_SECRET`.
- **Passwords:** hashed with bcrypt before storage.
- **Phone OTP and PIN:** phone one-time password (OTP) registration and six-digit personal identification number (PIN) login are implemented.
- **Endpoints:** `POST /auth/register`, `POST /auth/login`, the OTP routes, and the PIN routes.
- **RBAC roles:** `admin`, `manager`, `attendant`.

## 4. Repository structure

The backend follows the MVC pattern:

```
/src
  /config      → Configuration
  /controllers → Route handler logic
  /middleware  → RBAC, JWT verification, validation
  /models      → Mongoose schemas
  /routes      → Express route definitions
  server.js    → Application entry point
```

> The backend currently lives in the repository `Triaaz/StockSense-Backend-Devlopment`. The repository name predates the Inventra rename. The default branch is `develop`.

## 5. Environment setup

### 5.1 Prerequisites

- Node.js 18 long-term support (LTS) or higher
- MongoDB 6.0+ (local) or a MongoDB Atlas account
- Git 2.40+

### 5.2 Set up the backend

```bash
git clone https://github.com/Triaaz/StockSense-Backend-Devlopment.git
cd StockSense-Backend-Devlopment
npm install
cp .env.example .env
```

Set the following values:

```bash
PORT=5000
MONGO_URI=mongodb://localhost:27017/inventra
# For MongoDB Atlas, use a connection string in MONGO_URI
JWT_SECRET=<your secret>
```

```bash
npm run seed   # optional
npm run dev
```

The API is available at `http://localhost:5000/api`.

> Never commit your `.env` file. Confirm it is in `.gitignore` before you push.

## 6. API reference

### 6.1 Base path

| Environment | Base uniform resource locator (URL) |
| --- | --- |
| Production | `https://stocksense-backend-devlopment.onrender.com/api` |
| Local | `http://localhost:5000/api` |

### 6.2 Authentication header

```
Authorization: Bearer <token>
```

RBAC middleware checks the role in the JWT payload on every protected request. Roles in ascending permission: `attendant` < `manager` < `admin`.

### 6.3 Endpoint groups

| Group | Base path | Key endpoints |
| --- | --- | --- |
| Authentication | `/auth` | register, login, OTP, PIN |
| Users | `/users` | list, get, update, delete |
| Categories | `/categories` | create, list, get, update, delete |
| Products | `/products` | create, list, get, update, delete |
| Inventory | `/inventory` | stock-in, stock-out, update, reconcile, history |
| Reconciliation | `/reconciliation` | sessions, count submit, approve, recount, finalize |
| Suppliers | `/suppliers` | create, list, get, update, delete |
| Purchase orders | `/purchase-orders` | create, list, update status, delete |
| Sales | `/sales` | record, history, by attendant, void, daily summary |
| Alerts | `/alerts` | check, check-expiry, list, summary, mark read, delete |
| Reports | `/reports` | summary, low-stock, supplier-performance, order-status, recent-orders |
| Business profile | `/profile` | get, update |

For full request and response details, see the [API reference](07_complete-api-reference.md).

> **Planned (not yet built):** the `/sync` endpoint group for the offline background sync engine.

## 7. Database schema overview

Conventions:

- Primary keys are MongoDB ObjectIds (auto-generated `_id`).
- Monetary values are stored as Number in Nigerian Naira.
- Timestamps are stored in coordinated universal time (UTC) through Mongoose.

Collections in the current build:

| Collection | References |
| --- | --- |
| `users` | — |
| `products` | `categoryId`, `supplierLink` |
| `categories` | — |
| `suppliers` | — |
| `purchaseOrders` | supplier and product references |
| `sales` | `attendantId` |
| `saleItems` | `saleId`, `productId` |
| `inventoryHistory` | `productId`, `userId` (add-only movement log) |
| `reconciliation` / `reconciliationItem` | session and per-product counts |
| `alerts` | `productId` |
| `businessProfile` | — |

> Treat `inventoryHistory` as an add-only movement log: do not update or delete its records.

## 8. Security requirements

- Hypertext Transfer Protocol Secure (HTTPS) only in production (TLS 1.2 or higher).
- Valid JWT bearer token on all protected endpoints.
- RBAC middleware runs before any business logic.
- Input validation on all request parameters.
- Mongoose query methods only; never raw string concatenation.
- Reconciliation approvals write to stock and to the movement log: enforce RBAC on the approve and finalize routes and log the approving `userId` with every adjustment.

## 9. Deployment

The backend deploys to Render; the database is hosted on MongoDB Atlas. Production deployments require engineering-lead approval.

## 10. Development standards

- **Branch naming:** `feature/name`, `fix/name`, `chore/name`.
- **Commit messages:** Conventional Commits (`feat:`, `fix:`, `docs:`, `chore:`).
- **Merge** through a pull request with at least one reviewer.
- **Error codes:** 400 (bad request), 401 (unauthorized), 403 (forbidden), 404 (not found), 500 (server error).
- **Testing:** verify endpoints with Postman, including happy-path and error cases, and confirm RBAC for each role.

---

*Inventra · Developer onboarding guide · v2.0 · June 2026*
