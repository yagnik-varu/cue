# Build plan: step-by-step task cards

> Status: Ready · Last updated: 2026-10-15 · Related: [roadmap](../03-roadmap.md), [links](links.md), [AGENTS.md](../../AGENTS.md), [learning mode](../learning/README.md)
>
> This folder is **the source of truth for the build order**. It is written for **Antigravity CLI on free-tier models**:
> every task is small, names exact files and commands, and says how to check it's done.

## Files
| File | Milestone | Tasks |
|---|---|---|
| [M0-setup.md](M0-setup.md) | Tools, accounts, monorepo, skeletons, CI | 11 |
| [M1-foundations.md](M1-foundations.md) | Data model, signup, sync | 14 |
| [M2-capture-organise.md](M2-capture-organise.md) | Capture, organise, Kotlin tile/share/widget, phone login, web app | 14 |
| [M3-reminders.md](M3-reminders.md) | Time parsing, alarms, escalation, battery, Web Push, cron | 11 |
| [M4-extension.md](M4-extension.md) | Extension auth, popup, selection, badge, side panel, reminders | 7 |
| [M5-search-hooks.md](M5-search-hooks.md) | Search, AI-ready hooks | 5 |
| [M6-hardening.md](M6-hardening.md) | App lock, devices, TOTP, recovery, vault, export, backups, audio, MVP acceptance | 9 |
| [links.md](links.md) | All documentation links by topic | — |

## Session protocol (one task per session)
1. Open a **new** Antigravity session in the repo folder. One task per session keeps the model's context small.
2. Paste the **standard prompt** below, with the task ID filled in.
   - **UI tasks:** the agent must also read the design sections mapped to the task in [ui-ux-design §11](../07-design/ui-ux-design.md#11-implementation-guide-for-low-tier-models), and copy the look from [mockups.html](../07-design/mockups.html). It must not invent colours, sizes or components.
3. The agent **briefs** you (learning mode). For tasks marked ⭐, reply **"go"** only once you understand the plan.
4. The agent builds, then runs the task's **Done when** checks. **You** verify them too (run the app, click around).
5. The agent writes the learning note and ticks the concepts in `docs/learning/concept-map.md`.
6. Commit: `feat(M1-T05): signup endpoint [SEC-06]`. Then tick the task in the tracker below and push.

### Standard prompt (copy and paste)
```text
Read AGENTS.md first and follow all its rules, especially Learning mode and the zero-cost rule.
Then open docs/06-build-plan/<FILE>.md and do ONLY task <TASK-ID>.
Read only the docs listed in that task's "Read first" line (and nothing else unless you need it).
If the task builds any screen or UI, ALSO read its sections in docs/07-design/ui-ux-design.md (see the §11 task map)
and follow them exactly: same tokens, components, copy and states. Don't invent design.
Before writing code: brief me (what, why, alternatives, new concepts with Node.js analogies).
If the task is marked ⭐, wait for my "go".
Follow the Steps in order. Use the exact file paths. Don't add features that aren't in the task.
When finished: run the "Done when" checks and show me the results, explain what you built,
give me the small "try it" exercises, write the learning note, and tick the concept map.
If something in the design doesn't work, STOP and tell me. Don't change decisions silently.
```

### Short follow-up prompts
- **Continue:** `Continue task <TASK-ID> from step <N>. Re-read the task card first.`
- **Explain more:** `Explain <thing> like I know Node.js and Express but not <X>. Use a small example.`
- **Review:** `Review the changes for task <TASK-ID> against its "Done when" list and the design doc. List gaps only.`
- **Quiz:** `Quiz me with 5 questions on the learning note for <TASK-ID>.`

## When the model gets stuck (playbook)
1. **Paste the exact error** (full message + the command you ran). Ask: *"Give ONE most likely cause and ONE fix."*
2. **Check the official doc** linked on the task card. Ask the model to compare its code with that doc page.
3. **Shrink the task.** Ask it to do only step N, or to build a 10-line minimal example first.
4. **Restart the session** if the model loops. Paste the task card again plus a summary of what's done.
5. **Version problems** (Expo, React Native, Gradle, Kotlin): check the versions in `package.json` against the Expo SDK docs, and run `npx expo install --check` in `apps/mobile`.
6. **Never let the model:**
   - add paid services (D-21)
   - add AI (D-24)
   - delete or rewrite design docs
   - change a decision without adding a row in `docs/02-product/decisions.md`
7. **Native Android changes** (anything in `modules/` or `app.json` plugins) need `expo prebuild -p android --clean` and a new dev build. A JavaScript-only reload is not enough.

## Antigravity setup (verified 2026-10-15, D-49)
- **Install the CLI** on Windows (PowerShell): `irm https://antigravity.google/cli/install.ps1 | iex`, then run `agy` in the repo folder. Update with `agy update`.
- **Rules load automatically.** Antigravity reads `AGENTS.md` at the repo root in every session. No extra file is needed.
- **The `/cue-task` slash command.** The skill `.agents/skills/cue-task/SKILL.md` becomes `/cue-task` in the CLI. Type `/cue-task M1-T05`. It does the same as the standard prompt above.
- **Check the context** with `/context` if answers get vague. Start a new session per task (`/resume` continues an old one).
- **Free tier facts:**
  - Weekly quota, no fixed published numbers. Sonnet/Opus 4.6 and GPT-OSS are **removed on 2026-11-02**; after that the free tier is **Gemini 3.8 Flash** only.
  - Spend quota on ⭐ tasks first (sync, auth, Kotlin alarms, vault). For easy tasks use Gemini Flash.
  - When the quota runs out, read the learning notes and docs until it refreshes.
- **Don't use legacy "workflows".** They stop working on 2026-10-19. Skills replace them.
- Docs: [Rules](https://antigravity.google/docs/rules/) · [Skills](https://antigravity.google/docs/skills/) · [CLI best practices](https://antigravity.google/docs/cli/best-practices/) · [Plans](https://antigravity.google/docs/plans/)

## Progress tracker
Tick when **Done when** passes **and** the learning note exists.

**M0 Setup:**
- [ ] M0-T01 Install developer tools
- [ ] M0-T02 Free accounts
- [ ] M0-T03 Git repository
- [ ] M0-T04 Monorepo scaffold
- [ ] M0-T05 packages/shared + Vitest
- [ ] M0-T06 API on Workers + D1
- [ ] M0-T07 Web via Worker static assets
- [ ] M0-T08 Expo app on phone
- [ ] M0-T09 Extension skeleton
- [ ] M0-T10 CI
- [ ] M0-T11 Antigravity setup check

**M1 Foundations:**
- [ ] M1-T01 Shared domain model
- [ ] M1-T02 D1 schema + migrations
- [ ] M1-T03 API foundations
- [ ] M1-T04 Rate limiting + audit
- [ ] M1-T05 Crypto helpers
- [ ] M1-T06 Signup + tokens ⭐
- [ ] M1-T07 applyChange ⭐
- [ ] M1-T08 Sync endpoints
- [ ] M1-T09 API client
- [ ] M1-T10 Mobile local DB + outbox
- [ ] M1-T11 Mobile signup onboarding
- [ ] M1-T12 Mobile sync engine
- [ ] M1-T13 Notes screen + conflict banner
- [ ] M1-T14 Prod deploy + acceptance

**M2 Capture & organise:**
- [ ] M2-T01 UI foundation (design system) ⭐
- [ ] M2-T02 Tabs + fast capture + Inbox
- [ ] M2-T03 Editor, todos, groups, trash
- [ ] M2-T04 Swipe triage
- [ ] M2-T05 Voice capture
- [ ] M2-T06 Kotlin tile ⭐
- [ ] M2-T07 Kotlin share target
- [ ] M2-T08 Kotlin Glance widget
- [ ] M2-T09 FCM push + sync nudge
- [ ] M2-T10 Phone-approval login ⭐
- [ ] M2-T11 QR login
- [ ] M2-T12 Notes REST API
- [ ] M2-T13 Web app
- [ ] M2-T14 M2 acceptance

**M3 Reminders:**
- [ ] M3-T01 Time utilities
- [ ] M3-T02 Reminders API + schedules
- [ ] M3-T03 Date suggestions
- [ ] M3-T04 Kotlin exact alarms ⭐
- [ ] M3-T05 Notification actions
- [ ] M3-T06 Nag + escalation
- [ ] M3-T07 Battery onboarding
- [ ] M3-T08 Web Push
- [ ] M3-T09 Cron dispatcher
- [ ] M3-T10 Reminder UI
- [ ] M3-T11 M3 acceptance

**M4 Extension:**
- [ ] M4-T01 Extension auth
- [ ] M4-T02 Popup + capture queue
- [ ] M4-T03 Save selection + text fragment
- [ ] M4-T04 Page badge
- [ ] M4-T05 Side panel
- [ ] M4-T06 Reminder tick + office hours
- [ ] M4-T07 M4 acceptance

**M5 Search & hooks:**
- [ ] M5-T01 SearchProvider
- [ ] M5-T02 Android FTS5
- [ ] M5-T03 Server search
- [ ] M5-T04 Command router
- [ ] M5-T05 privacyGate + ai.enabled

**M6 Hardening:**
- [ ] M6-T01 App lock
- [ ] M6-T02 Devices + revoke
- [ ] M6-T03 TOTP
- [ ] M6-T04 Recovery
- [ ] M6-T05 Locked notes vault ⭐
- [ ] M6-T06 Export (client zip)
- [ ] M6-T07 Backups
- [ ] M6-T08 Keep audio (optional)
- [ ] M6-T09 MVP acceptance (14 days)
