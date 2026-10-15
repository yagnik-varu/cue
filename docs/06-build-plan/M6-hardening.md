# M6 · Hardening: security, vault, export, backups

> Status: Ready · Last updated: 2026-10-15 · Related: [README](README.md), [links](links.md), [auth-flows](../05-system-design/auth-flows.md), [hosting §4](../05-system-design/hosting-and-distribution.md#4-backups-data-04)
>
> Features: SEC-03, SEC-04, SEC-05, SEC-06 (rest), SEC-07, DATA-03, DATA-04, CAP-05 AC1 (optional audio, D-37/D-40)
> **Milestone done when:** the server stores **only ciphertext** for locked notes (verified by a test), the export opens
> cleanly in a Markdown viewer, and you can recover the account on a new phone. Then the **MVP acceptance** (M6-T09) starts.

---

### M6-T01 · App lock (SEC-04)
- **Depends on:** M2-T02
- **Read first:** [mobile-android.md §6](../05-system-design/mobile-android.md#6-security-on-device)
- **Goal:** an optional PIN or biometric lock when the app opens, and when it resumes after N minutes.
- **Steps:**
  1. Settings toggle + timeout (1 / 5 / 15 min).
  2. Use `expo-local-authentication`, with a PIN fallback (the PIN hash goes in secure store).
  3. `/capture` (opened from the tile or widget) can **save** while locked but shows nothing else (SEC-04 AC2).
- **Done when:** lock on, close the app, wait → reopening asks for a fingerprint. The tile can still save a note.
- **Docs:** [expo-local-authentication](https://docs.expo.dev/versions/latest/sdk/local-authentication/)
- **Learning note:** `M6-SEC-04-app-lock.md`

### M6-T02 · Device list, revoke, promote, alerts (SEC-05)
- **Depends on:** M2-T10
- **Read first:** [auth-flows.md §7](../05-system-design/auth-flows.md#7-device-management-sec-05)
- **Goal:** see all your devices, revoke any instantly, and promote a new Android phone to trusted.
- **Steps:**
  1. API:
     - `GET /devices`
     - `POST /devices/:id/revoke` (biometric-confirmed on a trusted device)
     - `POST /devices/:id/promote`
     - `GET /me/audit`
  2. Mobile and web Settings → Devices: name, kind, last seen, city, schedule, Revoke button. Mobile also shows a security log.
  3. Every new sign-in, recovery or revocation sends a push to all trusted devices.
- **Done when:** after you revoke the web browser, its next request gets a 401 and it shows the login screen.
- **Learning note:** `M6-SEC-05-devices.md` (concepts: session revocation, audit logs)

### M6-T03 · TOTP backup login (SEC-03)
- **Depends on:** M6-T02
- **Read first:** [auth-flows.md §4](../05-system-design/auth-flows.md#4-totp-backup-factor-sec-03)
- **Goal:** enrol Google or Microsoft Authenticator. If the phone is unavailable, sign in on the web with username + 6-digit code.
- **Steps:**
  1. Use `otpauth` (bare build with WebCrypto HMAC). The secret is encrypted with a Worker secret (AES-GCM).
  2. Enrol from the trusted phone (QR of the `otpauth://` URI, then confirm a code). `POST /auth/totp/login` with strict limits (5 fails → 15 min lock; 20 a day → 24 h), and `last_used_step` blocks reuse.
  3. A device that logs in by TOTP is **untrusted**, and every trusted device is alerted.
- **Done when:** TOTP login works, a wrong code 5 times locks the account for 15 min, and reusing the same code fails.
- **Docs:** [otpauth](https://github.com/hectorm/otpauth) · [RFC 6238](https://datatracker.ietf.org/doc/html/rfc6238)
- **Learning note:** `M6-SEC-03-totp.md` (concepts: HMAC, time steps, why TOTP alone needs strict rate limits)

### M6-T04 · Recovery on a new phone + regenerate codes (SEC-06)
- **Depends on:** M6-T03
- **Read first:** [auth-flows.md §5](../05-system-design/auth-flows.md#5-recovery-sec-06)
- **Goal:** a lost phone is not a lost account.
- **Steps:**
  1. New install → "Recover account" → username + one recovery code (+ TOTP if enabled) → this device becomes trusted, and the code is burned.
  2. Prompt to regenerate the codes (`POST /auth/recovery-codes/regenerate`), then revoke the old phone from the device list.
  3. A nag banner until there are ≥ 2 recovery paths.
- **Done when:** you uninstall the app, reinstall it, recover with a code, and all notes come back after sync.
- **Learning note:** `M6-SEC-06-recovery.md` (concepts: account recovery design, one-time codes)

### M6-T05 · Locked notes vault, E2EE (SEC-07) ⭐ wait for "go"
- **Depends on:** M6-T01
- **Read first:** [auth-flows.md §6](../05-system-design/auth-flows.md#6-locked-notes-vault-sec-07)
- **Goal:** any note can be locked. Its content is encrypted on the device, and the server only ever sees ciphertext.
- **Steps:**
  1. `packages/shared/crypto/vault.ts` (with tests):
     - `createVaultKey()`
     - `encryptNote(vk, {title, body})`
     - `decryptNote(vk, envelope)`
     - `wrapKey(vk, kek)` and `unwrapKey(...)`
     - `kekFromPin(pin, salt)` (PBKDF2-SHA256, high iterations, **runs on the client**)
     - `kekFromRecoveryKey(key)` (HKDF)
  2. First lock: create the VK, show the **vault recovery key** once, set the vault PIN, store the wrapped keys via the API. On Android, keep the VK in secure store with biometric required.
  3. Lock a note → `body = ""`, `ciphertext = envelope`, `locked = 1`, `ai_visible = 0`, all through `applyChange`.
  4. Reminder titles for locked notes show "🔒 Locked reminder". Auto-lock after 5 min or when the app goes to the background.
  5. Web: unlock with the vault PIN.
  6. **Test:** after locking, query D1 directly and assert there's no plaintext in `body`, and that `ciphertext` decrypts only with the key.
- **Done when:** the test passes, and a locked note opens on the phone (fingerprint) and on the web (PIN).
- **Docs:** [Web Crypto API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API) · [Android Keystore](https://developer.android.com/privacy-and-security/keystore) · [expo-secure-store](https://docs.expo.dev/versions/latest/sdk/securestore/)
- **Learning note:** `M6-SEC-07-vault.md` (concepts: AES-GCM, IVs, key wrapping, KDFs, E2EE trade-offs)
- **Watch out:** never log decrypted content, and never send the VK, PIN or recovery key to the server.

### M6-T06 · Full export, built on the client (DATA-03, D-48)
- **Depends on:** M2-T13, M6-T05
- **Goal:** one click in the web app downloads `cue-export-YYYY-MM-DD.zip`, containing Markdown files and `cue.json`.
- **Steps:**
  1. The web app fetches all notes, groups, tags and reminders through the API (paginated).
  2. Build the zip **in the browser** with `fflate` (D-48: zipping in the Worker would exceed 10 ms CPU). Use one `.md` per note with YAML front matter (id, group, tags, todo, reminder, source), plus `cue.json`.
  3. Locked notes: decrypted only if the vault is unlocked. Otherwise skipped, with a notice file.
- **Done when:** the zip opens, and the Markdown renders in VS Code or Obsidian.
- **Docs:** [fflate](https://github.com/101arrowz/fflate)
- **Learning note:** `M6-DATA-03-export.md` (concepts: data portability, client-side processing to save server CPU)

### M6-T07 · Backups (DATA-04)
- **Depends on:** M6-T06
- **Read first:** [hosting §4](../05-system-design/hosting-and-distribution.md#4-backups-data-04)
- **Goal:** know how to restore, and get reminded to download a backup monthly.
- **Steps:**
  1. Write `docs/runbooks/restore.md`: how to use D1 Time Travel (7 days) with the exact `wrangler d1 time-travel` commands from the docs.
  2. Create a recurring reminder note "Cue: download your monthly backup" (1st of each month, 10:00 IST) that deep-links to Export.
  3. Practise a restore on `cue-dev` once.
- **Done when:** the runbook exists and a practice restore works on dev.
- **Docs:** [D1 Time Travel](https://developers.cloudflare.com/d1/reference/time-travel/)
- **Learning note:** `M6-DATA-04-backups.md` (concepts: RPO/RTO, point-in-time recovery, runbooks)

### M6-T08 · Optional "Keep audio" for voice notes (CAP-05 AC1, D-37, D-40)
- **Depends on:** M2-T05
- **Goal:** a per-note toggle to keep the voice recording, stored in D1 chunks with a 100 MB cap.
- **Steps:**
  1. Record a compressed audio file at the same time as the transcription (check what expo-speech-recognition supports, or record with an Expo audio library).
  2. Upload in ≤ 1.5 MB chunks to `attachments` / `attachment_chunks`. `GET /files/:ref` streams it back. Show a usage meter in Settings, and block new audio at 100 MB.
- **Done when:** a kept recording plays back on the web, and the usage meter updates.
- **Learning note:** `M6-T08-audio-chunks.md` (concepts: chunked uploads, storage quotas, BLOBs)

### M6-T09 · MVP acceptance: 2 weeks of daily use
- **Depends on:** all tasks
- **Goal:** prove Cue solves the original problem.
- **Steps:**
  1. Use Cue daily for 14 days. Keep a short log in `docs/learning/notes/MVP-diary.md`: missed reminders, lost notes, slow captures, annoyances.
  2. Fix anything that breaks P1 (capture speed), P4 (reminders) or P5 (no data loss) first.
  3. Review [features.md](../02-product/features.md): every MVP acceptance criterion passes.
- **Done when:** 14 days with **no lost notes and no missed reminders**, and every learning note and concept-map tick is done.
- **Learning note:** `MVP-retrospective.md` (what you learned, what you'd do differently, what to build next from v2/Future)
