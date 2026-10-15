# Concept map: what Cue teaches

> Status: Living · Last updated: 2026-10-15 · Related: [learning README](README.md), [roadmap](../03-roadmap.md), [system design](../05-system-design/README.md)
>
> Tick `[x]` when a learning note covers the concept and the owner feels comfortable explaining it.
> "Where" names the milestone and the doc or feature where Cue uses the concept. Links go to official docs.

## A. Backend and system design
| ✓ | Concept | Where in Cue | Career value | Learn more |
|---|---|---|---|---|
| [ ] | REST API design (resources, status codes, validation, error shape) | M1–M5 · [api.md](../05-system-design/api.md) | Every backend job | [Hono](https://hono.dev/) |
| [ ] | Serverless and edge functions, CPU limits, cold starts | M1 · D-32, D-43 | Modern deployment model | [Cloudflare Workers](https://developers.cloudflare.com/workers/) |
| [ ] | Request validation with shared schemas | M1 · `packages/shared` | Type-safe APIs | [zod](https://zod.dev/) |
| [ ] | Relational schema design, indexes, migrations (SQLite) | M1 · [data-model.md](../05-system-design/data-model.md) | Core database skill | [D1](https://developers.cloudflare.com/d1/), [SQLite query planner](https://www.sqlite.org/queryplanner.html) |
| [ ] | ORM vs raw SQL | M1 | Choosing tools | [Drizzle](https://orm.drizzle.team/) |
| [ ] | Offline-first sync: versions, outbox, idempotency (`op_id`), soft deletes | M1 · [sync.md](../05-system-design/sync.md) | Distributed-systems thinking | [sync.md](../05-system-design/sync.md) |
| [ ] | Conflict handling (keep both vs last-writer-wins) | M1 · DATA-01, D-17 | Data integrity | [sync.md §3](../05-system-design/sync.md#3-the-server-write-rule-used-by-syncpush-and-by-the-webextension-rest-endpoints) |
| [ ] | Auth without passwords: device keypairs, signatures | M2 · SEC-01 | Modern auth | [Web Crypto API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API) |
| [ ] | JWT access tokens + rotating refresh tokens | M2 · auth-flows §0 | Asked in most interviews | [RFC 7519 (JWT)](https://datatracker.ietf.org/doc/html/rfc7519) |
| [ ] | Push approval with number matching; QR login; QRLJacking defence | M2 · SEC-01, SEC-02 | Security design | [auth-flows.md](../05-system-design/auth-flows.md) |
| [ ] | TOTP (authenticator codes) | M6 · SEC-03 | Common 2FA | [RFC 6238](https://datatracker.ietf.org/doc/html/rfc6238) |
| [ ] | Rate limiting and brute-force protection | M2 · SEC-08 | API security | [auth-flows §9](../05-system-design/auth-flows.md#9-rate-limit-table-sec-08) |
| [ ] | End-to-end encryption: AES-GCM, key wrapping, PBKDF2 | M6 · SEC-07 | Applied cryptography | [Web Crypto API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API) |
| [ ] | Cron and background jobs, batching under limits | M3 · reminders-engine §4 | Scheduling systems | [Cron Triggers](https://developers.cloudflare.com/workers/configuration/cron-triggers/) |
| [ ] | Push notifications: FCM (Android) and Web Push (VAPID) | M2–M3 | Mobile and web backends | [FCM](https://firebase.google.com/docs/cloud-messaging), [Web Push](https://web.dev/articles/push-notifications-overview) |
| [ ] | Time zones, UTC storage, recurrence rules | M3 · D-28, REM-02 | Classic source of bugs | [RFC 5545 (RRULE)](https://datatracker.ietf.org/doc/html/rfc5545) |
| [ ] | Full-text search (FTS5) | M5 · ORG-07 | Search basics | [SQLite FTS5](https://www.sqlite.org/fts5.html) |
| [ ] | Event-driven design (domain events), extension points | M5 · PLAT-04 | Architecture | [ai-hooks.md](../05-system-design/ai-hooks.md) |
| [ ] | Designing within free-tier limits (quotas, budgets) | All · [hosting](../05-system-design/hosting-and-distribution.md) | Cost-aware engineering | [hosting](../05-system-design/hosting-and-distribution.md#free-tier-limits) |

## B. React and React Native
| ✓ | Concept | Where in Cue | Career value | Learn more |
|---|---|---|---|---|
| [ ] | React components, props, state, hooks | M2 · web app | Front-end core | [react.dev](https://react.dev/) |
| [ ] | Server state with caching and refetching | M2 · web (D-39) | Real-world data fetching | [TanStack Query](https://tanstack.com/query/latest) |
| [ ] | PWA: manifest, service worker, installability | M2–M3 · D-35 | Web platform | [web.dev PWA](https://web.dev/learn/pwa/) |
| [ ] | React Native basics (views, styling, lists, navigation) | M2 · Android app | Mobile development | [React Native](https://reactnative.dev/) |
| [ ] | Expo: prebuild, dev builds, local release builds | M1 · D-31 | Shipping mobile apps | [Expo docs](https://docs.expo.dev/) |
| [ ] | JS ↔ native bridge with Expo Modules | M2–M3 | Bridging to native code | [Expo Modules](https://docs.expo.dev/modules/overview/) |
| [ ] | Local SQLite on the device + live queries | M1 · CAP-09 | Offline mobile apps | [Drizzle + Expo SQLite](https://orm.drizzle.team/docs/sqlite/connect-expo-sqlite) |
| [ ] | Performance budget (startup ≤ 1 s, lazy loading) | M2 · PLAT-01 | Performance work | [React Native performance](https://reactnative.dev/docs/performance) |
| [ ] | Browser extensions (MV3): service worker, popup, context menu, side panel, alarms | M4 · EXT-xx | Niche but valuable | [Chrome extensions](https://developer.chrome.com/docs/extensions), [WXT](https://wxt.dev/) |

## C. Android and Kotlin
| ✓ | Concept | Where in Cue | Career value | Learn more |
|---|---|---|---|---|
| [ ] | Kotlin basics: types, null safety, classes, lambdas, coroutines (intro) | M2 · first module (`cue-tile`) | New language | [Kotlin docs](https://kotlinlang.org/docs/home.html) |
| [ ] | Android app components: Activity, BroadcastReceiver, Service; the manifest | M2–M3 | Android fundamentals | [Android developers](https://developer.android.com/guide/components/fundamentals) |
| [ ] | Intents and the share target | M2 · CAP-04 | Android fundamentals | [Receiving shared data](https://developer.android.com/develop/ui/compose/sharing/receive) |
| [ ] | Quick Settings tile (`TileService`) | M2 · CAP-03 | Android UI surfaces | [QS tiles](https://developer.android.com/develop/ui/views/quicksettings-tiles) |
| [ ] | Home-screen widget with Jetpack Glance | M2 · CAP-02 | Modern Android UI | [Glance](https://developer.android.com/develop/ui/compose/glance) |
| [ ] | Exact alarms (`AlarmManager`), boot receiver, permissions | M3 · REM-04, D-42 | Reliable background work | [Schedule alarms](https://developer.android.com/develop/background-work/services/alarms) |
| [ ] | Notifications: channels, actions, full-screen alerts | M3 · REM-02, REM-07 | Every mobile app | [Notifications](https://developer.android.com/develop/ui/views/notifications) |
| [ ] | Battery optimisation and OEM restrictions (Oppo/Vivo/Realme/OnePlus) | M3 · REM-05, D-46 | Real-world Android pain | [dontkillmyapp.com](https://dontkillmyapp.com/) |
| [ ] | Keystore, biometrics, secure storage | M2/M6 · SEC-04, SEC-07 | Mobile security | [Android Keystore](https://developer.android.com/privacy-and-security/keystore) |
| [ ] | Debugging with `adb` and Logcat | All | Daily Android work | [Logcat](https://developer.android.com/tools/logcat) |

## E. UI/UX and design systems
| ✓ | Concept | Where in Cue | Career value | Learn more |
|---|---|---|---|---|
| [ ] | Design tokens (one source for colours, type, spacing) | M2-T01 · [ui-ux-design §3](../07-design/ui-ux-design.md#3-design-tokens) | Design-system engineering | [ui-ux-design §11.2](../07-design/ui-ux-design.md#112-design-tokens-pipeline-one-source--all-platforms) |
| [ ] | Utility-first CSS (Tailwind) | M2 · web, extension | Industry standard | [Tailwind + Vite](https://tailwindcss.com/docs/installation/using-vite) |
| [ ] | Headless + copy-in components (shadcn/ui, Radix) | M2 · web | Modern React UI | [shadcn/ui](https://ui.shadcn.com/docs/installation/vite) |
| [ ] | Tailwind on React Native (NativeWind) + Reusables | M2 · mobile | Cross-platform styling | [NativeWind](https://www.nativewind.dev/docs/getting-started/installation) |
| [ ] | Mobile UX patterns: FAB, bottom nav, swipe + undo, bottom sheets | M2 · [§6](../07-design/ui-ux-design.md#6-mobile-app-android) | Product sense | [M3 navigation bar](https://m3.material.io/components/navigation-bar/guidelines) |
| [ ] | Keyboard-first web UX: command palette, shortcuts | M2 · [§7.7–§7.8](../07-design/ui-ux-design.md#77-command-palette) | Pro tools | [Superhuman palette](https://blog.superhuman.com/how-to-build-a-remarkable-command-palette/) |
| [ ] | Accessibility: contrast, focus, screen readers, font scaling | All · [§10](../07-design/ui-ux-design.md#10-accessibility-checklist) | Required in serious teams | [WCAG 2.2](https://www.w3.org/TR/WCAG22/) |
| [ ] | UI states: empty, loading, error, offline | All · [§9](../07-design/ui-ux-design.md#9-states-checklist) | Polish that users notice | [NN/g empty states](https://www.nngroup.com/videos/empty-states-in-application-design-guidelines/) |

## D. Engineering practice
| ✓ | Concept | Where in Cue | Career value | Learn more |
|---|---|---|---|---|
| [ ] | Monorepo with workspaces and a task runner | M1 · D-36 | Common in companies | [pnpm workspaces](https://pnpm.io/workspaces), [Turborepo](https://turborepo.dev/docs) |
| [ ] | Shared TypeScript types across apps | M1 · `packages/shared` | Type safety end to end | [TypeScript handbook](https://www.typescriptlang.org/docs/handbook/intro.html) |
| [ ] | Unit tests (Vitest), RN tests (Jest), end-to-end tests (Playwright) | All · test cases in the design docs | Quality and confidence | [Vitest](https://vitest.dev/), [Playwright](https://playwright.dev/) |
| [ ] | CI: lint, test, build the APK on every push | M1 | DevOps basics | [GitHub Actions](https://docs.github.com/en/actions) |
| [ ] | Git workflow: small commits, branches, clear messages with feature IDs | All | Team work | [Conventional Commits](https://www.conventionalcommits.org/) |
| [ ] | Secrets and environments (dev and prod) | M1 · hosting §5 | Production safety | [Wrangler commands](https://developers.cloudflare.com/workers/wrangler/commands/) |
| [ ] | Decision records (why, not just what) | All · [decisions.md](../02-product/decisions.md) | Senior-level habit | [ADR](https://adr.github.io/) |
| [ ] | Debugging across layers (Worker logs, Chrome DevTools, Logcat) | All | Problem solving | [Chrome DevTools](https://developer.chrome.com/docs/devtools) |
