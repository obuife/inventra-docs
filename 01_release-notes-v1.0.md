# StockSense — Release Notes v1.0

**Product:** StockSense — SME Inventory & Sales Management System
**Version:** 1.0 — MVP Launch Release
**Release Date:** May 2026
**Status:** Generally Available (GA)
**Platform:** Android 8.0+ · Progressive Web App (PWA)
**Prepared By:** StockSense Product & Technical Writing Team

---

## 1. Overview

StockSense v1.0 is the inaugural release of the StockSense MVP — a mobile-first, offline-capable inventory and sales management platform built specifically for small and medium-sized enterprises (SMEs) across Nigeria and Sub-Saharan Africa.

This release delivers the full core feature set required for business owners, store managers, and sales attendants to manage inventory, record sales, manage suppliers, reconcile stock, and generate reports — all from a smartphone, with or without an internet connection.

---

## 2. What's New in v1.0

### 2.1 User Authentication & Access Control

| Feature ID | Feature | Description |
|---|---|---|
| AUTH-01 | Phone OTP Registration | Business owners register using a Nigerian phone number and receive an SMS OTP for verification. No email address required. |
| AUTH-02 | Email/Password Login | Secondary login option for users who prefer email-based authentication. |
| AUTH-03 | 4-Digit PIN Login | Sales attendants can log in quickly with a 4-digit PIN — ideal for shared devices during busy periods. |
| AUTH-04 | Role-Based Access Control | Three roles defined: Admin (Owner), Manager, and Attendant. Each role controls what screens and actions are accessible. |
| AUTH-05 | Staff Invitation System | Admins invite team members via phone number or a shareable business invite code. |
| AUTH-06 | Account Lockout | Accounts lock for 15 minutes after 5 consecutive failed login attempts. A visible countdown is displayed. |

### 2.2 Inventory Management

| Feature ID | Feature | Description |
|---|---|---|
| INV-01 | Product Catalogue | Add, edit, and manage products with name, category, cost price, selling price, opening stock, and reorder threshold. |
| INV-02 | Real-Time Stock Tracking | Every sale and stock addition updates current stock automatically. Formula: `current_stock = opening_stock + stock_in − sales − manual_out` |
| INV-03 | Stock In / Stock Out | Record new stock arrivals and manual removals (wastage, damage, returns) with mandatory reason logs. |
| INV-04 | Soft Delete | Products are archived, not permanently deleted. All transaction history is preserved. |
| INV-05 | Product Search & Filter | Live search as the user types. Filter by category or low-stock status. |
| INV-06 | Expiry Date Tracking | Optional expiry date field on each product — required for pharmacy use cases. |

### 2.3 Sales Management

| Feature ID | Feature | Description |
|---|---|---|
| SALE-01 | Quick Sale Recording | Attendants can record a single-item sale in under 30 seconds, 3 taps maximum. |
| SALE-02 | Multi-Item Transactions | Cart-style view supports multiple products in a single sale transaction. |
| SALE-03 | Attendant Attribution | Every sale is permanently tagged with the logged-in attendant's name and user ID. |
| SALE-04 | Sale Void | Admin and Manager can void incorrect sales with a mandatory reason. Original sale record is preserved; stock is reversed. |
| SALE-05 | Offline Sales | Sales recorded without internet are queued locally and automatically sync when connectivity returns. |
| SALE-06 | Payment Method Capture | Optional: Cash, Transfer, POS, or Other. |

### 2.4 Alerts System

| Feature ID | Feature | Description |
|---|---|---|
| ALERT-01 | Low-Stock Alerts | Fires when product stock reaches or falls below the configured reorder threshold. |
| ALERT-02 | Expiry Alerts | Fires at 30, 14, and 7 days before product expiry date. Each fires independently — even if an earlier alert was dismissed. |
| ALERT-03 | Alert Actions | Each alert offers: Create Purchase Order, View Product, or Dismiss. |
| ALERT-04 | Expired Product Escalation | Products past their expiry date immediately escalate to URGENT status. |

### 2.5 Supplier Management

| Feature ID | Feature | Description |
|---|---|---|
| SUP-01 | Supplier Directory | Maintain a list of suppliers with name, phone, email, contact person, and payment terms. |
| SUP-02 | Product-Supplier Linking | Each product can be linked to a primary and a backup supplier. |
| SUP-03 | Purchase Order Creation | Generate purchase orders from a low-stock alert or manually. POs pre-fill product and supplier details. |
| SUP-04 | PO Status Tracking | Track purchase orders through: Draft → Sent → Confirmed → Delivered. |
| SUP-05 | Delivery Confirmation | Confirming delivery on a PO automatically increments the product's stock. |

### 2.6 Inventory Reconciliation

| Feature ID | Feature | Description |
|---|---|---|
| REC-01 | Stock Count Sessions | Owner or Manager initiates a reconciliation session at any time. |
| REC-02 | Variance Calculation | System displays: Expected Qty vs. Actual Qty vs. Variance (monetary value in NGN). |
| REC-03 | Accept or Flag | Owner can accept variance (adjusts system stock) or flag individual items for investigation. |
| REC-04 | Offline Reconciliation | Full reconciliation works using locally cached data — no internet required. |

### 2.7 Offline Mode

| Feature ID | Feature | Description |
|---|---|---|
| OFFLINE-01 | Full Offline Operation | All core features — sales, stock updates, product browsing — work with no internet connection. |
| OFFLINE-02 | Sync Queue Indicator | A visible badge shows the number of operations pending sync. |
| OFFLINE-03 | Auto-Sync on Reconnect | Background sync triggers automatically when connectivity is restored. No user action needed. |
| OFFLINE-04 | Conflict Notification | If sync creates a conflict, the owner is notified for manual review. |

### 2.8 Dashboard & Reports

| Feature ID | Feature | Description |
|---|---|---|
| DASH-01 | Business Dashboard | Displays: today's sales total, stock status summary, low-stock alert list, recent sales feed, and quick-action buttons. |
| DASH-02 | Personalised Greeting | Dashboard shows a greeting with the business name and a count of active alerts. |
| DASH-03 | Sales Report | Daily, weekly, or custom date range. Exportable to PDF and CSV. |
| DASH-04 | Inventory Value Report | Current stock value in NGN across all products. |
| DASH-05 | Attendant Performance Report | Sales volume and transaction count per staff member. |
| DASH-06 | Audit Log Export | Full immutable transaction history export. Admin access only. |

---

## 3. Performance Benchmarks

| Metric | Target |
|---|---|
| App cold start | Under 3 seconds on mid-range Android |
| Dashboard load — online | Under 2 seconds on 3G |
| Dashboard load — offline | Under 0.5 seconds from local storage |
| Sale recording (end-to-end) | Under 30 seconds from open to confirmation |
| Product search results | Under 500ms |
| API response time (P95) | Under 500ms |
| Sync — 100 pending operations | Under 30 seconds on 3G |
| APK download size | Maximum 30MB |
| Crash-free session rate | Above 99% |

---

## 4. Known Limitations & Out-of-Scope Items

| Feature | Reason Deferred | Target Phase |
|---|---|---|
| WhatsApp Integration | Valuable but not blocking launch | Phase 2 |
| Barcode Scanning | Manual entry sufficient at launch | Phase 2 |
| Multi-Store / Multi-Location | Not majority MVP use case | Phase 2 |
| Native iOS Application | Android-first strategy | Phase 2 |
| Local Language UI (Yoruba, Igbo, Hausa) | English MVP; local languages follow | Phase 2 |
| AI Demand Forecasting | No training data at launch | Phase 3 |
| Full Accounting Integration | Separate product territory | Future |

---

## 5. Security Highlights

- All API traffic encrypted via TLS 1.2+ (TLS 1.3 preferred)
- JWT access tokens signed with RS256 — 24-hour expiry; 30-day refresh token rotation
- Passwords and PINs hashed with bcrypt (minimum cost factor: 12)
- Account lockout after 5 failed login attempts (15-minute cooldown)
- API rate limiting: 100 requests per minute per authenticated user
- Input validation and SQL injection prevention on all endpoints
- Sensitive fields encrypted at rest with AES-256
- Audit log is immutable — no UPDATE or DELETE ever permitted on the log table
- NDPR-compliant data handling — owner account deletion with 30-day recovery window

---

## 6. Upgrade Instructions

### Android App
1. Download the StockSense APK from the official distribution link.
2. Install on any Android 8.0 (Oreo) or later device.
3. Grant required permissions: SMS (OTP), Storage (PDF export), Network access.
4. Launch the app and follow the onboarding flow.

### Progressive Web App (PWA)
1. Open the StockSense URL in Chrome on Android or Chrome/Firefox on desktop.
2. Tap **"Add to Home Screen"** when prompted.
3. The app will cache all required assets for offline use on first load.

---

## 7. Support

For questions or feedback during the v1.0 pilot period, contact the StockSense support team via the in-app Help Centre or the pilot user onboarding packet provided at registration.

---

*StockSense · Release Notes v1.0 · May 2026 · Internal / Confidential*
