# System design

> Status: **Draft written 2026-10-15, waiting for the owner's review** · Last updated: 2026-10-15 · Related: [features](../02-product/features.md), [open questions](../04-open-questions.md), [zero-cost](../02-product/zero-cost.md)
>
> ⛔ Agents: do not write application code until the owner approves these docs (Status → Approved).

## Decisions already made
| Area | Decision |
|---|---|
| Language | TypeScript everywhere (D-30) |
| Android | React Native + TS, small native Kotlin modules for widget/tile/alarms/share (D-31) |
| Backend | Cloudflare Workers + D1 + Cron Triggers, free plan (D-32) |
| Web | React + Vite PWA on Cloudflare Pages (D-35) |
| Repo | Monorepo: apps/mobile, apps/web, apps/extension, apps/api, packages/shared (D-36) |
| Sync | Simple custom sync: local DB + outbox + server versions + keep-both on conflict (D-34) |
| Signup | Android app only; web/extension sign in via phone approval or QR (D-33) |
| Voice | Text by default, audio optional, in free object storage (D-37) |
| Hosting | Free cloud only, no home PC (D-27) |
| Distribution | APK sideload + unpacked extension (D-38) |

## Constraints the design must respect
- **Zero cost** (D-21): only free, open-source, self-hosted, on-device or permanent free-tier services.
- **Android first**, plus web and a Chrome extension (D-02). No messaging bot in MVP (D-44).
- **Offline capture and viewing** (D-13), with no silent overwrite on conflict (D-17).
- **Balanced privacy** (D-04): normal notes encrypted at rest; locked notes E2EE (and, when AI comes, never sent to it).
- **No AI in MVP, but AI-ready** (D-24, D-25, PLAT-04).
- **Login** by approving on the phone with number matching, QR fallback and TOTP backup (D-05). Recovery codes (D-06).
- **Per-device notification routing** with office hours (D-08).
- **Multi-user ready** (D-01), but no billing.
- **Hosting on a permanent free cloud tier only**, with no server on the owner's PC (D-27). Must handle sleep/idle limits (K-10).
- **Office hours are user-configurable; the default time zone is India (Asia/Kolkata)**. Store times in UTC (D-28).

## Discussion agenda (mobile + web)
1. **High-level architecture.** Clients (Android, web, extension, bot) ↔ API ↔ DB. Where AI runs. Push paths.
2. **Tech stack.** Android (Kotlin vs Flutter vs React Native), web framework, extension framework, backend language, DB. All free (Q-04).
3. **Hosting plan.** Which free cloud provider(s) to use (D-27): limits, sleep rules, free scheduled jobs.
4. **Data model.** User, Device, Note, Todo/Reminder, Group, Tag, Source, LockedVault, Session, AuditEvent.
5. **Sync design.** Outbox, versioning, conflict copies (Q-03).
6. **Auth design.** Signup and first-device bootstrap (Q-05), push approval with number matching, QR flow, TOTP, recovery codes, device sessions.
7. **Reminder engine.** On-device alarms vs server scheduler, de-duplication across devices, quiet-hours routing, alarm-style escalation on the phone.
8. **AI-ready hooks only (no AI, D-24).** Domain events, the Suggestion model, the search interface, `locked`/`ai_visible` flags, the command router and the `ai.enabled` flag (PLAT-04). AI itself: [future/ai-integration.md](../future/ai-integration.md).
9. **Extension design.** URL matching for page-linked notes, text fragments, side panel, auth.
10. **Distribution.** APK sideload, unpacked extension (Q-06).
11. **Testing strategy.** Especially reminder reliability, sync conflicts, and checking that AI never receives locked notes.

## Design docs (read in this order)
| # | Doc | Covers |
|---|---|---|
| 1 | [architecture.md](architecture.md) | Big picture, components, main flows, trust boundaries |
| 2 | [tech-stack.md](tech-stack.md) | Monorepo layout, libraries per layer, the `packages/shared` contents, Kotlin learning path |
| 3 | [data-model.md](data-model.md) | Tables, columns, indexes, local-only tables |
| 4 | [api.md](api.md) | Every endpoint |
| 5 | [sync.md](sync.md) | Android offline sync, the server write rule, conflicts, tests |
| 6 | [auth-flows.md](auth-flows.md) | Signup, phone approval, QR, TOTP, recovery, vault, rate limits |
| 7 | [reminders-engine.md](reminders-engine.md) | Android alarms, cron dispatcher, office hours, nagging, tests |
| 8 | [mobile-android.md](mobile-android.md) | Screens, speed budget, Kotlin modules, push, voice |
| 9 | [extension.md](extension.md) | Popup, context menu, page badge, auth, capture queue, reminders |
| 10 | [telegram-bot.md](telegram-bot.md) | **Future (D-44)**: bot design kept for later |
| 11 | [ai-hooks.md](ai-hooks.md) | PLAT-04 hooks (no AI) |
| 12 | [hosting-and-distribution.md](hosting-and-distribution.md) | Verified free-tier limits, backups, environments, installing |
