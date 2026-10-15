# Future: adding AI (integration guide)

> Status: Future, **not in MVP** (D-24) · Last updated: 2026-10-15 · Related: [features AI-01…AI-06, ORG-03, PLAT-04](../02-product/features.md), [decisions D-23–D-26](../02-product/decisions.md), [zero-cost §2](../02-product/zero-cost.md#2-ai-at-zero-cost-decision-d-23-open-question-q-01), [AI research](../01-research/emerging-features.md#a-ai-over-your-notes)

## Why AI is deferred
The core value works without AI: fast capture, one Inbox, quick triage, reminders that fire. AI (the chatbot,
organising, meaning search and the AI server) is postponed to keep the MVP small and free. The MVP is built
**AI-ready** (PLAT-04), so this guide should be all that's needed to add it later.

## Hooks the MVP already provides (PLAT-04)
| Hook | What it is | What AI will use it for |
|---|---|---|
| **Domain events** | `note.created`, `note.updated`, `note.deleted`, `todo.completed`, `reminder.fired` | Trigger AI jobs (tagging, grouping, indexing for meaning search) in the background |
| **Suggestion model** | `{id, type, target_note_id, payload, source, status: pending/accepted/rejected}` shown in the triage UI | AI creates suggestions (group, tags, "make todo", reminder date). The user accepts or rejects, so AI never changes data directly |
| **Search interface** | `search(query, filters) → results` with a keyword implementation | Add a meaning-based (embeddings) implementation, or combine both (AI-02) |
| **`locked` / `ai_visible` flags** | Stored on every note from day one | Enforce: locked or non-visible notes are **never** sent to AI (AI-06) |
| **Command router** | Bot and quick-add input → matching handler (`remind`, `find`, `done`, `snooze`, default `capture`) | Add an `ai` handler for questions and chat commands (AI-01, AI-03) |
| **Feature flag `ai.enabled`** | Default off | Turn AI UI on per user or device; roll out safely |
| **UI slots** | The extension side panel and the Android/web navigation leave room for a "Chat" view | Mount the chat UI without a redesign |

## How to add AI later (step by step)
1. **Pick the provider** under the zero-cost rule (D-21). The owner's PC can't host AI because it's off most of the time (D-27). That leaves on-device AI, a free cloud VM, or an opt-in free cloud tier. Options and trade-offs: [zero-cost.md §2](../02-product/zero-cost.md#2-ai-at-zero-cost-decision-d-23-open-question-q-01) (on-device, self-hosted Ollama, or an opt-in free cloud tier). Record a new decision D-xx.
2. **Implement one `AIProvider` interface** so providers can be swapped:
   ```
   AIProvider
     complete(prompt, context) -> text
     extract(text, schema)     -> structured JSON   (todos, dates, tags)
     embed(texts[])            -> vectors           (meaning search)
     transcribe(audio)         -> text              (optional: voice from a future bot)
   ```
3. **Add a privacy gate**, the only path to the provider. It drops any note where `locked = true` or `ai_visible = false`, and logs what was sent (count and IDs only, never the content).
4. **Build features in this order**, cheapest and most useful first:
   1. ORG-03 group and tag suggestions (event → `extract` → Suggestion)
   2. Splitting a voice ramble into several todos (CAP-05 Future)
   3. AI-02 meaning search (`embed` on `note.created` / `note.updated` → vector index → search interface)
   4. AI-01 ask your notes with citations (search → `complete` with the top notes → answer + links)
   5. AI-03 / AI-04 chat commands and "organise my Inbox" (command router `ai` handler → Suggestions)
5. **UX rules:** suggest, never force (P3). Every AI action is undoable. Show the AI-visible indicator (AI-05).
6. **Later:** expose the same operations through an MCP server (FUT-05).

## Must stay true
- Locked notes never reach any AI (SEC-07, AI-06).
- Notes are never used for training without the user's explicit opt-in and a clear warning.
- The app works fully with AI off or unreachable.
