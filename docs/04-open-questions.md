# Open questions and risks

> Status: Living · Last updated: 2026-10-15 · Related: [decisions](02-product/decisions.md), [zero-cost](02-product/zero-cost.md), [system design](05-system-design/README.md)
>
> Resolve these in the system-design discussion. When one is answered, record a decision (D-xx) and mark the question **Resolved → D-xx**.

## Open questions

| ID | Question | Options / notes | Status |
|---|---|---|---|
| Q-01 | **Which AI approach under zero cost?** | All AI is deferred; see [future/ai-integration.md](future/ai-integration.md). Revisit when AI starts. | **Resolved → D-24** |
| Q-02 | **Hosting: free cloud tier or self-host on the owner's PC?** | The owner's PC is off most of the time, so it can't be a server. The answer is a free cloud tier only. Which provider is chosen in the system design. | **Resolved → D-27** |
| Q-03 | **Sync approach** | Simple last-writer-wins per note + outbox + keep-both on conflict (recommended for one user) vs a sync engine vs CRDTs. | **Resolved → D-34** |
| Q-04 | **Cross-platform tech** | Native Kotlin vs Flutter vs React Native for Android; how much code to share with web and extension. Native modules are needed for the tile, widget and alarms either way. | **Resolved → D-30, D-31, D-32** |
| Q-05 | **First-device bootstrap and "public later" signup** | How does a brand-new user create an account with no trusted phone yet? Probably Android signup → recovery codes → app lock. | **Resolved → D-33** |
| Q-06 | **Distribution without fees** | Sideload the APK and load the extension unpacked while it's personal. Decide what to do when going public (the store fees break D-21). | **Resolved → D-38** |
| Q-07 | **Office-hours defaults** | User-configurable in Settings; time zone defaults to India (Asia/Kolkata). Holidays are not in MVP. | **Resolved → D-28** |
| Q-08 | **Voice: keep audio or only the transcript?** | Keeping audio costs storage (free-tier limits). Default proposal: transcript only, with an optional "keep audio". | **Resolved → D-37** |
| Q-09 | **Proposed decisions to confirm** | D-06 (recovery), D-17 (keep-both on conflict), D-22 (Telegram only; later superseded by D-44), D-26 (voice via the device's own speech recognition) | **Resolved: all confirmed 2026-10-15** |

## Risks

| ID | Risk | Impact | Mitigation |
|---|---|---|---|
| K-01 | **Free tiers change or disappear** | Service breaks | Keep the design portable (standard Postgres/SQLite, open protocols); export always works (DATA-03) |
| K-02 | **Android battery killers stop reminders** | Missed reminders break trust | On-device exact alarms (REM-04), onboarding (REM-05) tuned for Oppo/Vivo/Realme/OnePlus (D-46), alarm-style escalation (REM-07) |
| K-03 | *(Future)* **Free AI tiers may train on data** | Privacy promise broken | Applies when AI is added: opt-in with a warning; locked notes never sent (AI-06, SEC-07) |
| K-04 | *(Future)* **Self-hosted AI on a small free VM is slow or unavailable** | Chat and organising unavailable | Core app works without AI by design (D-24); AI degrades gracefully |
| K-05 | **Locked-note keys lost** | Permanent data loss | Recovery codes + trusted devices (SEC-06); clear warnings |
| K-06 | **QR-login phishing (QRLJacking)** | Account takeover | Short expiry, device details shown before approval, biometric, number matching, new-device alerts |
| K-07 | *(Future, D-44)* **Telegram messages not E2EE** | Bot content visible to Telegram | Bot notes are always "normal" notes; tell the user once (BOT-01 AC4) |
| K-08 | **MVP scope too big for one person** | Never ships | Milestones M1–M6 each deliver something usable; v2 and Future are strictly deferred |
| K-09 | **AI hooks skipped "because AI is later"** | Expensive rework when AI is added | PLAT-04 is an MVP feature with acceptance criteria; M5 checks it |
| K-10 | **Free cloud tiers sleep, pause or have limits** (D-27) | Web Push and extension reminders arrive late; the server is unavailable after a long idle period | Android reminders fire on the device anyway (REM-04). Prefer services that wake on request and offer free scheduled jobs. Check the pause and idle rules when choosing a provider |
| K-11 | **Android developer verification goes global in 2027** | Sideloaded APK installs may need a registered developer | Free "Limited Distribution" account (≤ 20 devices) or ADB install ([hosting §6](05-system-design/hosting-and-distribution.md#6-distribution-d-38)) |
| K-12 | **10 ms CPU per Worker invocation** (D-43) | Slow handlers fail | No server password hashing, cached tokens, capped batches, measure in staging |
| K-13 | **D1 100k writes/day and 500 MB DB** | Writes fail until 00:00 UTC | Lean writes, pruning, audio cap (D-40) |
