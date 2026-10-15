# UX principles: "easy to use" on every platform

> Status: Living · Last updated: 2026-10-16 · Related: [vision](../00-vision.md) (principles P1–P8), [features](features.md)
>
> **The detailed, final UI/UX spec is [07-design/ui-ux-design.md](../07-design/ui-ux-design.md)** (tokens, components, every screen). This page is the short summary. If they differ, the design doc wins (D-51, D-52).

## Global rules (all platforms)
| Rule | Target |
|---|---|
| Thought → saved note | **≤ 2 s and ≤ 2 actions** from any capture surface |
| Required fields at capture | **None.** No title, group or tag needed |
| Editor readiness | Typing works before sync, AI or the network finish (PLAT-01) |
| Suggestions | Suggest, never force. Every automatic action (e.g. a detected date, future AI) has **Undo** |
| Destructive actions | Go to Trash (30 days). No "are you sure?" pop-ups for normal deletes; offer Undo instead |
| Vocabulary | Use the same words everywhere: **Inbox, Notes, Todos, Groups, Important, Locked** |
| Empty states | Teach one thing ("Swipe right to make a todo") rather than showing a blank screen |
| Privacy cues | Locked notes show 🔒. (Future: an AI-visible indicator, AI-05) |
| Notifications | Few and meaningful. Only *Important* items repeat (D-09). Respect each device's quiet hours (REM-08) |

## Android (primary)
- **Capture surfaces:** home widget (New / Voice / Today), Quick Settings tile, share sheet, an in-app floating "+" button, and a hold-to-talk voice button.
- **Navigation:** bottom tabs **Inbox · Todos · Notes**. Settings and search sit in the top bar. *(Future: a 4th tab, **Chat**, when AI is added. Leave room for it.)*
- **Gestures in lists:** swipe right → Todo (Inbox) or Done (Todos). Swipe left → Archive. Long-press → actions sheet (with Select for multi-select).
- **Notifications:** Done and Snooze buttons on every reminder. Important reminders use a distinct sound and keep repeating until handled.
- **First run (3 screens, D-52):** username → recovery codes (shown and confirmed) → "You're ready". **No permissions in onboarding.** Notifications and the battery guide are asked **in context** the first time a reminder is set. App lock, office hours and time zone (default India) live in Settings.
- **Offline:** never block. Show a small "sync pending" dot only.

## Web app
- **Layout:** left sidebar (Inbox, Today, Upcoming, Groups, Important, Archive, Trash). Main list in the middle, note detail on the right. *(Future: chat opens as a panel.)*
- **Keyboard first** (final keymap in [ui-ux-design §7.8](../07-design/ui-ux-design.md#78-keyboard-map)):
  - `Q` or `Alt+N` quick add
  - `Ctrl/Cmd+K` command palette
  - `/` search
  - `T` toggle todo
  - `D` done
  - `E` archive
  - `H` remind
  - `M` move to group
  - `I` important
  - `Z` undo
  - `?` shortcut help
- **Quick-add bar** is always visible. Natural-language dates are highlighted as you type.
- **Login:** type the username, then approve on your phone. "Use QR instead" is shown right there.

## Browser extension
- **Popup:** a single text box with the cursor ready. Enter saves. The current page is attached automatically, with a small "×" to detach it.
- **Badge:** shows the count of notes linked to this page or site (EXT-04).
- **Right-click:** `Save "%s" to Cue` (selection) and "Save page to Cue".
- **Shortcuts:** `Alt+Shift+N` opens the popup, `Alt+Shift+S` opens the side panel (D-52).
- **Sign-in:** opens in a full extension tab (popups close when focus leaves; D-52).
- **Side panel:** this page's notes and the Inbox. *(Future: chat.)*
- **Quiet hours:** outside the configured office hours the extension shows **no pop-ups**. The badge still updates silently.

## Telegram bot (Future, D-44)
> Not in MVP. Kept for when a bot is added.
- **Keep it simple.** Any message becomes a note. "Remind me … tomorrow 6pm" becomes a reminder (rule-based). Voice messages are saved as audio notes. *(Future: questions answered from your notes.)*
- **Commands (optional, for power users):**
  - `/inbox`
  - `/today`
  - `/find <text>`
  - `/done`
  - `/snooze 1h`
  - `/help`
- **Every bot action replies with a short confirmation** and an "Undo" button.
- **Reminder messages** have inline buttons: ✅ Done · ⏰ Snooze 10m / 1h / Tomorrow.

## Voice and tone
Cue speaks **professionally, calmly and briefly**. It never nags in words, only through reminders the user asked for.

| Situation | ✅ Write | ❌ Avoid |
|---|---|---|
| Note saved | "Saved to Inbox." | "Awesome!! Your note is safe 🎉" |
| Empty Inbox | "Inbox clear. Capture anything with the + button." | "Nothing here… 😢" |
| Reminder | "Cue: Call the electrician · 6:00 PM" | "Hey! Don't forget!!!" |
| Important reminder repeat | "Still pending: Pay electricity bill" | "You forgot AGAIN" |
| Login approval (phone) | "Sign-in request from Chrome on Windows. Select 42 to approve." | "Someone is trying to log in!" |
| Error / offline | "Offline. Saved on this device and will sync when you're back online." | "Sync failed!" |

Notification titles start with **"Cue:"** so they're easy to recognise.
