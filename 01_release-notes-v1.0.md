# Inventra — Release notes v1.0

**Product:** Inventra — SME inventory and sales management system
**Version:** 1.0 — MVP launch release
**Release date:** May 2026 (revised June 2026)
**Status:** Generally available (GA)
**Platform:** Android 8.0+ · Progressive Web App (PWA)
**Prepared by:** Inventra Product and Technical Writing Team

---

## 1. Overview

Inventra v1.0 is the inaugural release of the Inventra MVP — a mobile-first inventory and sales management platform built for small and medium-sized enterprises (SMEs) across Nigeria and Sub-Saharan Africa.

This release lets business owners, managers, and attendants manage inventory, record sales, reconcile physical stock against system records, and generate reports from a smartphone.

> **About the status column**
> Each feature below is tagged **Available** (in the current build) or **Planned** (designed but not yet implemented). The Planned tags reflect a reconciliation with the backend team's implementation report. **Inventory reconciliation has now shipped and is Available; supplier management and purchase orders have not been built and are reclassified as Planned.**

---

## 2. What's new in v1.0

### 2.1 Authentication and access control

| ID | Feature | Description | Status |
|---|---|---|---|
| AUTH-01 | Email/password login | Register and log in with email and password. Login returns a JWT. | Available |
| AUTH-02 | JWT sessions | Sessions are issued as JWTs signed with a shared secret (`jsonwebtoken`). | Available |
| AUTH-03 | Role-based access control | Three roles: Admin (owner), Manager, Attendant. Each controls accessible screens and actions. | Available |
| AUTH-04 | Staff invitation | Admins invite team members. | Available |
| AUTH-05 | Phone OTP registration | Register with a Nigerian phone number and an SMS OTP. | Planned |
| AUTH-06 | 4-digit PIN login | Quick attendant login on shared devices. | Planned |
| AUTH-07 | Account lockout | Lock after repeated failed login attempts. | Planned |

### 2.2 Inventory management

| ID | Feature | Description | Status |
|---|---|---|---|
| INV-01 | Product catalogue | Add and edit products with name, category, cost price, selling price, opening stock, and reorder threshold. | Available |
| INV-02 | Real-time stock tracking | Every sale and stock movement updates current stock. | Available |
| INV-03 | Stock in / stock out | Record arrivals and manual removals via the inventory endpoints, written to `inventoryHistory`. | Available |
| INV-04 | Categories | Group products by category. | Available |
| INV-05 | Soft delete | Products are archived, not permanently removed. | Available |
| INV-06 | Expiry date tracking | Optional expiry date per product — required for pharmacy use. | Available |

### 2.3 Sales management

| ID | Feature | Description | Status |
|---|---|---|---|
| SALE-01 | Quick sale recording | Record a single-item sale in under 30 seconds. | Available |
| SALE-02 | Multi-item transactions | Multiple products in one sale. | Available |
| SALE-03 | Attendant attribution | Every sale is tagged with the attendant. | Available |
| SALE-04 | Sale void | Admin and Manager can void a sale with a reason; stock is reversed. | Available |
| SALE-05 | Daily summary | Daily sales summary endpoint. | Available |
| SALE-06 | Offline sales | Sales recorded without internet sync when connectivity returns. | Planned |

### 2.4 Alerts

| ID | Feature | Description | Status |
|---|---|---|---|
| ALERT-01 | Low-stock alerts | Fire when stock reaches or falls below the reorder threshold. | Available |
| ALERT-02 | Expiry alerts | Fire ahead of a product's expiry date. | Available |
| ALERT-03 | Alert management | Mark alerts as read, view summaries, and delete alerts. | Available |

### 2.5 Inventory reconciliation

Reconciliation lets the owner run a **stock count** that compares what Inventra says is in stock against a physical count, values the difference in Naira, and resolves each discrepancy. This feature group is new in this revision and reflects the shipped build.

| ID | Feature | Description | Status |
|---|---|---|---|
| REC-01 | Stock count sessions | Start a **New Count** session and track progress (products counted out of total, percentage complete). | Available |
| REC-02 | Discrepancy detection | Each item shows system count, physical count, and the difference, classified as **Match**, **Loss**, or **Surplus**. | Available |
| REC-03 | Variance valuation | Per-item **value impact** and a session-level **variance value**, shown in Naira as potential loss. | Available |
| REC-04 | Approve adjustment / request recount | Approve a discrepancy to update stock records, or send the item back for a recount. Approval shows a confirmation prompt because it writes to stock. | Available |
| REC-05 | Discrepancy details and reason | A discrepancy detail view shows category, unit, cost/selling price, counts, value impact, who counted, when, and a free-text reason (for example, "Broken bottles found on the shelf"). | Available |
| REC-06 | Filter and review | Filter the count list by custom date range and by status (All, Approved, Pending, Recount). | Available |

> **Note:** The reconciliation REST endpoints are documented in the [API reference](./07_complete-api-reference.md).

### 2.6 Business profile

| ID | Feature | Description | Status |
|---|---|---|---|
| PROF-01 | Business profile | View and update business name, contact details, currency, timezone, and low-stock threshold. | Available |

### 2.7 Reports and dashboard

| ID | Feature | Description | Status |
|---|---|---|---|
| DASH-01 | Dashboard summary | Sales totals, stock status, and recent activity. | Available |
| DASH-02 | Low-stock report | Products below threshold. | Available |
| DASH-03 | Order status and recent orders | Purchase order status counts and recent orders. | Planned (depends on purchase orders) |
| DASH-04 | Supplier performance report | Performance metrics per supplier. | Planned (depends on supplier management) |

### 2.8 Supplier management (planned)

Supplier management is designed but **not yet built**. The capabilities below are reclassified from Available to Planned and are targeted for Phase 2.

| ID | Feature | Description | Status |
|---|---|---|---|
| SUP-01 | Supplier directory | Maintain suppliers with name, phone, and contact details. | Planned |
| SUP-02 | Purchase orders | Create and manage purchase orders. | Planned |
| SUP-03 | Supplier performance | Supplier performance reporting. | Planned |

### 2.9 Planned for a later release

| ID | Feature | Status |
|---|---|---|
| SUP-01–03 | Supplier management and purchase orders (see 2.8) | Planned |
| OFFLINE-01 | Full offline operation and background sync engine | Planned |
| INT-01 | Redis caching and job queues | Planned |
| INT-02 | SMS/OTP provider integration | Planned |

---

## 3. Performance benchmarks

| Metric | Target |
|---|---|
| App cold start | Under 3 seconds on mid-range Android |
| Dashboard load — online | Under 2 seconds on 3G |
| Sale recording (end-to-end) | Under 30 seconds |
| Product search results | Under 500 ms |
| API response time (P95) | Under 500 ms |
| APK download size | Maximum 30 MB |
| Crash-free session rate | Above 99% |

---

## 4. Known limitations and out-of-scope items

| Feature | Reason deferred | Target phase |
|---|---|---|
| Supplier management (directory) | Not yet built | Phase 2 |
| Purchase orders | Not yet built | Phase 2 |
| Order-status and supplier-performance reports | Depend on suppliers / purchase orders | Phase 2 |
| Offline sync engine | Server sync endpoints not yet built | Phase 2 |
| Phone OTP / SMS provider | Out of MVP scope | Phase 2 |
| Redis caching | Not required for MVP load | Phase 2 |
| WhatsApp integration | Valuable but not blocking | Phase 2 |
| Barcode scanning | Manual entry sufficient at launch | Phase 2 |
| Multi-store / multi-location | Not a majority MVP use case | Phase 2 |
| Native iOS application | Android-first strategy | Phase 2 |
| Local language UI (Yoruba, Igbo, Hausa) | English MVP first | Phase 2 |
| AI demand forecasting | No training data at launch | Phase 3 |

---

## 5. Security highlights

- All production API traffic is encrypted via TLS 1.2+ (TLS 1.3 preferred).
- Sessions use JWTs signed with a shared secret (`JWT_SECRET`) via the `jsonwebtoken` library.
- Passwords are hashed before storage.
- Input validation runs on all endpoints.
- The `inventoryHistory` collection is treated as an append-only movement log.
- Reconciliation adjustments that update stock are logged with the approving user and require explicit confirmation.

> **Planned:** RS256 token signing, refresh-token rotation, account lockout, OTP rate limiting, and AES-256 field encryption at rest are designed but not yet implemented.

---

## 6. Upgrade instructions

### Android app

1. Download the Inventra APK from the official distribution link.
2. Install on any Android 8.0 (Oreo) or later device.
3. Grant required permissions: Storage (PDF export) and Network access.
4. Launch the app and follow onboarding.

### Progressive Web App (PWA)

1. Open the Inventra URL in Chrome on Android, or Chrome/Firefox on desktop.
2. Tap **Add to Home screen** when prompted.
3. The app caches required assets on first load.

---

## 7. Support

For questions or feedback during the v1.0 pilot, contact the Inventra support team via the in-app Help Centre or the pilot onboarding packet provided at registration.

---

*Inventra · Release notes v1.0 (revised) · v1.2 · June 2026 · Internal / Confidential*
