# Browser extension design (Chrome MV3)

> Status: Draft for review · Last updated: 2026-10-15 · Related: [architecture](architecture.md), [auth-flows](auth-flows.md), [reminders-engine](reminders-engine.md), features EXT-01…EXT-05, CAP-09, CAP-10, decisions D-38, D-39

## 1. Build and install
- Framework: **WXT** + React + TypeScript (proposed) in `apps/extension`.
- Distribution: build the folder → Chrome → `chrome://extensions` → Developer mode → **Load unpacked** (D-38). Edge works the same way. Firefox is later.
- Manifest permissions (minimum): `storage`, `alarms`, `notifications`, `contextMenus`, `sidePanel`, `activeTab`, `tabs` (to read the URL for badges). Host permission only for the API origin. **No `<all_urls>` content script in MVP.**

## 2. Parts
| Part | What it does | Feature |
|---|---|---|
| **Popup** | One text box, cursor ready. Enter saves. The current page (title + URL) is attached and can be removed. Detected dates appear as a chip | EXT-01, CAP-10, REM-01 |
| **Shortcut** | `commands` → `_execute_action` (default `Alt+Shift+N`; side panel `Alt+Shift+S`; user-changeable at `chrome://extensions/shortcuts`; D-52) | EXT-01 |
| **Context menu** | "Save selection to Cue" and "Save page + remind me…" | EXT-02 |
| **Side panel** | Notes for this page or site, the Inbox, quick capture, and a reserved slot for future chat | EXT-05 |
| **Badge** | Count of notes linked to the active tab's page or site | EXT-04 |
| **Service worker** | Auth tokens, capture queue, `chrome.alarms` (refresh + due reminders), notifications | CAP-09, REM-06/08 |

## 3. Save selection with a text fragment (EXT-02)
- The selected text comes from the context-menu `info.selectionText`. No content script is needed.
- Build the link `url + "#:~:text=" + encodeURIComponent(start)[,end]`. For long selections, use the first and last ~5 words as `start,end`.
- Store `source_url`, `source_title`, `source_fragment`, and `page_key` (§4).

## 4. Page-linked notes and badge (EXT-04)
- `page_key` normalisation in `packages/shared`:
  - lowercase host
  - strip `www.`
  - drop tracking params (`utm_*`, `fbclid`, `gclid`)
  - drop the hash
  - for `page` match: host + path (+ meaningful query); for `site` match: host only
- On `tabs.onActivated` and `tabs.onUpdated (status=complete)`: compute both keys → look them up in a **local cache** (`chrome.storage.local`, map `page_key → count`, refreshed every 5 min and after saves via `GET /notes/page-keys`) → `action.setBadgeText`.
- This uses no API call per tab switch, which saves free-tier requests and is instant.
- Opening the popup or side panel on that page lists those notes first.

## 5. Auth (EXT-03)
- First run: the popup shows one **"Sign in to Cue"** button that opens an **extension tab page** (`signin.html`), because popups close when focus leaves (D-52). The tab runs [auth-flows §2](auth-flows.md#2-sign-in-on-web-or-extension-approve-on-phone-with-number-matching-sec-01), with a QR option (§3).
- The device key is a non-extractable WebCrypto key in IndexedDB (the extension origin). Tokens are kept in `chrome.storage.session` (access) and `chrome.storage.local` (refresh).

## 6. Offline capture queue (D-39, CAP-09)
- If `POST /notes` fails with a network error, append `{op_id, payload, created_at}` to `chrome.storage.local.captureQueue` and show **"Saved offline. Will sync."**
- Retry on the `online` event, on each alarm tick, and when the popup opens. `op_id` makes retries safe.

## 7. Reminders and office hours (REM-06, REM-08)
- A `chrome.alarms` "tick" every 1 min (minimum allowed: 30 s) calls `GET /reminders/due?device_id=`.
- If `isWithinSchedule(thisDevice.schedule, now, tz)` → `chrome.notifications.create` with buttons **Done** and **Snooze 1 h**. Otherwise → no notification (the badge still updates), and it goes to "Missed reminders".
- Notification ids are `reminderId:fire_at`, so a later Done from another device clears them on the next tick.

## 8. Budget
- One tick per minute is 1,440 requests a day per extension, plus the badge cache refresh about every 5 min (288 a day). Small relative to the free limit of 100,000 Workers requests a day ([hosting](hosting-and-distribution.md#free-tier-limits)). Later option: Web Push into the extension service worker (Chrome 121+) instead of polling. Ticks pause when signed out.
