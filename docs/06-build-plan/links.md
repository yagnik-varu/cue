# Documentation links (verified 2026-10-15)

> Status: Living · Related: [build plan README](README.md), [concept map](../learning/concept-map.md)
>
> All links were checked on 2026-10-15 (redirects were updated to their final URLs). If a link breaks later, search
> the same title on the official site, then update this file.

## Tooling and engineering practice
- [Node.js download](https://nodejs.org/en/download) · [pnpm install](https://pnpm.io/installation) · [pnpm workspaces](https://pnpm.io/workspaces)
- [Turborepo docs](https://turborepo.dev/docs) · [TypeScript tsconfig](https://www.typescriptlang.org/tsconfig/)
- [ESLint config files](https://eslint.org/docs/latest/use/configure/configuration-files) · [Prettier install](https://prettier.io/docs/install)
- [Vitest guide](https://vitest.dev/guide/) · [Playwright intro](https://playwright.dev/docs/intro)
- [GitHub Actions quickstart](https://docs.github.com/en/actions/get-started/quickstart) · [Conventional Commits](https://www.conventionalcommits.org/)
- [Chrome DevTools](https://developer.chrome.com/docs/devtools)

## Antigravity CLI
- [Rules (AGENTS.md, .agents/rules)](https://antigravity.google/docs/rules/) · [Skills](https://antigravity.google/docs/skills/)
- [CLI overview](https://antigravity.google/docs/cli/overview/) · [CLI install](https://antigravity.google/docs/cli/install/) · [CLI best practices](https://antigravity.google/docs/cli/best-practices/)
- [Models](https://antigravity.google/docs/models/) · [Plans and quotas](https://antigravity.google/docs/plans/) · [Slash commands](https://antigravity.google/docs/slash-commands/)

## Cloudflare (API, database, cron, hosting)
- [Workers get started](https://developers.cloudflare.com/workers/get-started/guide/) · [Wrangler install](https://developers.cloudflare.com/workers/wrangler/install-and-update/) · [Wrangler commands](https://developers.cloudflare.com/workers/wrangler/commands/)
- [Hono on Workers](https://hono.dev/docs/getting-started/cloudflare-workers)
- [D1 get started](https://developers.cloudflare.com/d1/get-started/) · [D1 migrations](https://developers.cloudflare.com/d1/reference/migrations/) · [D1 Worker API](https://developers.cloudflare.com/d1/worker-api/) · [D1 SQL statements (FTS5 supported)](https://developers.cloudflare.com/d1/sql-api/sql-statements/) · [D1 Time Travel](https://developers.cloudflare.com/d1/reference/time-travel/)
- [Drizzle + D1](https://orm.drizzle.team/docs/sqlite/connect-cloudflare-d1) · [Drizzle migrations](https://orm.drizzle.team/docs/migrations)
- [Secrets](https://developers.cloudflare.com/workers/configuration/secrets/) · [Cron Triggers](https://developers.cloudflare.com/workers/configuration/cron-triggers/) · [Static assets](https://developers.cloudflare.com/workers/static-assets/)
- [Vitest integration](https://developers.cloudflare.com/workers/testing/vitest-integration/) · [Real-time logs](https://developers.cloudflare.com/workers/observability/logs/real-time-logs/) · [Limits](https://developers.cloudflare.com/workers/platform/limits/)
- [Web Crypto in Workers](https://developers.cloudflare.com/workers/runtime-apis/web-crypto/) · [Request (cf object)](https://developers.cloudflare.com/workers/runtime-apis/request/)

## Shared libraries
- [zod](https://zod.dev/) · [jose (JWT)](https://github.com/panva/jose) · [uuidv7](https://github.com/LiosK/uuidv7) · [fflate (zip)](https://github.com/101arrowz/fflate)
- [chrono (date parsing)](https://github.com/wanasit/chrono) · [rrule](https://github.com/jkbrzt/rrule) · [otpauth (TOTP)](https://github.com/hectorm/otpauth)
- Standards: [JWT RFC 7519](https://datatracker.ietf.org/doc/html/rfc7519) · [TOTP RFC 6238](https://datatracker.ietf.org/doc/html/rfc6238) · [iCalendar/RRULE RFC 5545](https://datatracker.ietf.org/doc/html/rfc5545) · [SQLite FTS5](https://www.sqlite.org/fts5.html)

## Android app (Expo / React Native)
- [Create a project](https://docs.expo.dev/get-started/create-a-project/) · [Monorepos (pnpm notes)](https://docs.expo.dev/guides/monorepos/) · [Development builds](https://docs.expo.dev/develop/development-builds/introduction/)
- [Local app development](https://docs.expo.dev/guides/local-app-development/) · [Local production build](https://docs.expo.dev/guides/local-app-production/) · [Signed APK](https://reactnative.dev/docs/signed-apk-android)
- [Expo Router](https://docs.expo.dev/router/introduction/) · [expo-sqlite](https://docs.expo.dev/versions/latest/sdk/sqlite/) · [Drizzle + Expo SQLite](https://orm.drizzle.team/docs/sqlite/connect-expo-sqlite)
- [expo-secure-store](https://docs.expo.dev/versions/latest/sdk/securestore/) · [expo-local-authentication](https://docs.expo.dev/versions/latest/sdk/local-authentication/) · [expo-notifications](https://docs.expo.dev/versions/latest/sdk/notifications/) · [expo-camera](https://docs.expo.dev/versions/latest/sdk/camera/)
- [Expo Modules overview](https://docs.expo.dev/modules/overview/) · [Expo Modules get started](https://docs.expo.dev/modules/get-started/) · [Config plugins](https://docs.expo.dev/config-plugins/introduction/) · [FCM credentials](https://docs.expo.dev/push-notifications/fcm-credentials/)
- [expo-speech-recognition](https://github.com/jamsch/expo-speech-recognition) · [React Native performance](https://reactnative.dev/docs/performance)

## Android native (Kotlin)
- [Kotlin tour](https://kotlinlang.org/docs/kotlin-tour-welcome.html) · [Kotlin basic syntax](https://kotlinlang.org/docs/basic-syntax.html)
- [Android Studio install](https://developer.android.com/studio/install) · [adb](https://developer.android.com/tools/adb) · [Logcat](https://developer.android.com/tools/logcat)
- [Quick Settings tiles](https://developer.android.com/develop/ui/views/quicksettings-tiles) · [Receive shared data](https://developer.android.com/develop/ui/compose/sharing/receive) · [Glance widgets](https://developer.android.com/develop/ui/compose/glance)
- [Alarms](https://developer.android.com/develop/background-work/services/alarms) · [Exact alarms (Android 14)](https://developer.android.com/about/versions/14/changes/schedule-exact-alarms) · [Broadcasts](https://developer.android.com/develop/background-work/background-tasks/broadcasts)
- [Notifications](https://developer.android.com/develop/ui/views/notifications) · [Notification channels](https://developer.android.com/develop/ui/compose/notifications/channels) · [Create a notification](https://developer.android.com/develop/ui/compose/notifications/create-notification)
- [Doze and App Standby](https://developer.android.com/training/monitoring-device-state/doze-standby) · [Android Keystore](https://developer.android.com/privacy-and-security/keystore)
- Battery killers by brand: [Oppo](https://dontkillmyapp.com/oppo) · [Vivo](https://dontkillmyapp.com/vivo) · [Realme](https://dontkillmyapp.com/realme) · [OnePlus](https://dontkillmyapp.com/oneplus)

## Push notifications
- [FCM HTTP v1 send + authorize](https://firebase.google.com/docs/cloud-messaging/send/v1-api)
- [Web Push overview](https://web.dev/articles/push-notifications-overview) · [Push API (MDN)](https://developer.mozilla.org/en-US/docs/Web/API/Push_API) · [@pushforge/builder](https://www.npmjs.com/package/@pushforge/builder) (npm blocks bots; opens fine in a browser)

## Web app
- [Vite guide](https://vite.dev/guide/) · [vite-plugin-pwa](https://vite-pwa-org.netlify.app/guide/) · [React Router](https://reactrouter.com/home) · [TanStack Query](https://tanstack.com/query/latest/docs/framework/react/overview)
- [Web Crypto API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API) · [Web Speech API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Speech_API)

## Design and UI kits (verified 2026-10-16)
- Cue design: [ui-ux-design.md](../07-design/ui-ux-design.md) · [mockups.html](../07-design/mockups.html)
- [Tailwind + Vite](https://tailwindcss.com/docs/installation/using-vite) · [shadcn/ui Vite install](https://ui.shadcn.com/docs/installation/vite) · [shadcn changelog](https://ui.shadcn.com/docs/changelog) · [Lucide](https://lucide.dev/guide/)
- [NativeWind install](https://www.nativewind.dev/docs/getting-started/installation) · [React Native Reusables](https://github.com/founded-labs/react-native-reusables) · [@gorhom/bottom-sheet](https://github.com/gorhom/react-native-bottom-sheet) · [Expo SDK 57 pins](https://github.com/expo/expo/blob/sdk-57/packages/expo/bundledNativeModules.json)
- Guidelines: [M3 navigation bar](https://m3.material.io/components/navigation-bar/guidelines) · [M3 bottom sheets](https://m3.material.io/components/bottom-sheets/guidelines) · [NN/g empty states](https://www.nngroup.com/videos/empty-states-in-application-design-guidelines/) · [WCAG 2.2](https://www.w3.org/TR/WCAG22/)

## Browser extension
- [WXT installation](https://wxt.dev/guide/installation.html) · [Chrome extensions get started](https://developer.chrome.com/docs/extensions/get-started)
- APIs: [contextMenus](https://developer.chrome.com/docs/extensions/reference/api/contextMenus) · [sidePanel](https://developer.chrome.com/docs/extensions/reference/api/sidePanel) · [alarms](https://developer.chrome.com/docs/extensions/reference/api/alarms) · [notifications](https://developer.chrome.com/docs/extensions/reference/api/notifications) · [action](https://developer.chrome.com/docs/extensions/reference/api/action) · [storage](https://developer.chrome.com/docs/extensions/reference/api/storage) · [commands](https://developer.chrome.com/docs/extensions/reference/api/commands)
- [Text fragments (MDN)](https://developer.mozilla.org/en-US/docs/Web/URI/Reference/Fragment/Text_fragments)
