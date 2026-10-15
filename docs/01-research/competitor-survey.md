# Competitor survey (about 30 apps)

> Status: Final · Research date: 2026-10-15 · Related: [research index](README.md), [features](../02-product/features.md)
>
> † = from general knowledge, not re-verified in this research session. Prices are from mixed sources and change often, so check them before relying on them.

## Comparison table

| App | Quick capture | Reminders | Note→todo / organising | AI | Security | Platforms / offline / price |
|---|---|---|---|---|---|---|
| **Google Keep** | Android widgets†, share sheet†, Chrome extension†, Gemini creates notes, Keep Live voice dump (Android, paid AI tiers), Keep in Google Messages | Moved to Google Tasks; **location reminders removed (Jan 2026)** | Checklists, labels, colours | Gemini list generation; Gemini can read and edit notes | No E2EE, no app lock | Android, iOS, web; free |
| **Apple Notes + Reminders** | Siri, Watch (watchOS 26), share sheet, Lock Screen and Control Center† | Time, location†, recurring; typed dates (iOS 27); Siri reminders from Mail/Messages | Folders, tags†, checklists | Siri AI writes notes and suggests "Add to Notes" | Locked notes E2EE; Advanced Data Protection = all notes E2EE (opt-in) | Apple only + iCloud web; offline; free |
| **OneNote / Sticky Notes** | Sticky Notes: Win+Alt+S, one-click screenshot, **saves the source link and resurfaces it**; email-to-OneNote† | None native† | Notebooks / sections / pages† | Copilot Notebooks; voice → structured notes | Password-protected sections†; Microsoft account 2FA | All platforms; mostly free |
| **Microsoft To Do** | Widgets†, flagged Outlook email† | Time, recurring; no location | Lists, My Day, steps† | Copilot reads tasks (Premium) | Microsoft account | All; free; seen as neglected |
| **Evernote** | Web Clipper, email-in, document scan†, mobile AI cleanup | Tasks with due dates, reminders | Notebooks, tags, spaces | AI Assistant (v11, Jan 2026), semantic search, MCP | 2FA; no E2EE | All; offline; free = 50 notes / 1 device; ~$99–250/yr |
| **Notion** | Web clipper†, widgets†, Agents iPhone app (text, voice, photo → filed automatically) | Date reminders† | Databases, pages | Notion Agent, Custom Agents, MCP | 2FA, passkeys; no E2EE | All; offline since Aug 2025 (limits) |
| **Obsidian** | Official Web Clipper (highlights, AI interpreter), iOS share sheet, plugins | Plugins only | Folders, tags, links, Bases | Plugins (bring your own model) | Sync is E2EE; local files | All; offline; free, Sync $4–5/mo |
| **Bear** | Web Clipper 2.0, widgets†, Shortcuts | None native | Nested tags | None built in; MCP (2.8 beta) | Note encryption† | Apple + web beta; ~$30/yr |
| **Joplin** | Web clipper† | One alarm per to-do; may be missed if the app isn't running | Notebooks, tags, **note ↔ to-do toggle** | Plugins | E2EE | All; open source; free |
| **Standard Notes** | Web clipper (paid) | None† | Tags, folders (paid) | None | E2EE, 2FA, built-in authenticator | All; $0 / $90 / $120 per year |
| **Notesnook** | Web clipper, Inbox API (Jun 2026) | One-time and recurring, synced | Notebooks, tags, colours | None | E2EE, app lock, vault | All; free tier |
| **Simplenote** | Basic | None | Tags | None | Not E2EE† | **Maintenance mode since Mar 2026** |
| **UpNote** | Web clipper | None | Notebooks, tags | None | Note lock; not E2EE | All; ~$40 lifetime |
| **Todoist** | Quick Add with natural-language parsing†, **Ramble voice → several tasks** (40+ languages, Wear OS), email, extension† | Time, recurring, location (Pro) | Projects, labels, filters | Ramble, Task Assist | SOC 2 Type II | All; Pro ~$5/mo |
| **TickTick** | Natural-language dates, widgets, voice† | Time, location, **"constant" nagging reminders** | Lists, tags | MCP (May 2026) | Standard | All; $49.99/yr |
| **Mem** | Voice Mode, hold-hotkey "Push-to-Remember", save@mem.ai, clipper, SMS/WhatsApp/Telegram | Agent nudges | Auto-organised | Agentic chat, Heads Up (related notes while you type), Clean Up | SOC 2; no E2EE | All; offline-first; $9–199 |
| **Reflect** | Voice memos, Chrome clip | — | Daily notes, backlinks | Q&A with citations; bring your own key (Open) | E2EE with opt-in AI decryption | Classic $10/mo; Open free (Mac) |
| **Tana** | Meeting capture | — | Supertags | Meeting agents | — | **Pivoted to meetings** |
| **Capacities** | **WhatsApp, Telegram, email (all plans)** | Tasks (Dec 2025) | Object types | AI assistant, MCP | Not stated | All; ~$10/mo Pro |
| **Heptabase** | Home widget (search, write, voice), share sheet, **unified Inbox** | Inbox reminders | Whiteboards, cards | Chat over the knowledge base | Not stated | All; $9–18/mo |
| **Saner.ai** | Chrome extension, voice | **Surfaces items when relevant** | Brain dumps → tasks | Chat over notes, email, calendar | Not E2EE | Web + mobile; $8–16 |
| **Supernotes** | Email-in | None | Cards | AI features | — | All; £6/mo |
| **Amplenote** | Jots stream, extension, email, widgets | Push, recurring, calendar sync | **Tasks inside notes, auto-scored** | Agent, MCP | Vault notes E2EE | All; ~$6–10/mo |
| **Drafts** | **Opens to a blank page**, Lock Screen, Watch, background dictation | Sends to Reminders/Todoist | Tags, flags | Chat Console | Advanced Data Protection | Apple only; ~$20/yr |

Newer AI-first apps:
- **Fabric**: auto-organises everything; not E2EE.
- **mymind**: no folders, AI tags everything.
- **Voicenotes**: voice-first with "Ask AI".
- **Granola**: meeting notes without a bot joining the call.
- **NotebookLM**: research over your own sources.
- **Twos**: notes, tasks and reminders in one stream.
- **Second Brain AI** (iOS): one thought is routed to Reminders, Calendar or Markdown.
- **Anytype**: local-first, E2EE.

## Standout ideas to learn from
- **Google Keep:** fastest load time and capture. Keep Live adds spoken items to the right existing list. Its removal of location reminders is a lesson about platform risk.
- **OneNote Sticky Notes:** saves the *source* of each note and resurfaces it when you return to that page. This is the model for our **EXT-04 / CAP-10**.
- **Todoist Ramble:** one spoken ramble becomes separate dated tasks, and you can correct them by voice. Model for **CAP-05**.
- **TickTick:** "constant" reminders that nag until done. Model for **REM-07**.
- **Joplin:** a one-tap note ↔ to-do toggle. Model for **ORG-01**. Its alarms can be missed if the app isn't running, which is a reliability warning.
- **Mem:** "AI remembers for you": hold a key and speak, and the agent nudges you later. Model for the Future "smart nudges".
- **Capacities:** capture by messaging an app you already use (WhatsApp/Telegram). Model for **BOT-01/02**.
- **Heptabase:** one Inbox for notes, reminders and to-dos, with "capture now, triage later". Model for **CAP-01 / ORG-04**.
- **Amplenote:** tasks stay inside the note they came from. Useful for "convert part of a note to a todo".
- **Drafts:** the text exists before you decide where it goes. The editor is ready instantly.
- **Notesnook:** the closest match to our security goals (E2EE, app lock, vault, recurring reminders, inbox API), but it has no AI.
- **Reflect:** E2EE, and AI only sees the text you explicitly select. An alternative model for privacy-friendly AI.

## Market gaps (our opportunity)
1. **Nobody combines E2EE (or a strong privacy promise), a trusted AI chatbot, reliable reminders and passwordless phone/QR login.** Encrypted apps have no AI; AI apps aren't encrypted.
2. **No note app documents QR-code device login.** The closest is QOwnNotes pairing via a QR token.
3. **Location reminders on Android are now rare** after Keep dropped them. This is a Future opportunity.
4. **Page-linked notes that resurface in the browser** are only offered by small extensions (NoterLight, Annoto, Sidenote).
5. **Device- and time-aware routing** ("not on my work PC after hours") — none of the apps surveyed offers it.

## Risks seen in the market
- Pricing is moving to AI credits (Mem, Heptabase, Tana), so AI cost per user matters.
- Products change fast: Simplenote frozen, Notion Mail closed, Tana pivoted, Keep features removed. Users value **export** and stability.

## Sources
- Google Keep:
  - [Location reminder removal](https://9to5google.com/2026/01/15/google-keep-reminders-migration-tasks-wide/)
  - [Keep Live](https://9to5google.com/2026/09/03/gmail-docs-keep-live/)
  - [Workspace updates](https://workspaceupdates.googleblog.com/search/label/Google%20Keep)
- Apple:
  - [Apple Notes iOS 27](https://www.macrumors.com/2026/06/17/ios-27-apple-notes-app-new-features/)
  - [Reminders iOS 27](https://www.macrumors.com/2026/06/08/ios-27-calendar-reminders/)
  - [Apple security guide](https://support.apple.com/guide/security/sec1782bcab1/web)
- Microsoft:
  - [Sticky Notes](https://www.thurrott.com/cloud/303220/new-sticky-notes-experience-starts-rolling-out-to-windows-users)
  - [Microsoft To Do status](https://windowscentral.com/microsoft/windows-11/windows-wrap-microsofts-new-copilot-reminders-show-us-what-not-to-do-abandon-a-once-great-app)
- Evernote:
  - [Plans](https://evernote.com/compare-plans)
  - [v11](https://getcoai.com/news/evernote-v11-adds-openai-search-features-at-130-annual-price/)
- Notion:
  - [Releases](https://www.notion.com/releases)
  - [Offline](https://www.notion.com/releases/2025-08-19)
  - [Passkeys](https://www.notion.com/help/passkeys)
- Obsidian:
  - [Pricing](https://obsidian.md/pricing)
  - [Clipper review](https://obsidian.rocks/the-obsidian-web-clipper-fixes-everything/)
- Joplin:
  - [3.4](https://joplinapp.org/news/20250922-release-3-4)
  - [To-dos](https://joplinapp.org/help/apps/to-dos)
  - [Alarms forum](https://discourse.joplinapp.org/t/alarms/1458)
- Standard Notes:
  - [Plans](https://standardnotes.com/plans)
- Notesnook:
  - [Pricing](https://notesnook.com/pricing)
  - [3.4 Inbox API](https://alternativeto.net/news/2026/6/notesnook-3-4-brings-an-inbox-api-enhanced-desktop-security-unique-url-slugs-and-more/)
- Simplenote:
  - [Maintenance mode](https://alternativeto.net/news/2026/3/automattic-ends-simplenote-active-development-only-basic-maintenance-continues)
- Todoist:
  - [Ramble](https://www.todoist.com/help/articles/dictate-and-add-tasks-with-ramble-P1Raq7vVF)
  - [TechCrunch on Ramble](https://techcrunch.com/2026/01/21/todoists-app-now-lets-you-add-tasks-to-your-to-do-list-by-speaking-to-its-ai/)
- TickTick:
  - [Upgrade](https://ticktick.com/about/upgrade)
- Mem:
  - [Mem 2.0](https://get.mem.ai/blog/mem-2-dot-0)
  - [Text to Mem](https://support.mem.ai/article/150-save-ideas-recommendations-on-the-go)
- Reflect:
  - [Privacy](https://next.reflect.app/privacy)
- Capacities:
  - [WhatsApp integration](https://docs.capacities.io/reference/integrations/whatsapp)
  - [Pricing](https://capacities.io/pricing)
- Heptabase:
  - [Roadmap](https://wiki.heptabase.com/roadmap)
- Saner.ai:
  - [Saner.ai](https://www.saner.ai/)
- Amplenote:
  - [Help](https://www.amplenote.com/help/all_articles)
- Drafts:
  - [Changelog](https://docs.getdrafts.com/changelog)
- QOwnNotes:
  - [Web app / QR pairing](https://qownnotes.readthedocs.io/getting-started/web-app.html)
