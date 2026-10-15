# Research: method and key takeaways

> Status: Final · Research date: 2026-10-15 · Related: [vision](../00-vision.md), [decisions](../02-product/decisions.md)

## Method
We ran three web research tracks in parallel. Each one first explored many sources, then reviewed them one by one:
1. **Competitor survey** ([competitor-survey.md](competitor-survey.md)): about 30 note and task apps. For each we looked at capture paths, reminders, note-to-todo conversion, AI, security, platforms and pricing.
2. **Real user pain points** ([user-pain-points.md](user-pain-points.md)): Hacker News, the Obsidian, Apple and Microsoft forums, Reddit (via mirrors and quotes, because Reddit blocks direct access), Product Hunt, and blogs.
3. **Emerging features, security and auth** ([emerging-features.md](emerging-features.md), [security-and-auth.md](security-and-auth.md)).

**Caveats:**
- Many Product Hunt posts and blogs come from people selling a fix. Treat them as evidence that a problem *exists*, not proof of how common it is.
- Prices change often, so check them before you rely on them.
- Evidence on login lockouts is thin.

## Top 10 takeaways
1. **The #1 problem is notes that never come back** (the "note graveyard"), not capture alone. Bringing notes back matters as much as capturing them.
2. **Capture speed decides whether a note exists at all.** Users drop apps that take 4–15 seconds to open. Google Keep wins on load speed.
3. **Reminders fail in two ways:** they're dismissed once and forgotten, or they never fire (Android battery killers). TickTick's "constant" reminders and Due's repeating alerts are what users praise.
4. **Keyword search breaks as the archive grows.** Users want search by meaning and the ability to ask their notes questions, with citations. That's our future chatbot (deferred by D-24).
5. **Hurried notes lose their meaning.** Saving the source page or app automatically helps a lot. Microsoft's OneNote Sticky Notes does this.
6. **Voice dumps that split into several tasks are the capture trend of 2025–26** (Todoist Ramble, Keep Live, Mem).
7. **Messaging-bot capture works.** Capacities, Mem and Coco already offer capture via WhatsApp or Telegram.
8. **There's a market gap.** Encrypted apps (Standard Notes, Notesnook, Joplin) have little or no AI, and AI apps (Mem, Notion, Evernote, Saner) aren't end-to-end encrypted. Nobody combines a trusted AI, reliable reminders and phone-approved login.
9. **Google Keep removed location reminders (Jan 2026),** and products change fast (Simplenote frozen, Tana pivoted, Notion Mail closed). Users fear lock-in, so full export matters.
10. **A short username plus a TOTP code as the only login is unsafe** (it can be brute-forced and phished). Phone approval with number matching, plus a QR fallback, delivers the same convenience safely.

## How the research shaped the product
> Update 2026-10-15: all AI features (AI-xx, ORG-03) moved to Future by D-24. Keyword search (ORG-07) covers search in MVP.

| Finding | Product response (feature IDs in [features.md](../02-product/features.md)) |
|---|---|
| Slow capture loses thoughts | CAP-01…CAP-09 (widget, tile, share, voice, extension, bot, offline) |
| Note graveyard | AI-01 chat over notes, EXT-04 page-linked notes, REM-07 important nagging; Future: digest/resurfacing |
| Dismissed or unfired reminders | REM-03…REM-08 (on-device alarms, battery onboarding, nagging, channels, routing) |
| Search failure | AI-01, AI-02 (meaning-based search) |
| Hurried notes lose meaning | CAP-10 (auto source context) |
| Organising fatigue | ORG-03 (AI-suggested groups), ORG-04 (swipe triage) |
| Sync data loss / lock-in | DATA-01…DATA-04 |
| AI privacy fear | SEC-07 locked notes, AI-06 no training |
| Unsafe TOTP-only login | SEC-01…SEC-05 |
