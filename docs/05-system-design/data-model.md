# Data model

> Status: Draft for review · Last updated: 2026-10-15 · Related: [sync](sync.md), [auth-flows](auth-flows.md), [features](../02-product/features.md)
>
> The same logical model is used on the server (D1/SQLite) and Android (SQLite, full offline copy). Web and extension don't store data locally (D-39); they use the API.
> Conventions:
> - IDs are **UUIDv7 strings**, generated on the client (time-ordered and safe offline).
> - Times are **UTC epoch milliseconds** (D-28). The display time zone comes from `user.timezone`.
> - Every synced table has `user_id`, `version`, `updated_at`, `deleted` (soft delete; tombstones are needed for sync).

## 1. Entities (overview)
```
User 1─* Device 1─* Session
User 1─* Note  *─1 Group          Note *─* Tag (via NoteTag)
Note 1─0..1 Todo (fields on Note)  Note 1─* Reminder
Note 1─0..1 Source (fields on Note)
User 1─* Suggestion ─→ Note        User 1─* Event (outbox of domain events, server side)
User 1─0..1 Vault (locked-notes key material)   User 1─* RecoveryCode   User 1─0..1 Totp
Device 1─0..1 NotificationSchedule      (TelegramLink: Future, D-44)
```

## 2. Tables

### users
| Column | Type | Notes |
|---|---|---|
| id | text PK | UUIDv7 |
| username | text unique | Short, lowercase, `[a-z0-9_]{3,20}`. Used to start a login (SEC-01) |
| timezone | text | IANA zone, **default `Asia/Kolkata`** (D-28) |
| settings | json | Theme, nag interval default, `ai.enabled` (default false, PLAT-04) |
| created_at | int | |

### devices
| Column | Type | Notes |
|---|---|---|
| id | text PK | |
| user_id | text FK | |
| name | text | e.g. "Pixel 8", "Work PC – Chrome", "Chrome extension" |
| kind | text | `android` \| `web` \| `extension` |
| public_key | text | ECDSA P-256 public key (JWK). The private key never leaves the device |
| trusted | int | 1 = can approve logins (Android only in MVP) |
| push_token | text null | FCM token (Android) or Web Push subscription JSON (PWA) |
| created_at, last_seen_at | int | |
| revoked_at | int null | Set by SEC-05 |

### notification_schedules (REM-08, D-28)
| Column | Type | Notes |
|---|---|---|
| device_id | text PK FK | |
| mode | text | `always` \| `office_hours` \| `never` |
| days | text | e.g. `1,2,3,4,5` (Mon=1) |
| start_min, end_min | int | Minutes after midnight in the user's time zone, e.g. 540 → 09:00, 1080 → 18:00 |
| version, updated_at | int | Synced |

### sessions
| Column | Type | Notes |
|---|---|---|
| id | text PK | |
| device_id | text FK | |
| refresh_hash | text | SHA-256 of the refresh token |
| expires_at | int | |
| created_at, last_used_at | int | |

### notes (also holds todo, source and flags; one row per note)
| Column | Type | Notes |
|---|---|---|
| id | text PK | |
| user_id | text FK | |
| body | text | Plaintext for normal notes; **empty for locked notes** |
| title | text null | Optional |
| is_todo | int | ORG-01 |
| done_at | int null | ORG-02 |
| state | text | `inbox` \| `active` \| `archived` |
| group_id | text null FK | ORG-05 |
| pinned, important | int | ORG-06, REM-07 |
| locked | int | SEC-07 |
| ciphertext | text null | Locked notes only: base64 AES-GCM envelope ([auth-flows §6](auth-flows.md#6-locked-notes-vault-sec-07)) |
| ai_visible | int | Default 1; forced to 0 when locked (PLAT-04) |
| source_kind | text | `android_share` \| `widget` \| `tile` \| `voice` \| `web` \| `extension` (future: `telegram`) |
| source_app | text null | Android package or label |
| source_url, source_title | text null | CAP-10, EXT-02 |
| source_fragment | text null | Text-fragment link `#:~:text=` (EXT-02) |
| page_match | text null | `page` \| `site`: how EXT-04 matches this note |
| page_key | text null | Normalised URL or host used for badge lookups (indexed) |
| audio_ref | text null | `d1:<attachment_id>` for app audio (D-40); future bot audio `tg:<file_id>` |
| conflict_of | text null | If this is a conflict copy, the original note id (DATA-01) |
| content_version | int | The `version` at which `body`/`title`/`ciphertext` last changed (used for conflict detection, [sync.md](sync.md)) |
| created_at, updated_at, version | int | |
| deleted | int | Soft delete. Trash = deleted with `deleted_at` < 30 days (ORG-06) |
| deleted_at | int null | |

### reminders
| Column | Type | Notes |
|---|---|---|
| id | text PK | |
| note_id | text FK | |
| fire_at | int | Next occurrence (UTC ms) |
| rrule | text null | RFC 5545 rule for recurring reminders (REM-02) |
| snoozed_until | int null | |
| last_ack_at | int null | When the current occurrence was handled (Done or Snooze) on any device (REM-03) |
| nag | int | 1 if the note is important (REM-07); repeats every `nag_every_min` |
| nag_every_min | int | Default 10 |
| escalated_at | int null | When it escalated to the alarm-style phone alert (REM-07, D-44) |
| channels | text | e.g. `device,web` (REM-06) |
| server_dispatched_at | int null | Last time the server cron sent this occurrence (web PWA) ([reminders-engine](reminders-engine.md)) |
| repeat_count | int | Nag repeats for the current occurrence; reset on the next occurrence |
| last_fired_at | int null | When the current occurrence first fired (for the "Missed reminders" list) |
| version, updated_at, deleted | int | |

### groups, tags, note_tags
- `groups(id, user_id, name, color, sort, version, updated_at, deleted)`
- `tags(id, user_id, name, version, updated_at, deleted)`
- `note_tags(note_id, tag_id, version, updated_at, deleted)` (composite PK)

### suggestions (PLAT-04)
| Column | Type | Notes |
|---|---|---|
| id | text PK | |
| note_id | text FK | |
| type | text | `reminder_date` (MVP, rule-based) · future: `group`, `tags`, `make_todo`, `split` |
| payload | json | e.g. `{ "fire_at": 1760530000000, "text": "tomorrow 6pm" }` |
| source | text | `rules` (MVP) · future: `ai:<provider>` |
| status | text | `pending` \| `accepted` \| `rejected` |
| version, updated_at | int | |

### Auth and security tables (server only, never synced to clients)
| Table | Columns | Feature |
|---|---|---|
| `login_requests` | id, user_id, requesting_device_info (json), number_shown, nonce, status (`pending/approved/denied/expired`), expires_at, approved_by_device | SEC-01, SEC-02 |
| `recovery_codes` | id, user_id, code_hash (SHA-256), used_at | SEC-06 |
| `totp` | user_id, secret_enc, enabled_at, last_used_step | SEC-03 |
| `vaults` | user_id, wrapped_keys (json: one entry per device plus recovery-code wrap), salt, params | SEC-07 |
| `telegram_links` *(Future, D-44; don't create in MVP)* | user_id, chat_id, linked_at; `link_codes(code_hash, user_id, expires_at)` | BOT-01 |
| `audit_events` | id, user_id, type (`login`, `device_added`, `device_revoked`, `recovery_used`, …), device_id, ip_hint, at | SEC-08 |
| `rate_limits` | key, window_start, count | SEC-01, SEC-03, SEC-08 |
| `change_log` | version (autoincrement PK), user_id, table, row_id, op_id (unique, for idempotent retries) | Sync pull index ([sync.md](sync.md)) |
| `domain_events` | id, user_id, name, payload, created_at | PLAT-04 (kept small; pruned after N days) |
| `attachments`, `attachment_chunks` | attachments(id, user_id, note_id, mime, bytes, created_at); chunks(attachment_id, idx, data BLOB ≤ 1.5 MB) | D-40: optional audio, 100 MB cap per user |
| `kv_cache` | key, value, expires_at | e.g. the cached FCM OAuth token (D-43) |

## 3. Indexes (minimum)
- `notes(user_id, version)`, `notes(user_id, state, updated_at)`, `notes(user_id, page_key)`
- `reminders(fire_at)` (for the cron dispatcher), `reminders(note_id)`
- `change_log(user_id, version)`
- `devices(user_id)`, `sessions(device_id)`

## 4. Local-only tables (Android only)
- `outbox(seq, op_id, table, row_id, op, base_version, changed_fields, payload, created_at, tries)`: pending changes ([sync.md](sync.md)).
- `sync_state(last_pulled_version, last_push_at)`.
- `scheduled_alarms(reminder_id, occurrence_at, alarm_request_code)`, for cancelling and rescheduling.
- Search index: SQLite FTS5 on Android (ORG-07). Web and extension use server search (`GET /search`, D1 FTS5, supported).
- Extension only: a capture queue in `chrome.storage.local` (`[{op_id, payload, created_at}]`), not a table.

## 5. Size and free-tier notes
- Text notes are tiny. Even 50,000 notes is only a few tens of MB.
- Audio is the only large data. It is **off by default** (D-37), chunked in D1 and capped at 100 MB (D-40) inside the 500 MB D1 limit.
- `change_log` and `domain_events` must be pruned (e.g. keep 90 days). A device offline for longer does a full resync.
