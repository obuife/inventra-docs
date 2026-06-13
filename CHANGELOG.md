# Changelog

All notable changes to Inventra documentation are recorded in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/). Versions align with the Inventra product release cycle.

---

## [1.2.0] — June 2026 — Reconciliation shipped; supplier management deferred

A reconciliation with the engineering team: **inventory reconciliation (REC-01) has been built and is now Available**, while **supplier management — including the supplier directory, purchase orders, and the supplier-performance report — has not been built and is reclassified as Planned**.

### Added

- **Inventory reconciliation** documented as a shipped feature across the set, based on the engineering team's screenshots of the Stock count screen: count sessions (**New Count**), progress tracking, system vs physical count, difference, **Result** (Match/Loss/Surplus), **value impact** and session **variance value** in Naira, **Status** (Approved/Pending/Recount), date/status filters, the **Product Discrepancy Details** view with a reason field, and the **Approve Adjustment** / **Request Recount** actions with an approval confirmation prompt.
  - **01 Release notes** — new section 2.5 (REC-01–REC-06) marked Available.
  - **07 API reference** — new section 7, `/reconciliation`, with the reconciliation route set and discrepancy-item response shape.
  - **03 Developer onboarding** — added the reconciliation endpoint group, `reconciliation` / `reconciliationItem` collections, and reconciliation testing requirements.
  - **05 Admin guide** — section 3 reconciliation promoted from "planned feature" to Available and rewritten to match the shipped UI; reconciliation rows added to the RBAC matrix; a reconciliation smoke test added.
  - **04 User help guide** — new "Run a stock count (reconciliation)" walkthrough and FAQ entry.
  - **02 Demo video script** — new Scene 09 demonstrating reconciliation.

### Changed

- **Supplier management → Planned** across all documents. The supplier directory (SUP-01), purchase orders (SUP-02), supplier performance (SUP-03), and the supplier-dependent reports (order status, recent orders, supplier-performance) are reclassified from Available to Planned (Phase 2).
  - **README** — features, project status, and roadmap updated; suppliers/POs moved to Planned, reconciliation added as Available.
  - **01 Release notes** — section 2.8 "Supplier management (planned)"; DASH-03 and DASH-04 marked Planned; known-limitations and planned tables updated.
  - **07 API reference** — Suppliers and Purchase orders sections removed from the live API and moved to Planned; `/reports/order-status`, `/reports/recent-orders`, `/reports/supplier-performance` marked Planned; sections renumbered.
  - **03 Developer onboarding** — `/suppliers` and `/purchase-orders` moved to the planned note; `supplier` and `purchaseOrder` collections marked planned; `product.supplierId` noted as reserved.
  - **05 Admin guide** — "Manage suppliers" and "Create purchase orders" removed from the RBAC matrix (planned note added); supplier/PO services marked planned in the infrastructure table.
  - **04 User help guide** — "Manage suppliers" walkthrough replaced with a "coming soon" note; the low-stock alert step no longer creates a purchase order; supplier FAQ added.
  - **06 Offline guide** — supplier-directory caching row flagged as supporting a planned feature; the duplicate-PO conflict scenario removed; reconciliation sessions added to offline-cached data.

- **Document set version** bumped to **v1.2** across all footers.

### Removed

- The "Suppliers and purchase orders" available-feature claims and the standalone "Inventory reconciliation (planned)" entries from the prior revision.

---

## [1.1.0] — June 2026 — Rename and backend reconciliation

### Changed

- **Product rename** — Renamed the product from StockSense to **Inventra** across all documents. Repository and Render deployment URLs are unchanged because the backend repo hasn't been renamed.

- **07 API reference** — Rewrote against the live backend. Base path changed from `/v1` to `/api`; base URL updated to the Render deployment on port 5000. Added the **Categories**, **Inventory** (`/inventory/stock-in`, `/stock-out`, `/update`, `/history`), and **Business profile** (`/profile`) groups. Updated **Sales**, **Alerts**, and **Reports** endpoints to match the backend. Simplified the error format to `{ message, error }`. Moved OTP, PIN, token refresh, reconciliation, and sync endpoints to a **Planned** section.

- **03 Developer onboarding guide** — Updated auth to JWT via `jsonwebtoken` with `JWT_SECRET` (removed RS256, refresh tokens, OTP, PIN as current features). Updated env vars to `PORT`, `MONGO_URI` / `LIVE_URL`, `JWT_SECRET`. Marked Redis and S3 as not implemented. Updated the collection list to the actual schema (`user`, `product`, `supplier`, `category`, `purchaseOrder`, `sale`, `saleItem`, `inventoryHistory`, `alert`, `businessProfile`). Noted the removal of `stockEntry`.

- **05 Admin guide and deployment runbook** — Updated production infrastructure to Render (port 5000). Reduced the required env-var checklist to the active set. Marked Redis, OTP, push, S3, and monitoring as planned. Updated deploy and rollback commands and the health-check URL. Marked reconciliation as a planned feature.

- **01 Release notes** — Added an Available/Planned status to every feature. Reclassified phone OTP, PIN login, account lockout, offline sync, reconciliation, and Redis as planned.

- **06 Offline architecture and sync guide** — Retitled as a design specification with a status banner. Marked the `/sync` and `/sync/pull` endpoints and conflict resolution as planned. Renamed identifiers (`inventra_sync`, `InventraDB`, `com.inventra.app`).

- **04 End-user help guide and FAQ** — Updated onboarding to email/password (the current auth). Added a "coming soon" note for SMS OTP and removed OTP-specific FAQ answers.

- **02 Demo video script** — Renamed throughout. Updated the onboarding scene to email-based sign-up. Flagged the offline scene as **[In development]** with production guidance.

- **README** — Rebuilt to a top-GitHub-project structure: centred header with badges, table of contents, features, tech stack, quick start, project status, and roadmap. Updated repo and live API links.

### Removed

- References to the `stockEntry` model (replaced by the inventory tracking system).

---

## [1.0.1] — June 2026 — Database correction

### Changed

- Updated database technology from PostgreSQL + Prisma to MongoDB (Atlas / local) + Mongoose across the developer, admin, offline, and API docs. Replaced Prisma migration commands with `npm run seed`, `pg_dump`/`pg_restore` with `mongodump`/`mongorestore`, and `DATABASE_URL` with `MONGODB_URI`.

---

## [1.0.0] — May 2026 — MVP launch

### Added

- Initial release of all seven documentation deliverables plus the README, produced for the v1.0 MVP launch.

### Product version

- Aligned to the product PRD v1.1 (Merged) — May 2026

---

## [Unreleased] — Upcoming

### Planned for Phase 2 (Months 2–4)

- Supplier management and purchase order guide (once `/suppliers` and `/purchase-orders` ship)
- Order-status and supplier-performance reporting (once suppliers/POs ship)
- Offline sync engine guide (once `/sync` ships)
- SMS/OTP integration guide (once the SMS gateway is selected)
- WhatsApp integration guide
- Barcode scanning guide
- Multi-store management guide
- iOS user guide
- Local language supplement — Yoruba, Hausa, Igbo

### Planned for Phase 3 (Months 7–12)

- AI demand forecasting guide
- WhatsApp chatbot user guide
- Supplier e-ordering integration guide
- Billing integration guide

---

## Version format guide

| Tag | Meaning |
|---|---|
| `Added` | New documents or major sections |
| `Changed` | Updates to existing content |
| `Fixed` | Corrections to errors or broken links |
| `Removed` | Documents or sections removed |
| `Deprecated` | Outdated content kept for reference |

---

*Inventra documentation changelog · Maintained by the Technical Writing Team*
