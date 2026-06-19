# Inventra — Offline architecture and sync guide (design)

**Audience:** Mobile and backend developers
**Applies to:** Inventra v1.0
**Owner:** Mobile and backend leads

> **Status: design specification.** This guide describes Inventra's target offline-first architecture. The client-side local store and queue are part of the design, but the server sync endpoints (`/sync`, `/sync/pull`) are not yet implemented. Treat the endpoint examples here as the planned contract, not as routes you can call against the current backend. The current backend base path is `/api`.

## 1. Why offline-first

Mobile internet in the target market is intermittent. Operators report losing network access several times a working day, and gaps can last hours in non-urban areas. This is the normal operating environment for Inventra users, not a rare edge case.

| Approach | Behaviour |
| --- | --- |
| Offline-first (Inventra) | All writes go to local storage first; sync happens in the background; the app always responds instantly. |
| Online-first (rejected) | Writes go to the server first; the app is slow or broken without internet. |
| Online-only (rejected) | No local storage; unusable without internet. |

## 2. Offline architecture overview

### 2.1 Local database

The client keeps a local database that mirrors the server collections, so offline and online queries use the same logic.

### 2.2 The sync queue

Every write is recorded in a local `sync_queue` before it is applied to the main local tables.

```sql
CREATE TABLE sync_queue (
  id          TEXT PRIMARY KEY,   -- generated on device
  operation   TEXT NOT NULL,      -- { type, entity, payload }
  entity_id   TEXT,
  created_at  INTEGER NOT NULL,   -- device time
  status      TEXT DEFAULT 'pending', -- pending | synced | failed | conflict
  retry_count INTEGER DEFAULT 0,
  last_error  TEXT
);
```

> The `sync_queue` is add-only. Never update or delete rows; only transition the `status` field.

### 2.3 Write flow

1. A user action triggers a write (for example, an attendant confirms a sale).
2. The operation is written to `sync_queue` with `status = 'pending'`.
3. The local tables update immediately (optimistic update).
4. The user interface updates instantly.
5. The background sync service checks connectivity.
6. If online, the queue is processed; if offline, it waits.

> The user interface must never wait for a server response before updating. All updates are optimistic.

### 2.4 Read flow

All reads come from the local database. If online, a background delta sync keeps local data fresh. Reads never show a server-waiting spinner.

### 2.5 Connectivity status indicator

Every screen shows a status bar: online and synced; online with operations syncing; offline with operations queued; or sync failed (tap to retry).

## 3. Sync engine (planned)

> **Status:** the server endpoints in this section are planned, not implemented.

### 3.1 Trigger conditions

Sync fires when connectivity is restored, when the app returns to the foreground while online, when the user taps Retry, or on a periodic background check.

### 3.2 Sync process

1. Confirm connectivity.
2. Lock the queue to prevent concurrent runs.
3. Fetch pending items in creation order.
4. Send each operation to the planned `/sync` endpoint.
5. On success, set `status = 'synced'`.
6. On conflict, set `status = 'conflict'` and notify the owner.
7. On error, increment `retry_count`, apply backoff, and keep `status = 'pending'`.
8. After the maximum retries, set `status = 'failed'`.

```jsonc
// POST /api/sync (planned)
{
  "device_id": "device-identifier",
  "last_sync_at": "2026-06-01T08:00:00Z",
  "operations": [
    { "id": "queue-id-1", "type": "sale", "entity": "sale", "payload": {} }
  ]
}
```

### 3.3 Conflict resolution (planned)

| Scenario | Resolution | Notify owner? |
| --- | --- | --- |
| Same product stock modified on two offline devices | Most recent server time wins; both logged | Yes, if stock goes negative |
| Sale recorded offline for a product deleted online | Sale preserved; product shown as archived | No |
| Same sale voided on two devices | First void wins; second ignored | No |
| Stock goes negative after offline sales sync | All sales applied; stock set to zero | Yes, mandatory |

> Negative-stock conflicts are never auto-resolved; they always require owner review.

## 4. Data cached offline (target design)

| Data type | Refresh |
| --- | --- |
| Product catalogue | On every sync |
| Current stock levels | On every sync |
| Pending sales | Always local |
| Recent sales | On every sync |
| Active alerts | On every sync |
| Reconciliation sessions | On every sync |
| User profile and permissions | On login or role change |

## 5. Testing offline behaviour (when sync ships)

| Test case | Expected result |
| --- | --- |
| Record a sale offline | Sale appears immediately; badge shows one pending |
| Reconnect after an offline sale | Badge clears; sale appears on the owner's device |
| Close the app with a queued write, then reopen online | The write syncs; no data loss |
| Two devices sell the last unit offline | Both applied; stock goes negative; conflict raised |

---

*Inventra · Offline architecture and sync guide (design) · v2.0 · June 2026*
