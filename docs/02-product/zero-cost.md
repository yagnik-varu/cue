# Zero-cost guide (rule D-21)

> Status: Living · Last updated: 2026-10-15 · Related: [decisions](decisions.md) (D-21, D-22, D-23), [open questions](../04-open-questions.md)
>
> **Rule:** the app is free, and building and running it must cost the owner **nothing**. Every dependency must be open source, self-hosted, on-device, or on a permanent free tier.
>
> ⚠ Free tiers and store fees change. Re-check each item when it is chosen in the system design and record the date. Items marked *(verify)* were not re-checked in research.
>
> This guide lists **candidates**, not final choices. The tech stack is decided in [05-system-design](../05-system-design/README.md).

## 1. Things that normally cost money, and the free path

| Need | Usual paid option | Free path for us | Trade-off |
|---|---|---|---|
| Publish the Android app | Google Play developer account (one-time fee, about $25) *(verify)* | **Install the APK directly** (sideload from GitHub Releases or a direct link). F-Droid if the app is fully open source | No Play Store listing; fine while it's "me first". Revisit when going public |
| Publish the browser extension | Chrome Web Store (one-time fee, about $5) *(verify)* | **Load unpacked** in Chrome/Edge for personal use. Firefox Add-ons and Edge Add-ons publishing *(verify: believed free)* | Unpacked extensions show a developer-mode warning |
| iOS app | Apple Developer Program (about $99/yr) | **Not possible for free.** iOS stays in Future (FUT-11) | — |
| Domain name | Paid domain | Free `*.workers.dev` subdomain (chosen) | Less pretty URL |
| Server / API hosting | Paid VPS or cloud | **Permanent free cloud tiers only (D-27)**, e.g. Oracle Cloud Always Free, Cloudflare Workers, Firebase Spark, Supabase free. ~~Self-host on the owner's PC~~ is ruled out because the PC is off most of the time | Free tiers have limits; some pause when inactive (risk K-10) |
| Database | Managed paid DB | **Cloudflare D1 free** (chosen, D-32): 500 MB per DB, 100k writes/day | Size and write limits |
| File / audio storage | Paid object storage. ⚠ **Cloudflare R2 requires a card even for its free tier** | **D1 BLOB chunks, capped** (D-40) | 100 MB audio cap |
| Push notifications | Paid push services | **Firebase Cloud Messaging** (free), **Web Push / VAPID** (free, open standard) | None significant |
| Messaging bot | WhatsApp Business (pay per message), Twilio SMS (paid) | **None in MVP** (D-44). Future: Telegram Bot API (free) | WhatsApp deferred (D-22) |
| Speech-to-text | Paid cloud speech-to-text (OpenAI, Google Cloud) | **Android on-device SpeechRecognizer**, the browser **Web Speech API**, or self-hosted **Whisper / whisper.cpp** | Chrome's Web Speech may send audio to Google; on-device or Whisper is more private |
| AI chat / organising | Paid LLM APIs | **Not in MVP (D-24).** Later: see §2 | — |
| Embeddings (meaning search) | Paid embedding API | **Not in MVP (D-24).** Later: small open-source models run locally | — |
| Login: push approval, QR, TOTP | Paid auth providers (some) | Build with free open-source libraries (TOTP and WebAuthn libraries exist for every stack). FCM for push | More work than a paid service |
| Email (recovery, alerts) | Paid email API | **Avoid needing email.** Recovery uses codes + trusted devices (SEC-06), alerts go via push | No email recovery |
| Monitoring / error tracking | Paid APM | Free tiers or self-hosted, plus simple logs | — |
| Backups | Paid storage | D1 Time Travel (7 days, free) + monthly export reminder + manual export | See [hosting §4](../05-system-design/hosting-and-distribution.md#4-backups-data-04) |

## 2. AI at zero cost (decision D-23, open question Q-01)
> **AI is deferred to Future (D-24).** This section is kept as input for when AI is added; see [future/ai-integration.md](../future/ai-integration.md).

"Free" and "never trains on my notes" pull in opposite directions:

| Option | Cost | Privacy | Quality | Notes |
|---|---|---|---|---|
| **A. On-device AI** (Android on-device models such as Gemini Nano via ML Kit GenAI; Chrome built-in Prompt API in the extension) | Free | ✅ Best: nothing leaves the device | Medium; only some devices | Good for tagging, date extraction, short summaries |
| **B. Self-hosted open model** (Ollama / llama.cpp on a free cloud VM) | Free (within VM limits) | ✅ Data stays with you | Medium; free VMs are small | ⚠ The owner's PC is **not** an option because it's off most of the time (D-27) |
| **C. Free cloud AI tier** (e.g. Gemini API free tier, other free-model providers) | Free within limits | ⚠ Free tiers **may use your data to improve products** *(verify each provider's terms)* | Good | Only as an **opt-in**, with a warning (AI-06) |
| D. Paid API with no-training terms | 💰 Costs money | ✅ | Best | ❌ Breaks D-21 |

**Proposed default:** A + B first. C only when the user turns it on per feature. Locked notes never go anywhere (SEC-07).

## 3. Rules for agents writing code
- Before adding any dependency or service, check that it is free (license + pricing) and note it in the system-design doc.
- Prefer open-source libraries with permissive licenses (MIT, Apache-2.0, BSD).
- Design for free-tier limits: small payloads, batching, caching, and no polling loops when push is available.
- Never add a feature that requires a credit card on file to run.
