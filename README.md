# Inventra

### Know your stock. Run your business.

**A mobile-first inventory and sales management system for small and medium-sized enterprises (SMEs) across Nigeria and Sub-Saharan Africa.**

![Status](https://img.shields.io/badge/status-v1.0-success)
![Platform](https://img.shields.io/badge/platform-Android%20%C2%B7%20Web-blue)
![Backend](https://img.shields.io/badge/backend-Node.js%20%2B%20Express-339933)
![Database](https://img.shields.io/badge/database-MongoDB-47A248)
![License](https://img.shields.io/badge/license-Internal-lightgrey)

[Documentation](#documentation) · [Quick start](#quick-start) · [API reference](07_complete-api-reference.md) · [Contributing](#contributing)

---

> **Note:** Inventra was previously named StockSense. The backend repository and the Render deployment URL still use the original `stocksense` slug because they have not been renamed.

## Table of contents

- [Overview](#overview)
- [Features](#features)
- [Tech stack](#tech-stack)
- [Documentation](#documentation)
- [Quick start](#quick-start)
- [Project status](#project-status)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [Team](#team)

---

## Overview

Inventra helps Nigerian shop owners, pharmacists, and wholesalers:

- Track inventory in real time from a smartphone
- Record sales in a few taps
- Manage suppliers and purchase orders
- Reconcile physical stock against system records and act on the variance
- Get alerts before stock runs out or medicines expire
- See exactly what every staff member has sold
- Generate and export sales and inventory reports

It is built for users with basic smartphone skills and unreliable internet. If you can use WhatsApp, you can use Inventra.

| Field | Detail |
| --- | --- |
| Product name | Inventra |
| Version | 1.0 |
| Target market | Nigerian SMEs: retail shops, pharmacies, wholesalers, mini-marts |
| Client | React Native mobile application (in development) and a web interface; user experience designed in Figma |
| Backend repository | [`Triaaz/StockSense-Backend-Devlopment`](https://github.com/Triaaz/StockSense-Backend-Devlopment) |
| Live API base URL | `stocksense-backend-devlopment.onrender.com/api` |

> The live API is served under the base path shown above. Individual endpoints sit beneath it (for example, `/api/auth/login`), so the base path on its own is not a browsable web page.

---

## Features

- **Authentication and access** — email and password sign-in, phone sign-up with a one-time password (OTP), a six-digit personal identification number (PIN) for shared devices, JSON Web Token (JWT) sessions, bcrypt password hashing, and three roles (Admin, Manager, Attendant)
- **Inventory management** — products, categories, stock-in and stock-out, adjustments, and an add-only movement history
- **Inventory reconciliation** — stock-count sessions that compare the system count against a physical count, value the variance in Naira, and let the owner approve the adjustment or request a recount
- **Sales** — single and multi-item sales, attendant attribution, void with a reason, and a daily summary
- **Suppliers and purchase orders** — supplier directory and purchase-order management
- **Alerts** — low-stock and expiry checks, with read, summary, and delete management
- **Reports and dashboard** — dashboard summary, low-stock report, supplier-performance report, purchase-order status, and recent orders
- **Role-based access control (RBAC)** — Admin, Manager, and Attendant roles enforced on every endpoint

See [Project status](#project-status) for what is built versus planned.

---

## Tech stack

| Layer | Technology |
| --- | --- |
| Client | React Native mobile application (in development); web interface; designs in Figma |
| Application programming interface (API) | Node.js and Express.js (representational state transfer, or REST) |
| Database | MongoDB (Atlas or local) with Mongoose object data modelling (ODM) |
| Authentication | JWT (`jsonwebtoken`) with bcrypt password hashing |
| Hosting | Render (API); MongoDB Atlas (database) |
| Tooling | Git and GitHub; Postman; Visual Studio (VS) Code |

---

## Documentation

| No. | Document | Audience |
| --- | --- | --- |
| 1 | [Release notes v1.0](01_release-notes-v1.0.md) | All teams |
| 2 | [Demo video script](02_demo-video-script.md) | Product, design, video |
| 3 | [Developer onboarding guide](03_developer-onboarding-guide.md) | Developers |
| 4 | [End-user help guide and FAQ](04_user-help-guide-faq.md) | End users and pilot participants |
| 5 | [Admin guide and deployment runbook](05_admin-guide-deployment-runbook.md) | Admins and operations |
| 6 | [Offline architecture and sync guide](06_offline-architecture-sync-guide.md) | Mobile and backend developers |
| 7 | [Complete API reference](07_complete-api-reference.md) | Backend and mobile developers |

New to the project? Start with the [Developer onboarding guide](03_developer-onboarding-guide.md).

---

## Quick start

Get the backend running locally in five steps.

### Prerequisites

- Node.js 18 long-term support (LTS) or higher
- MongoDB 6.0+ (local) or a MongoDB Atlas account
- Git 2.40+

### Steps

```bash
# 1. Clone the backend repository
git clone https://github.com/Triaaz/StockSense-Backend-Devlopment.git
cd StockSense-Backend-Devlopment

# 2. Install dependencies
npm install

# 3. Configure environment variables
cp .env.example .env
```

Set these values in `.env`:

```bash
PORT=5000
MONGO_URI=mongodb://localhost:27017/inventra
JWT_SECRET=your_secret_here
```

```bash
# 4. Seed the database (optional)
npm run seed

# 5. Start the development server
npm run dev
```

The API is available at `http://localhost:5000/api`. For the full walkthrough, see the [Developer onboarding guide](03_developer-onboarding-guide.md).

---

## Project status

| Area | Status |
| --- | --- |
| Authentication (email/password, JWT, bcrypt) | Available |
| Phone OTP and six-digit PIN login | Available |
| Products, categories, inventory | Available |
| Inventory reconciliation (stock counts, variance, approve/recount) | Available |
| Sales (record, void, history, daily summary) | Available |
| Suppliers and purchase orders | Available |
| Alerts (low-stock, expiry) | Available |
| Reports and dashboard (summary, low-stock, supplier performance, purchase-order status, recent orders) | Available |
| Business profile | Available |
| Mobile application (React Native) | In development |
| Offline background sync engine (`/sync`) | Planned |
| Redis caching and queues | Planned |

---

## Roadmap

**Phase 2** — Offline background sync engine, Redis caching, WhatsApp integration, barcode scanning, multi-store support, and a local-language interface.

**Phase 3** — Artificial-intelligence (AI) demand forecasting, a WhatsApp chatbot, supplier e-ordering, and billing integrations.

---

## Contributing

Spotted an inaccuracy, such as a wrong endpoint, an outdated permission, or a feature description that does not match what was built?

1. Open an **issue** describing the change and why, or
2. Contact the technical writing team through the team channel.

Documentation follows a docs-as-code workflow: every document is GitHub Flavored Markdown, version-controlled in this repository, and changes are tracked in the [CHANGELOG](CHANGELOG.md).

---

## Team

Produced by the Inventra capstone team (Techcrush Cohort 6, Group 14), working across six tracks.

| Track | Responsibility |
| --- | --- |
| Product Management | Direction, scope, backlog, prioritisation, and coordination |
| Product Design | Product experience and screen flows in Figma |
| Frontend | Onboarding, landing page, and the main screen flows |
| Mobile Development | React Native application and its modules |
| Backend | Node.js and Express API, MongoDB data layer, deployment, and testing |
| Technical Writers | This documentation set, kept in step with the product as built |

---

*Inventra · Technical documentation · v2.0 · June 2026*
