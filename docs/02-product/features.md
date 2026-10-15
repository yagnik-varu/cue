# Feature catalogue

> Status: Living · Last updated: 2026-10-15 · Related: [decisions](decisions.md), [UX principles](ux-principles.md), [roadmap](../03-roadmap.md), [zero-cost guide](zero-cost.md)
>
> **How to use this file**
> - Each feature has a stable **ID**. Reference it in tasks, commits and code. Don't renumber; mark removed features *Dropped*.
> - **Priority:** `MVP` (build first), `v2` (next), `Future` (later/optional).
> - **Platforms:** A = Android, W = Web, E = Browser extension, B = messaging bot (Future, D-44), S = Server.
> - **Cost:** every feature must follow the zero-cost rule (D-21). The 💰 notes say how to keep it free.
> - **Acceptance criteria (AC)** describe *behaviour*, not implementation. The system design decides the how.
> - **No AI in the MVP (D-24).** AI features are kept in this file with priority `Future`. The MVP must still be *AI-ready* (PLAT-04, [future/ai-integration.md](../future/ai-integration.md)).

## Module overview
| Module | Prefix | MVP features |
|---|---|---|
| Capture | CAP | 9 (+1 Future) |
| Organise | ORG | 6 (+1 Future) |
| Remind | REM | 9 (+1 v2) |
| AI chatbot and search | AI | 0 (6 Future, D-24) |
| Browser extension | EXT | 5 |
| Messaging bot | BOT | 0 (3 Future, D-44) |
| Security and login | SEC | 8 |
| Data safety | DATA | 4 |
| Platform basics | PLAT | 4 |
| Future backlog | FUT | — |

---

## CAP: Capture (goal: under 2 s from thought to saved note)

### CAP-01 · Universal Inbox · MVP · A W E B S
Every capture from every surface lands in one **Inbox**, with no title, folder or tag required.
- AC1: A note can be saved with only body text (or only audio).
- AC2: New captures appear at the top of the Inbox on all devices after sync.
- AC3: Each Inbox item shows its source icon (widget, share, voice, extension, bot, web) and its capture time.
- AC4: An item leaves the Inbox when it is grouped, turned into a todo, archived or deleted (see ORG-04).

### CAP-02 · Android home-screen widget · MVP · A
One-tap buttons: **New note**, **Voice note**, and a **Today's todos** list.
- AC1: Tapping New note opens a blank editor with the keyboard up, ready to type in under 1 s. The editor does not wait for sync or the network.
- AC2: The todo list can be ticked from the widget.

### CAP-03 · Android Quick Settings tile · MVP · A
A tile labelled "Quick note" (≤ 18 characters) opens the capture editor. It works from the lock screen (the editor unlocks after biometric if app lock is on).
- AC1: Tile → editor in ≤ 2 taps.

### CAP-04 · Android share target ("Share to Notes") · MVP · A
Share text, links or images from any app into a new Inbox note.
- AC1: Shared content is saved with its **source app name**, and the URL if it is a link (CAP-10).
- AC2: Saving needs one tap. An optional "add reminder" chip is shown.

### CAP-05 · Voice note → text · MVP · A W
Hold to record. The device's own speech recognition turns it into text (D-26).
- AC1: The transcript is saved as the note body. The user can choose whether to keep the audio file.
- AC2: Natural-language dates in the transcript are detected (REM-01), e.g. "call Ravi tomorrow 6pm" → reminder at tomorrow 18:00.
- *Future (D-24):* one ramble split into several todos ("buy milk, call Ravi, pay bill" → 3 todos); voice messages from a future bot (BOT-01).
- 💰 Android on-device speech recognition and the browser Web Speech API are free. No speech-to-text server.

### CAP-06 · Web quick-add bar + global hotkey · MVP · W
An always-visible "Capture…" bar. Keyboard shortcuts `Q` (when not typing) and `Alt+N` (anywhere) open quick add (D-52).
- AC1: Type + Enter saves to the Inbox. Natural-language dates in the text create a reminder (REM-01).

### CAP-07 · Extension quick note · MVP · E
Covered by EXT-01. Listed here so all capture surfaces are visible in one place.

### CAP-08 · Messaging-bot capture · Future (D-44) · B
Covered by BOT-01.

### CAP-09 · Offline capture and outbox · MVP · A (E: capture queue only)
> Web app is online-only in MVP (D-39). Web offline → FUT-16.
- AC1 (Android): With no network, capturing, editing and viewing existing notes all work.
- AC1b (Extension): With no network, a quick note or saved selection is queued and sent automatically when back online. Viewing needs the network.
- AC2: Offline changes queue in an outbox and sync automatically when the device is back online. The user sees a small "pending sync" indicator.
- AC3: No offline action is ever lost (see DATA-01).

### CAP-10 · Auto source context · MVP · A W E B
Each capture automatically stores its **source**: page URL and title (extension, shared links), source app (Android share), or channel (bot, web, widget).
- AC1: The source is shown on the note and links back (for web pages, to the exact highlighted passage when available, EXT-02).
- AC2: No location is stored (D-10).

---

## ORG: Organise (goal: no organising work at capture time)

### ORG-01 · Note ↔ todo in one tap · MVP · A W E
- AC1: Any note can become a todo (and back) with one action (a swipe on Android, a button or shortcut on web).
- AC2: A checklist line inside a note can be made into its own todo, linked to the original note.

### ORG-02 · Mark complete · MVP · A W E B
- AC1: Completing a todo removes it from active views. It stays in a "Completed" view and can be restored.
- AC2: Completing it from a notification, the widget or the bot (reply "done") updates every device.

### ORG-03 · AI-suggested groups and tags · Future (D-24) · A W
- AC1: For each Inbox item the AI suggests a group (e.g. "Shopping", "Work – Project X") and up to 3 tags.
- AC2: The user accepts with one tap or edits. **Nothing is moved without confirmation** (principle P3).
- AC3: Locked notes never get AI suggestions (SEC-07).
- MVP prep: suggestions are shown through the generic *Suggestion* model (PLAT-04), so the triage UI can show them later without redesign.

### ORG-04 · Inbox triage (swipe or keyboard) · MVP · A W
Process the Inbox quickly: **→ Todo**, **→ Group**, **Archive**, **Delete**, **Remind me…**
- AC1: Each action is one swipe or one key.
- AC2: An "Inbox zero" empty state confirms you're done.

### ORG-05 · Groups (collections) and tags · MVP · A W E
- AC1: Users can create, rename and delete groups. A note belongs to at most one group and can have many tags.
- AC2: Views show all notes, todos (today, upcoming, no date), each group, and archive.

### ORG-06 · Pin, archive, trash · MVP · A W E
- AC1: Pinned notes stay on top. Archived notes are hidden but searchable.
- AC2: Deleted notes go to **Trash for 30 days** and can be restored (DATA-02).

### ORG-07 · Keyword search (offline) · MVP · A W E
- AC1: Instant search over titles, bodies, tags, group names and source URLs.
- AC2: Works offline on Android (over notes already synced). Web and extension search on the server (online).
- AC3: Locked notes are searched only after the vault is unlocked, on the device.
- Built behind a search interface so meaning-based search (AI-02) can be added later (PLAT-04).

---

## REM: Remind (goal: a reminder that doesn't fire is a bug)

### REM-01 · Natural-language dates · MVP · A W E B
- AC1: Recognises "tomorrow 6pm", "in 2 hours", "every Monday 9am", "on 25th", "next Friday" in typed text, voice and bot messages.
- AC2: The detected date is highlighted and can be accepted or removed before saving.
- 💰 A free open-source date parser (rule-based, no AI).

### REM-02 · Time-based and recurring reminders · MVP · A W E B S
- AC1: One-time and recurring (daily, weekly, monthly, custom) reminders on notes and todos.
- AC2: Notification actions: **Done**, **Snooze** (10 min, 1 h, this evening, tomorrow morning, custom).

### REM-03 · Reminder sync across devices · MVP · S
- AC1: A reminder marked done or snoozed on one device stops alerting on all other devices.

### REM-04 · Reliable on-device alarms (Android) · MVP · A
- AC1: Reminders are scheduled locally on the phone so they fire **offline** and without a sync.
- AC2: They survive a reboot and an app update.

### REM-05 · Battery-optimisation onboarding · MVP · A
- AC1: The first-run setup detects aggressive battery savers (e.g. Samsung, Xiaomi) and guides the user to exempt the app, with a "test reminder now" button.
- AC2: The app warns if exact-alarm or notification permission is missing.

### REM-06 · Delivery channels · MVP · A W E B
Android local alarm notification, browser/desktop notification (web PWA via Web Push, extension). *(Telegram → Future, D-44.)*
- AC1: Each reminder is delivered on the channels the user enabled, following routing rules (REM-08).
- 💰 Local Android alarms (free), Firebase Cloud Messaging for sync nudges (free), standard Web Push (free).

### REM-07 · Important = nag until done · MVP · A W E B
- AC1: The user can flag a note or todo as **Important**.
- AC2: An important reminder repeats every N minutes (default 10, user-adjustable) until it is Done or Snoozed.
- AC3: If it is still not handled after 3 repeats, it **escalates to an alarm-style alert** on the phone (full-screen or heads-up, alarm sound, "Important" channel) that keeps repeating until Done or Snoozed. The PC shows it too if within office hours (D-44).
- AC4: Normal (non-important) reminders alert once (D-09).

### REM-08 · Per-device quiet hours and routing (office hours) · MVP · A W E S
- AC1: Each device has a notification schedule. Example: **Work PC browser: Mon–Fri 09:00–18:00 only**.
- AC2: Outside a device's window, reminders are **not shown on that device**. They go to the phone only.
- AC3: The user can set the schedule per device from the device list (SEC-05).
- AC4: Missed reminders show in an in-app "Reminders you missed" list the next time the app is opened.
- AC5: **Office hours are set by the user** in Settings (D-28): the working days, plus a start and end time. They can be edited at any time and applied to chosen devices (e.g. "Work PC"). A suggested default is Mon–Fri 09:00–18:00, which the user can change.
- AC6: **Time zone defaults to India (Asia/Kolkata, IST, UTC+05:30)** and can be changed in Settings. Office hours and all reminder times follow the selected time zone.

### REM-09 · Upcoming and overdue view · MVP · A W E
- AC1: A "Today / Upcoming / Overdue" list of todos with reminders. Overdue items are highlighted.

### REM-10 · "Next time I'm on my laptop" · v2 · W E S
- AC1: The user can choose "remind me on my PC" with no time. The reminder fires the next time the extension or web app is active **within office hours**.

---

## AI: Chatbot and search (all Future, D-24)
> Not built in MVP. Kept here as the target spec. How to add these later: [future/ai-integration.md](../future/ai-integration.md).

### AI-01 · Ask your notes (chat with citations) · Future · A W E B
- AC1: Questions like "what did I note about the car service?" return an answer **with links to the source notes**.
- AC2: If nothing relevant is found, it says so. It never invents notes.
- AC3: Available in the app and the extension side panel (EXT-05); later also a messaging bot (BOT-01).
- 💰 Must follow the zero-cost rule (D-21). Options are recorded in D-23 and [zero-cost.md §2](zero-cost.md#2-ai-at-zero-cost-decision-d-23-open-question-q-01).

### AI-02 · Meaning-based search · Future · A W E
- AC1: Meaning-based search, so "doctor" finds a note saying "clinic appointment".
- AC2: Plugs into the same search interface as keyword search (ORG-07).

### AI-03 · Create notes, todos and reminders by chat · Future · A W E B
- AC1: "Remind me to call mom tomorrow 6pm" creates a todo and reminder and shows a confirmation card with **Undo**.

### AI-04 · "Organise my Inbox" command · Future · A W B
- AC1: The chatbot proposes a batch plan (group these 5, make these 3 todos, archive these 2). The user approves all, some, or none.

### AI-05 · AI-visible indicator · Future · A W E
- AC1: Each note clearly shows whether AI can read it. Locked notes show 🔒 "AI never sees this".
- MVP prep: the `ai_visible` / `locked` flags exist in the data model from day one (PLAT-04).

### AI-06 · AI privacy rules · Future (rule applies from day one) · S
- AC1: Locked notes are never sent to any AI model.
- AC2: Notes are never used for model training. When a free cloud AI tier that may use the data is enabled, the user must have opted in with a clear warning (D-23).
- AC3: AI features can be fully switched off in settings.

---

## EXT: Browser extension (Chrome first; Firefox and Edge later)

### EXT-01 · Quick-note popup + keyboard shortcut · MVP · E
- AC1: A shortcut (`Alt+Shift+N`, D-52) opens a popup with the cursor ready. Enter saves to the Inbox with the current page as its source (CAP-10).

### EXT-02 · Save selection (right-click) · MVP · E
- AC1: Right-click → "Save to Notes" saves the selected text, page title, URL and a **text-fragment link** (`#:~:text=`) back to the exact passage.
- AC2: An optional reminder ("read on Friday") can be added in the same step.

### EXT-03 · Extension sign-in by approving on phone · MVP · E
- AC1: The extension signs in through the SEC-01 or SEC-02 flow (no password typed).

### EXT-04 · Page-linked notes and badge · MVP · E
- AC1: When you visit a page or site that has notes, the toolbar icon shows a **count badge**.
- AC2: Opening the popup lists those notes first.
- AC3: The user chooses matching per note: exact page or whole site.

### EXT-05 · Side panel · MVP · E
- AC1: The side panel shows the Inbox and the notes for this page, with quick capture at the top.
- AC2: It leaves a slot for the future chatbot (AI-01) without a redesign.

---

## BOT: Messaging bot

### BOT-01 · Telegram bot · Future (D-44) · B S
> Moved to Future because the owner doesn't use Telegram. Design kept in [telegram-bot.md](../05-system-design/telegram-bot.md).
- AC1: Linking: the user gets a one-time code in the app and sends it to the bot. **No linking by phone number alone.**
- AC2: Text message → Inbox note. Voice message → audio note (CAP-05 AC3). "remind me…" → reminder (REM-01, rule-based).
- AC3: Replies like "done" or "snooze 1h" to a reminder message work. Commands: `/inbox`, `/today`, `/find <text>` (keyword search, ORG-07).
- AC5: Messages go through a **command router**, so a future AI handler (AI-01, AI-03) can be added as one more route (PLAT-04).
- *Future (D-24):* questions answered from notes, voice transcription, chat-style commands.
- AC4: The bot warns once that Telegram messages are not end-to-end encrypted, so bot notes are normal (not locked) notes.
- 💰 The Telegram Bot API is free.

### BOT-02 · Reminder delivery via Telegram · Future (D-44) · B S
- AC1: Reminders and escalations (REM-06, REM-07) are delivered as Telegram messages with Done and Snooze buttons.

### BOT-03 · WhatsApp bot · Future (only if free) · B S
- Blocked by the zero-cost rule (D-22). Revisit only if a free path exists.

---

## SEC: Security and login

### SEC-01 · Login: short username → approve on trusted phone · MVP · A W E S
- AC1: The new device asks only for the short username.
- AC2: The trusted phone receives an approval request showing device type, browser, approximate location and time.
- AC3: **Number matching.** The new device shows a 2-digit number, and the user selects the same number on the phone, then confirms with fingerprint or face.
- AC4: Requests expire in ≤ 120 s. Repeated requests are rate-limited (stops push-bombing).
- 💰 Push via Firebase Cloud Messaging (free).

### SEC-02 · QR-scan login (fallback) · MVP · A W E S
- AC1: The new device shows a QR code. The user scans it with the logged-in phone app and approves it on the same screen as SEC-01.
- AC2: The QR code expires in ≤ 60–120 s and refreshes automatically. The phone shows device details before approval (QRLJacking mitigation).

### SEC-03 · TOTP authenticator (backup factor) · MVP · A W S
- AC1: The user can enrol Google or Microsoft Authenticator (or similar) by scanning a QR code.
- AC2: TOTP is used only when phone approval isn't possible, and **always together with the username + recovery checks**. Never as the sole factor without strict rate limiting.

### SEC-04 · App lock (Android) · MVP · A
- AC1: An optional PIN, fingerprint or face lock when the app opens or resumes after N minutes.
- AC2: Quick-capture surfaces (tile, widget) can **save** without unlocking but can't **read** notes until unlocked.

### SEC-05 · Device list, revoke, alerts · MVP · A W E S
- AC1: A list of signed-in devices with last-active time, where each can be revoked instantly.
- AC2: Every new sign-in triggers a push alert on all trusted devices.
- AC3: The device list also holds each device's notification schedule (REM-08).

### SEC-06 · Account recovery · MVP · A W S
- AC1: Signup generates recovery codes. The user must confirm they saved them (for example by re-typing one).
- AC2: Any trusted device can approve a new device.
- AC3: The app nags until at least two recovery paths are set up.
- AC4: The app clearly warns that locked notes cannot be recovered if all keys are lost.

### SEC-07 · Locked notes (end-to-end encrypted vault) · MVP · A W E
- AC1: Any note can be locked. Its content is encrypted on the device, and the server stores only ciphertext.
- AC2: Opening a locked note needs a vault PIN or biometric.
- AC3: Locked notes are excluded from AI, server search and bots. Their reminders show a generic "🔒 Locked reminder" title.

### SEC-08 · Baseline protections · MVP · S
- AC1: TLS everywhere, encryption at rest, and each user's data isolated (ready for multiple users).
- AC2: Rate limits on all auth endpoints. Short-lived sessions per device.
- AC3: A user-visible log of login and security events.

---

## DATA: Data safety

### DATA-01 · Conflict-safe sync · MVP · A W E S
- AC1: If the same note is edited on two devices offline, **both versions are kept** and the note is marked "conflict". The user picks or merges.
- AC2: No silent overwrite in any sync path.

### DATA-02 · Trash with 30-day restore · MVP · A W E
- Covered by ORG-06, AC2.

### DATA-03 · Full export · MVP · W
- AC1: One click exports all notes as **Markdown files + a JSON file** (with groups, tags, todos, reminders and source info) in a zip.
- AC2: Locked notes are exported decrypted only after a vault unlock. Otherwise they are left out, with a notice.

### DATA-04 · Backups · MVP · S
- AC1: Automatic server backups, with restore tested.
- 💰 Must fit the free hosting chosen in the system design. See [zero-cost.md](zero-cost.md).

---

## PLAT: Platform basics

### PLAT-01 · Instant start · MVP · A W E
- AC1: The capture editor is usable before sync, AI or the network finish loading (principle P1). On web, the editor shell loads instantly and saving shows a clear error with retry if offline (D-39).

### PLAT-02 · Same model everywhere · MVP · A W E B
- AC1: Inbox, Notes, Todos, Groups, Important and Locked mean the same thing and look alike on every platform.

### PLAT-03 · Light and dark theme, accessible text sizes · MVP · A W E

### PLAT-04 · AI-ready extension points · MVP · S A W E B
No AI is built, but these hooks must exist so AI can be added later without rework (D-25). Details: [future/ai-integration.md](../future/ai-integration.md).
- AC1: Domain events are emitted for note and todo changes (`note.created`, `note.updated`, `note.deleted`, `todo.completed`, `reminder.fired`).
- AC2: A generic **Suggestion** record (type, target note, payload, status pending/accepted/rejected) exists, and the triage UI can show suggestions. In MVP only rule-based sources create them, e.g. detected dates.
- AC3: Search goes through one **search interface** (keyword implementation now).
- AC4: Notes carry `locked` and `ai_visible` flags from day one.
- AC5: The in-app quick-add (and a future bot) parse input through a **command router** with pluggable handlers.
- AC6: A feature flag `ai.enabled` (default **off**) controls all future AI UI.

---

## FUT: Future backlog (not scheduled)

| ID | Feature | Why later | Zero-cost note |
|---|---|---|---|
| FUT-01 | Location reminders (arrive or leave) | Not chosen for MVP; needs background-location permission | Android geofencing is free |
| FUT-02 | Daily or weekly digest (today's todos, open loops, forgotten notes) | Not chosen for MVP | Sent via push, free |
| FUT-03 | Resurfacing (on this day, spaced review, related notes while you write) | Fixes the note graveyard further | — |
| FUT-04 | AI "expand hurried note" (keeps the original text) | Not chosen for MVP | D-23 |
| FUT-05 | MCP server (Claude and other AI tools can read and write notes) | Owner wants it later (D-15) | Free and open protocol |
| FUT-06 | "Next time I talk to X" person reminders | Complex | — |
| FUT-07 | Email-in capture | Not chosen | Needs a free inbound email option |
| FUT-08 | Snip a region → OCR → note (extension) | Nice to have | Tesseract.js (free) |
| FUT-09 | Imports from wearables (Plaud, Omi), Keep or Evernote | Nice to have | — |
| FUT-10 | Wear OS voice tile | After Android stabilises | — |
| FUT-11 | iOS app | D-02 | ⚠ The Apple developer program costs about $99/yr. Conflicts with D-21 |
| FUT-12 | Passkey-first login | Adoption still growing | Free |
| FUT-13 | Sharing a note or list with others | Not chosen | — |
| FUT-14 | Version history | Not chosen; DATA-01 covers the critical case | — |
| FUT-15 | WhatsApp bot | D-22 | Only if free |
| FUT-16 | Offline web app (local DB + outbox in the browser) | Not required for MVP (D-39) | Free (IndexedDB) |
| FUT-17 | Telegram bot (BOT-01, BOT-02, CAP-08) | Owner doesn't use Telegram (D-44) | Free |
| FUT-18 | Hindi UI | English-only MVP (D-45) | Free |
