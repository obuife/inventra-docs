# Inventra — Admin guide and deployment runbook

**Includes:** RBAC permission matrix, reconciliation guide, and deployment procedures
**Version:** 2.0 · June 2026

## Part 1: Admin (owner) guide

### 1. Interpret your dashboard

| Indicator | Colour | Action |
| --- | --- | --- |
| Stock status | Green | All products above reorder level. No action needed. |
| Stock status | Amber | Some products approaching threshold. Review the low-stock list. |
| Stock status | Red | Products at or below reorder level. Arrange a restock. |
| Active alerts badge | Red number on the Alerts icon | Tap Alerts to review and action each one. |

### 2. Role and permission matrix (RBAC)

| Feature / action | Admin | Manager | Attendant |
| --- | --- | --- | --- |
| Log in and access the app | Yes | Yes | Yes |
| View dashboard (full revenue data) | Yes | Yes | No |
| View dashboard (no revenue) | Yes | Yes | Yes |
| Add / edit products | Yes | Yes | No |
| Record stock-in and manual stock-out | Yes | Yes | No |
| Record sales | Yes | Yes | Yes |
| Void a sale | Yes | Yes | No |
| View all sales history | Yes | Yes | No |
| Manage suppliers | Yes | Yes | No |
| Create / manage purchase orders | Yes | Yes | No |
| Enter physical counts during a stock count | Yes | Yes | Yes |
| View reconciliation discrepancies | Yes | Yes | No |
| Approve a reconciliation adjustment | Yes | Yes | No |
| Request a recount / finalize a count | Yes | Yes | No |
| View and export reports | Yes | Yes (export delegated by Admin) | No |
| Manage users and roles | Yes | No | No |
| Change system settings | Yes | No | No |

> Attendants can enter physical counts during a stock count, but only Admins and Managers can view discrepancies, approve adjustments, request recounts, and finalize a count.

### 3. Inventory reconciliation

Reconciliation lets you confirm that what Inventra says you have matches what is physically on your shelves, value any difference in Naira, and resolve it.

**Run a reconciliation when** you suspect pilferage, a large delivery arrives, stock-outs are higher than expected, an audit is due, or a new staff member takes over stock management.

**The stock-count screen** shows summary cards for products counted, pending approval, discrepancies, and the variance value (net financial impact in Naira). Each row shows the product, who counted it and when, the value impact, the system count, the physical count, the difference, a result (Match, Loss, Surplus), and a status (Approved, Pending, Recount).

**To resolve a discrepancy,** open the item to see the counts, value impact, who counted, when, and a free-text reason. Then choose **Approve Adjustment** (updates system stock to the physical count and logs the adjustment with your name, after a confirmation prompt) or **Request Recount** (sets the item to Recount without changing stock).

### 4. Manage users

**Invite a staff member:** Settings, Manage staff, Invite user. Enter the details, select Manager or Attendant, and send the invite.

**Change a role:** Settings, Manage staff, select the person, Change role. This ends their current session; they must log in again.

**Deactivate a staff member:** Settings, Manage staff, select the person, Deactivate. Access is revoked immediately. Past sales and actions stay in the records.

### 5. Subscription plans

| Plan | Price | Includes |
| --- | --- | --- |
| Basic | ₦0 / month | Up to 4 users, basic reports, and core stock tracking and counts. For a single shop with one to three staff. |
| Premium | ₦2,500 / month | Up to 10 users, monthly reports, stock reconciliation, support for multiple devices, and customer support. For growing businesses. |

## Part 2: Deployment runbook

### 6. Production environment

| Component | Provider | Notes |
| --- | --- | --- |
| Application programming interface (API) server | Render | Node.js. Deployed at `stocksense-backend-devlopment.onrender.com`. |
| Database | MongoDB Atlas | Connection through `MONGO_URI`. |
| Continuous integration and deployment (CI/CD) | GitHub Actions | Automated build, test, and deploy. |

> Redis (caching and queues) is in the target architecture but not implemented in the v1.0 backend.

### 7. Pre-deployment checklist

- All pull requests merged and approved by the engineering lead
- All checks passing (tests, lint, build)
- Mongoose schema changes reviewed
- No open priority-zero or priority-one bugs

**Environment variables** (set in production, never committed): `PORT`, `MONGO_URI`, `JWT_SECRET`.

**Security:** valid Secure Sockets Layer (SSL) certificate for the production domain; cross-origin resource sharing (CORS) restricted to the production frontend; input validation confirmed on all routes; RBAC enforced on the reconciliation approve and finalize routes.

**Database:** production backup taken before deploying; indexes created on `productId`, `businessId`, and `createdAt`.

### 8. Deployment steps

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
```

### 9. Post-deployment smoke tests

- Register a new business account and confirm `POST /api/auth/register` returns 201.
- Log in and confirm `POST /api/auth/login` returns a token.
- Add a product and confirm it appears in inventory.
- Record a sale and verify stock is deducted.
- Create a supplier and a purchase order, and confirm they are saved.
- Start a stock count, enter a count that differs from the system count, approve the adjustment, and confirm stock updates and the adjustment is logged.
- Invite a staff member as Attendant and verify restricted access.

### 10. Incident response

| Severity | Definition | Response target |
| --- | --- | --- |
| Priority 1 (critical) | System down or data at risk | Under 2 hours |
| Priority 2 (high) | Major feature unavailable | Under 4 hours |
| Priority 3 (medium) | Feature degraded | Under 24 hours |
| Priority 4 (low) | Minor issue | Next sprint |

**Rollback:** revert through the Render dashboard; if the database was modified, restore from the backup with `mongorestore`; verify the service responds; run the smoke tests; document the incident.

### 11. Maintenance schedule

| Task | Schedule |
| --- | --- |
| Database backup | Regular, automated, with retention |
| Dependency security audit | Monthly (`npm audit`) |
| SSL certificate renewal | Before expiry |

---

*Inventra · Admin guide and deployment runbook · v2.0 · June 2026*
