# Roadmap

> Status: Living · Last updated: 2026-10-15 · Related: [features](02-product/features.md), [decisions](02-product/decisions.md), [open questions](04-open-questions.md)
>
> **The detailed, step-by-step plan is in [06-build-plan](06-build-plan/README.md)** (task cards with files, commands, checks and documentation links). This page is the overview.
> Every milestone must respect the zero-cost rule (D-21).

## Phase 0: System design (drafted 2026-10-15, waiting for review)
- Decide the stack and architecture for Android, web, extension, server, sync and bot (all free). No AI server (D-24), only the AI-ready hooks (PLAT-04).
- Open questions Q-01…Q-09 are all resolved (see [04-open-questions.md](04-open-questions.md)).
- **Done when:** `05-system-design/` is approved by the owner (architecture, data model, API, sync, auth, reminders, hosting).

## Phase 1: MVP (personal use)
Start with [M0 setup](06-build-plan/M0-setup.md) (tools, accounts, monorepo, skeletons).
Built as milestones so every step is usable on its own.

| Milestone | Delivers | Feature IDs | Done when |
|---|---|---|---|
| **M1 Foundations** ([cards](06-build-plan/M1-foundations.md)) | Account on the first device (Android), recovery codes, data model, sync with outbox, free hosting deployed | SEC-06, SEC-08, CAP-09, DATA-01, PLAT-01 | A note created offline on Android syncs to the server after reconnecting; a conflict keeps both copies (web is online-only, D-39) |
| **M2 Capture & organise** ([cards](06-build-plan/M2-capture-organise.md)) | Android: Inbox, editor, widget, tile, share target, voice (device speech-to-text). Web: Inbox, quick-add, triage. Login on web by approving on the phone or by QR | CAP-01…CAP-06, CAP-10, ORG-01, ORG-02, ORG-04…ORG-06, SEC-01, SEC-02, PLAT-02, PLAT-03 | Thought → saved note in ≤ 2 s on Android; web login works with no password |
| **M3 Reminders** ([cards](06-build-plan/M3-reminders.md)) | Natural-language dates, time and recurring reminders, on-device alarms, battery onboarding, push and web notifications, Important nagging, quiet hours per device, overdue view | REM-01…REM-09 | A test reminder fires offline on the phone; the PC gets no notification outside office hours |
| **M4 Browser extension** ([cards](06-build-plan/M4-extension.md)) | Popup, save selection with text-fragment link, page badge, side panel, sign-in by approving on phone | EXT-01…EXT-05, CAP-07 | Revisiting a page shows the badge with the right count |
| **M5 Search & hooks** ([cards](06-build-plan/M5-search-hooks.md)) | Keyword search (offline on Android, server search on web and extension); the AI-ready hooks verified | ORG-07, PLAT-04 | Search finds a note by any word in body, tag or URL; the hooks exist with `ai.enabled = false` |
| **M6 Hardening** ([cards](06-build-plan/M6-hardening.md)) | Locked notes vault, app lock, TOTP backup, device list and revoke, full export, backups | SEC-03, SEC-04, SEC-05, SEC-07, DATA-03, DATA-04 | The server stores only ciphertext for locked notes (verified by test); the export opens cleanly in a Markdown viewer |

**Every milestone is also done only when** its learning notes exist in `docs/learning/notes/` and its concepts are ticked in [concept-map.md](learning/concept-map.md) (D-47).

**MVP done when:** the owner uses it daily for 2 weeks with no lost notes and no missed reminders.

## Phase 2: v2
- FUT-17 Telegram bot, only if the owner starts using Telegram (D-44)
- REM-10 "Next time I'm on my laptop"
- FUT-02 daily digest and FUT-03 resurfacing (if the note graveyard persists)
- FUT-05 MCP server
- FUT-12 passkeys
- Publish more widely if wanted. A Play Store and Chrome Web Store listing costs a one-time fee, which conflicts with D-21 (see [zero-cost.md](02-product/zero-cost.md)).

## Phase 3: Future AI (deferred by D-24)
Follow [future/ai-integration.md](future/ai-integration.md): choose a free provider → privacy gate → ORG-03 suggestions → voice split → AI-02 meaning search → AI-01 chat with citations → AI-03/AI-04 chat commands → FUT-05 MCP server.

## Phase 4: Future (going public, optional)
Location reminders (FUT-01), AI expand (FUT-04), person reminders (FUT-06), email-in (FUT-07), OCR snip (FUT-08), imports (FUT-09), Wear OS (FUT-10), iOS (FUT-11, paid), sharing (FUT-13), version history (FUT-14), WhatsApp (FUT-15, only if free).
