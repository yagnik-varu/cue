# M5 · Search + AI-ready hooks

> Status: Ready · Last updated: 2026-10-15 · Related: [README](README.md), [links](links.md), [ai-hooks](../05-system-design/ai-hooks.md), decisions D-24, D-25
>
> Features: ORG-07 (keyword search), PLAT-04 (AI-ready hooks)
> **Milestone done when:** search finds a note by any word in its body, tags or URL (offline on Android), and the
> [ai-hooks §7 checklist](../05-system-design/ai-hooks.md#7-done-when-m5-checklist) is all ticked. Learning notes are written.

---

### M5-T01 · `SearchProvider` interface (PLAT-04)
- **Depends on:** M1-T01
- **Read first:** [ai-hooks.md §3](../05-system-design/ai-hooks.md#3-search-interface)
- **Goal:** one interface in `packages/shared/search`, which every search screen uses.
- **Steps:** add the interface from ai-hooks §3, plus a tiny in-memory implementation for tests.
- **Done when:** types compile, and the tests use the in-memory provider.
- **Learning note:** `M5-PLAT-04-search-interface.md` (concepts: programming to interfaces, swapping implementations)

### M5-T02 · Android full-text search (FTS5) (ORG-07)
- **Depends on:** M5-T01, M2-T03
- **Goal:** instant offline search on the phone.
- **Steps:**
  1. A migration on the device: an FTS5 virtual table `notes_fts(body, title, tags, source_url)`, kept in sync with triggers on insert, update and delete.
  2. `KeywordSearchAndroid implements SearchProvider`, with prefix matching (`word*`) and a highlighted snippet.
  3. A `/search` screen, opened from the top-bar search icon.
  4. Locked notes are excluded (ORG-07 AC3; unlocked search comes in M6).
- **Done when:** in airplane mode, searching "elec" finds "Pay electricity bill" in < 100 ms.
- **Docs:** [SQLite FTS5](https://www.sqlite.org/fts5.html) · [expo-sqlite](https://docs.expo.dev/versions/latest/sdk/sqlite/)
- **Learning note:** `M5-ORG-07-fts5.md` (concepts: inverted indexes, FTS5, triggers)

### M5-T03 · Server search for web and extension (ORG-07)
- **Depends on:** M5-T01, M2-T12
- **Goal:** `GET /api/search?q=` used by the web app and the extension.
- **Steps:**
  1. Create a D1 **FTS5** virtual table `notes_fts` with a new migration (D1 supports FTS5, verified 2026-10-15) and keep it in sync in `applyChange`. Filter results by `user_id` and exclude locked notes. Limit 50.
  2. `KeywordSearchServer implements SearchProvider` in the web and extension clients.
  3. The web `/` shortcut opens search, and the extension side panel gets a search box.
- **Done when:** searching from the web finds notes created on the phone.
- **Docs:** [D1 SQL statements](https://developers.cloudflare.com/d1/sql-api/sql-statements/) · [SQLite FTS5](https://www.sqlite.org/fts5.html)
- **Learning note:** `M5-ORG-07-server-search.md` (concepts: search on server vs client, query cost on free tiers)

### M5-T04 · Command router + quick-add commands (PLAT-04)
- **Depends on:** M3-T01
- **Read first:** [ai-hooks.md §5](../05-system-design/ai-hooks.md#5-command-router)
- **Goal:** `packages/shared/router` with handlers `find`, `remind`, `capture` (default). Quick-add on mobile and web uses it (e.g. `/find car`).
- **Steps:**
  1. A `Handler` type plus a `route(input)` that picks the first match.
  2. Tests: `/find x` → find; "pay bill tomorrow" → capture + reminder suggestion; plain text → capture.
- **Done when:** tests pass, and adding a handler needs no change outside `router/`.
- **Learning note:** `M5-PLAT-04-command-router.md` (concepts: the chain-of-responsibility pattern, extension points)

### M5-T05 · `privacyGate`, `ai.enabled`, events checklist (PLAT-04)
- **Depends on:** M1-T07
- **Read first:** [ai-hooks.md §1, §4, §6, §7](../05-system-design/ai-hooks.md)
- **Goal:** the AI-ready hooks exist and are tested, but no AI is used.
- **Steps:**
  1. `privacyGate(notes)` in shared: removes `locked` or `ai_visible = 0` notes. Unit test.
  2. `users.settings.ai.enabled` defaults to `false`, and the UI hides any AI slot (the "Chat" tab is not rendered).
  3. Check the domain events: a test that create, update, delete and complete each write a `domain_events` row.
  4. Tick every box in ai-hooks §7.
- **Done when:** the checklist is fully ticked.
- **Learning note:** `M5-PLAT-04-ai-ready.md` (concepts: designing for future features without building them, privacy by design)
