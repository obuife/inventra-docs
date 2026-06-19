# Changelog

All notable changes to the Inventra documentation are recorded in this file. The format follows Keep a Changelog, and versions align with the Inventra product release cycle.

## [2.0.0] — June 2026 — Documentation reconciled with the as-built product

A second reconciliation with the engineering and design teams, based on the deployed backend, the Postman test runs, the Figma designs, and the in-product pricing screen. Several features that earlier revisions had marked as planned are confirmed built, and the whole set is brought into one consistent story.

### Changed

- **Suppliers and purchase orders → Available.** The supplier directory, purchase-order management, and the supplier-dependent reports (supplier performance, purchase-order status, recent orders) are implemented in the deployed backend and verified in Postman. They are reclassified from Planned to Available across all documents.
- **Phone OTP and six-digit PIN login → Available.** One-time password registration and PIN login are implemented in the backend authentication module and reflected in the designs. They are reclassified from Planned to Available.
- **Client technology corrected.** The client is described as a React Native mobile application (in development) and a web interface, with the user experience designed in Figma, replacing the earlier "React and Next.js Progressive Web App / Android (Kotlin or Flutter)" description.
- **No quality-assurance (QA) track.** References to a QA audience or role are removed, because the project team had no dedicated QA track. Testing was carried out by the backend track using Postman.
- **Abbreviations expanded on first use** across the set (for example, SME, GDP, API, REST, MVC, JWT, RBAC, OTP, PIN, HTTPS, URI, CRUD, ODM).
- **Pricing aligned to the product.** The plans are Basic (free) and Premium (₦2,500 per month), matching the in-product pricing screen.
- Document set version bumped to v2.0 across all footers.

### Still planned

- The offline background sync engine and its server endpoints (`/sync`, `/sync/pull`) are designed but not implemented; the offline guide remains a design specification.
- Redis caching and queues remain planned.

## [1.2.0] — June 2026 — Reconciliation shipped; supplier management deferred

A reconciliation with the engineering team at the time: inventory reconciliation was confirmed built and marked Available, while supplier management was understood to be unbuilt and was reclassified as Planned. (This decision was later superseded in v2.0, once suppliers and purchase orders were confirmed built.)

### Added

- Inventory reconciliation documented as a shipped feature across the set: count sessions, progress tracking, system versus physical count, difference, result (Match, Loss, Surplus), value impact and session variance in Naira, status (Approved, Pending, Recount), filters, the discrepancy detail view, and the approve/recount actions with a confirmation prompt.

### Changed

- Supplier management marked Planned across all documents. (Superseded in v2.0.)

## [1.1.0] — June 2026 — Rename and backend rework

### Changed

- Renamed the product from StockSense to Inventra across all documents. The repository and Render URLs were left unchanged because the backend repository was not renamed.
- Rewrote the API reference against the live backend: base path changed from `/v1` to `/api`; added the Categories, Inventory, and Business profile groups; simplified the error format.
- Updated authentication to JWT through `jsonwebtoken` with bcrypt password hashing.

## [1.0.1] — June 2026 — Database correction

### Changed

- Updated the database technology from PostgreSQL and Prisma to MongoDB (Atlas or local) with Mongoose across the developer, admin, offline, and API documents.

## [1.0.0] — May 2026 — Initial release

### Added

- Initial release of all seven documents plus the README, produced for the v1.0 launch.

---

*Maintained by the Inventra technical writing team.*
