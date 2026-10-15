# Tech stack

> Status: Draft for review · Last updated: 2026-10-15 · Related: [architecture](architecture.md), [hosting](hosting-and-distribution.md), [zero-cost](../02-product/zero-cost.md)
>
> Every item must be **free** (open-source license or a free plan with no card). Free-plan facts were verified on 2026-10-15 (see [hosting §Free-tier limits](hosting-and-distribution.md#free-tier-limits)). ✅ = verified. ✅ (lib) = open-source license, latest version checked on npm.
> Library choices marked **(proposed)** are Claude's recommendations. They become final when this doc is approved.

## 1. Monorepo layout (D-36)
```
cue/
├─ apps/
│  ├─ mobile/        React Native + Expo (Android), with native Kotlin modules in modules/
│  ├─ web/           React + Vite PWA
│  ├─ extension/     Chrome MV3 extension (WXT)
│  └─ api/           Cloudflare Worker (Hono) + D1 migrations
├─ packages/
│  ├─ shared/        types, zod schemas, sync protocol, date parsing, command router, search interface
│  ├─ ui-tokens/     design tokens (HEX) → web.css, native.css, NativeWind preset (D-50, ui-ux-design §11.2)
│  └─ config/        tsconfig, eslint, prettier presets
├─ docs/             these docs
├─ AGENTS.md
├─ package.json      pnpm workspaces
└─ turbo.json        Turborepo task pipeline
```

## 2. Choices by layer

| Layer | Choice | Why | License / cost | Cost check |
|---|---|---|---|---|
| Language | **TypeScript** everywhere (D-30) | The owner knows Node.js; shared code | Free | ✅ |
| Monorepo | **pnpm workspaces + Turborepo** (proposed) | Fast installs, cached builds, simple | MIT | ✅ (lib) |
| **Android app** | **React Native via Expo SDK 57** (RN 0.86), using the *prebuild / dev-build* workflow (proposed) | Fast development, native modules allowed, **local Gradle builds are free** (no EAS needed) | MIT | ✅ (lib) |
| Native modules | **Kotlin** via the **Expo Modules API** (proposed) | One clean way to write Kotlin modules. Lets the owner learn Kotlin step by step | MIT | ✅ (lib) |
| Android widget | **Jetpack Glance** (Kotlin) (proposed) | Modern Kotlin widget API; good Kotlin practice | Apache-2.0 | ✅ (lib) |
| Android local DB | **expo-sqlite** + **Drizzle ORM** (proposed) | SQLite on the device; Drizzle also supports D1, so the schema style is shared | MIT / Apache-2.0 | ✅ (lib) |
| Android speech | **expo-speech-recognition** (on-device Android SpeechRecognizer) (proposed) | Free, no server (D-26) | MIT | ✅ (lib) |
| Android secure storage | **expo-secure-store** (Keystore-backed) + **expo-local-authentication** (biometric) | Device private key, vault key, app lock | MIT | ✅ |
| Navigation / state | **Expo Router**; **Zustand** for UI state; data comes from the local DB | Simple and common | MIT | ✅ |
| **Web app** | **React + Vite**, PWA via **vite-plugin-pwa** (D-35); **online-only** (D-39) | Installable, Web Push. No offline data | MIT | ✅ |
| Web / extension data | **TanStack Query** (cache + refetch on focus) calling the typed API client (proposed). The extension capture queue lives in `chrome.storage.local` | Simple online data layer; no local DB on web | MIT | ✅ |
| Web routing / UI | React Router; **Tailwind v4 + shadcn/ui (Radix base) + lucide-react + sonner + cmdk** (D-50) | Low-tier models know these best; components are copied into the repo | MIT/ISC | ✅ |
| Mobile UI | **NativeWind 4.2 (Tailwind 3.4) + React Native Reusables + lucide-react-native + sonner-native + @gorhom/bottom-sheet + FlashList v2** (D-50) | shadcn-style on mobile, the same look as the web | MIT/ISC | ✅ |
| Design tokens | `packages/ui-tokens` (HEX) → web CSS, native CSS, NativeWind preset ([ui-ux-design §11.2](../07-design/ui-ux-design.md#112-design-tokens-pipeline-one-source--all-platforms)) | One source of truth | — | ✅ |
| Font | **Inter** (OFL): `@fontsource-variable/inter` (web/ext), `@expo-google-fonts/inter` (mobile) | Legible, the shadcn default | OFL-1.1 | ✅ |
| **Extension** | **WXT** (Vite-based MV3 framework) + React (proposed) | Hot reload, TypeScript, builds the unpacked folder | MIT | ✅ (lib) |
| **API + web hosting** | **Cloudflare Workers + Hono**, serving the PWA as static assets (D-32, D-41) | TypeScript, tiny, Workers-native; one deploy | MIT / Free plan | ✅ |
| Server DB | **Cloudflare D1** (SQLite) + **Drizzle** migrations | Free tier; same SQL dialect as the device | Free plan | ✅ (lib) |
| JWT | **jose** (works in Workers, WebCrypto-based) (D-48) | Sign and verify access tokens | MIT | ✅ (lib) |
| IDs | **uuidv7** (D-48) | Time-ordered client-side IDs | MIT | ✅ (lib) |
| Zip export | **fflate**, in the browser (D-48) | Client-side export zip | MIT | ✅ (lib) |
| Validation | **zod** schemas in `packages/shared` (used by both clients and the API) | One source of truth for payloads | MIT | ✅ |
| Date parsing | **chrono-node** (rule-based, no AI) (proposed) | Parses "tomorrow 6pm", "every Monday" (recurrence needs a small wrapper) | MIT | ✅ (lib) |
| Recurrence | **rrule** (RFC 5545 rules) (proposed) | Standard recurring reminders | BSD | ✅ (lib) |
| TOTP | **otpauth**, *bare* build with WebCrypto HMAC (proposed) | RFC 6238; works in Workers and RN | MIT | ✅ (lib) |
| Crypto | **WebCrypto** (built into Workers, browsers, and RN via `react-native-quick-crypto` or Expo crypto) | ECDSA P-256 device keys, AES-GCM for locked notes, SHA-256 | Free | ✅ (lib) |
| Telegram bot | *Future (D-44)*: **grammY** (webhook mode on Workers) | Typed and well maintained | MIT | ✅ (lib) |
| Push to Android | **FCM HTTP v1**, called from the Worker with a service-account JWT | Free | Free | ✅ (lib) |
| Web Push (PWA) | **@pushforge/builder** (WebCrypto, Workers-compatible). Not `web-push` (Node-only) | Free standard | MIT | ✅ (lib) |
| Testing | **Vitest** (shared, api, web), **Jest** (RN), **Playwright** (web E2E) | Standard | MIT / Apache | ✅ |
| CI | **GitHub Actions** (free for public repos; limited free minutes for private repos) | Lint, test, build the APK artifact | Free tier | ✅ (lib) |

## 3. `packages/shared`: what lives there
| Module | Contents | Used by |
|---|---|---|
| `model/` | TypeScript types + zod schemas for Note, Todo, Reminder, Group, Tag, Source, Suggestion, Device | all |
| `sync/` | Change and outbox types, the push/pull protocol, the conflict rule ([sync.md](sync.md)) | all clients + api |
| `time/` | chrono-node wrapper (`parseWhen(text, tz)`), rrule helpers, office-hours check `isWithinSchedule(device, instant)` | all |
| `router/` | Command router: `parse(input) → {command, args}` for quick-add, and later a bot (PLAT-04) | web, mobile, extension, api |
| `search/` | `SearchProvider` interface + keyword implementation (PLAT-04, ORG-07) | clients |
| `events/` | Domain event names and payload types (PLAT-04) | all |
| `api-client/` | Typed fetch client for the Worker endpoints | clients |
| `crypto/` | Locked-note envelope format, key-wrapping helpers (thin wrappers over WebCrypto) | clients |

## 4. Versions
Pin exact versions in `package.json` when the repo is created, and record the date. Use the current Expo SDK, React Native and Node LTS at that time.

## 5. Learning path for Kotlin (owner goal)
The native modules are small and isolated. Build them in this order (easiest first):
1. `CueTile`: a Quick Settings tile that opens the capture screen (about 50 lines).
2. Share target: an intent filter plus passing the text to RN.
3. `CueWidget`: a Glance widget with New / Voice / Today buttons.
4. `CueAlarms`: AlarmManager exact alarms, a boot receiver, notification actions (the most important one; see [reminders-engine.md](reminders-engine.md)).
