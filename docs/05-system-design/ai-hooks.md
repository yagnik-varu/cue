# AI-ready hooks (no AI in MVP)

> Status: Draft for review · Last updated: 2026-10-15 · Related: [future/ai-integration.md](../future/ai-integration.md), decisions D-24, D-25, feature PLAT-04
>
> This doc says **how** the PLAT-04 hooks are built in the MVP. What AI does with them later is in the future guide.

## 1. Domain events
- **Where:** emitted inside `applyChange` ([sync.md §3](sync.md#3-the-server-write-rule-used-by-syncpush-and-by-the-webextension-rest-endpoints)).
- **Names** (in `packages/shared/events`): `note.created`, `note.updated`, `note.deleted`, `todo.completed`, `reminder.fired`, `reminder.acked`.
- **Payload:** `{ event_id, user_id, note_id, changed_fields, at }`. **Never the note content** (consumers load it, and the privacy gate checks it).
- **Delivery in MVP:** an in-process `EventBus` in the Worker with **zero subscribers by default**, plus a row in `domain_events` (pruned after 90 days). Later, AI workers subscribe or read the table.

## 2. Suggestions
- Table `suggestions` ([data-model](data-model.md#suggestions-plat-04)), synced to Android and readable by web.
- **MVP producer:** the rule-based date detector (`type: reminder_date`, `source: rules`).
- **UI:** the triage view and the note editor show pending suggestions as chips: **Accept** or **Dismiss**. The same component later shows AI suggestions such as group, tags, make-todo or split.
- **Contract:** a suggestion never changes data on its own. Accepting it runs a normal `applyChange`.

## 3. Search interface
```ts
// packages/shared/search
export interface SearchProvider {
  search(q: string, opts: { limit: number; filters?: { state?: string; groupId?: string; isTodo?: boolean } })
    : Promise<Array<{ noteId: string; score: number; snippet: string }>>;
}
// MVP: KeywordSearch (Android: SQLite FTS5; server: D1 FTS5)
// Future: SemanticSearch (embeddings) and HybridSearch(keyword, semantic)
```

## 4. Privacy flags
- `notes.locked` and `notes.ai_visible` exist from the first migration. `locked = 1` forces `ai_visible = 0` (enforced in `applyChange`).
- Future code that sends anything to AI must go through **one** function, `privacyGate(notes)`, which removes locked and non-visible notes. Write this function and its unit test in the MVP, even though nothing calls it yet. It's cheap, and it locks the rule in.

## 5. Command router
```ts
// packages/shared/router
type Handler = { name: string; match(input: string): boolean; run(ctx, input): Promise<Reply> };
// MVP handlers (quick-add), in order: find, remind (rule-based date), capture (default)
// Future bot handlers: link, done, snooze, today, inbox
// Future: an "ai" handler inserted before "capture" when ai.enabled = true
```
- The router serves the in-app quick-add commands (e.g. `/find car`) now, and a future bot later.

## 6. Feature flag
- `users.settings.ai.enabled` defaults to `false`. Clients hide all AI UI when it's false. The "Chat" tab slot is reserved but not rendered (see [ux-principles](../02-product/ux-principles.md)).

## 7. Done when (M5 checklist)
- [ ] Events written for every note and todo change (test).
- [ ] Rule-based date suggestions appear and can be accepted or dismissed.
- [ ] `SearchProvider` used by every search screen.
- [ ] `privacyGate` exists, and its test proves locked notes are removed.
- [ ] The router handles all quick-add commands. Adding a handler needs no change outside `router/`.
- [ ] `ai.enabled` is false and no AI UI is visible.
