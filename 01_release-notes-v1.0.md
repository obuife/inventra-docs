# Inventra — Release notes v1.0

**Product:** Inventra, an inventory and sales management system for small and medium-sized enterprises (SMEs)
**Version:** 1.0
**Release date:** June 2026
**Status:** Available
**Client:** React Native mobile application (in development) and a web interface
**Prepared by:** Inventra product and technical writing teams

## 1. Overview

Inventra v1.0 is an inventory and sales management platform for SMEs across Nigeria and Sub-Saharan Africa. It lets business owners, managers, and attendants manage inventory, record sales, manage suppliers and purchase orders, reconcile physical stock against system records, and generate reports.

Each feature below is tagged **Available** (built in the current release) or **Planned** (designed but not yet implemented). The tags reflect the deployed backend, the Postman test runs, and the product designs.

## 2. What is new in v1.0

### 2.1 Authentication and access control

| ID | Feature | Description | Status |
| --- | --- | --- | --- |
| AUTH01 | Email and password login | Register and log in with email and password. Login returns a JSON Web Token (JWT). | Available |
| AUTH02 | JWT sessions | Sessions are issued as JWTs through the `jsonwebtoken` library. | Available |
| AUTH03 | Role-based access control (RBAC) | Three roles: Admin, Manager, Attendant. Each controls the screens and actions available. | Available |
| AUTH04 | Staff invitation | Admins invite team members. | Available |
| AUTH05 | Phone one-time password (OTP) | Register and verify with a phone number and a one-time password. | Available |
| AUTH06 | Six-digit personal identification number (PIN) login | Quick login on shared counter devices. | Available |

### 2.2 Inventory management

| ID | Feature | Description | Status |
| --- | --- | --- | --- |
| INV01 | Product catalogue | Add and edit products with name, category, cost price, selling price, opening stock, and reorder threshold. | Available |
| INV02 | Real-time stock tracking | Every sale and stock movement updates the current stock. | Available |
| INV03 | Stock-in and stock-out | Record arrivals and manual removals through the inventory endpoints, written to the movement history. | Available |
| INV04 | Categories | Group products by category. | Available |
| INV05 | Soft delete | Products are archived, not permanently removed. | Available |
| INV06 | Expiry date tracking | Optional expiry date per product, useful for pharmacy stock. | Available |

### 2.3 Sales management

| ID | Feature | Description | Status |
| --- | --- | --- | --- |
| SALE01 | Quick sale recording | Record a single-item sale in seconds. | Available |
| SALE02 | Multi-item transactions | Multiple products in one sale. | Available |
| SALE03 | Attendant attribution | Every sale is tagged with the attendant who recorded it. | Available |
| SALE04 | Sale void | Admin and Manager can void a sale with a reason; stock is reversed. | Available |
| SALE05 | Daily summary | Daily sales summary. | Available |

### 2.4 Suppliers and purchase orders

| ID | Feature | Description | Status |
| --- | --- | --- | --- |
| SUP01 | Supplier directory | Maintain suppliers with name, phone, and contact details. | Available |
| SUP02 | Purchase orders | Create, view, update the status of, and delete purchase orders. | Available |
| SUP03 | Supplier performance | Supplier-performance reporting. | Available |

### 2.5 Inventory reconciliation

Reconciliation lets the owner run a stock count that compares the system stock against a physical count, values the difference in Naira, and resolves each discrepancy.

| ID | Feature | Description | Status |
| --- | --- | --- | --- |
| REC01 | Stock count sessions | Start a count and track progress (products counted out of total, percentage complete). | Available |
| REC02 | Discrepancy detection | Each item shows system count, physical count, and the difference, classified as Match, Loss, or Surplus. | Available |
| REC03 | Variance valuation | Per-item value impact and a session-level variance value, shown in Naira as potential loss. | Available |
| REC04 | Approve adjustment / request recount | Approve a discrepancy to update stock (after a confirmation prompt), or send the item back for a recount. | Available |
| REC05 | Discrepancy details and reason | A detail view shows counts, value impact, who counted, when, and a free-text reason. | Available |
| REC06 | Filter and review | Filter the count list by date range and by status (All, Approved, Pending, Recount). | Available |

### 2.6 Alerts

| ID | Feature | Description | Status |
| --- | --- | --- | --- |
| ALERT01 | Low-stock alerts | Fire when stock reaches or falls below the reorder threshold. | Available |
| ALERT02 | Expiry alerts | Fire ahead of a product's expiry date. | Available |
| ALERT03 | Alert management | Mark alerts as read, view summaries, and delete alerts. | Available |

### 2.7 Reports and dashboard

| ID | Feature | Description | Status |
| --- | --- | --- | --- |
| DASH01 | Dashboard summary | Sales totals, stock status, and recent activity. | Available |
| DASH02 | Low-stock report | Products below threshold. | Available |
| DASH03 | Purchase-order status and recent orders | Purchase-order status counts and recent orders. | Available |
| DASH04 | Supplier-performance report | Performance metrics per supplier. | Available |

### 2.8 Business profile

| ID | Feature | Description | Status |
| --- | --- | --- | --- |
| PROF01 | Business profile | View and update business name, contact details, currency, timezone, and low-stock threshold. | Available |

### 2.9 Planned for a later release

| ID | Feature | Status |
| --- | --- | --- |
| OFFLINE01 | Offline background sync engine and server sync endpoints | Planned |
| INT01 | Redis caching and queues | Planned |
| INT02 | WhatsApp integration | Planned |
| INT03 | Barcode scanning | Planned |

## 3. Security highlights

- Production API traffic is encrypted with Transport Layer Security (TLS) 1.2 or higher.
- Sessions use JWTs issued through the `jsonwebtoken` library.
- Passwords are hashed with bcrypt before storage.
- Input validation runs on all endpoints.
- The movement history is treated as an add-only log.
- Reconciliation adjustments that update stock are logged with the approving user and require explicit confirmation.

## 4. Known limitations

| Feature | Reason deferred | Target phase |
| --- | --- | --- |
| Offline background sync engine | Server sync endpoints not yet built | Phase 2 |
| Redis caching | Not required for current load | Phase 2 |
| WhatsApp integration | Valuable but not blocking | Phase 2 |
| Barcode scanning | Manual entry sufficient for now | Phase 2 |
| Multi-store / multi-location | Not a majority use case yet | Phase 2 |
| AI demand forecasting | No training data yet | Phase 3 |

## 5. Support

For questions or feedback, contact the Inventra team through the in-app help centre or the onboarding packet provided at registration.

---

*Inventra · Release notes · v2.0 · June 2026*
