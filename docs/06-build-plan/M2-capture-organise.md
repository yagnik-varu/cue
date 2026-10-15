# M2 · Capture & organise + phone-approval login + web app

> Status: Ready · Last updated: 2026-10-15 · Related: [README](README.md), [links](links.md), [mobile-android](../05-system-design/mobile-android.md), [auth-flows](../05-system-design/auth-flows.md), [ux-principles](../02-product/ux-principles.md)
>
> Features: CAP-01…CAP-06, CAP-10, ORG-01, ORG-02, ORG-04, ORG-05, ORG-06, DATA-02, SEC-01, SEC-02, PLAT-02, PLAT-03
> **Milestone done when:** on Android, thought → saved note takes **≤ 2 s** from the widget or tile, and the web app logs in
> **without a password** (phone approval), then shows the Inbox. Learning notes are written.

---

### M2-T01 · UI foundation: tokens, Tailwind + shadcn, NativeWind + Reusables, Inter, Lucide (PLAT-02, PLAT-03, D-45, D-50, D-51) ⭐ wait for "go"
- **Depends on:** M0-T07, M0-T08, M0-T09
- **Read first:** [ui-ux-design §3](../07-design/ui-ux-design.md#3-design-tokens) (all), [§11.1–§11.5](../07-design/ui-ux-design.md#11-implementation-guide-for-low-tier-models), §5 (component list), and [mockups.html](../07-design/mockups.html) (open in a browser)
- **Goal:** one shared design system on all three platforms. After this task, every later UI task only composes existing components.
- **Steps:**
  1. Create `packages/ui-tokens`: put the values from §3 into `src/tokens.ts` **exactly as HEX**. Add `scripts/build.ts` to generate `dist/web.css`, `dist/native.css` (HSL triples) and `dist/preset.js` (§11.2). Add a contrast test for the §3.1 pairs.
  2. **Web:**
     - Tailwind v4 + `@tailwindcss/vite`.
     - `npx shadcn@latest init`, choosing the **Radix** base (not Base UI).
     - Import `@cue/ui-tokens/web.css` and map it with `@theme inline`.
     - Add the shadcn components listed in §11.3, plus `@fontsource-variable/inter`, `lucide-react` and `sonner`.
  3. **Extension:** add Tailwind to `wxt.config.ts`, the same tokens CSS and fontsource Inter (no CDN), and the shadcn components copied from web. Set the popup body width to 380 px.
  4. **Mobile:**
     - Install NativeWind **4.2** with `npx expo install` (+ `tailwindcss@3.4`).
     - Use the preset from `@cue/ui-tokens`.
     - Add React Native Reusables components (nativewind registry).
     - Add `lucide-react-native`, `sonner-native`, `@gorhom/bottom-sheet`, `react-native-gesture-handler`, `@shopify/flash-list`, `expo-haptics`, and Inter via `@expo-google-fonts/inter` (weights mapped as in §11.2).
  5. `strings.ts` per app with the copy from §4.2 (D-45).
  6. A **"Design check" screen** per platform (dev only) showing Button, Input, Chip (all variants), Checkbox, NoteRow, TodoRow, Toast and EmptyState in light and dark.
- **Commands:** `pnpm --filter @cue/ui-tokens build` · `pnpm --filter @cue/web dev` · `pnpm --filter @cue/mobile exec expo start --dev-client`
- **Done when:**
  - the Design-check screens match `mockups.html` in light and dark
  - the contrast test passes
  - no hex colours appear anywhere outside `packages/ui-tokens`
- **Docs:** [Tailwind + Vite](https://tailwindcss.com/docs/installation/using-vite) · [shadcn/ui install (Vite)](https://ui.shadcn.com/docs/installation/vite) · [NativeWind install](https://www.nativewind.dev/docs/getting-started/installation) · [React Native Reusables](https://github.com/founded-labs/react-native-reusables) · [Lucide](https://lucide.dev/guide/) · [Expo SDK 57 pins](https://github.com/expo/expo/blob/sdk-57/packages/expo/bundledNativeModules.json)
- **Learning note:** `M2-PLAT-03-design-system.md` (concepts: design tokens, utility-first CSS, shadcn's "copy the component" model, NativeWind vs StyleSheet, why HEX is used for mobile)
- **Watch out:**
  - never `npm i` native libraries on mobile
  - if a Reusables component is missing, build it with NativeWind classes from §5 (same behaviour)

### M2-T02 · Mobile tabs, fast capture screen, Inbox (CAP-01, PLAT-01)
- **Depends on:** M1-T13, M2-T01
- **Read first:** [mobile-android.md §2–§3](../05-system-design/mobile-android.md#2-screens-expo-router)
- **Goal:** bottom tabs **Inbox · Todos · Notes**, and a `/capture` screen that opens with the keyboard up in ≤ 1 s.
- **Steps:**
  1. Expo Router: `(tabs)/inbox`, `(tabs)/todos`, `(tabs)/notes`, plus a modal route `/capture`. Deep link `cue://capture` (and `?voice=1`).
  2. `/capture`: one multiline input with `autoFocus`, and a Save button that calls `localWrite` with `state: "inbox"` and `source_kind` from the query string (`widget`/`tile`/`app`). Don't import sync or search code in this screen.
  3. Inbox: a live list of `state = inbox`, newest first, each showing its source icon and time (CAP-01 AC3).
  4. A floating "+" button opens `/capture`.
- **Done when:** with a **release** build, tap "+" → typing within 1 s. The new note shows in the Inbox instantly, even offline.
- **Docs:** [Expo Router](https://docs.expo.dev/router/introduction/) · [React Native performance](https://reactnative.dev/docs/performance)
- **Learning note:** `M2-CAP-01-fast-capture.md` (concepts: navigation, deep links, lazy imports, measuring startup)

### M2-T03 · Note editor, todos, groups, tags, pin, archive, trash (ORG-01, ORG-02, ORG-05, ORG-06, DATA-02)
- **Depends on:** M2-T02
- **Read first:** [features.md ORG-01…ORG-06](../02-product/features.md)
- **Goal:** full organise features on mobile.
- **Steps:**
  1. `/note/[id]`: edit the body and title, toggle **Todo** (ORG-01), **Done** (ORG-02), **Important**, **Pin**. Pick a **group** and **tags**.
  2. The Todos tab shows Today / Upcoming / Overdue / No date / Completed (reminders come in M3; for now: open todos, then completed).
  3. Notes tab: all, by group, pinned first. Plus **Archive** and **Trash** views. Delete → Trash (soft delete with `deleted_at`). Restore works. A "Delete forever" button sits in Trash.
  4. Each change goes through `localWrite` with the correct `changed_fields` (so merges work, sync.md §3).
- **Done when:** every ORG acceptance criterion in features.md can be shown on the phone, and the changes sync.
- **Learning note:** `M2-ORG-organise.md` (concepts: soft delete, field-level change tracking, list rendering)

### M2-T04 · Swipe triage in Inbox (ORG-04)
- **Depends on:** M2-T03
- **Goal:** swipe right → Todo, swipe left → Archive. Long-press menu → Group / Remind / Delete. An "Inbox clear" empty state.
- **Steps:** use React Native Gesture Handler swipeables (already part of the Expo template) and show an Undo toast after each action.
- **Done when:** processing 5 Inbox items takes about 5 swipes, and Undo restores the item.
- **Learning note:** `M2-ORG-04-triage.md` (concepts: gestures, optimistic UI + undo)

### M2-T05 · Voice capture (CAP-05, D-26)
- **Depends on:** M2-T02
- **Read first:** [mobile-android.md §7](../05-system-design/mobile-android.md#7-voice-cap-05-d-26-d-37)
- **Goal:** hold the mic button on `/capture` → on-device speech-to-text → the transcript fills the input.
- **Steps:**
  1. Add `expo-speech-recognition` (needs a dev build → re-run prebuild). Prefer on-device recognition. Locale setting `en-IN` (default) / `hi-IN`.
  2. Request the microphone permission with a clear explanation. `cue://capture?voice=1` starts listening immediately.
  3. Audio is **not** kept (keeping audio is M6-T08).
- **Done when:** speaking "buy milk tomorrow" puts that text in the note, offline if the phone supports on-device recognition.
- **Docs:** [expo-speech-recognition](https://github.com/jamsch/expo-speech-recognition)
- **Learning note:** `M2-CAP-05-voice.md` (concepts: native permissions, on-device vs cloud speech)

### M2-T06 · Kotlin #1: Quick Settings tile `cue-tile` (CAP-03) ⭐ wait for "go" (first Kotlin)
- **Depends on:** M2-T02
- **Read first:** [tech-stack.md §5](../05-system-design/tech-stack.md#5-learning-path-for-kotlin-owner-goal) · [mobile-android.md §4](../05-system-design/mobile-android.md#4-native-kotlin-modules-learning-path-order)
- **Goal:** a "Quick note" tile in the notification shade that opens `cue://capture?source=tile`.
- **Steps:**
  1. `npx create-expo-module@latest --local` → `apps/mobile/modules/cue-tile`.
  2. Kotlin `CueTileService : TileService`. On `onClick()` → start the app with the deep link (use `startActivityAndCollapse`; check the API-level differences in the docs).
  3. Register the service in the module's `AndroidManifest.xml` with the `BIND_QUICK_SETTINGS_TILE` permission, a label of ≤ 18 characters, and an icon.
  4. **Learning mode:** comment every Kotlin line the first time a concept appears (class, override, Intent, PendingIntent).
- **Commands:** `pnpm --filter @cue/mobile exec expo prebuild -p android --clean` · `pnpm --filter @cue/mobile exec expo run:android --device`
- **Done when:** add the tile via "Edit tiles", tap it → the capture screen opens with the keyboard up.
- **Docs:** [Quick Settings tiles](https://developer.android.com/develop/ui/views/quicksettings-tiles) · [Expo Modules get started](https://docs.expo.dev/modules/get-started/) · [Kotlin tour](https://kotlinlang.org/docs/kotlin-tour-welcome.html)
- **Learning note:** `M2-CAP-03-kotlin-tile.md` (concepts: Kotlin syntax basics, Android Service, the manifest, Intent/PendingIntent, how an Expo module plugs native code in)

### M2-T07 · Kotlin #2: Share target `cue-share` (CAP-04, CAP-10)
- **Depends on:** M2-T06
- **Goal:** "Share → Cue" from any app saves text or a link into the Inbox, with `source_app` / `source_url`.
- **Steps:**
  1. Add an `ACTION_SEND` intent filter (`text/plain`, `image/*`) to the main activity using a config plugin, or handle it in the module.
  2. Kotlin: read `EXTRA_TEXT`, `EXTRA_SUBJECT`, and the referrer package. Expose `getPendingShare()` to JS. If the text is a URL, set `source_url`.
  3. JS: on launch with a share, open a small "Save to Inbox" sheet (one tap; CAP-04 AC2).
- **Done when:** share a link from Chrome → the note saves with the URL and "Chrome" as the source app.
- **Docs:** [Receiving shared data](https://developer.android.com/develop/ui/compose/sharing/receive) · [Config plugins](https://docs.expo.dev/config-plugins/introduction/)
- **Learning note:** `M2-CAP-04-share-target.md` (concepts: intent filters, extras, config plugins)

### M2-T08 · Kotlin #3: Home widget `cue-widget` with Glance (CAP-02)
- **Depends on:** M2-T06
- **Goal:** a widget with **New**, **Voice** and a short **Today** list (tick to complete).
- **Steps:**
  1. Add Jetpack Glance in the module's Gradle file. Create a `GlanceAppWidget` and a receiver, and register them in the manifest with widget metadata XML.
  2. Buttons → deep links `cue://capture?source=widget` and `cue://capture?voice=1&source=widget`.
  3. Today list: JS writes a small JSON snapshot (top 5 todos) to SharedPreferences via the module whenever todos change. The widget reads it. A tick → a Glance `ActionCallback` writes a **pending action** that JS drains into `localWrite` on next launch.
- **Done when:** the widget shows on the home screen, buttons open capture, and ticking a todo marks it done after the app opens.
- **Docs:** [Glance](https://developer.android.com/develop/ui/compose/glance)
- **Learning note:** `M2-CAP-02-glance-widget.md` (concepts: app widgets, Compose-style UI, why widgets read snapshots instead of the DB)
- **Watch out:** if Glance is too heavy for the model, the fallback is `react-native-android-widget` (MIT). Record it as a decision if used.

### M2-T09 · FCM push: device token + Worker sender + "sync" nudge
- **Depends on:** M1-T14, M0-T02
- **Read first:** [mobile-android.md §5](../05-system-design/mobile-android.md#5-push-fcm) · [hosting §2](../05-system-design/hosting-and-distribution.md#2-libraries-confirmed-for-workers)
- **Goal:** the server can send data messages to the phone; the first use is `{"type":"sync"}` after changes from other devices.
- **Steps:**
  1. Firebase console (Spark): add an Android app with your package name → download `google-services.json` (git-ignored). Point `app.json` at it.
  2. Mobile: `expo-notifications` → `getDevicePushTokenAsync()` → `PATCH /api/devices/:id { push_token }`.
  3. Worker `services/fcm.ts`: sign a service-account JWT with WebCrypto (RS256) → exchange it for an OAuth token → **cache it in `kv_cache` for ~55 min** (D-43) → POST to the FCM HTTP v1 send endpoint.
  4. After `applyChange` from a non-Android device, send a collapsed `sync` data message to the user's Android devices.
  5. Mobile: on a data message, run `sync()`.
- **Done when:** editing a note via `curl` makes the phone update within seconds while the app is in the background.
- **Docs:** [FCM send](https://firebase.google.com/docs/cloud-messaging/send/v1-api) · [FCM auth server](https://firebase.google.com/docs/cloud-messaging/send/v1-api#authorize-http-v1-send-requests) · [Expo FCM credentials](https://docs.expo.dev/push-notifications/fcm-credentials/) · [expo-notifications](https://docs.expo.dev/versions/latest/sdk/notifications/)
- **Learning note:** `M2-T09-fcm-push.md` (concepts: push tokens, data vs notification messages, OAuth service accounts, token caching)
- **Watch out:** the service-account key is a **secret**: `wrangler secret put FCM_SERVICE_ACCOUNT`.

### M2-T10 · Login by phone approval (SEC-01) ⭐ wait for "go"
- **Depends on:** M2-T09, M1-T06
- **Read first:** [auth-flows.md §2](../05-system-design/auth-flows.md#2-sign-in-on-web-or-extension-approve-on-phone-with-number-matching-sec-01)
- **Goal:** a new browser signs in with: username → pick the matching number on the phone + biometric → signed in.
- **Steps:**
  1. API:
     - `POST /auth/login/start` (rate-limited; fake response for unknown usernames)
     - `POST /auth/login/approve` (trusted device signature; wrong number → deny + alert)
     - `GET /auth/login/status`
     - `POST /auth/login/complete` (new device signature over the nonce)
  2. FCM `login_request` data message → a high-priority notification → `/approve/[requestId]` screen showing device, browser, `<city>`, time and 3 numbers. Require `expo-local-authentication` biometric. Use `FLAG_SECURE` on this screen.
  3. A new-device alert push to all trusted devices + an audit event.
  4. Tests: approve, wrong number, expired, unknown username (same response shape), rate limit.
- **Done when:** the web login (M2-T13) works end to end with the phone. *(Until M2-T13, test with `curl` plus a script that signs the nonce.)*
- **Docs:** [expo-local-authentication](https://docs.expo.dev/versions/latest/sdk/local-authentication/) · [auth-flows](../05-system-design/auth-flows.md)
- **Learning note:** `M2-SEC-01-phone-approval.md` (concepts: number matching vs push-bombing, challenge-response, user enumeration)

### M2-T11 · QR login fallback (SEC-02)
- **Depends on:** M2-T10
- **Read first:** [auth-flows.md §3](../05-system-design/auth-flows.md#3-qr-fallback-sec-02)
- **Goal:** the new device shows a QR code. The phone scans it, sees the same approval screen, and approves.
- **Steps:**
  1. `POST /auth/qr/start` and `GET /auth/qr/details`. `POST /auth/qr/approve` checks the pubkey fingerprint. Auto-expire after 60 s.
  2. Mobile `/scan` with `expo-camera` (barcode scanning). Location-mismatch warning text.
- **Done when:** QR login works, and an expired QR is rejected.
- **Docs:** [expo-camera](https://docs.expo.dev/versions/latest/sdk/camera/)
- **Learning note:** `M2-SEC-02-qr-login.md` (concepts: QRLJacking and its mitigations, key fingerprints)

### M2-T12 · Notes REST API for web and extension (D-39)
- **Depends on:** M1-T07
- **Read first:** [api.md "Notes"](../05-system-design/api.md#notes-web-extension-all-writes-use-applychange) · [sync.md §6](../05-system-design/sync.md#6-web-and-extension-online-d-39)
- **Goal:** the online endpoints, all writing through `applyChange`.
- **Steps:**
  1. Endpoints:
     - `GET /notes` (filters, cursor)
     - `GET /notes/:id`
     - `POST /notes`
     - `PATCH /notes/:id` (`base_version`, `changed_fields`)
     - `DELETE /notes/:id`
     - `POST /notes/:id/restore`
     - `POST /notes/:id/resolve-conflict`
     - CRUD for `/groups` and `/tags`
  2. Each write from web or extension triggers the FCM sync nudge (M2-T09).
  3. Tests for each endpoint, including a PATCH conflict that produces a conflict copy.
- **Done when:** tests pass, and `api-client` has functions for all these endpoints.
- **Learning note:** `M2-T12-rest-notes.md` (concepts: REST resource design, PATCH semantics, one write path for all clients)

### M2-T13 · Web app: login, layout, Inbox, editor, quick-add (CAP-06, ORG-xx on web)
- **Depends on:** M2-T10, M2-T12, M2-T01
- **Read first:** [ux-principles.md "Web app"](../02-product/ux-principles.md#web-app)
- **Goal:** a usable desktop web app (online-only).
- **Steps:**
  1. React Router routes: `/login`, `/inbox`, `/todos`, `/notes`, `/groups/:id`, `/archive`, `/trash`, `/note/:id`, `/settings`.
  2. Login: username → show "Select **42** on your phone" → poll the status → done. A "Use QR instead" link shows the QR (M2-T11). The device key is non-extractable WebCrypto, kept in IndexedDB.
  3. TanStack Query for data (refetch on focus and every 60 s). Quick add opens with `Q` / `Alt+N` (CAP-06, keymap in [ui-ux-design §7.8](../07-design/ui-ux-design.md#78-keyboard-map)).
  4. Keyboard shortcuts from ux-principles (`/`, `T`, `E`, `I`, `R`, `?`). Triage actions as buttons + keys.
  5. Offline: show "You're offline. Not saved." with Retry (D-39).
  6. Show the source (CAP-10): an icon + link for notes with `source_url`.
- **Done when:** sign in from Chrome with phone approval → see the Inbox → create a note → it appears on the phone within seconds.
- **Docs:** [React Router](https://reactrouter.com/home) · [TanStack Query](https://tanstack.com/query/latest/docs/framework/react/overview) · [Web Crypto API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API)
- **Learning note:** `M2-CAP-06-web-app.md` (concepts: SPA routing, server-state caching, keyboard UX, non-extractable keys)

### M2-T14 · M2 acceptance + deploy
- **Goal:** prove M2 works on the real phone and in Chrome, then deploy.
- **Steps:**
  1. Deploy the Worker (API + web).
  2. Build and sideload the release APK.
  3. Check every M2 feature's acceptance criteria in [features.md](../02-product/features.md).
- **Done when:**
  - the capture timing is ≤ 2 s from the widget and the tile
  - the web login works with no password
  - all M2 learning notes are written
