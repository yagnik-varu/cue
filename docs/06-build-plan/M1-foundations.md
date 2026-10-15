# M1 · Foundations: data model, signup, sync

> Status: Ready · Last updated: 2026-10-15 · Related: [README](README.md), [links](links.md), [data-model](../05-system-design/data-model.md), [sync](../05-system-design/sync.md), [auth-flows](../05-system-design/auth-flows.md)
>
> Features: SEC-06 (recovery codes at signup), SEC-08 (baseline), CAP-09 (offline outbox), DATA-01 (conflict-safe sync), PLAT-01
> **Milestone done when:** a note created **offline** on Android syncs to the server after reconnecting, and editing the
> same note on two devices keeps **both** versions. Learning notes for M1 are written.

---

### M1-T01 · Shared domain model (zod schemas + types)
- **Depends on:** M0-T05
- **Read first:** [data-model.md §1–2](../05-system-design/data-model.md) (tables `users`, `devices`, `notes`, `reminders`, `groups`, `tags`, `note_tags`, `suggestions`, `notification_schedules`)
- **Goal:** one zod schema and TypeScript type per synced entity in `packages/shared/model`.
- **Steps:**
  1. Add the `uuidv7` library and an `newId()` helper in `model/ids.ts`.
  2. `model/time.ts`: `now(): number` (UTC epoch ms). All times are numbers (D-28).
  3. For each entity create `model/<entity>.ts` with a zod schema matching the data-model columns exactly (snake_case field names, same as the DB). Export `type Note = z.infer<typeof NoteSchema>` and so on.
  4. `model/sync-meta.ts`: a common `SyncMeta = { version, updated_at, deleted }` merged into every synced schema.
  5. Constants: `DEFAULT_TIMEZONE = "Asia/Kolkata"`, `USERNAME_REGEX = /^[a-z0-9_]{3,20}$/`.
  6. Tests: valid and invalid examples for Note, Reminder and Device.
- **Files:** `packages/shared/src/model/*.ts`, `packages/shared/src/model/*.test.ts`
- **Commands:** `pnpm --filter @cue/shared add uuidv7` · `pnpm --filter @cue/shared test`
- **Done when:** tests pass, and every column in data-model.md §2 for these tables exists in a schema.
- **Docs:** [zod](https://zod.dev/) · [uuidv7](https://github.com/LiosK/uuidv7)
- **Learning note:** `M1-T01-domain-model.md` (concepts: schema-first design, runtime validation vs TypeScript types, UUIDv7 and why client-generated IDs make offline creation possible)

### M1-T02 · D1 schema and migrations (Drizzle)
- **Depends on:** M1-T01, M0-T06
- **Read first:** [data-model.md](../05-system-design/data-model.md) (all of it) · decisions D-40, D-44
- **Goal:** all MVP tables exist in local and prod D1, created through versioned migrations.
- **Steps:**
  1. Add `drizzle-orm` (and `drizzle-kit` as a dev dependency) to `apps/api`.
  2. `apps/api/src/db/schema.ts`: define every table from data-model.md §2, **except `telegram_links` and `link_codes`** (Future, D-44). Include `change_log`, `domain_events`, `rate_limits`, `audit_events`, `login_requests`, `recovery_codes`, `totp`, `vaults`, `attachments`, `attachment_chunks`, `kv_cache`, and the reminder columns `server_dispatched_at`, `repeat_count`, `last_fired_at`, `escalated_at`.
  3. Add the indexes from data-model.md §3.
  4. `drizzle.config.ts` → migrations output to `apps/api/migrations/`. Generate the first migration.
  5. Apply locally, then to prod.
- **Commands:** `pnpm --filter @cue/api exec drizzle-kit generate` · `pnpm --filter @cue/api exec wrangler d1 migrations apply cue-dev --local` · `pnpm --filter @cue/api exec wrangler d1 migrations apply cue-prod --remote --env production`
- **Done when:** `wrangler d1 execute cue-dev --local --command "SELECT name FROM sqlite_master WHERE type='table'"` lists all the tables.
- **Docs:** [Drizzle + D1](https://orm.drizzle.team/docs/sqlite/connect-cloudflare-d1) · [Drizzle migrations](https://orm.drizzle.team/docs/migrations) · [D1 migrations](https://developers.cloudflare.com/d1/reference/migrations/) · [D1 Worker API](https://developers.cloudflare.com/d1/worker-api/)
- **Learning note:** `M1-T02-schema-migrations.md` (concepts: migrations as version control for the database, indexes and why `(user_id, version)` matters, soft deletes)
- **Watch out:** never edit an applied migration. Always generate a new one.

### M1-T03 · API foundations (structure, errors, validation, logging)
- **Depends on:** M1-T02
- **Read first:** [api.md](../05-system-design/api.md) (header + error shape)
- **Goal:** a clean API structure that every later endpoint follows.
- **Steps:**
  1. Folders: `src/routes/` (one file per area: auth, sync, notes, reminders, devices, me), `src/services/` (business logic), `src/db/` (schema + queries), `src/lib/` (errors, validation, logging).
  2. `lib/errors.ts`: an `AppError(code, message, status)` class plus a Hono `onError` handler that returns `{ error: { code, message } }`.
  3. `lib/validate.ts`: a small middleware using the zod schemas from `@cue/shared`.
  4. A request-id middleware, plus compact logging (method, path, status, ms).
  5. Typed `Env` bindings (`DB`, secrets). Secrets come from `.dev.vars` locally (never committed).
- **Done when:** an unknown route returns a 404 in the error shape, an invalid body returns a 400 with a zod message, and tests cover both.
- **Docs:** [Hono on Workers](https://hono.dev/docs/getting-started/cloudflare-workers) · [Workers secrets](https://developers.cloudflare.com/workers/configuration/secrets/)
- **Learning note:** `M1-T03-api-structure.md` (concepts: layered architecture routes → services → db, consistent error contracts, middleware like Express)

### M1-T04 · Rate limiting and audit helpers
- **Depends on:** M1-T03
- **Read first:** [auth-flows.md §9](../05-system-design/auth-flows.md#9-rate-limit-table-sec-08)
- **Goal:** reusable `rateLimit(key, limit, windowSec)` (D1 fixed window) and `audit(userId, type, deviceId, ipHint)`.
- **Steps:**
  1. `services/rate-limit.ts`: an upsert into `rate_limits` keyed by `key + window_start`. Throws `AppError("rate_limited", …, 429)` when over the limit.
  2. `services/audit.ts`: inserts an `audit_events` row. `ip_hint` comes from `request.cf` city and country.
  3. Tests: the 6th call inside the window is rejected, and a new window resets.
- **Done when:** tests pass.
- **Docs:** [Workers request (cf object)](https://developers.cloudflare.com/workers/runtime-apis/request/)
- **Learning note:** `M1-T04-rate-limiting.md` (concepts: fixed vs sliding windows, why limits stop brute force and push-bombing)
- **Watch out:** each check costs a D1 write (100k writes/day free, K-13). Only rate-limit auth endpoints.

### M1-T05 · Crypto helpers (shared)
- **Depends on:** M1-T01
- **Read first:** [auth-flows.md §0](../05-system-design/auth-flows.md#0-building-blocks)
- **Goal:** small, tested WebCrypto wrappers that work in Workers, browsers and React Native.
- **Steps:**
  1. `crypto/keys.ts`: `generateDeviceKeyPair()` (ECDSA P-256), `exportPublicJwk()`, `sign(privateKey, data)`, `verify(publicJwk, data, sig)`.
  2. `crypto/hash.ts`: `sha256Hex(text)`, `randomToken(bytes = 32)` (base64url).
  3. `crypto/recovery.ts`: `generateRecoveryCodes(10)`, formatted like `ABCD-EFGH-IJKL`, high entropy.
  4. Tests in Node/Vitest using the global `crypto.subtle`.
- **Done when:** tests pass, and sign → verify round-trips.
- **Docs:** [Web Crypto API (MDN)](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API) · [Web Crypto in Workers](https://developers.cloudflare.com/workers/runtime-apis/web-crypto/)
- **Learning note:** `M1-T05-crypto-basics.md` (concepts: public/private keys, signatures, hashing vs encryption, why recovery codes don't need slow hashing)
- **Watch out:** React Native may need a WebCrypto polyfill. Check in M1-T11 and record the choice.

### M1-T06 · Signup + tokens + auth middleware (SEC-06 part 1, D-33)
- **Depends on:** M1-T04, M1-T05
- **Read first:** [auth-flows.md §0–§1](../05-system-design/auth-flows.md#1-signup-android-only-d-33)
- **Goal:** `POST /api/auth/signup` creates a user, a trusted Android device, a session, and 10 recovery codes (returned once).
- **Steps:**
  1. Add `jose` for JWT (HS256, 15 min) in `services/tokens.ts`: `issueAccess(userId, deviceId)`, `issueRefresh()` (random token whose SHA-256 is stored in `sessions`).
  2. `routes/auth.ts`:
     - `GET /auth/username-available` (rate-limited)
     - `POST /auth/signup` (rate-limited; body validated; username regex; timezone default `Asia/Kolkata`; recovery code hashes stored)
  3. `POST /auth/refresh`: the body has `refresh_token`, `device_id`, `ts`, and a signature over `refresh_token|ts` by the device key. Rotate tokens.
  4. `middleware/auth.ts`: verify the JWT, then load the device and reject it if `revoked_at` is set (instant revocation).
  5. Config flag `SIGNUP_OPEN` (env var). When `false`, signup returns 403.
  6. Tests: happy path, duplicate username, bad signature on refresh, revoked device → 401.
- **Done when:** tests pass, and `curl` signup locally returns tokens + 10 codes.
- **Docs:** [jose](https://github.com/panva/jose) · [JWT RFC 7519](https://datatracker.ietf.org/doc/html/rfc7519) · [Workers secrets](https://developers.cloudflare.com/workers/configuration/secrets/)
- **Learning note:** `M1-SEC-06-signup-and-tokens.md` (concepts: access vs refresh tokens, rotation, device-bound tokens, why no passwords)
- **Watch out:** keep CPU low. HS256 and ECDSA verify are cheap, but **no** bcrypt/PBKDF2 on the server (D-43).

### M1-T07 · `applyChange`: the server write rule (DATA-01) ⭐ big task, wait for "go"
- **Depends on:** M1-T02, M1-T03
- **Read first:** [sync.md §2–§3](../05-system-design/sync.md#3-the-server-write-rule-used-by-syncpush-and-by-the-webextension-rest-endpoints)
- **Goal:** one function `applyChange(env, userId, deviceId, change)` that implements sync.md §3 exactly.
- **Steps:**
  1. `services/apply-change.ts`: check `op_id` idempotency in `change_log`; create; fast-path update when `base_version >= row.version`; otherwise a content conflict creates a conflict copy (`conflict_of`), while a metadata-only change merges the changed fields.
  2. Allocate the version by inserting into `change_log` (autoincrement) and using its id. Set `content_version` when content fields change.
  3. `locked = 1` forces `ai_visible = 0`.
  4. Emit domain events via a tiny `EventBus` (`services/events.ts`) and insert them into `domain_events` (PLAT-04).
  5. Tests = sync.md §9 cases 2, 3 and 4, plus the create case.
- **Done when:** all tests pass.
- **Docs:** [D1 Worker API](https://developers.cloudflare.com/d1/worker-api/) · [sync.md](../05-system-design/sync.md)
- **Learning note:** `M1-DATA-01-apply-change.md` (concepts: optimistic concurrency, idempotency keys, conflict copies vs last-writer-wins, event emission)
- **Watch out:** D1 allows 50 queries per invocation. Batch with `db.batch()` where possible.

### M1-T08 · Sync endpoints `/api/sync/push` and `/api/sync/pull`
- **Depends on:** M1-T06, M1-T07
- **Read first:** [sync.md §4](../05-system-design/sync.md#4-api-android-sync)
- **Goal:** the Android sync API works exactly as specified.
- **Steps:**
  1. `POST /sync/push`: auth required; max 100 changes; call `applyChange` for each; return results.
  2. `GET /sync/pull?since&limit`: join `change_log` → current rows; return `cursor`, `has_more`, and `resync_required` (when `since` is older than the oldest retained version).
  3. Tests: push then pull returns the rows; a pull with an old cursor sets `resync_required`.
- **Done when:** tests pass.
- **Learning note:** `M1-T08-sync-api.md` (concepts: cursor pagination, pull vs push sync, tombstones)

### M1-T09 · Shared API client
- **Depends on:** M1-T08
- **Goal:** `packages/shared/api-client`, a typed `createApiClient({ baseUrl, getAccessToken, refresh })` used by all apps.
- **Steps:**
  1. A typed function for each endpoint built so far, validating responses with zod.
  2. On a 401: call `refresh()` once, then retry. Turn the error shape into a thrown `ApiError`.
- **Done when:** unit tests with a mocked `fetch` pass.
- **Learning note:** `M1-T09-api-client.md` (concepts: typed contracts end to end, retry on 401, why the client is shared)

### M1-T10 · Mobile local database (expo-sqlite + Drizzle) + outbox
- **Depends on:** M0-T08, M1-T01
- **Read first:** [data-model.md §4](../05-system-design/data-model.md#4-local-only-tables-android-only) · [sync.md §5](../05-system-design/sync.md#5-android-client-loop)
- **Goal:** the phone has a local copy of the synced tables, plus `outbox` and `sync_state`.
- **Steps:**
  1. Add `expo-sqlite` and `drizzle-orm`. Create `apps/mobile/src/db/schema.ts`, mirroring the synced tables (no server-only auth tables).
  2. Add `outbox(seq, op_id, table, row_id, op, base_version, changed_fields, payload, created_at, tries)` and `sync_state`.
  3. Run migrations at app start (Drizzle `useMigrations`).
  4. `src/db/write.ts`: `localWrite(table, row, changedFields)` writes the row **and** an outbox item in **one transaction**.
- **Done when:** creating a note offline puts the row and one outbox item in SQLite (verify with a debug screen or log).
- **Docs:** [expo-sqlite](https://docs.expo.dev/versions/latest/sdk/sqlite/) · [Drizzle + Expo SQLite](https://orm.drizzle.team/docs/sqlite/connect-expo-sqlite)
- **Learning note:** `M1-CAP-09-local-db-outbox.md` (concepts: local-first, transactions, the outbox pattern)

### M1-T11 · Mobile device key, secure storage, signup onboarding (SEC-06)
- **Depends on:** M1-T05, M1-T06, M1-T09
- **Read first:** [auth-flows.md §1](../05-system-design/auth-flows.md#1-signup-android-only-d-33) · [mobile-android.md §2, §6](../05-system-design/mobile-android.md)
- **Goal:** on first launch, 3 screens (D-52): pick a username → sign up → recovery codes (Copy/Save/Share + "I saved these codes" + re-type one) → "You're ready" → Inbox. No permission prompts here.
- **Design:** [ui-ux-design §6.13](../07-design/ui-ux-design.md#613-onboarding), §5.26 RecoveryCodesGrid.
- **Steps:**
  1. Generate the device key (WebCrypto or a polyfill; record the choice in the learning note). Store the private key and tokens in `expo-secure-store`.
  2. Screens: `/onboarding/username` (live availability), `/onboarding/recovery` (codes shown, "I saved them" + re-type check).
  3. The API base URL comes from config (dev: your PC's LAN IP `:8787`; prod: the `workers.dev` URL).
- **Done when:** fresh install → onboarding → home. The codes can't be seen again later.
- **Docs:** [expo-secure-store](https://docs.expo.dev/versions/latest/sdk/securestore/) · [Expo Router](https://docs.expo.dev/router/introduction/)
- **Learning note:** `M1-SEC-06-onboarding.md` (concepts: secure storage / Keystore, first-device trust)

### M1-T12 · Mobile sync engine
- **Depends on:** M1-T10, M1-T11
- **Read first:** [sync.md §5](../05-system-design/sync.md#5-android-client-loop)
- **Goal:** `sync()` pushes the outbox and pulls changes, triggered on app start, foreground, after writes (debounced 2 s), and on network regained.
- **Steps:**
  1. `src/sync/engine.ts`: push in batches of 100 → delete sent outbox items and set the row version → pull until `has_more = false` → upsert locally (skip rows with pending outbox items) → save `last_pulled_version`.
  2. Handle `resync_required`: wipe synced tables (keep the outbox) and pull from 0.
  3. Triggers via `AppState` and NetInfo. Background periodic sync comes later (WorkManager/expo-background-task), so note it as a TODO with a feature ID.
- **Done when:** airplane mode → create 3 notes → airplane off → the notes appear in D1 (check with `wrangler d1 execute`).
- **Learning note:** `M1-T12-sync-engine.md` (concepts: the sync loop, debouncing, network events, eventual consistency)

### M1-T13 · Minimal notes screen + conflict banner (DATA-01 UI)
- **Depends on:** M1-T12
- **Goal:** a simple list + create/edit screen to exercise sync, plus a banner for conflict copies.
- **Steps:**
  1. A list of notes (live query from SQLite) and a basic editor.
  2. If a note has `conflict_of`, or other notes point to it, show "Edited on two devices: 2 versions" with **Keep this** / **Keep other** (soft-delete the other).
- **Done when:** sync.md §9 test 2 passes manually. Use a second client: call `/api/sync/push` with `curl`, or use a second phone/emulator.
- **Learning note:** `M1-DATA-01-conflict-ui.md`

### M1-T14 · Production deploy + M1 acceptance
- **Depends on:** M1-T01…M1-T13
- **Goal:** prod works end to end.
- **Steps:**
  1. Set prod secrets: `wrangler secret put JWT_SECRET --env production` (and any others so far).
  2. Apply migrations to prod. Deploy. Set `SIGNUP_OPEN=true`, sign up from the phone (release APK pointing at prod), then set `SIGNUP_OPEN=false`.
  3. Run the M1 acceptance: offline create → sync, and the two-device conflict.
- **Done when:** both acceptance checks pass on prod. All M1 learning notes exist, and the M1 concepts are ticked in the concept map.
- **Docs:** [Workers secrets](https://developers.cloudflare.com/workers/configuration/secrets/) · [D1 Time Travel](https://developers.cloudflare.com/d1/reference/time-travel/)
- **Learning note:** `M1-T14-deploy.md` (concepts: environments, secrets, migrations in prod, rollback with Time Travel)
