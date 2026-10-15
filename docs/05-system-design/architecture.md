# Architecture

> Status: Draft for review · Last updated: 2026-10-15 · Related: [tech-stack](tech-stack.md), [data-model](data-model.md), [sync](sync.md), [auth-flows](auth-flows.md), [reminders-engine](reminders-engine.md), [hosting](hosting-and-distribution.md)
>
> Implements decisions D-21 (zero cost), D-27 (free cloud only), D-30…D-38 (stack).

## 1. The big picture

```
 ┌────────────────────────────────── CLIENTS (all TypeScript) ─────────────────────────────────┐
 │                                                                                             │
 │  ANDROID (React Native + Expo)          WEB PWA (React + Vite)        EXTENSION (Chrome MV3) │
 │  OFFLINE-FIRST                          ONLINE-ONLY (D-39)            ONLINE + capture queue │
 │  ├─ UI (RN)                             ├─ UI (React)                 ├─ popup / side panel   │
 │  ├─ Local DB: SQLite (full copy)        ├─ API calls + in-memory      ├─ API calls + small    │
 │  ├─ Sync client + outbox                │   cache (TanStack Query)    │   cache; capture queue│
 │  └─ Kotlin modules:                     ├─ Service worker: Web Push   │   in chrome.storage   │
 │     alarms · widget · tile · share      │   only (no offline data)    ├─ Service worker:      │
 │     · speech · keystore                 └─ Web Speech API             │   chrome.alarms poll, │
 │                                                                       │   notifications,      │
 │                                                                       │   context menu, badge │
 │                    all share  packages/shared  (types, schemas, sync, date parser, router)    │
 └──────────────┬───────────────────────────────┬──────────────────────────────┬───────────────┘
                │ HTTPS (JSON)                  │                              │
                ▼                               ▼                              ▼
 ┌──────────────────────────── CLOUDFLARE (free plan) ─────────────────────────────────────────┐
 │  Worker "cue-api" (Hono)                                                                     │
 │   ├─ /auth/*      signup (Android), login approval, QR, TOTP, recovery, sessions              │
 │   ├─ /sync/*      push outbox changes, pull changes since version                            │
 │   ├─ /devices/*   device list, revoke, notification schedule (office hours)                  │
 │   └─ scheduled()  Cron Trigger every minute → reminder dispatcher (Web Push, nag repeats)     │
 │  D1 (SQLite) ← all data (+ optional audio chunks)   Static assets ← web PWA (same Worker)   │
 └──────────────┬──────────────────────────────┬──────────────────────────────┬────────────────┘
                ▼                              ▼                              ▼
        Firebase Cloud Messaging         Web Push services                (Telegram: Future, D-44)
        (Android: login approvals,       (browser notifications           (capture, reminders,
         "sync now" nudges)               for the PWA)                     /find, done/snooze)
```

## 2. Components and responsibilities

| Component | Responsibility | Feature IDs |
|---|---|---|
| **Android app** | The main capture surface; first and trusted device; approves logins; fires reminders locally | CAP-01…06, REM-04/05, SEC-01/02/04/06/07 |
| Kotlin: `CueAlarms` | Exact alarms, survives reboot, notification actions (Done/Snooze) even when JS isn't running | REM-02, REM-04, REM-07 |
| Kotlin: `CueWidget`, `CueTile` | Home-screen widget and Quick Settings tile → open the capture screen instantly | CAP-02, CAP-03 |
| Kotlin: share target | Receive shared text, links and images from other apps | CAP-04 |
| **Web PWA** | Desktop capture and organising; installable; **online-only** (D-39); Web Push within office hours | CAP-06, ORG-xx, REM-08 |
| **Extension** | Quick note, save selection, page-linked badge, side panel, quiet-hours-aware notifications; offline captures are queued | EXT-01…05, CAP-09 |
| **API Worker** | Auth, sync, devices, scheduled reminder dispatch | SEC-xx, DATA-01, REM-06/07/08 |
| **D1** | The single source of truth on the server (ciphertext only for locked notes) | — |
| **packages/shared** | Everything both sides must agree on (see [tech-stack §3](tech-stack.md#3-packagesshared-what-lives-there)) | PLAT-02, PLAT-04 |

## 3. Key principles in the design
1. **Local-first on Android, online on web.** The Android app reads and writes its **local DB first**, then syncs in the background, so the UI never waits for the network (P1, PLAT-01, CAP-09). The web app talks to the API directly (D-39). The extension calls the API but queues captures when offline.
2. **The server is the ordering authority.** It assigns a monotonically increasing `version` to every change. Clients pull "everything after version N" ([sync.md](sync.md)).
3. **Reminders fire on the device where possible.** Android uses local exact alarms (works offline). The server cron covers only Web Push (PWA) and nag repeats for the PC ([reminders-engine.md](reminders-engine.md)).
4. **Cheap CPU per request.** The Workers free plan has a tight CPU budget per request, so the design avoids slow password hashing. There are no passwords: login uses device keys, and recovery codes are high-entropy and hashed with SHA-256 ([auth-flows.md](auth-flows.md)).
5. **Zero cost.** Every external dependency is on a free plan or open source. No credit card ([hosting-and-distribution.md](hosting-and-distribution.md)).
6. **AI-ready, no AI.** Events, Suggestions, a search interface, flags and a command router exist from day one ([ai-hooks.md](ai-hooks.md)).

## 4. Main flows (summary)

| Flow | Path |
|---|---|
| Capture on Android | Widget/tile → RN capture screen → SQLite + outbox → (background) `POST /sync/push` |
| Capture on web | Quick-add → `POST /notes` (online-only; error with retry if offline) |
| Capture in extension | Hotkey popup → `POST /notes`; if offline → `chrome.storage` capture queue → sent on reconnect |
| Another device sees it | Android: FCM "sync" nudge + on open → `GET /sync/pull?since=N`. Web: refetch on focus / every 60 s while visible. Extension: `chrome.alarms` refresh |
| Reminder on phone | Pull → schedule `CueAlarms` locally → fires offline → Done/Snooze → outbox → server → other devices cancel |
| Reminder on PC | Cron every minute → due reminders → routing rules (office hours per device) → Web Push / extension poll |
| Sign in on web | Username → `POST /auth/login/start` → FCM to phone → approve with number + biometric → web gets a session |

## 5. Trust boundaries
- **Device ↔ server:** HTTPS only. Each device has its own keypair and session. A device can be revoked (SEC-05).
- **Locked notes:** encrypted on the client. The server, the bot and future AI only ever see ciphertext (SEC-07).
- **Free providers:** Cloudflare and Google (FCM) see metadata. Normal notes are encrypted at rest by the provider (see [hosting](hosting-and-distribution.md)).

## 6. What is deliberately NOT here
- No AI server (D-24).
- No home-PC server (D-27).
- No real-time co-editing or CRDTs (D-34).
- No email service (recovery needs no email; D-06).
- No messaging bot (Telegram → Future, D-44).
