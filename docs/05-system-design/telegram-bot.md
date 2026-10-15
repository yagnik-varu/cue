# Telegram bot design

> Status: **Future (D-44): not built in MVP.** The owner doesn't use Telegram. Kept as a ready design · Last updated: 2026-10-15 · Related: [auth-flows §8](auth-flows.md#8-telegram-linking-bot-01), [ai-hooks §5](ai-hooks.md#5-command-router), [reminders-engine](reminders-engine.md), features BOT-01, BOT-02, decisions D-22, D-26, D-37

## 1. Setup
- Create the bot with @BotFather (free). Suggested name **Cue**, handle e.g. `@CueNotesBot` (check availability).
- `setWebhook` → `https://<api>/telegram/webhook` with `secret_token`. The token and the secret are stored as Worker secrets.
- Library: **grammY** in webhook mode inside the Worker (proposed).

## 2. Message handling (through the command router)
| Input | Result |
|---|---|
| `/start` | Welcome + "Link your account: open Cue → Settings → Link Telegram" |
| `/link CODE` | Links `chat_id` to the user ([auth-flows §8](auth-flows.md#8-telegram-linking-bot-01)) |
| Any text | New Inbox note (`source_kind: telegram`). Reply: "Saved to Inbox." + [Undo] |
| Text with a date ("pay bill tomorrow 6pm") | Note + reminder (rule-based). Reply: "Saved · Reminder Tue 6:00 PM" + [Undo] [Change time] |
| Voice message | Audio note: store the Telegram `file_id` in `audio_ref` (no download, no transcript in MVP; D-26). Reply: "Voice note saved (audio)." |
| Photo / document | Note with the caption + `file_id` reference |
| `/inbox`, `/today` | Lists (max 10) with [Done] buttons |
| `/find words` | Keyword search, top 5, with links that open the note in the web app |
| `/unlink` | Removes the link |
| Unlinked chat | Every message → "Please link first: /link CODE" |

The first message after linking explains: **"Telegram chats aren't end-to-end encrypted. Notes sent here are normal (not locked) notes."** (BOT-01 AC4)

## 3. Reminder messages (BOT-02)
- Format: `Cue: Pay electricity bill · 6:00 PM`, with inline buttons **✅ Done · ⏰ 10m · 1h · Tomorrow**.
- `callback_query` → `applyChange` (ack or snooze) → the message is edited to "✅ Done" or "⏰ Snoozed until 7:00 PM".
- Escalations for important reminders say: `Still pending: Pay electricity bill (3rd reminder)`.

## 4. Limits and safety
- Rate limits: about 1 msg/s per chat, about 30 msg/s overall. For one user this is far below them.
- Store both `file_id` and `file_unique_id` (no documented expiry). Playback: the API calls `getFile` (≤ 20 MB, link valid ≥ 1 h) and streams it through `/files/:ref`.
- No AI in MVP. A future `ai` router handler answers questions ([future/ai-integration.md](../future/ai-integration.md)).
