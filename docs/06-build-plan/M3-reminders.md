# M3 · Reminders that actually fire

> Status: Ready · Last updated: 2026-10-15 · Related: [README](README.md), [links](links.md), [reminders-engine](../05-system-design/reminders-engine.md), decisions D-08, D-09, D-28, D-42, D-44, D-46
>
> Features: REM-01…REM-09 (+ PLAT-04 suggestions for detected dates)
> **Milestone done when:** a test reminder fires **offline** with the app closed, survives a reboot, an important one escalates
> to an alarm-style alert, and the PC shows **no** notification outside office hours. Learning notes are written.

---

### M3-T01 · Time utilities: parse dates, recurrence, office hours (REM-01, REM-08)
- **Depends on:** M1-T01
- **Read first:** [reminders-engine.md §2, §6](../05-system-design/reminders-engine.md#2-creating-a-reminder-rem-01-rem-02)
- **Goal:** pure, fully tested functions in `packages/shared/time`.
- **Steps:**
  1. Add `chrono-node` and `rrule`.
  2. `parseWhen(text, tz, now)` → `{ fire_at, rrule?, matchedText } | null`. Use `tz` (default `Asia/Kolkata`) as the reference.
  3. Recurrence wrapper: map "every day / every Monday 9am / every month on 5th / every weekday" to an RRULE string. Unknown phrases return no RRULE.
  4. `nextOccurrence(rrule, after, tz)`.
  5. `isWithinSchedule({mode, days, start_min, end_min}, instant, tz)`.
  6. Tests (≥ 20), all in IST:
     - "tomorrow 6pm"
     - "in 2 hours"
     - "every Monday 9am"
     - "on 25th"
     - midnight edge cases
     - Mon–Fri 09:00–18:00 at 08:59, 09:00, 17:59, 18:00, Saturday
- **Done when:** all tests pass.
- **Docs:** [chrono](https://github.com/wanasit/chrono) · [rrule](https://github.com/jkbrzt/rrule) · [RFC 5545](https://datatracker.ietf.org/doc/html/rfc5545)
- **Learning note:** `M3-REM-01-time-parsing.md` (concepts: UTC vs local time, time zones, RRULE, why pure functions are easy to test)

### M3-T02 · Reminders API, device schedules, user time zone (REM-02, REM-03, REM-08, REM-09)
- **Depends on:** M3-T01, M2-T12
- **Read first:** [api.md "Reminders" and "Devices"](../05-system-design/api.md#reminders) · [data-model.md reminders](../05-system-design/data-model.md#reminders)
- **Goal:** reminders can be created, changed, acknowledged and listed from any client.
- **Steps:**
  1. Reminder writes go through `applyChange` (so they sync to Android). Also add `POST/PATCH/DELETE /reminders`.
  2. `POST /reminders/:id/ack {action: done|snooze, until?}`: sets `last_ack_at` or `snoozed_until`. A recurring reminder marked done moves `fire_at` to the next occurrence.
  3. `GET /reminders/due?device_id` (applies that device's schedule) and `GET /reminders/missed`.
  4. `PATCH /devices/:id` with a `notification_schedules` body. `GET/PATCH /me` with `timezone`.
  5. Tests: ack stops a due reminder; a recurring reminder advances; a device outside office hours gets nothing from `/due`.
- **Done when:** tests pass, and `api-client` has these functions.
- **Learning note:** `M3-REM-02-reminders-api.md` (concepts: state machines (fired → acked/snoozed), recurrence on the server)

### M3-T03 · Detected-date suggestions (REM-01 AC2, PLAT-04)
- **Depends on:** M3-T01, M3-T02
- **Read first:** [ai-hooks.md §2](../05-system-design/ai-hooks.md#2-suggestions)
- **Goal:** typing "pay bill tomorrow 6pm" shows a chip **"⏰ Tue 6:00 PM"** that the user can accept or remove before saving.
- **Steps:**
  1. Shared helper `detectReminder(text, tz)` → a suggestion payload. Mobile `/capture` and the editor, and the web quick-add, show the chip.
  2. Accepting creates the reminder (and a `suggestions` row with status `accepted`, `source: rules`). Dismissing records `rejected`.
- **Done when:** it works on mobile and web, and is never applied without the user's tap.
- **Learning note:** `M3-PLAT-04-suggestions.md` (concepts: suggest-don't-force UX, extension points for future AI)

### M3-T04 · Kotlin #4a: `cue-alarms`, schedule exact alarms (REM-04) ⭐ wait for "go"
- **Depends on:** M2-T06, M3-T02
- **Read first:** [reminders-engine.md §3](../05-system-design/reminders-engine.md#3-android-local-alarms-kotlin-module-cuealarms) · D-42
- **Goal:** alarms fire at the exact minute, offline, with the app killed, and survive a reboot.
- **Steps:**
  1. `npx create-expo-module@latest --local` → `modules/cue-alarms`. Manifest: `USE_EXACT_ALARM`, `RECEIVE_BOOT_COMPLETED`, `POST_NOTIFICATIONS`.
  2. A native store (SharedPreferences JSON, or a tiny SQLite table) of upcoming alarms: `reminder_id`, `occurrence_at`, title (or "🔒 Locked reminder"), `important`, `nag_every_min`.
  3. JS API:
     - `sync(list)`: replace all scheduled alarms with this list, max 50
     - `cancel(reminderId)`
     - `canScheduleExact()`
  4. Scheduling: `setAlarmClock()` for important reminders, `setExactAndAllowWhileIdle()` for normal ones. Use a stable `requestCode` per reminder.
  5. `AlarmReceiver` shows a basic notification (channels and actions come in the next task). A `BootReceiver` (also `MY_PACKAGE_REPLACED`) reschedules everything from the native store.
  6. JS: after each sync, compute the next 50 occurrences from SQLite → `CueAlarms.sync(list)`.
- **Done when:** set a reminder 2 min ahead → airplane mode → swipe the app away → it fires on time. Reboot the phone with a reminder 5 min ahead → it still fires.
- **Docs:** [Schedule alarms](https://developer.android.com/develop/background-work/services/alarms) · [Exact alarm changes (Android 14)](https://developer.android.com/about/versions/14/changes/schedule-exact-alarms) · [Broadcasts](https://developer.android.com/develop/background-work/background-tasks/broadcasts)
- **Learning note:** `M3-REM-04-exact-alarms.md` (concepts: AlarmManager, PendingIntent, BroadcastReceiver, Doze, why native code is needed when JS isn't running)
- **Watch out:** `adb logcat | Select-String Cue` (PowerShell) to see native logs.

### M3-T05 · Kotlin #4b: notification channels and Done/Snooze actions (REM-02, REM-03)
- **Depends on:** M3-T04
- **Read first:** [reminders-engine.md §3, §5](../05-system-design/reminders-engine.md#5-acknowledging-rem-03)
- **Goal:** reminder notifications have **Done / Snooze 10 min / 1 h / Tomorrow**, and they work without opening the app.
- **Steps:**
  1. Channels: "Reminders" (default) and "Important" (high importance, its own sound). Titles look like "Cue: Pay bill · 6:00 PM" (ux-principles tone).
  2. `ActionReceiver` handles each action natively: update the native store, cancel or reschedule, dismiss the notification, and append a **pending action** `{reminder_id, action, until, at}`.
  3. JS: `CueAlarms.drainPendingActions()` on app start and resume (plus a headless task if feasible) → `localWrite` ack/snooze → sync.
  4. When a pull shows an ack from another device → `cancel` + dismiss the shown notification.
- **Done when:** tap Done on the lock screen → the web app shows the reminder as done after the phone syncs.
- **Docs:** [Notifications](https://developer.android.com/develop/ui/views/notifications) · [Notification channels](https://developer.android.com/develop/ui/compose/notifications/channels) · [Build a notification](https://developer.android.com/develop/ui/compose/notifications/create-notification)
- **Learning note:** `M3-REM-03-notification-actions.md` (concepts: channels, notification actions, handing events from native to JS)

### M3-T06 · Kotlin #4c: nag repeats + alarm-style escalation (REM-07, D-44)
- **Depends on:** M3-T05
- **Read first:** [reminders-engine.md §3 (Nag, Escalation rows)](../05-system-design/reminders-engine.md#3-android-local-alarms-kotlin-module-cuealarms)
- **Goal:** an unhandled important reminder repeats every `nag_every_min` (10). After 3 repeats it shows a **full-screen / heads-up alarm-style alert** that keeps repeating until Done or Snooze.
- **Steps:**
  1. On each fire of an important reminder with no ack: schedule the next repeat and increment the counter in the native store.
  2. At 3 or more repeats: show a notification with `setFullScreenIntent(...)` to an `AlarmActivity` (big Done / Snooze buttons) and alarm audio attributes. If the full-screen permission isn't granted, guide the user to the setting and fall back to a heads-up alert with sound.
  3. Settings screen: the nag interval (default 10).
- **Done when:** an ignored important reminder escalates to the alarm screen on your Oppo/Vivo/Realme/OnePlus phone and stops once Done is tapped anywhere.
- **Docs:** [Build a notification: time-sensitive](https://developer.android.com/develop/ui/compose/notifications/create-notification) · [Notifications](https://developer.android.com/develop/ui/views/notifications)
- **Learning note:** `M3-REM-07-escalation.md` (concepts: full-screen intents, Activities, OS restrictions per Android version)

### M3-T07 · Battery onboarding for Oppo/Vivo/Realme/OnePlus (REM-05, D-46)
- **Depends on:** M3-T04
- **Read first:** [reminders-engine.md §3 "Battery onboarding"](../05-system-design/reminders-engine.md#3-android-local-alarms-kotlin-module-cuealarms)
- **Goal:** an **in-context permission primer** shown the first time a reminder is set (D-52): notifications → a battery guide for the phone's brand → **Test reminder**.
- **Design:** [ui-ux-design §6.14](../07-design/ui-ux-design.md#614-permission-primer-and-battery-guide).
- **Steps:**
  1. Detect the brand (`Build.MANUFACTURER` via a tiny module function, or `expo-device`).
  2. Brand-specific steps with deep links where possible: Auto-launch, Allow background activity, Battery: Unrestricted, Lock app in recents. Request `REQUEST_IGNORE_BATTERY_OPTIMIZATIONS`.
  3. "Test reminder" schedules an alarm in 10 s. Settings shows a ⚠ warning if exact alarms or notifications are disabled.
- **Done when:** after the guide, a reminder still fires after leaving the phone idle for 1+ hour.
- **Docs:** [Doze and App Standby](https://developer.android.com/training/monitoring-device-state/doze-standby) · [dontkillmyapp: Oppo](https://dontkillmyapp.com/oppo) · [Vivo](https://dontkillmyapp.com/vivo) · [Realme](https://dontkillmyapp.com/realme) · [OnePlus](https://dontkillmyapp.com/oneplus)
- **Learning note:** `M3-REM-05-battery.md` (concepts: Doze, OEM restrictions, why reliability needs user setup)

### M3-T08 · Web Push for the PWA (REM-06)
- **Depends on:** M2-T13
- **Read first:** [hosting §2](../05-system-design/hosting-and-distribution.md#2-libraries-confirmed-for-workers)
- **Goal:** the web app (installed PWA) receives reminder notifications from the server.
- **Steps:**
  1. Generate VAPID keys once (store the private key as a Worker secret, put the public key in the web config).
  2. Web: register a service worker (vite-plugin-pwa `injectManifest` mode), ask for notification permission on a button click, `pushManager.subscribe` → `POST /push/web/subscribe`.
  3. The service worker shows the notification (tag = `reminderId:fire_at`) with **Done / Snooze 1h** actions → calls `/reminders/:id/ack`.
  4. Worker `services/webpush.ts` using **@pushforge/builder** (not `web-push`).
- **Done when:** a reminder created on the phone shows as a desktop notification in Chrome (within office hours).
- **Docs:** [Push notifications overview](https://web.dev/articles/push-notifications-overview) · [Push API](https://developer.mozilla.org/en-US/docs/Web/API/Push_API) · [@pushforge/builder](https://www.npmjs.com/package/@pushforge/builder) · [vite-plugin-pwa](https://vite-pwa-org.netlify.app/guide/)
- **Learning note:** `M3-REM-06-web-push.md` (concepts: service workers, VAPID, push subscription, payload encryption)

### M3-T09 · Cron dispatcher + daily maintenance (REM-06, REM-07, REM-08)
- **Depends on:** M3-T02, M3-T08
- **Read first:** [reminders-engine.md §4, §7](../05-system-design/reminders-engine.md#4-server-the-cron-dispatcher-every-minute) · D-43
- **Goal:** every minute, due reminders go out by Web Push to devices within their schedule, PC nags repeat, and recurrences advance.
- **Steps:**
  1. `wrangler.jsonc` triggers: `"* * * * *"` and `"0 21 * * *"` (02:30 IST maintenance). One `scheduled()` handler branches on `event.cron`.
  2. Dispatcher: the query from reminders-engine §4 with `LIMIT 50`, **at most 20 sends per run**, then update `server_dispatched_at`, `repeat_count`, `last_fired_at`, and advance non-nag recurring reminders.
  3. Maintenance: purge Trash older than 30 days, prune `change_log`/`domain_events` older than 90 days, expire login requests and old rate-limit windows.
  4. Measure CPU locally and in prod (`wrangler tail`), and keep it under 10 ms.
- **Commands:** `pnpm --filter @cue/api exec wrangler dev --test-scheduled` then `curl "http://localhost:8787/__scheduled?cron=*+*+*+*+*"`
- **Done when:** tests for the dispatcher logic pass, and a due reminder arrives on the PC only within office hours.
- **Docs:** [Cron Triggers](https://developers.cloudflare.com/workers/configuration/cron-triggers/) · [Real-time logs](https://developers.cloudflare.com/workers/observability/logs/real-time-logs/) · [Workers limits](https://developers.cloudflare.com/workers/platform/limits/)
- **Learning note:** `M3-T09-cron-dispatcher.md` (concepts: scheduled jobs, batching under CPU limits, idempotent dispatch)

### M3-T10 · Reminder UI everywhere (REM-09, REM-08 settings, Missed list)
- **Depends on:** M3-T02, M3-T05
- **Goal:** Today / Upcoming / Overdue views, a reminder picker, Settings for office hours and time zone, and a **Missed reminders** list, on mobile and web.
- **Steps:**
  1. A reminder picker: quick chips (Later today, Tonight, Tomorrow 9am, Next week) + date/time + repeat.
  2. Settings → Devices → each device's schedule (`always` / `office_hours` / `never`, days, start, end). Settings → Time zone (default India).
  3. The Missed list (REM-08 AC4) on mobile and web.
- **Done when:** every REM acceptance criterion in features.md can be shown.
- **Learning note:** `M3-REM-09-reminder-ui.md`

### M3-T11 · M3 acceptance tests
- **Read first:** [reminders-engine.md §8](../05-system-design/reminders-engine.md#8-test-cases)
- **Goal:** run all 6 test cases from reminders-engine §8 on your real phone and PC, and record the results in the learning note.
- **Done when:** all 6 pass (case 3 now means alarm-style escalation, not Telegram). The M3 learning notes are written, and the concepts are ticked.
- **Learning note:** `M3-T11-reminder-acceptance.md` (concepts: testing on real devices, what to do when an OEM still kills the app)
