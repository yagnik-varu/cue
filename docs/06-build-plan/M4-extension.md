# M4 · Browser extension (Chrome MV3)

> Status: Ready · Last updated: 2026-10-15 · Related: [README](README.md), [links](links.md), [extension design](../05-system-design/extension.md), decisions D-38, D-39
>
> Features: EXT-01…EXT-05, CAP-07, CAP-09 (capture queue), CAP-10, REM-06/REM-08 (extension side)
> **Milestone done when:** revisiting a page shows the badge with the right note count, offline captures sync later,
> and reminders appear only within office hours. Learning notes are written.

---

### M4-T01 · Extension auth: sign in with your phone (EXT-03)
- **Depends on:** M0-T09, M2-T10, M2-T11
- **Read first:** [extension.md §1, §5](../05-system-design/extension.md#5-auth-ext-03)
- **Goal:** the extension signs in with the same phone-approval or QR flow as the web app.
- **Steps:**
  1. Manifest permissions: `storage`, `alarms`, `notifications`, `contextMenus`, `sidePanel`, `activeTab`, `tabs`, plus a host permission for the API origin only.
  2. The device key is non-extractable WebCrypto in IndexedDB. The access token goes in `chrome.storage.session`, the refresh token in `chrome.storage.local`.
  3. Reuse `@cue/shared/api-client`. The popup shows only a "Sign in to Cue" button that opens the **extension tab page** `signin.html`, where the number (or the QR) is shown (D-52, [ui-ux-design §8.3](../07-design/ui-ux-design.md#83-sign-in-tab-page)).
- **Done when:** the extension shows "Signed in as <username>" after you approve on the phone, and the session survives a Chrome restart.
- **Docs:** [WXT installation](https://wxt.dev/guide/installation.html) · [chrome.storage](https://developer.chrome.com/docs/extensions/reference/api/storage)
- **Learning note:** `M4-EXT-03-extension-auth.md` (concepts: extension storage areas, service worker lifecycle, origins)

### M4-T02 · Popup quick note + shortcut + capture queue (EXT-01, CAP-09, CAP-10)
- **Depends on:** M4-T01, M3-T03
- **Read first:** [extension.md §2, §6](../05-system-design/extension.md#6-offline-capture-queue-d-39-cap-09)
- **Goal:** `Alt+Shift+N` → type → Enter → saved to the Inbox, with the current page attached. Offline captures are queued. Design: [ui-ux-design §8.2](../07-design/ui-ux-design.md#82-popup).
- **Steps:**
  1. The popup: a textarea with autofocus, the current tab's title and URL as a removable chip, and the detected-date chip (M3-T03).
  2. `commands`: `_execute_action` = `Alt+Shift+N`; a second command `open-side-panel` = `Alt+Shift+S` (D-52).
  3. `POST /notes` with an `op_id`. On a network error, push to `chrome.storage.local.captureQueue` and show "Saved offline. Will sync."
  4. Flush the queue on the `online` event, on each alarm tick (M4-T06), and when the popup opens.
- **Done when:** with Wi-Fi off, save 2 notes → turn Wi-Fi on → both appear on the phone once, with no duplicates.
- **Docs:** [chrome.commands](https://developer.chrome.com/docs/extensions/reference/api/commands) · [chrome.storage](https://developer.chrome.com/docs/extensions/reference/api/storage)
- **Learning note:** `M4-EXT-01-popup-queue.md` (concepts: idempotent retries, offline queues, extension UI surfaces)

### M4-T03 · Context menu "Save selection" + text-fragment link (EXT-02)
- **Depends on:** M4-T02
- **Read first:** [extension.md §3](../05-system-design/extension.md#3-save-selection-with-a-text-fragment-ext-02)
- **Goal:** right-click on selected text → "Save selection to Cue" saves the text plus a link that jumps back to the exact passage.
- **Steps:**
  1. Register context menus on install: "Save selection to Cue" (selection) and "Save page + remind me…" (page).
  2. Build the text fragment: `url#:~:text=<start>[,<end>]` (first and last ~5 words for long selections, URL-encoded).
  3. Store `source_url`, `source_title`, `source_fragment`, and `page_key` (M4-T04).
- **Done when:** clicking the note's source link in the web app opens the page scrolled to and highlighting the passage.
- **Docs:** [chrome.contextMenus](https://developer.chrome.com/docs/extensions/reference/api/contextMenus) · [Text fragments (MDN)](https://developer.mozilla.org/en-US/docs/Web/URI/Reference/Fragment/Text_fragments)
- **Learning note:** `M4-EXT-02-text-fragments.md` (concepts: URL fragments, context menus, the selection API)

### M4-T04 · Page-linked notes + badge (EXT-04)
- **Depends on:** M4-T02, M2-T12
- **Read first:** [extension.md §4](../05-system-design/extension.md#4-page-linked-notes-and-badge-ext-04)
- **Goal:** the toolbar badge shows how many notes you have for the current page or site.
- **Steps:**
  1. `packages/shared`: `pageKey(url, match: "page" | "site")` normalisation with tests (lowercase host, strip `www.`, drop `utm_*`/`fbclid`/`gclid` and the hash).
  2. API: `GET /notes/page-keys` → `{ [page_key]: count }` and `GET /notes/by-page?key=`.
  3. The extension caches the page-keys map in `chrome.storage.local` (refresh every 5 min and after saves). On `tabs.onActivated` / `tabs.onUpdated` → compute both keys → `action.setBadgeText`.
  4. Each note can choose `page_match` (exact page or whole site).
- **Done when:** save a note on a page → leave → come back → the badge shows "1" with no API call on tab switch.
- **Docs:** [chrome.action](https://developer.chrome.com/docs/extensions/reference/api/action)
- **Learning note:** `M4-EXT-04-page-badge.md` (concepts: URL normalisation, client caching to save free-tier requests)

### M4-T05 · Side panel (EXT-05)
- **Depends on:** M4-T04
- **Goal:** the side panel shows quick capture, **notes for this page** first, then the Inbox, plus a reserved (hidden) chat slot.
- **Steps:** add a WXT side panel entrypoint, reuse the popup components, and follow the active tab with `chrome.tabs` events.
- **Done when:** the panel content changes as you switch tabs.
- **Docs:** [chrome.sidePanel](https://developer.chrome.com/docs/extensions/reference/api/sidePanel)
- **Learning note:** `M4-EXT-05-side-panel.md`

### M4-T06 · Reminder tick + notifications with office hours (REM-06, REM-08)
- **Depends on:** M4-T01, M3-T02
- **Read first:** [extension.md §7–§8](../05-system-design/extension.md#7-reminders-and-office-hours-rem-06-rem-08)
- **Goal:** the extension shows due reminders as Chrome notifications **only within office hours**.
- **Steps:**
  1. `chrome.alarms.create("tick", { periodInMinutes: 1 })`. On each tick: flush the capture queue, then `GET /reminders/due?device_id`.
  2. Check `isWithinSchedule` locally too. If allowed → `chrome.notifications.create` (id `reminderId:fire_at`, buttons Done / Snooze 1h). Otherwise → only the badge.
  3. Button clicks → `/reminders/:id/ack`. Clear notifications already acked elsewhere on the next tick.
- **Done when:** a reminder at 20:00 IST does **not** pop up on the PC, and one at 11:00 on a weekday does.
- **Docs:** [chrome.alarms](https://developer.chrome.com/docs/extensions/reference/api/alarms) · [chrome.notifications](https://developer.chrome.com/docs/extensions/reference/api/notifications)
- **Learning note:** `M4-REM-08-extension-reminders.md` (concepts: polling vs push, MV3 service workers that sleep, quiet hours)

### M4-T07 · M4 acceptance
- **Goal:** prove every extension feature works in your office Chrome.
- **Done when:**
  - every EXT acceptance criterion in [features.md](../02-product/features.md) can be shown
  - the extension is rebuilt and reloaded unpacked
  - the M4 learning notes are written and the concepts are ticked
