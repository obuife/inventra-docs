# Inventra — Offline architecture and sync guide (design)

**Audience:** Mobile developers, backend engineers, QA engineers, support
**Applies to:** Inventra v1.0 — Android app and Progressive Web App (PWA)
**Key technologies:** SQLite/Room (Android), IndexedDB/Dexie.js (PWA), WorkManager, Service Worker
**Owner:** Mobile Lead and Backend Lead

> **Status: design specification**
> This guide describes Inventra's target offline-first architecture. The **client-side local store and queue** are part of the v1.0 design, but the **server sync endpoints (`/sync`, `/sync/pull`) aren't implemented yet.** Treat the endpoint examples here as the planned contract, not as routes you can call against the current backend. The current backend base path is `/api`.

---

## 1. Why offline-first

Nigeria's mobile internet is fast on paper but highly intermittent. SME operators in our target market report losing network access **3 to 6 times per working day**. In non-urban areas, gaps can last hours.

This is the **normal operating environment** for Inventra users, not a rare edge case.

| Design approach | Behaviour |
|---|---|
| Offline-first (Inventra) | All writes go to local storage first. Sync happens in the background. The app always responds instantly. |
| Online-first (rejected) | Writes go to the server first. The app is slow or broken without internet. |
| Online-only (rejected) | No local storage. Unusable without internet. |

---

## 2. Offline architecture overview

### 2.1 The two local databases

| Platform | Engine | Library | Target max footprint |
|---|---|---|---|
| Android (Kotlin / Flutter) | SQLite | Room Persistence Library | Under 50 MB |
| PWA (Chrome / web) | IndexedDB | Dexie.js | Under 50 MB |

> **Note:** The local schema mirrors the MongoDB server collections so offline and online queries use the same logic and sync mapping stays straightforward.

### 2.2 The sync queue

Every write is recorded in a local `sync_queue` table before it's applied to the main local tables.

```sql
CREATE TABLE sync_queue (
  id            TEXT PRIMARY KEY,        -- UUID generated on device
  operation     TEXT NOT NULL,           -- JSON: { type, entity, payload }
  entity_id     TEXT,                    -- ID of the affected record
  created_at    INTEGER NOT NULL,        -- Unix timestamp (device time, ms)
  status        TEXT DEFAULT 'pending',  -- pending | synced | failed | conflict
  retry_count   INTEGER DEFAULT 0,
  last_error    TEXT
);
```

**Example operation payload (sale):**

```json
{
  "id": "a1b2c3d4-...",
  "movement_type": "sale",
  "entity": "sale",
  "payload": {
    "id": "a1b2c3d4-...",
    "attendant_id": "u-001",
    "device_time": "2026-05-01T09:15:00Z",
    "items": [{ "product_id": "p-001", "quantity": 2, "unit_price": "350.00" }],
    "total_amount": "700.00",
    "payment_method": "cash"
  }
}
```

> **Caution:** The `sync_queue` is append-only. Never `UPDATE` or `DELETE` rows. Only transition the `status` field.

### 2.3 Write flow (online or offline)

Every write follows this sequence regardless of connectivity:

1. A user action triggers a write (for example, an attendant confirms a sale).
2. The operation is written to `sync_queue` with `status = 'pending'`.
3. The local tables update immediately (optimistic update).
4. The UI updates instantly.
5. The background sync service checks connectivity.
6. If online, the queue is processed. If offline, the queue waits.

> **Caution:** The UI must never wait for a server response before updating. All updates are optimistic.

### 2.4 Read flow

All reads come from the local database, never directly from the server:

1. The UI requests data.
2. Data is served from local SQLite / IndexedDB.
3. If online, a background delta sync keeps local data fresh.
4. Reads never show a server-waiting spinner.

### 2.5 Connectivity detection

| Platform | API | Behaviour |
|---|---|---|
| Android | `ConnectivityManager` + `NetworkCallback` | Fires sync when the network becomes available. |
| PWA | `navigator.onLine` + online/offline events | Also runs a periodic beacon ping to confirm a real route. |

> **Caution:** `navigator.onLine` returns `true` whenever a network adapter is active, even without an internet route. Confirm with a lightweight beacon ping before syncing.

### 2.6 Connectivity status indicator

Every screen shows a status bar:

- 🟢 **Online — all data synced:** connected and the queue is empty
- 🟡 **Online — X actions syncing:** sync in progress
- 🔴 **Offline — X actions queued:** no connection detected
- ⚠️ **Sync failed — tap to retry:** one or more items failed after max retries

---

## 3. Sync engine (planned)

> **Status:** The server endpoints in this section are planned, not implemented.

### 3.1 Trigger conditions

Sync fires when:

- Connectivity is restored
- The app returns to the foreground while online
- The user taps **Retry** on a failed-sync notification
- A periodic background check runs (Android WorkManager)

### 3.2 Sync process

1. Confirm connectivity via a beacon ping.
2. Lock the queue to prevent concurrent runs.
3. Fetch pending items: `SELECT * FROM sync_queue WHERE status = 'pending' ORDER BY created_at ASC`.
4. Process in creation order.
5. `POST` each operation to the planned `/sync` endpoint.
6. On `200`, set `status = 'synced'`.
7. On `409` (conflict), set `status = 'conflict'` and notify the owner.
8. On error, increment `retry_count`, apply backoff, and keep `status = 'pending'`.
9. After max retries, set `status = 'failed'` and show a **Sync failed** notification.
10. Release the lock when the queue is empty.

**Planned batch sync endpoint:**

```json
// POST /api/sync   (planned)
{
  "device_id": "device-uuid",
  "last_sync_at": "2026-05-01T08:00:00Z",
  "operations": [
    { "id": "queue-uuid-1", "movement_type": "sale", "entity": "sale", "payload": {} }
  ]
}

// Response
{
  "applied": ["queue-uuid-1"],
  "conflicts": [{ "id": "queue-uuid-2", "reason": "STOCK_NEGATIVE", "details": {} }],
  "server_time": "2026-05-01T09:12:45Z"
}
```

### 3.3 Delta pull (planned)

```
GET /api/sync/pull?since=2026-05-01T08:00:00Z&device_id=device-uuid   (planned)
```

Returns records changed after `since`, excluding changes from this `device_id`.

### 3.4 Retry strategy

| Attempt | Delay |
|---|---|
| 1st | 30 seconds |
| 2nd | 2 minutes |
| 3rd | 10 minutes |
| 4th | 30 minutes |
| 5th+ | Hourly |
| After 5th | No further retries — item marked `failed` |

---

## 4. Conflict resolution (planned)

### 4.1 Rules

| Scenario | Resolution | Notify owner? |
|---|---|---|
| Same product stock modified on two offline devices | Most recent `server_time` wins. Both logged. | Yes, if stock goes negative |
| Sale recorded offline for a product deleted online | Sale preserved. Product shown as `[Archived]`. | No |
| Same sale voided on two devices | First void wins. Second ignored (idempotent). | No |
| Price changed while offline sales were made at the old price | Sales preserved at the offline price. | Yes — price-mismatch alert |
| Stock goes negative after all offline sales sync | All sales applied. Stock set to 0. | Yes — mandatory |

> **Caution:** Negative-stock and price-mismatch conflicts are never auto-resolved. They always require owner review.

### 4.2 Conflict notification

When a conflict needs attention, a **Conflict** badge appears on the Alerts icon, the alert type is `SYNC_CONFLICT`, and the owner taps it to act.

---

## 5. Platform-specific implementation

### 5.1 Android — WorkManager

```kotlin
val syncRequest = PeriodicWorkRequestBuilder<SyncWorker>(
    repeatInterval = 15, TimeUnit.MINUTES
)
  .setConstraints(
    Constraints.Builder()
      .setRequiredNetworkType(NetworkType.CONNECTED)
      .build()
  )
  .setBackoffCriteria(BackoffPolicy.EXPONENTIAL, 30, TimeUnit.SECONDS)
  .build()

WorkManager.getInstance(context).enqueueUniquePeriodicWork(
  "inventra_sync",
  ExistingPeriodicWorkPolicy.KEEP,
  syncRequest
)
```

> **Caution:** Run sync on a background thread. Use Kotlin coroutines with `Dispatchers.IO` inside `doWork()`. Never run on the main thread.

> **Note:** Use Room's `@Transaction` annotation when a method writes to multiple tables (for example, recording a sale and deducting stock). If either write fails, both roll back.

### 5.2 PWA — Service Worker caching

| Asset type | Strategy | Rationale |
|---|---|---|
| App shell (HTML, CSS, JS) | Cache-first | Static assets — serve instantly |
| API reads (GET) | Network-first with cache fallback | Try fresh; serve cache if offline |
| API writes (POST, PUT, DELETE) | Queue (Background Sync API) | Never cache writes; queue them |
| Report PDFs | No caching | Generated on demand |

```javascript
// Register sync when an operation is queued
navigator.serviceWorker.ready.then(registration => {
  registration.sync.register('inventra-sync');
});

// In the service worker
self.addEventListener('sync', event => {
  if (event.tag === 'inventra-sync') {
    event.waitUntil(processSyncQueue());
  }
});
```

> **Caution:** Background Sync is supported in Chrome for Android but limited in Safari/iOS. For iOS (Phase 2), fall back to processing the queue on the `visibilitychange` event.

---

## 6. Data cached offline

| Data type | Strategy | Refresh | Notes |
|---|---|---|---|
| Product catalogue | Persistent local copy | On every sync | Includes archived products |
| Current stock levels | Persistent local copy | On every sync | Updated on every movement |
| Pending sales | Full local storage | Always local | Never expire or purge |
| Recent sales (last 30 days) | Local copy | On every sync | For offline history |
| Active alerts | Local copy | On every sync | Visible without connectivity |
| Reconciliation sessions | Local copy | On every sync | For offline stock counts |
| Supplier directory | Persistent local copy | On every sync | **Planned** — supports the not-yet-built supplier feature |
| User profile and permissions | Persistent local copy | On login / role change | RBAC works offline |

---

## 7. Troubleshooting (when sync ships)

### 7.1 User-facing issues

**"My sale doesn't appear on the owner's phone."**

1. Check whether a sync badge shows pending operations.
2. If yes, the sale is queued — check the internet connection.
3. If online but not syncing, tap the badge to retry.
4. If the badge shows **Sync failed**, check the device clock.

**"I see a Sync failed notification."**

1. Tap the notification to see which operations failed.
2. Tap **Retry**.
3. If it fails again, verify the connection by opening a browser.
4. If the connection is good, note the error and contact support with the device model and error text.

### 7.2 Developer debugging

**Inspect the sync queue on Android:**

```bash
adb shell run-as com.inventra.app sqlite3 databases/inventra.db
SELECT id, json_extract(operation, '$.type'), status, retry_count, last_error
FROM sync_queue WHERE status != 'synced' ORDER BY created_at ASC;
```

**Inspect IndexedDB on the PWA:** open Chrome DevTools (F12) > **Application > Storage > IndexedDB > InventraDB > sync_queue**.

---

## 8. Testing offline behaviour

| Test case | Steps | Expected result |
|---|---|---|
| Basic offline sale | Airplane mode → record a sale → confirm | Sale in the attendant's list immediately. Badge shows 1 pending. |
| Sync on reconnect | After an offline sale, re-enable data | Badge clears within 30 seconds. Sale appears on the owner's list. |
| App close during queue | Airplane mode → record 3 sales → close app → re-enable data → reopen | All 3 sync. No data loss. |
| Concurrent negative stock | Two devices offline both sell the last unit → reconnect both | Both applied. Stock goes to -1. Conflict alert raised. |
| Large queue sync | Record 50 sales offline → reconnect | All 50 sync within 30 seconds on 3G. No losses or duplicates. |

---

*Inventra · Offline architecture and sync guide · v1.2 (design) · June 2026 · Internal / Confidential*
