# Hosting and distribution (zero cost)

> Status: Draft for review · Facts verified: **2026-10-15** (official docs; re-check before relying on them) · Related: [zero-cost](../02-product/zero-cost.md), [tech-stack](tech-stack.md), decisions D-21, D-27, D-32, D-38, D-40…D-43

## 1. What runs where
| Piece | Where | Cost |
|---|---|---|
| API + **web PWA static files** | **One Cloudflare Worker** with *static assets* (D-41). Same origin, so no CORS | Free plan |
| Database | Cloudflare D1, one database `cue-prod` (plus `cue-dev`) | Free plan |
| Scheduled jobs | Cron Triggers on the same Worker: `* * * * *` (reminders) and `0 21 * * *` (daily maintenance at 02:30 IST) | Free (5 triggers per account) |
| Audio (optional) | **D1 BLOB chunks** (D-40) | Free |
| Android push | Firebase Cloud Messaging (Spark plan, no card) | Free |
| Web Push | Standard VAPID, sent from the Worker | Free |
| Code + CI + APK downloads | GitHub (repo, Actions, Releases) | Free tier |
| URL | `cue-api.<account>.workers.dev` (no custom domain needed) | Free |

## Free-tier limits
Verified 2026-10-15. **H** = an official page states it; **M** = official but indirect.

| Service | Limit | Impact on Cue | Conf |
|---|---|---|---|
| Workers requests | **100,000 / day** (resets 00:00 UTC = 05:30 IST) | One user: extension tick about 1.5k + cron 1.4k + app traffic, so plenty of headroom | H |
| Workers CPU | **10 ms per invocation, cron included** | Keep handlers light: no password hashing, cache the FCM token, batch sends (§3) | H |
| Subrequests | 50 per invocation | Cap pushes per cron run | H |
| Cron Triggers | 5 per account; minimum 1 minute | We use 2 | H |
| D1 storage | **500 MB per database**, 5 GB per account | Text is tiny; audio must be capped (D-40) | H |
| D1 rows | **5M read / 100k written per day**; queries fail until 00:00 UTC if exceeded | Sync about 3 writes per change. Rate limits and events also write, so prune and keep them lean | H |
| D1 row / BLOB size | **2 MB** per row; SQL statement 100 KB; **50 queries per invocation** | Audio chunked at 1.5 MB; batch queries per request | H |
| D1 encryption at rest | AES-256 (GCM) | Satisfies D-04 for normal notes | H |
| D1 Time Travel | Included on Free, **7 days** point-in-time restore | DATA-04 base layer (§4) | H |
| R2 | Free tier exists, but **enabling it needs a payment method** | ❌ **Not used** (D-40) | M-H |
| Durable Objects | Available on Free (SQLite-backed) | Not needed in MVP (we poll for login status) | H |
| Rate Limiting binding | GA; not confirmed on Free | Use the D1 `rate_limits` table; try the binding later | M |
| Pages | 500 builds/month; docs say "start new projects with Workers" | Use Workers static assets instead (D-41) | H |
| FCM HTTP v1 | No cost on Spark; callable from any server via a service-account JWT | Fine | H |
| Telegram | Free. About 1 msg/s per chat, 30 msg/s overall. `getFile` ≤ 20 MB; `file_id` has no documented expiry (store `file_unique_id` too) | Future only (D-44) | H/M |
| Expo / EAS | Local Gradle builds are free and need no EAS. EAS Free = 15 Android builds/month | Build locally; EAS optional | H |
| Expo SDK | **57** stable (RN 0.86, React 19.2); SDK 58 beta | Start on SDK 57 | H |
| Chrome `chrome.alarms` | Minimum 30 s (no limit when unpacked) | 1-min tick is fine | H |
| Extension Web Push | Supported in the MV3 service worker (Chrome 121+) | Option to replace polling later | H |
| Android exact alarms | **`USE_EXACT_ALARM`** is granted at install; the alarm-app-only rule is Play policy, not the OS. Sideloaded apps can declare it | Use it (D-42) | M-H |
| Store fees | Play $25 one-time; Chrome Web Store about $5 one-time (card) | Not paid (D-38) | H/M |
| **Android developer verification** | Starts 2026-09-30 in 4 countries; **global in 2027**. Free paths remain: ADB install, a free "Limited Distribution" account (≤ 20 devices), or the "advanced flow" (one-day wait) | Personal use stays free; see risk K-11 | H |

## 2. Libraries confirmed for Workers
- **grammY:** has an official Workers guide (`webhookCallback(bot, "cloudflare-mod")`).
- **Web Push:** use **`@pushforge/builder`** (MIT, WebCrypto, lists Workers support). `web-push` is Node-only and not recommended.
- **FCM:** RS256 JWT signed with WebCrypto. **Cache the OAuth access token (about 1 h) in D1** so most invocations don't sign.
- **otpauth:** use its *bare* build with WebCrypto HMAC (Workers and React Native aren't officially listed).

## 3. Staying inside 10 ms CPU
- No slow password hashing on the server (there are no passwords). Vault PBKDF2 runs on the client ([auth-flows](auth-flows.md)).
- The cron dispatcher: one indexed query, **at most 20 sends per run**, and the rest carry over to the next minute.
- Web Push payload encryption is the costliest operation. Keep payloads tiny (`{id, title, time}`) and measure in staging.
- CPU time doesn't count time waiting on I/O (fetch, D1). Measure with `wrangler tail` and the dashboard CPU metrics.

## 4. Backups (DATA-04)
1. **D1 Time Travel** (free, 7 days) for "undo the last week".
2. **Monthly export reminder**: a normal reminder note "Cue: download your monthly backup" that opens Export (DATA-03). Save the zip anywhere you like, e.g. a personal Google Drive.
3. Manual **Export** in the web app at any time.

## 5. Environments
| Env | Worker | D1 | Notes |
|---|---|---|---|
| dev | `wrangler dev` (local) | local D1 | Use `wrangler dev --test-scheduled` to trigger the cron |
| prod | `cue-api` | `cue-prod` | Secrets via `wrangler secret put` (JWT secret, TOTP key, VAPID keys, FCM service account) |

## 6. Distribution (D-38)
- **Android:** `./gradlew assembleRelease` locally, or a GitHub Actions job builds the APK → GitHub Release → download on the phone → install (allow "install unknown apps" once). Keep the **signing keystore backed up**, because losing it means you can't update the app.
- **2027 verification:** when it goes global, register a free *Limited Distribution* account or install via ADB (K-11).
- **Extension:** `pnpm --filter extension build` → load `.output/chrome-mv3` unpacked in Chrome or Edge.
- **Web:** `wrangler deploy` publishes the API and the PWA together.
