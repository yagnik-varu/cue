# Auth and security flows

> Status: Draft for review · Last updated: 2026-10-15 · Related: [security research](../01-research/security-and-auth.md), [data-model](data-model.md), decisions D-04, D-05, D-06, D-33, features SEC-01…SEC-08

## 0. Building blocks
| Piece | Design |
|---|---|
| **Device key** | Every device creates an **ECDSA P-256 keypair** on first run. Android keeps the private key in `expo-secure-store` (Keystore-backed). Web and extension keep it as a **non-extractable WebCrypto key** in IndexedDB. The server stores only public keys. |
| **Access token** | JWT (HS256, Worker secret), **15 min**, containing `user_id` and `device_id`. On each request the server also checks that the device isn't revoked (one indexed D1 read), so revocation is instant. |
| **Refresh token** | 32 random bytes, stored as a SHA-256 hash, **rotated on every use**, lasting 60 days. A refresh request must be **signed by the device key**, so a stolen refresh token alone is useless. |
| **No passwords** | There are no passwords, so the server does no slow password hashing. That fits the tight CPU limit of the Workers free plan. Server crypto is cheap: SHA-256, HMAC, ECDSA verify, AES-GCM. |
| **Location hint** | The Worker reads Cloudflare's free `request.cf` data (city and country) for approval screens and audit logs. No paid geo-IP. |

## 1. Signup: Android only (D-33)
```
Android app                                   API
 1. generate device keypair
 2. user picks username (live availability check, rate-limited)
 3. POST /auth/signup {username, timezone:"Asia/Kolkata", device:{name, kind:"android", public_key, push_token}}
                                               → create user + device(trusted=1) + session
                                               → generate 10 recovery codes (shown ONCE), store SHA-256 hashes
                                               ← {tokens, recovery_codes[10]}
 4. show recovery codes → user must re-type one to continue (SEC-06 AC1)
 5. set up app lock (PIN/biometric, SEC-04) → notifications + battery exemption + test reminder (REM-05)
```
- Signup is rate-limited per IP. While the app is personal, signup can be **closed by config** (`SIGNUP_OPEN=false`) after the owner's account exists.

## 2. Sign in on web or extension: approve on phone with number matching (SEC-01)
```
New device (web/ext)            API                                   Trusted phone
 1. generate keypair
 2. POST /auth/login/start {username, device:{kind,name,public_key}}
                                 create login_request: number=42, nonce, expires 120 s
                                 FCM → all trusted devices ────────────▶ "Sign-in request"
 ◀── {request_id, number: 42, nonce}                                     shows: Chrome on Windows ·
 3. show "Select 42 on your phone"                                         <city>, <country> · 10:42
                                                                         numbers [17] [42] [88]
                                                                         user taps 42 + biometric
                                 ◀── POST /auth/login/approve {request_id, chosen:42,
                                      sig_by_phone_key(request_id|42)}
                                 wrong number → deny + alert "possible attack"
 4. poll GET /auth/login/status?id (every 2 s, ≤120 s)  → "approved"
 5. POST /auth/login/complete {request_id, sig_by_new_device_key(nonce)}
                                 verify signature → register device(trusted=0) → session
 ◀── {tokens}                    push "New sign-in: Chrome on Windows" to all trusted devices; audit event
```
- **Unknown username:** the server returns the same response shape with a request that never gets approved, so attackers can't discover which usernames exist.
- **Rate limits:** `login/start` allows 5 per username per 10 min and 20 per IP per hour. This stops push-bombing.
- Step 5 proves the device that *asked* is the one that *receives* the session (someone who only saw `request_id` can't take it).

## 3. QR fallback (SEC-02)
```
New device: POST /auth/qr/start {device:{...}} → {request_id, nonce, qr: "cue://link?r=<request_id>&f=<pubkey-fingerprint>"}
            show QR; auto-refresh every 60 s (old request expires)
Phone: scan (expo-camera) → GET /auth/qr/details?r= → same approval screen as §2 (device, browser, city, time)
       + warning if the requester's country/city differs from the phone's current location (both from request.cf)
       → biometric → POST /auth/qr/approve {request_id, fingerprint, sig_by_phone_key}
New device: poll status → POST /auth/login/complete (same as §2 step 5)
```
- Phone screen text: **"Only approve if you are looking at this QR on your own screen right now."** This mitigates QRLJacking.
- The fingerprint in the QR must match the public key the new device registered, so an attacker's relayed QR can't be swapped.

## 4. TOTP backup factor (SEC-03)
- **Enrol** from the trusted Android app: the server creates a secret (stored AES-GCM-encrypted with a Worker secret) and returns an `otpauth://` URI, shown as a QR for Google or Microsoft Authenticator. The user confirms with one code.
- **Use** when the phone isn't available: new device → `username + 6-digit code` → device added as **untrusted** → all trusted devices alerted.
- **Strict limits:** 5 failures → 15 min lock; 20 failures per day → 24 h lock. `last_used_step` prevents reusing the same code.

## 5. Recovery (SEC-06)
| Situation | Path |
|---|---|
| New phone, old phone still works | Sign in on the new phone via §2 or §3. The old phone then **promotes** it to trusted (biometric on the old phone) |
| Phone lost | New Android → "Recover account" → `username + one recovery code` (+ TOTP if enabled) → becomes the trusted device. The old phone is revoked from the device list. That recovery code is burned, and the app prompts to generate a new set |
| All recovery paths lost | Account cannot be recovered (by design). The UI warns about this |

- **Nag** until there are at least 2 recovery paths: recovery codes confirmed, plus (TOTP enabled **or** a second trusted Android device).

## 6. Locked notes vault (SEC-07)
| Item | Design |
|---|---|
| Vault key (VK) | A random 256-bit AES-GCM key, generated on the client the first time a note is locked |
| Note encryption | `ciphertext = base64(JSON{v:1, iv, ct})`, where `ct = AES-GCM(VK, JSON{title, body})` with a random 96-bit IV per save |
| Unlock on Android | VK is stored in `expo-secure-store` with **biometric required** |
| Unlock with vault PIN (any device) | KEK = **PBKDF2-SHA256** (high iteration count, run **on the client**, not on the Worker) of the vault PIN with a salt. `wrap(VK, KEK)` is stored in `vaults.wrapped_keys` |
| Vault recovery key | A 24-character key shown once when the vault is created (the user saves it). `wrap(VK, HKDF(recovery key))` is also stored |
| Server sees | Only ciphertext and wrapped keys. Never VK, the PIN, the recovery key or plaintext |
| Auto-lock | 5 min after the last use, and when the app goes to the background |
| Reminders on locked notes | Notifications show **"Cue: 🔒 Locked reminder"**. The device decrypts the title only after unlock |
| Search | Only on the device, after unlock, in memory |

## 7. Device management (SEC-05)
- `GET /devices`: name, kind, trusted, last seen, city, and the notification schedule (REM-08).
- `POST /devices/:id/revoke` (needs biometric on a trusted device): sets `revoked_at` and deletes its sessions. Its next request gets a 401.
- Every new sign-in, recovery and revocation → push alert + an `audit_events` row (SEC-08).

## 8. Telegram linking (BOT-01)
> **Future (D-44).** Not built in MVP; kept for when a bot is added.
- In the app: Settings → **Link Telegram** → `POST /telegram/link-code` → an 8-character code, valid for 10 min.
- In Telegram: send `/link ABCD1234` to the bot → the server maps `chat_id` to the user. The phone number is never used.
- **Webhook security:** `setWebhook` uses a `secret_token`. The Worker rejects requests without the matching `X-Telegram-Bot-Api-Secret-Token` header.
- `/unlink` in the bot, or in app settings, removes the link.

## 9. Rate-limit table (SEC-08)
| Endpoint | Limit |
|---|---|
| `/auth/signup` | 3 / IP / day |
| `/auth/username-available` | 30 / IP / hour |
| `/auth/login/start`, `/auth/qr/start` | 5 / username / 10 min; 20 / IP / hour |
| `/auth/login/status` | 1 every 2 s per request id |
| TOTP and recovery attempts | 5 failures → 15 min lock; 20 / day → 24 h lock |
| `/telegram/link-code` (Future) | 5 / user / hour |

Implemented with the D1 `rate_limits` table (fixed windows), or the Workers rate-limiting binding if it's free (see [hosting](hosting-and-distribution.md#free-tier-limits)).
