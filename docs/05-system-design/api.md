# API reference (Cloudflare Worker, Hono)

> Status: Draft for review · Last updated: 2026-10-15 · Related: [architecture](architecture.md), [sync](sync.md), [auth-flows](auth-flows.md), [reminders-engine](reminders-engine.md)
>
> - JSON over HTTPS. Request and response bodies are validated with zod schemas from `packages/shared`.
> - Auth is `Authorization: Bearer <access JWT>` unless marked *public*.
> - Errors use the shape `{ "error": { "code": "string", "message": "string" } }`.

## Auth (see [auth-flows](auth-flows.md))
| Method | Path | Auth | Purpose |
|---|---|---|---|
| GET | `/auth/username-available?u=` | public | Signup helper (rate-limited) |
| POST | `/auth/signup` | public | Android-only signup (D-33) |
| POST | `/auth/login/start` | public | Start phone approval (SEC-01) |
| POST | `/auth/qr/start` | public | Start QR linking (SEC-02) |
| GET | `/auth/qr/details?r=` | trusted device | Approval screen data |
| POST | `/auth/login/approve` | trusted device | Approve or deny (number match, signature) |
| POST | `/auth/qr/approve` | trusted device | Approve a QR request |
| GET | `/auth/login/status?id=` | public | Poll the request status |
| POST | `/auth/login/complete` | public + device signature | Exchange for tokens |
| POST | `/auth/totp/login` | public | Backup login (SEC-03) |
| POST | `/auth/recover` | public | Recovery-code login on a new Android (SEC-06) |
| POST | `/auth/refresh` | device signature | Rotate tokens |
| POST | `/auth/logout` | ✓ | End this device's session |
| POST | `/auth/totp/enroll`, `/auth/totp/confirm`, `/auth/totp/disable` | trusted device | Manage TOTP |
| POST | `/auth/recovery-codes/regenerate` | trusted device + biometric | New set of codes |

## Sync (Android)
| Method | Path | Purpose |
|---|---|---|
| POST | `/sync/push` | Push outbox changes ([sync §4](sync.md#4-api-android-sync)) |
| GET | `/sync/pull?since=&limit=` | Pull changes |

## Notes (web, extension; all writes use `applyChange`)
| Method | Path | Purpose |
|---|---|---|
| GET | `/notes?state=&group=&is_todo=&cursor=` | List |
| GET | `/notes/:id` | One note |
| POST | `/notes` | Create (with `op_id`) |
| PATCH | `/notes/:id` | Update (`base_version`, `changed_fields`) |
| DELETE | `/notes/:id` | Soft delete (Trash) |
| POST | `/notes/:id/restore` | Restore from Trash |
| POST | `/notes/:id/resolve-conflict` | Keep this / keep other / merged body (DATA-01) |
| GET | `/notes/by-page?key=` | Notes for a page or site (EXT-04) |
| GET | `/notes/page-keys` | Map `page_key → count` for the extension badge cache |
| GET | `/search?q=` | Keyword search (ORG-07) |
| CRUD | `/groups`, `/tags` | Groups and tags (ORG-05) |
| POST | `/suggestions/:id/accept`, `/suggestions/:id/dismiss` | PLAT-04 |

## Reminders
| Method | Path | Purpose |
|---|---|---|
| POST/PATCH/DELETE | `/reminders[/:id]` | Manage reminders (also via notes/sync) |
| POST | `/reminders/:id/ack` | `{action: "done" \| "snooze", until?}` (REM-03) |
| GET | `/reminders/due?device_id=` | Extension polling (applies the device schedule) |
| GET | `/reminders/missed` | Missed list (REM-08 AC4) |

## Devices and settings
| Method | Path | Purpose |
|---|---|---|
| GET | `/devices` | Device list (SEC-05) |
| PATCH | `/devices/:id` | Rename, `push_token`, notification schedule (REM-08) |
| POST | `/devices/:id/revoke` | Revoke (SEC-05) |
| POST | `/devices/:id/promote` | Make a new Android trusted (approved by a trusted device) |
| GET/PATCH | `/me` | Username, `timezone` (default Asia/Kolkata), settings (`ai.enabled` stays false) |
| GET | `/me/audit` | Security events (SEC-08) |
| POST | `/push/web/subscribe` | Save a Web Push subscription for this device |

## Export and internal
| Method | Path | Purpose |
|---|---|---|
| — | *(no `/export` endpoint)* | Export is built **in the web app** from the normal list endpoints and zipped with `fflate` (D-48, DATA-03) |
| GET | `/files/:ref` | Audio download proxy (D-37) |
| — | *(Future, D-44)* `/telegram/link-code`, `/telegram/webhook` | See [telegram-bot.md](telegram-bot.md) |
| — | `scheduled()` | Cron: reminder dispatch every minute; daily: purge Trash > 30 days, prune `change_log`/`domain_events` > 90 days, expire login requests |
