# Android app design (React Native + Expo + Kotlin modules)

> Status: Draft for review · Last updated: 2026-10-16 · Screen designs: [ui-ux-design §6](../07-design/ui-ux-design.md#6-mobile-app-android) · Related: [tech-stack](tech-stack.md), [sync](sync.md), [reminders-engine](reminders-engine.md), [auth-flows](auth-flows.md), [ux-principles](../02-product/ux-principles.md), decisions D-31, D-33

## 1. Project setup
- `apps/mobile`: Expo (current SDK) with the **prebuild / dev-build** workflow (not Expo Go, because native modules are needed).
- Native Kotlin code lives in **local Expo modules**: `apps/mobile/modules/cue-alarms`, `cue-widget`, `cue-tile`, `cue-share`.
- Build an APK **locally for free**: `npx expo prebuild` → `cd android && ./gradlew assembleRelease` (or `npx expo run:android --variant release`). The signing key is generated once and stored outside the repo (back it up). EAS Build is optional (free tier: 15 Android builds a month).
- Install: copy the APK to the phone, or download it from a GitHub Release (D-38).

## 2. Screens (Expo Router)
| Route | Purpose | Features |
|---|---|---|
| `/capture` | **The capture screen.** Opens with the keyboard up. Mic button. Detected-date chip. Save | CAP-01/02/03/05, REM-01 |
| `/(tabs)/inbox` | Inbox with swipe triage | CAP-01, ORG-04 |
| `/(tabs)/todos` | Today / Upcoming / Overdue / No date / Completed | ORG-02, REM-09 |
| `/(tabs)/notes` | All notes, groups, tags, pinned, archive, trash | ORG-05/06 |
| `/note/[id]` | Editor + reminder + group/tags + important + lock + source link | many |
| `/search` | Keyword search (FTS5) | ORG-07 |
| `/approve/[requestId]` | Login approval with number matching + biometric | SEC-01/02 |
| `/scan` | QR scanner for device linking | SEC-02 |
| `/settings/*` | Office hours and time zone, devices, TOTP, recovery, app lock, vault, export, battery helper | REM-08, SEC-xx, DATA-03 |
| `/onboarding/*` | 3 screens: username → recovery codes → ready (D-52). Permissions are **not** asked here | D-33, SEC-06 |
| `/permissions/notifications`, `/permissions/battery` | In-context permission primer at the first reminder: notifications → battery guide per brand → test reminder (D-52) | REM-05 |
| `/note/[id]/conflict` | Conflict compare (stacked) | DATA-01 |
| `/settings/device/[id]` | Device detail: notification schedule (office hours), revoke | REM-08, SEC-05 |
| `/settings/vault/setup` | Vault setup (PIN, recovery key) | SEC-07 |
| `/missed` | Missed reminders | REM-08 AC4 |

## 3. Speed budget (P1, CAP-02)
- `/capture` must not import heavy modules. Defer search, sync and anything else until after the first frame.
- The widget and tile launch the app with a deep link `cue://capture` (or `cue://capture?voice=1`).
- Saving writes to SQLite + outbox in **one transaction**. Sync runs later (debounced).
- Target: tap → typing ≤ 1 s on a mid-range phone (measure with a release build).

## 4. Native Kotlin modules (learning path order)
| Module | What it does | Key Android APIs |
|---|---|---|
| `cue-tile` | Quick Settings tile "Quick note" → opens `cue://capture` | `TileService`, `startActivityAndCollapse` |
| `cue-share` | Receives `ACTION_SEND` (text, URL, image) → passes `{text, url, sourceApp}` to JS | Intent filters, `Activity.referrer` / calling package |
| `cue-widget` | Home widget: **New**, **Voice**, a **Today** list (tick to complete; the tick goes into a native pending-actions queue) | Jetpack Glance, `GlanceAppWidget`, `ActionCallback` |
| `cue-alarms` | Exact alarms, native store, boot reschedule, notification actions, nag repeats ([reminders-engine §3](reminders-engine.md#3-android-local-alarms-kotlin-module-cuealarms)) | `AlarmManager`, `BroadcastReceiver`, `NotificationCompat`, `BOOT_COMPLETED` |

The JS ↔ native contract is in `packages/shared` types. Each module exposes a small async API, e.g. `CueAlarms.sync(list)`, `CueAlarms.drainPendingActions()`.

## 5. Push (FCM)
- `expo-notifications` gets the **device FCM token**, which is sent to `/devices` as `push_token`.
- **Data-only messages:**
  - `{"type":"sync"}` → run sync (a headless task if the app is in the background).
  - `{"type":"login_request","id":…}` → show a high-priority notification that opens `/approve/[id]`.
- Reminders are **not** sent by FCM. They are local alarms ([reminders-engine](reminders-engine.md)).

## 6. Security on device
- App lock (SEC-04): `expo-local-authentication` on start and on resume after N min. The capture screen can **save** while locked but shows nothing else (SEC-04 AC2).
- Device private key and tokens in `expo-secure-store`. Vault key in secure store with biometric required (SEC-07).
- Screenshots blocked on the approval and vault screens (`FLAG_SECURE`).

## 7. Voice (CAP-05, D-26, D-37)
- `expo-speech-recognition` with on-device recognition preferred, and the `en-IN` and `hi-IN` locales selectable.
- The transcript becomes the note body. Audio is **kept only if the user toggles "Keep audio"** (D-37). It is then compressed and uploaded in ≤ 1.5 MB chunks to D1, with a 100 MB total cap and a usage meter (D-40).
