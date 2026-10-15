# Sync design

> Status: Draft for review · Last updated: 2026-10-15 · Related: [data-model](data-model.md), [architecture](architecture.md), decisions D-13, D-17, D-34, D-39, features CAP-09, DATA-01, REM-03

## 1. Scope
| Client | Model |
|---|---|
| **Android** | Full offline copy in SQLite + **outbox** + **pull by version** (this doc) |
| **Web** | Online-only (D-39). It writes through the API, but uses the **same server write rule** (§3), so conflicts are handled the same way |
| **Extension** | Online. Offline captures go into a queue and are sent as normal creates (creates never conflict) |

## 2. Core ideas
1. **The server owns ordering.** Every accepted change gets the next `version` from a per-database autoincrement (`change_log.version`). Clients remember the last version they pulled.
2. **Clients send what they based the edit on.** Each change carries `base_version` (the row version the client last saw) and `changed_fields`.
3. **Retries are idempotent.** Each change has a client-generated `op_id`. If the server has already applied that `op_id`, it returns the earlier result.
4. **Deletes are soft** (`deleted = 1`), so other devices learn about them through pull. Rows are hard-deleted (purged) after 30 days in Trash.

## 3. The server write rule (used by `/sync/push` and by the web/extension REST endpoints)
```
applyChange(change):
  if op_id already in change_log → return previous result            # idempotent
  row = load(table, row_id)
  if row is null:                → insert (create)                   # creates never conflict
  else if change.base_version >= row.version:
                                 → apply changed_fields              # nobody else changed it
  else:                                                              # someone changed it since
     contentFields = {body, title, ciphertext}
     if changed_fields ∩ contentFields ≠ ∅ and row.content_version > change.base_version:
         → CONFLICT: keep the server row as is, and insert a NEW note
           {conflict_of: row.id, body/title/ciphertext from the change, state: same}
           (DATA-01: both versions kept; the user resolves)
     else:
         → MERGE: apply only changed_fields (metadata such as done, group, pinned,
           important, state, tags is last-writer-wins by field)
  version = next(); write change_log(version, user_id, table, row_id, op_id)
  if content fields changed → row.content_version = version
  emit domain event (note.created / note.updated / note.deleted / todo.completed …)   # PLAT-04
  return {row_id, version, conflict_copy_id?}
```
- **Reminders:** `last_ack_at` and `snoozed_until` use last-writer-wins. The latest Done or Snooze from any device wins, and it stops alerts on all devices (REM-03).
- **Locked notes:** only `ciphertext` changes. The server never looks inside it. A conflict creates a conflict copy containing the other ciphertext.

## 4. API (Android sync)
### Push
```
POST /sync/push
{ "device_id": "...", "changes": [
    { "op_id": "uuid", "table": "notes", "row_id": "uuid", "op": "upsert"|"delete",
      "base_version": 1234, "changed_fields": ["body","important"], "payload": { ... } }
  ] }                                              # max 100 changes per request
→ { "results": [ { "op_id": "...", "version": 1240, "conflict_copy_id": null } ] }
```
### Pull
```
GET /sync/pull?since=1234&limit=500
→ { "changes": [ { "table": "notes", "row": {...full row...} }, ... ],
    "cursor": 1400, "has_more": false,
    "resync_required": false }          # true if `since` is older than the retained change_log
```
- Pull returns the **current full row** for each changed row (not a diff), which keeps clients simple.
- If `resync_required`, the client wipes its local synced data (the outbox is kept) and pulls from 0.

## 5. Android client loop
```
on local write:  write SQLite row + outbox item (same transaction) → UI updates instantly
sync() runs:     on app start/foreground, after each local write (debounced 2 s),
                 on FCM data message {"type":"sync"}, on network regained, every 15 min (WorkManager)
sync():
   1. push outbox in batches of 100 → for each result: delete outbox item, set row.version
      (if conflict_copy_id → pull will deliver the copy)
   2. pull since last_pulled_version until has_more = false → upsert rows locally
      (skip rows that have pending outbox items; they'll be re-based on the next push)
   3. reschedule local alarms for reminders that changed (reminders-engine)
   4. save last_pulled_version
```
- **The server sends an FCM "sync" nudge** to the user's Android devices after any accepted change that came from *another* device (web, extension). The nudge is collapsed so bursts don't spam.

## 6. Web and extension (online, D-39)
- They read through REST endpoints (`GET /notes?state=inbox`, `GET /notes/:id`, `GET /search?q=`, `GET /notes/by-page?key=`). They write with `POST /notes`, `PATCH /notes/:id` (with `base_version` + `changed_fields`), and `DELETE /notes/:id`. All writes go through **the same `applyChange`**.
- TanStack Query refetches on window focus, every 60 s while visible, and after writes.
- **Extension capture queue:** when `POST /notes` fails because the network is down, `{op_id, payload}` is appended to `chrome.storage.local`. A `chrome.alarms` job and the `online` event retry the queue. The `op_id` makes retries safe.

## 7. Conflict UX (DATA-01)
- A conflict copy shows a banner: **"Edited on two devices: 2 versions"** with **Keep this**, **Keep other**, and **Merge** (opens both side by side).
- Keeping one soft-deletes the other. Nothing is ever silently lost.

## 8. Free-tier budget (rough)
- Each accepted change costs about 2–3 D1 row writes (row + change_log + event). 100 notes a day is about 300 writes, far below the free daily write limit (see [hosting](hosting-and-distribution.md#free-tier-limits)).
- Pulls are cheap: indexed by `(user_id, version)`.
- Pruning: `change_log` and `domain_events` keep 90 days. A device older than that gets `resync_required`.

## 9. Test cases that must pass
1. Create offline on Android → reconnect → the note appears on web.
2. Edit the same note body on Android (offline) and on web → Android reconnects → **both versions exist**, with a banner.
3. Mark done on web while Android has the same note offline and only changes `group` → both changes merge, with no conflict.
4. The same push is sent twice (a retry) → one row, one version.
5. Delete on web → Android pulls → the note is in Trash. After 30 days it is purged everywhere.
6. Reminder done on Android → web and extension stop alerting within one cron cycle.
