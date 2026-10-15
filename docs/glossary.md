# Glossary

> Status: Living · Last updated: 2026-10-15

| Term | Meaning in this project |
|---|---|
| **Inbox** | The single place every quick capture lands before it is organised (CAP-01). |
| **Note** | Free text (plus an optional source and attachments). Can be turned into a todo. |
| **Todo** | A note with a done/not-done state. It may have a reminder. |
| **Group** | A collection of notes (e.g. "Shopping"). A note is in at most one group. |
| **Tag** | A label. A note can have many. |
| **Important** | A flag. Its reminders repeat until done or snoozed (REM-07). |
| **Locked note** | A note encrypted on the device (E2EE). The server, AI and bots never see its content (SEC-07). |
| **Source / auto-context** | Where a capture came from: page URL, app, or channel (CAP-10). |
| **Page-linked note** | A note tied to a URL or site. The extension shows a badge when you revisit it (EXT-04). |
| **Text fragment** | A URL ending `#:~:text=…` that opens a page scrolled to and highlighting specific text. |
| **Quiet hours / office hours** | Per-device time windows when that device may show notifications (REM-08). |
| **Outbox** | The queue of changes made offline, waiting to sync (CAP-09). |
| **Conflict copy** | A second version kept when the same note was edited on two devices offline (DATA-01). |
| **E2EE** | End-to-end encryption: only the user's devices hold the key. |
| **TOTP** | Time-based one-time password: the 6-digit code from an authenticator app. Backup factor only here. |
| **Push approval** | Logging in by approving a request on the trusted phone (SEC-01). |
| **Number matching** | The new device shows a number, and the user picks the same number on the phone. Stops accidental or spam approvals. |
| **QRLJacking** | A phishing attack where someone relays a live login QR code to a victim. |
| **Passkey** | Passwordless login based on WebAuthn (fingerprint or face on the device). Future (FUT-12). |
| **Recovery codes** | One-time codes saved at signup to regain access (SEC-06). |
| **RAG** | Retrieval-augmented generation: the AI first finds relevant notes, then answers using them (AI-01). |
| **Embeddings** | Number vectors that represent meaning. Used for meaning-based search (AI-02). |
| **On-device AI** | A model running on the phone or in the browser. Free and private. |
| **Ollama** | A free tool to run open-source AI models on your own PC. |
| **MCP** | Model Context Protocol: an open standard that lets AI tools like Claude read and write app data (FUT-05). |
| **FCM** | Firebase Cloud Messaging: Google's free push-notification service for Android and web. |
| **Sideload** | Installing an Android APK directly, without the Play Store. |
| **Zero-cost rule** | D-21: building and running the app must cost nothing. |
