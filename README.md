<div align="center">

# Inventra

### Know your stock. Run your business.

**A mobile-first, offline-capable inventory and sales management system for SMEs across Nigeria and Sub-Saharan Africa.**

[![Status](https://img.shields.io/badge/status-MVP%20v1.0-success)](#)
[![Platform](https://img.shields.io/badge/platform-Android%208.0%2B%20%C2%B7%20PWA-blue)](#)
[![Backend](https://img.shields.io/badge/backend-Node.js%20%2B%20Express-339933)](#)
[![Database](https://img.shields.io/badge/database-MongoDB-47A248)](#)
[![License](https://img.shields.io/badge/license-Internal-lightgrey)](#)

[Documentation](#documentation) · [Quick start](#quick-start) · [API reference](./07_complete-api-reference.md) · [Contributing](#contributing)

</div>

---

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

- Track inventory in real time from any Android smartphone
- Record sales in a few taps
- Reconcile physical stock against system records and act on the variance
- Get alerts before stock runs out or medicines expire
- See exactly what every staff member has sold
- Generate and export sales and inventory reports

It's built for users with basic smartphone skills and unreliable internet. If you can use WhatsApp, you can use Inventra.

> **Note:** Inventra was previously named StockSense. The backend repository and Render deployment URL still use the original `stocksense` slug because they haven't been renamed.

| Field | Detail |
|---|---|
| Product name | Inventra |
| Version | 1.0 — MVP launch |
| Target market | Nigerian SMEs — retail shops, pharmacies, wholesalers, mini-marts |
| Primary platform | Android 8.0+ · Progressive Web App (PWA) |
| Backend repository | [`Triaaz/StockSense-Backend-Devlopment`](https://github.com/Triaaz/StockSense-Backend-Devlopment) |
| Live API | [`stocksense-backend-devlopment.onrender.com/api`](https://stocksense-backend-devlopment.onrender.com/) |

---

## Features

- **Inventory management** — products, categories, stock-in/stock-out, and an append-only movement history
- **Sales** — quick single and multi-item sales, attendant attribution, void with reason, daily summaries
- **Inventory reconciliation** — stock-count sessions that compare system count against a physical count, value the variance in Naira, and let the owner approve the adjustment or request a recount
- **Alerts** — low-stock and expiry checks, with read/summary/delete management
- **Reports and dashboard** — dashboard summary and low-stock report
- **Role-based access control** — Admin, Manager, and Attendant roles enforced on every endpoint

See [Project status](#project-status) for what's shipped versus planned.

---

## Tech stack

| Layer | Technology |
|---|---|
| Client | React + Next.js PWA · Android (Kotlin or Flutter) |
| API | Node.js + Express.js |
| Database | MongoDB (Atlas / local) + Mongoose |
| Auth | JWT (`jsonwebtoken`, `JWT_SECRET`) |
| Local storage | SQLite via Room (Android) · IndexedDB via Dexie.js (PWA) |
| Hosting | Render (API) · Vercel/Netlify (PWA) |

---

## Documentation

| # | Document | Audience |
|---|---|---|
| 1 | [Release notes v1.0](./01_release-notes-v1.0.md) | All teams |
| 2 | [Demo video script](./02_demo-video-script.md) | Product · Design · Video |
| 3 | [Developer onboarding guide](./03_developer-onboarding-guide.md) | All developers |
| 4 | [End-user help guide and FAQ](./04_user-help-guide-faq.md) | End users · Pilot participants |
| 5 | [Admin guide and deployment runbook](./05_admin-guide-deployment-runbook.md) | Admins · DevOps |
| 6 | [Offline architecture and sync guide](./06_offline-architecture-sync-guide.md) | Mobile · Backend · QA |
| 7 | [Complete API reference](./07_complete-api-reference.md) | Backend · Mobile · QA |

New to the project? Start with the [Developer onboarding guide](./03_developer-onboarding-guide.md).

---

## Quick start

Get the backend running locally in five steps.

### Prerequisites

- Node.js 18 LTS or higher
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

```
PORT=5000
MONGO_URI=mongodb://localhost:27017/inventra
JWT_SECRET=your_secret_here
```

```bash
# 4. Seed the database (optional)
npm run seed

# 5. Start the server
npm run dev
```

The API is available at `http://localhost:5000/api`. For the full walkthrough, see the [Developer onboarding guide](./03_developer-onboarding-guide.md).

---

## Project status

| Area | Status |
|---|---|
| Auth (email/password + JWT) | ✅ Available |
| Products, categories, inventory | ✅ Available |
| Sales (record, void, history, summary) | ✅ Available |
| Inventory reconciliation (stock counts, variance, approve/recount) | ✅ Available |
| Alerts (low-stock, expiry) | ✅ Available |
| Reports and dashboard (summary, low-stock) | ✅ Available |
| Business profile | ✅ Available |
| Suppliers and purchase orders | 🚧 Planned |
| Order-status and supplier-performance reports | 🚧 Planned |
| Phone OTP / SMS provider | 🚧 Planned |
| Redis caching and queues | 🚧 Planned |
| Offline sync engine (`/sync`) | 🚧 Planned |

---

## Roadmap

**Phase 2** — Supplier management and purchase orders, order-status and supplier-performance reporting, offline sync engine, phone OTP, Redis, WhatsApp integration, barcode scanning, multi-store, native iOS, local-language UI.

**Phase 3** — AI demand forecasting, WhatsApp chatbot, supplier e-ordering, billing integrations.

---

## Contributing

Spotted an inaccuracy — a wrong endpoint, an outdated permission, or a feature description that doesn't match what was built?

1. Open an **issue** describing the change and why, or
2. Contact the Technical Writing team via the team channel.

Documentation follows a docs-as-code workflow: every doc is GitHub Flavored Markdown, version-controlled in this repo, and changes are tracked in the [CHANGELOG](./CHANGELOG.md).

---

## Team

Produced by the **Inventra Technical Writing Team** as a capstone deliverable for the Techcrush Technical Writing Bootcamp, with the Inventra product and engineering teams.

| Role | Responsibility |
|---|---|
| Technical Writers | All 7 documents — research, structure, writing, review |
| Engineering Lead | Technical accuracy review |
| Product Manager | Scope and PRD alignment |
| Design Lead | UX copy and wireframe review |

---

<div align="center">

*Inventra · Technical documentation · v1.2 · June 2026*

</div>
