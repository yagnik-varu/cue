# Reminders engine

> Status: Draft for review · Last updated: 2026-10-15 · Related: [data-model](data-model.md#reminders), [sync](sync.md), decisions D-07, D-08, D-09, D-28, D-44, D-46, features REM-01…REM-09

## 1. Rule of thumb
**The phone fires its own alarms. The server handles only the channels the phone can't** (web PWA, extension). A reminder must fire on Android **even with no internet and the app closed** (REM-04).

## 2. Creating a reminder (REM-01, REM-02)
- `packages/shared/time.parseWhen(text, tz)` wraps **chrono-node** with a reference time in the user's time zone (default `Asia/Kolkata`, D-28). It returns `{fire_at (UTC ms), rrule?, matchedText}`.
- Recurrence phrases ("every Monday 9am", "daily", "every month on 5th") are mapped to an **RFC 5545 RRULE** by a small wrapper. Unsupported phrases → the user picks from the recurrence picker.
- The detected date is shown as a **Suggestion** (`type: reminder_date`, `source: rules`). The user accepts or removes it before saving (PLAT-04, REM-01 AC2).

## 3. Android: local alarms (Kotlin module `CueAlarms`)
| Concern | Design |
|---|---|
| Scheduling | `AlarmManager.setAlarmClock()` for important reminders (most reliable; shows the alarm icon). `setExactAndAllowWhileIdle()` for normal ones. Permission: **`USE_EXACT_ALARM`** (D-42; allowed for sideloaded apps) |
| Works without JS | `CueAlarms` keeps its **own small native store** of upcoming alarms (`reminder_id`, `occurrence_at`, title or 🔒, `important`, `nag_every_min`). Alarms can fire and be rescheduled after reboot without starting React Native |
| Reboot / update | A `BOOT_COMPLETED` + `MY_PACKAGE_REPLACED` receiver reschedules everything from the native store |
| Notification | Channel "Reminders" (normal) and "Important" (high priority, distinct sound). Title is "Cue: <title> · 6:00 PM". Actions: **Done**, **Snooze 10 min**, **1 h**, **Tomorrow** |
| Actions without opening the app | A `BroadcastReceiver` handles Done and Snooze natively: it updates the native store, cancels or reschedules, and writes a **pending action** that the JS sync layer turns into an outbox change the next time JS runs (or via a headless task) |
| Nag (REM-07) | If important and not handled → the native side schedules a repeat in `nag_every_min` (default 10) until Done or Snooze |
| Escalation (REM-07 AC3, D-44) | After 3 unhandled repeats → **alarm-style alert**: `setFullScreenIntent` + alarm audio attributes on the "Important" channel, plus `setAlarmClock` for the next repeat. Android 14+ restricts full-screen intents to alarm and calling apps by default. If the permission isn't granted, the app guides the user to Settings → Special app access → Full-screen notifications; otherwise it falls back to a heads-up notification with the alarm sound |
| Sync link | After every pull, JS calls `CueAlarms.sync(upcomingList)` with the next 50 occurrences. Changes from other devices (Done on web) cancel the alarm and dismiss any shown notification |
| Phone quiet hours | Default `always`. The phone is the always-on channel. The user can set a schedule for the phone too (REM-08) |

**Battery onboarding (REM-05):**
- Detect the manufacturer. **First targets: Oppo (ColorOS), Vivo (Funtouch/OriginOS), Realme (realme UI), OnePlus (OxygenOS)** (D-46). Then Samsung and Xiaomi.
- Guide the user to the brand's *Auto-launch / Allow background activity / Battery: Unrestricted* settings and *Lock app in recents*.
- Show a short guide with deep links to battery or auto-start settings, and request `REQUEST_IGNORE_BATTERY_OPTIMIZATIONS`.
- A **"Test reminder"** button schedules an alarm 10 s later.
- A warning appears in Settings if a permission is later revoked.

## 4. Server: the cron dispatcher (every minute)
`scheduled()` in the Worker, triggered by a Cron Trigger `* * * * *`:
```
now = Date.now()
due = SELECT reminders r JOIN notes n
      WHERE r.fire_at <= now AND r.deleted = 0 AND n.deleted = 0 AND n.done_at IS NULL
        AND (r.last_ack_at IS NULL OR r.last_ack_at < r.fire_at)
        AND (r.snoozed_until IS NULL OR r.snoozed_until <= now)
        AND r.server_dispatched_at IS NULL or (r.nag = 1 AND r.server_dispatched_at <= now - r.nag_every_min*60000)
      LIMIT 50
for each reminder:
   for each web device of the user with a push subscription:
       if isWithinSchedule(device.schedule, now, user.timezone): send Web Push (tag = reminderId:fire_at)
   set server_dispatched_at = now; repeat_count += 1
   if not nag and recurring: fire_at = next occurrence (rrule in user.timezone); reset counters
```
- **The extension doesn't need push.** It polls `GET /reminders/due?device_id=` every minute with `chrome.alarms`. The server applies the device's schedule (REM-08), and the extension checks it again locally. Outside office hours → **no notification** (badge only).
- **The Android channel is not sent by the server** (local alarms). The server only sends an FCM **sync nudge** when a reminder is created or changed elsewhere.
- Columns used: `server_dispatched_at`, `repeat_count`, `last_fired_at` (see [data-model](data-model.md#reminders)).

## 5. Acknowledging (REM-03)
- **Done or Snooze from any channel** (Android notification, web notification click, extension) → `applyChange` sets `last_ack_at` or `snoozed_until` (last-writer-wins).
- Other devices then stop:
  - Android: the pull triggers `CueAlarms` to cancel and dismiss.
  - Web: Web Push sends a silent replacement with the same `tag` to close the notification.
  - Extension: the next poll clears it.
- A recurring reminder marked Done completes **that occurrence**: `fire_at` moves to the next occurrence. Marking the *note* done stops the series.

## 6. Quiet hours and office hours (REM-08, D-28)
- `isWithinSchedule(schedule, instant, tz)` lives in `packages/shared/time` and is used by the server, the extension and the web app.
- A schedule is `{mode: always|office_hours|never, days:[1..7], start_min, end_min}` in the user's time zone (default Asia/Kolkata, UTC+05:30, no DST).
- Example "Work PC": `office_hours, days 1–5, 540–1080` → notifications Mon–Fri 09:00–18:00 IST only.
- A reminder that is due outside a device's window is **not shown there later either**. It shows in the **"Missed reminders"** list in every app (REM-08 AC4), and the phone covers it.

## 7. Free-tier budget
- The cron runs 1,440 times a day, with one indexed query plus a few sends per run. Within the free limits ([hosting](hosting-and-distribution.md#free-tier-limits)), but **CPU is 10 ms per run** (D-43).
- Cap at **20 sends per run**; the rest carry over to the next minute. The query uses `LIMIT 50`.

## 8. Test cases
1. Phone in airplane mode, app swiped away → a reminder fires at the exact minute.
2. Reboot the phone → pending reminders still fire.
3. Important reminder ignored → it repeats every 10 min. After 3 repeats → alarm-style alert (full-screen or heads-up, alarm sound) that keeps repeating. Done on the PC → the phone stops within seconds (FCM sync nudge).
4. Work PC schedule Mon–Fri 9–18 IST. A reminder at 20:00 → **no** PC notification; the phone fires; the PC shows it under "Missed reminders" the next morning.
5. "Every Monday 9am" → fires on consecutive Mondays in IST. Done on one Monday doesn't stop the next one.
6. Locked-note reminder → the notification text is "🔒 Locked reminder" on every channel.
