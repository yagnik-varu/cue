# Emerging and novel features (2025–2026)

> Status: Final · Research date: 2026-10-15 · Related: [research index](README.md), [features](../02-product/features.md), [security](security-and-auth.md)
>
> Complexity: L = low, M = medium, H = high. "→" names the feature ID in our catalogue.

## A. AI over your notes

| Idea | Who does it | User value | Complexity | → |
|---|---|---|---|---|
| **Chat with your notes ("what did I note about X?")**: answers with citations | Mem 2.0 Agentic Chat, Reflect (on-device embeddings), Notion Agent, Obsidian Smart Connections and Khoj | Very high | M (cloud), H (E2EE, on-device) | AI-01 |
| **Auto-organise messy quick notes**: tag, group, clean up | Mem "Clean Up", mymind auto-tags, Apple Reminders "Auto-Categorize" (iOS 26) | High | L–M | ORG-03 |
| **Pull tasks and dates out of free text** ("call mom re passport next Tue 6pm") | Todoist Ramble, Apple Intelligence, Saner.ai | Very high | L–M (LLM structured output plus a deterministic date-parser fallback) | REM-01, AI-03 |
| **Daily or weekly digest** | ChatGPT Pulse, Readwise Daily Review | High | L | FUT-02 |
| **Resurfacing**: spaced repetition, "on this day", random | Readwise, Recall, Obsidian plugins | Medium-high | L | FUT-03 |
| **Contextual resurfacing**: related notes appear while you write or browse | Mem "Heads Up", Recall Augmented Browsing | High | M | FUT-03 |

Sources:
- [Mem 2.0](https://get.mem.ai/blog/mem-2-dot-0)
- [Reflect AI search](https://www.producthunt.com/p/reflect-notes/reflect-ai-search)
- [Notion agents](https://www.trendingtopics.eu/notion-launches-ai-agents/)
- [Smart Connections](https://www.obsidianstats.com/plugins/smart-connections)
- [Local RAG in React Native](https://swmansion.com/blog/building-an-ai-powered-note-taking-app-in-react-native-part-3-local-rag-868ba75f818b)
- [mymind tagging](https://alternativeto.net/news/2025/10/mymind-introduces-automatic-note-summarization-and-tagging)
- [iOS 26 Auto-Categorize](https://9to5mac.com/2025/08/15/reminders-in-ios-26-has-a-powerful-new-tool-for-organizing-tasks/)
- [Todoist Ramble](https://www.todoist.com/help/articles/from-voice-to-tasks-ramble-july-1)
- [ChatGPT Pulse](https://www.tomsguide.com/ai/chatgpt-pulse-is-here-now-ai-starts-the-chat-and-curates-your-feed)
- [Recall Augmented Browsing](https://www.hunted.space/product/recall-6/launches/recall-augmented-browsing)

## B. Voice and wearables

| Idea | Who does it | User value | Complexity | → |
|---|---|---|---|---|
| **Voice capture → transcript → note or several todos** | Todoist Ramble (no audio stored), Mem Voice Mode, Reflect (Whisper, audio deleted), Apple Notes call transcription | Very high | L (cloud speech-to-text) to M (on-device) | CAP-05 |
| **AI pendants and pins** | Limitless (bought by Meta, Dec 2025), Bee (bought by Amazon, Jul 2025), Plaud NotePin S, Omi (open source) | Medium, niche, privacy concerns | H to build; M to integrate | FUT-09 (imports only) |

Takeaway: big tech is buying up the wearable market. Be the place captures *land* (imports and webhooks) rather than building hardware. A tap-to-record watch tile gets most of the value.

## C. Capture via messaging and email

| Channel | Who does it | Complexity | → |
|---|---|---|---|
| Telegram bot (text and voice) | Mem, Capacities, Coco, Remind Me Bot | **L** (free API) | BOT-01 |
| WhatsApp bot | Mem, Capacities | **M** (Meta Business verification, per-conversation pricing) | BOT-02 |
| Email-in (forward to a capture address) | Mem save@, Evernote, Notesnook Inbox API | L | FUT-07 |
| SMS | Mem | M (Twilio cost) | not planned |

Security note: bot messages pass through Telegram or Meta in plaintext. Treat bot capture as "normal" (not locked) notes and say so in the UI.

Sources:
- [Mem on the go](https://support.mem.ai/article/150-save-ideas-recommendations-on-the-go)
- [Capacities WhatsApp](https://docs.capacities.io/reference/integrations/whatsapp)

## D. Context-aware reminders

| Idea | Who does it | User value | Complexity | → |
|---|---|---|---|---|
| **Location reminders (arrive or leave)** | Apple Reminders, TickTick, Todoist Pro. **Google Keep removed them (Jan 2026)** | High (gap on Android) | M (OS geofence limits: ~20 on iOS, ~100 on Android; background-location policy review) | FUT-01 |
| **"Next time I'm on my laptop"**: deliver on whichever device becomes active first | Almost nobody (WakeMinder, Mac only) | Medium-high, clear gap | L–M | REM-10 (v2) |
| **"Next time I talk to person X"** | Apple "When Messaging" (iMessage only) | High | M (via calendar or bot chats); H (via the OS) | FUT-06 |
| **Device- and time-aware routing** (no work-PC pop-ups after hours) | Not found as a note-app feature | High for this user | L–M | REM-08 |

Sources:
- [Keep Wear OS reminders](https://9to5google.com/2026/01/23/google-keep-wear-reminders/)
- [Geofence limits](https://docs.mapp.com/limitations)
- [WakeMinder](https://peerpush.com/p/wakeminder)

## E. Browser extension

| Idea | Who does it | Complexity | → |
|---|---|---|---|
| Quick-note popup plus keyboard shortcut | Most clippers | L | EXT-01 |
| Right-click "save selection" with a **text-fragment link** (`#:~:text=`) back to the exact passage | Obsidian clipper. Text fragments are supported in Chrome 81+, Firefox 131+, Safari 18.4+ | L | EXT-02 |
| **Page-linked notes**: toolbar badge "2 notes on this site" when you revisit | NoterLight, Annoto, Sidenote, OneNote Sticky Notes | L–M (URL normalisation) | EXT-04 |
| Side panel with chat and notes (Chrome `sidePanel` API, Chrome 114+) | Sidenote | L–M | EXT-05 |
| Snip a region → OCR → note | (Microsoft Recall is a warning about "capture everything") | L–M | FUT-08 |

Sources:
- [Chrome sidePanel](https://developer.chrome.com/docs/extensions/reference/api/sidePanel)
- [Text fragments](https://jamesg.blog/2024/12/13/text-fragments)
- [NoterLight](https://chromewebstore.google.com/detail/fcchemoehjimdhajnfbnognhnkekbajh)
- [Annoto](https://addons.mozilla.org/firefox/addon/annoto/)

## F. OS-level quick capture (Android first)

| Surface | Notes | → |
|---|---|---|
| Home-screen widget (new note / voice / today's todos) | Very high value | CAP-02 |
| Quick Settings tile (`TileService`, label ≤ 18 characters) | Reachable from the lock screen | CAP-03 |
| Share sheet target ("Share to Notes" from any app) | Also saves the source app and URL | CAP-04 |
| Wear OS voice tile | Future | FUT-10 |
| iOS Control Center, Action Button, Shortcuts (App Intents) | When the iOS app exists | FUT-11 |

Source: [Android Quick Settings tiles](https://developer.android.com/develop/ui/views/quicksettings-tiles)

## G. Offline-first sync
- Options:
  - **Yjs or Automerge** (CRDTs): needed only for live co-editing of the same note.
  - **PowerSync or Electric**: Postgres synced to a SQLite database on the device.
  - **Simple approach**: last-writer-wins per note, an offline outbox, and keep-both on conflict.
- **Recommendation for a single user:** keep it simple. Capture must never fail on a bad network (CAP-09, DATA-01).
- The final choice belongs to the system design ([05-system-design](../05-system-design/README.md)).
- Sources: [local-first guide](https://devtoollab.com/blog/local-first-apps-crdt-sync-engine-guide-2026), [FOSDEM 2026 local-first track](https://archive.fosdem.org/2026/schedule/track/local-first/)

## H. AI on device vs in the cloud (for privacy)
- **Apple Foundation Models** (iOS 26), **Gemini Nano** (ML Kit GenAI, limited Android devices), and the **Chrome built-in Prompt API** (usable from extensions) are free and work offline. They are good for tagging, date extraction and embeddings, and weak at long reasoning across many notes.
- See [security-and-auth.md §B4](security-and-auth.md#b4-encryption-vs-ai) for the trade-offs.
- Sources: [Gemini Nano](https://developer.android.com/ai/gemini-nano/ml-kit-genai), [Chrome built-in AI](https://developer.chrome.com/docs/ai/built-in-apis)

## Most promising differentiators (shortlist)
1. Capture in under 2 s from anywhere into one Inbox, and AI sorts it later.
2. Context reminders beyond time: device routing now; laptop, location and person later.
3. Notes tied to the page you're on.
4. Chat with your notes, plus (later) digest and resurfacing, to close the forgetting loop.
5. Visible privacy: locked notes excluded from AI, no training.
6. Easy login: username → approve on phone → fingerprint.
