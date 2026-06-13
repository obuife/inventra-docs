# Inventra — Admin guide and deployment runbook

*Includes: RBAC permission matrix, reconciliation guide, and deployment procedures*
**Version 1.2 · June 2026 · Internal / Confidential**

---

# Part 1: Admin (owner) guide

---

## 1. Interpret your dashboard

Open your dashboard every morning and check these areas:

| Indicator | Colour | Action |
|---|---|---|
| Stock status — green | All products above reorder level | No action needed |
| Stock status — amber | Some products approaching threshold | Review low-stock list and consider reordering |
| Stock status — red | Products at or below reorder level | Act now — arrange a restock |
| Active alerts badge | Red number on the Alerts icon | Tap Alerts to review and action each one |
| Sync badge | Number of pending sync operations | Check your internet connection |

---

## 2. Role and permission matrix (RBAC)

| Feature / action | Admin (owner) | Manager | Attendant |
|---|:---:|:---:|:---:|
| Log in and access the app | ✓ | ✓ | ✓ |
| View dashboard — full revenue data | ✓ | ✓ | — |
| View dashboard — limited (no revenue) | ✓ | ✓ | ✓ |
| Add / edit products | ✓ | ✓ | — |
| Archive products | ✓ | ✓ | — |
| View product list | ✓ | ✓ | ✓ |
| Record stock in | ✓ | ✓ | — |
| Record manual stock out | ✓ | ✓ | — |
| Record sales | ✓ | ✓ | ✓ |
| Void a sale | ✓ | ✓ | — |
| View own sales history | ✓ | ✓ | ✓ |
| View all sales history | ✓ | ✓ | — |
| View per-attendant sales breakdown | ✓ | ✓ | — |
| Stock adjustments | ✓ | ✓ | — |
| Start / manage a stock count | ✓ | ✓ | — |
| Enter physical counts during a stock count | ✓ | ✓ | ✓ |
| View reconciliation discrepancies | ✓ | ✓ | — |
| Approve a reconciliation adjustment (updates stock) | ✓ | ✓ | — |
| Request a recount | ✓ | ✓ | — |
| Finalize a stock count | ✓ | ✓ | — |
| View audit log | ✓ | ✓ | — |
| View reports | ✓ | ✓ | — |
| Export reports | ✓ | Delegated by Admin | — |
| Manage users and roles | ✓ | — | — |
| Change system settings | ✓ | — | — |
| Set alert thresholds | ✓ | ✓ | — |
| Receive low-stock alerts | ✓ | ✓ | ✓ |
| Receive expiry alerts | ✓ | ✓ | — |

> **Note:** Manager export of reports requires explicit delegation from the Admin. Go to **Settings > Staff permissions** to grant or revoke it per Manager.

> **Note:** Reconciliation permissions reflect the shipped feature and its design. Attendants can enter physical counts, but only Admins and Managers can view discrepancies, approve adjustments, request recounts, and finalize a count.

> **Planned:** Supplier management and purchase orders are **not yet built**, so "Manage suppliers" and "Create purchase orders" are omitted from this matrix. They will return when the feature ships (Phase 2).

---

## 3. Inventory reconciliation

Reconciliation (the **Stock count** screen) lets you confirm that what Inventra says you have matches what's physically on your shelves, value any difference in Naira, and resolve it.

### 3.1 Why run a reconciliation

Reconciliation answers: *Does what Inventra says I have match what's actually on my shelf?*

Run a reconciliation when:

- You suspect theft or pilferage
- A large delivery arrives and you want to verify counts
- Your stockout frequency is higher than expected
- A monthly or quarterly audit is due
- A new staff member takes over stock management

### 3.2 Read the Stock count screen

The screen header shows summary cards:

- **Products counted** — progress through the session (for example, *50/120*, *44.6% complete*)
- **Pending Approval** — discrepancies awaiting your decision
- **Discrepancies** — total items where the physical count differs from the system count
- **Variance value** — the net financial impact in Naira, shown as potential loss

Each row in the list shows the product and SKU, who counted it and when, the **value impact**, the **system count**, the **physical count**, the **difference**, a **Result** (**Match**, **Loss**, or **Surplus**), and a **Status** (**Approved**, **Pending**, or **Recount**).

> **Tip:** Use the filter to narrow the list by custom date range and by status (All, Approved, Pending, Recount).

### 3.3 Run a reconciliation

1. Go to **Reconciliation** (or **More > Reconciliation**).
2. Tap **New Count**. A session is created and timestamped with your name.
3. The list shows each active product with its **system count**.
4. Count each item physically and enter the **physical count**.
5. The **difference** and **Result** update live; **Loss** means stock is missing, **Surplus** means there's more than expected.
6. The **value impact** column shows the financial effect of each difference in Naira.

### 3.4 Resolve a discrepancy

1. Tap an item to open **Product Discrepancy Details**. It shows the category, unit, cost price, selling price, system count, physical count, difference, value impact, who counted it, when, and a free-text **reason**.
2. Enter or review the reason (for example, "Broken bottles found on the shelf").
3. Choose one:
   - **Approve Adjustment** — updates system stock to the physical count and logs the adjustment with your name. A confirmation prompt appears first, warning that stock records will be updated.
   - **Request Recount** — sets the item's status to **Recount** without changing stock, so it can be counted again.

> **Caution:** Approved adjustments are permanent and write to your stock records. To correct a mistake, run a new count or create a stock adjustment with a correction reason.

---

## 4. Manage users

### 4.1 Invite a staff member

1. Go to **Settings > Manage staff**.
2. Tap **Invite user**.
3. Enter the staff member's phone number.
4. Select their role: Manager or Attendant.
5. Tap **Send invite**.

### 4.2 Change a staff member's role

1. Go to **Settings > Manage staff**.
2. Tap the staff member's name.
3. Tap **Change role** and select the new role.
4. Tap **Confirm**.

> **Caution:** Changing a role ends the staff member's current session. They must log in again.

### 4.3 Deactivate a staff member

1. Go to **Settings > Manage staff**.
2. Tap the staff member's name.
3. Tap **Deactivate account** and confirm.

Access is revoked immediately. Past sales and actions stay in the audit log.

---

## 5. System settings

### 5.1 Configure alerts

Go to **Settings > Notifications** to toggle:

- **Low-stock alerts** — globally or per product
- **Expiry alerts** — at 30, 14, and 7 days before expiry
- **Sync alerts** — when a sync operation fails

### 5.2 Subscription plans

| Plan | Price | Limits |
|---|---|---|
| Free | ₦0 | Up to 50 products, 3 users, basic reports |
| Starter | ₦500/month | Up to 200 products, 10 users, full reports, PDF export |
| Pro | ₦2,000/month | Unlimited products and users, all reports, priority support |

---

# Part 2: Deployment runbook

---

## 6. Production environment

| Component | Provider | Notes |
|---|---|---|
| API server | Render | Node.js 18 LTS. Deployed at `stocksense-backend-devlopment.onrender.com`. Runs on port 5000. |
| Database | MongoDB Atlas (cloud) or MongoDB 6.0 (local) | Connection via `MONGO_URI` or `LIVE_URL`. |
| File storage | Planned — AWS S3 or Cloudflare R2 | Not yet implemented. |
| CDN / frontend | Vercel or Netlify | Next.js PWA. |
| Monitoring | Planned — Sentry + Uptime Robot | Not yet wired up. |
| CI/CD | GitHub Actions | Automated build, test, and deploy. |

> **Note:** Redis (cache/queues), SMS/OTP providers, push notifications, and supplier/purchase-order services are in the target architecture but **aren't implemented** in the v1.0 backend.

---

## 7. Pre-deployment checklist

### Code and build

- [ ] All pull requests merged and approved by Engineering Lead
- [ ] All CI checks passing (tests, lint, build)
- [ ] Mongoose schema changes reviewed and approved
- [ ] No open P0 or P1 bugs
- [ ] APK size verified: under 30 MB

### Environment variables

Confirm these are set in production. **Never commit them to source control.**

- [ ] `PORT` (5000)
- [ ] `MONGO_URI` or `LIVE_URL`
- [ ] `JWT_SECRET`

> **Note:** Additional variables (`REDIS_URL`, `OTP_PROVIDER_KEY`, `S3_*`, `SENTRY_DSN`) belong to planned services and aren't required by the current build.

### Security

- [ ] SSL certificate valid for the production domain
- [ ] CORS restricted to the production frontend domain
- [ ] Input validation middleware confirmed on all routes
- [ ] RBAC enforced on reconciliation approve/finalize routes

### Database

- [ ] Production database backup taken before deploying
- [ ] Seed data verified on staging
- [ ] MongoDB indexes created on `productId`, `businessId`, `createdAt`

---

## 8. Deployment steps

> **Caution:** Always take a full database backup before deploying to production.

### 8.1 Deploy the backend

```bash
# 1. Back up the database
mongodump --uri="$MONGO_URI" --out=./backup_$(date +%Y%m%d_%H%M)

# 2. Pull the release branch
git checkout release/v1.0
git pull origin release/v1.0

# 3. Seed initial data (if required)
npm run seed

# 4. Build
npm run build

# 5. Deploy (Render auto-deploys on push to the release branch,
#    or trigger a manual deploy from the Render dashboard)

# 6. Verify the health check
curl https://stocksense-backend-devlopment.onrender.com/api/health
```

### 8.2 Deploy the frontend

```bash
npm run build
vercel --prod
```

### 8.3 Release the Android APK

```bash
# Build the release APK
./gradlew assembleRelease

# Sign the APK
jarsigner -verbose -sigalg SHA256withRSA -digestalg SHA-256 \
  -keystore inventra.keystore app-release-unsigned.apk alias_name

# Align and verify
zipalign -v 4 app-release-unsigned.apk inventra-v1.0.apk
```

Verify the APK is under 30 MB before distributing.

---

## 9. Post-deployment smoke tests

- [ ] Register a new business account — verify `POST /api/auth/register` returns `201`
- [ ] Log in — verify `POST /api/auth/login` returns a token
- [ ] Add a test product and confirm it appears in inventory
- [ ] Record a test sale and verify stock is deducted
- [ ] Record a stock-in and confirm `GET /api/inventory/history` reflects it
- [ ] Start a stock count, enter a physical count that differs from the system count, approve the adjustment, and confirm stock updates and the adjustment is logged
- [ ] Invite a staff member as Attendant and verify restricted access
- [ ] Run a low-stock check (`POST /api/alerts/check`) and confirm an alert is created

---

## 10. Incident response

### Severity levels

| Severity | Definition | Example | Response target |
|---|---|---|---|
| P1 — Critical | System down or data at risk | API unreachable, DB corruption | < 2 hours MTTR |
| P2 — High | Major feature unavailable | Sales can't be recorded | < 4 hours |
| P3 — Medium | Feature degraded | Reports generating slowly | < 24 hours |
| P4 — Low | Minor issue | UI misalignment | Next sprint |

### Rollback procedure

> **Caution:** Only roll back if a P1 can't be resolved within 30 minutes of deployment.

```bash
# 1. Revert via the Render dashboard (Rollback to a previous deploy)

# 2. If the database was seeded or modified, restore from backup:
mongorestore --uri="$MONGO_URI" ./backup_YYYYMMDD_HHMM

# 3. Verify the health check returns 200
# 4. Run smoke tests (Section 9)
# 5. Document the incident: timestamp, root cause, steps taken, resolution
```

---

## 11. Maintenance schedule

| Task | Schedule | Details |
|---|---|---|
| Database backup | Every 6 hours | Automated. 30-day retention. Verify monthly. |
| Maintenance window | Sunday 02:00–04:00 WAT | Notify users 24 hours in advance. |
| SSL certificate renewal | 60 days before expiry | Auto-renew via Let's Encrypt. |
| Dependency security audit | Monthly | Run `npm audit`. Patch critical vulnerabilities within 48 hours. |
| Log rotation | Daily | App logs: 90 days. Error logs: 1 year. |

---

*Inventra · Admin guide and deployment runbook · v1.2 · June 2026 · Internal / Confidential*
