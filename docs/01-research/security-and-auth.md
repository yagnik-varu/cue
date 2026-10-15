# Security and authentication research

> Status: Final · Research date: 2026-10-15 · Related: [decisions](../02-product/decisions.md) (D-05, D-06), [features](../02-product/features.md) (SEC-xx)

## B1. Login methods compared

| Method | Strength | Friction | Verdict for us |
|---|---|---|---|
| **Passkeys (WebAuthn)** | Phishing-resistant (bound to the domain). Synced passkeys count as AAL2 under NIST SP 800-63-4 | Lowest once set up | Future primary (FUT-12); adoption is still ~15–35% |
| **Phone approval (push + number matching)** | Strong when number matching and biometrics are used | Very low | **Primary login for MVP** (SEC-01) |
| **QR scan from a logged-in phone** | Strong with short expiry and context shown | Low | **Fallback** (SEC-02) |
| **TOTP authenticator app (6 digits)** | Fine as a *second* factor; can be phished in real time | Medium | **Backup only** (SEC-03) |
| Magic link / email OTP | Only as strong as the email account | Medium | Recovery only |
| SMS OTP | Weak; NIST no longer accepts it at AAL2 | Low | **Avoid** |

### ⚠ Why "short username + TOTP code" alone is not safe
- A 6-digit code has 1,000,000 combinations. Without strict rate limiting it can be **brute-forced in about 3 days**.
  ([lukeplant](https://lukeplant.me.uk/blog/posts/6-digit-otp-for-two-factor-auth-is-brute-forceable-in-3-days/))
- A fake login page can relay the code in real time, so TOTP is **phishable**.
  ([shkspr](https://shkspr.mobi/blog/2024/10/should-you-enable-totp-only-authentication/))
- **What the owner actually wants** is "type a short username on a new device, then confirm with my phone". Phone
  approval (B2) delivers exactly that, safely. See decision **D-05**.

Sources:
- [Passwordless guide](https://guptadeepak.com/ciam-compass/guides/passwordless-authentication/)
- [NIST passkeys](https://www.corbado.com/blog/nist-passkeys)
- [Apple passkey updates](https://www.authsignal.com/blog/articles/apples-wwdc25-passkey-updates-fast-forwarding-the-journey-to-passwordless)

## B2. Cross-device login: QR codes and phone approval

### Option 1: native WebAuthn hybrid (browser QR → phone passkey)
- The browser shows a QR code, the phone scans it, and Bluetooth confirms the phone is physically nearby. Phishing-resistant, and it comes free with passkeys.
- Weakness: dead ends on PCs without Bluetooth (some Windows 10 machines).
- Source: ([corbado](https://www.corbado.com/blog/webauthn-passkey-qr-code))

### Option 2: app-to-app QR (WhatsApp Web, Discord, Signal style) → our SEC-02
- **Signal's design is the gold standard.** The QR code carries the new device's *public key*, and the phone sends keys and data encrypted to that key. ([Signal blog](https://signal.org/blog/a-synchronized-start-for-linked-devices/))
- **Risk: QRLJacking.** A phisher relays a live QR code to the victim (MITRE ATT&CK T1676). Discord cut its QR lifetime to 2 minutes after scams. ([MITRE](https://attack.mitre.org/techniques/T1676), [Discord scam](https://portswigger.net/daily-swig/discord-users-warned-over-qr-code-login-scam-that-can-result-in-pwned-accounts))
- **Required mitigations:**
  - Short expiry (≤ 60–120 s).
  - Before approving, the phone shows the requesting device, browser, approximate location and IP.
  - Biometric confirmation on the phone.
  - A "new device linked" notification afterwards.

### Option 3: push approval ("Is this you signing in on Chrome/Windows?") → our SEC-01
- Works like Google Prompt. ([Google](https://support.google.com/accounts/answer/7026266))
- **Number matching is mandatory.** The new device shows 2 digits, and the user picks the same digits on the phone. This stops "push-bombing", where an attacker spams approval prompts until the user taps Approve. CISA recommends it. ([CISA](https://www.cisa.gov/sites/default/files/publications/fact-sheet-implement-number-matching-in-mfa-applications-508c.pdf))

### Recommended flow (adopted, D-05)
```
New device: enter short username ─▶ server sends push to trusted phone
                                      │
Phone: shows "Chrome on Windows · <city> · 10:42" + pick matching number + fingerprint ─▶ approve
                                      │
New device logged in ─▶ all devices get "new device signed in" alert; device appears in device list (revocable)
Fallback: phone unreachable → show QR on new device → scan with app (same approval screen)
Backup:   no phone at all → TOTP code + recovery code path (rate-limited)
```
> `<city>` is a placeholder. The real value comes from IP geolocation.

**First device bootstrap:** the very first login (no trusted phone yet) needs a separate path. For example, the signup
on Android creates the account, sets up the app lock and generates recovery codes. Details belong to the system design.

## B3. App lock and per-note locking
- **Notesnook** separates two locks: an *app lock* (PIN, biometrics) and a *vault* with its own password for individual notes. ([app lock](https://notesnook.com/help/app-lock), [vault](https://help.notesnook.com/lock-notes-with-private-vault))
- **Apple Notes** locks notes with the device passcode or Face ID, which reduces the chance of forgetting the password.
- **Real protection, not just a UI gate:** the note is encrypted with a vault key, and that key is wrapped in the Android Keystore behind biometrics.
- → our SEC-04 (app lock) and SEC-07 (locked notes).

## B4. Encryption vs AI
**The tension:** cloud AI needs plaintext. Notion, Evernote, Keep and OneNote are not E2EE.

| Pattern | Example | Trade-off |
|---|---|---|
| (a) On-device models | Reflect embeddings, Apple Foundation Models, Gemini Nano, Chrome Prompt API | Private but weaker; uneven device coverage |
| (b) Opt-in per-request decryption | Reflect (AI sees only the selected text) | Honest and simple; that text leaves the encrypted scope |
| (c) Confidential computing (enclaves) | Apple Private Cloud Compute, Tinfoil, Proton Lumo | Strongest cloud option; costly, fewer models |
| **(d) Balanced (our choice, D-04)** | Normal notes: encrypted in transit and at rest on the server, AI allowed, no training. **Locked notes: E2EE, never sent to AI** | Full AI power on normal notes; the user decides which notes are secret |

**Hidden costs of E2EE (they apply to our locked notes):**
- **Server search is impossible.** Locked notes are searched only on the device.
- **Reminders:** the server must know *when* to fire. A reminder on a locked note fires with a generic title ("🔒 Locked reminder") unless the device holds the key.
- **Bots:** Telegram and WhatsApp see plaintext in transit, so bot captures are always normal notes.

Sources:
- [Reflect privacy](https://next.reflect.app/privacy)
- [Apple PCC](https://security.apple.com/blog/private-cloud-compute/)
- [Tinfoil](https://www.llama.com/resources/case-studies/tinfoil/)
- [Why AI note apps aren't E2EE](https://calmara.app/blog/why-ai-note-apps-arent-end-to-end-encrypted)

## B5. Account recovery without lockout
- **Cautionary example:** Standard Notes has no password reset. Forget the password while signed out everywhere and the data is gone. ([Standard Notes](https://standardnotes.com/help/6/i-ve-forgotten-my-password-what-should-i-do))
- **Patterns that work:**
  - **Recovery codes / emergency kit** (Notesnook, 1Password). ([1Password](https://support.1password.com/recovery-codes/))
  - **Any logged-in device can approve a new one** (the same mechanism as SEC-01).
  - Passkey-protected key backup (WhatsApp, Oct 2025) and the **WebAuthn PRF extension** for unlocking with a passkey (Bitwarden, Nov 2025). These are future options. ([PRF](https://www.corbado.com/blog/passkeys-prf-webauthn))
  - PIN-gated escrow in hardware security modules (Signal SVR3, WhatsApp HSM) caps how many guesses are allowed. Probably overkill for us.
- **Adopted (D-06):**
  - Recovery codes are shown at signup, and the user must confirm they saved them.
  - Any trusted device can approve a new device.
  - TOTP is the backup factor.
  - Nag until at least two recovery paths are set up.
  - **Locked-note data is lost if every key is lost.** Warn the user clearly, and use the recovery code to re-wrap the vault key.

## B6. Baseline security checklist (for system design)
- TLS everywhere. Encryption at rest. Each user's data isolated (multi-user ready).
- Rate limiting and lockout backoff on every auth endpoint (TOTP, username lookup, QR polling).
- Session per device: listed, revocable, short-lived access tokens plus refresh tokens.
- An audit log of logins and device changes, visible to the user.
- AI providers used only with "no training / zero or limited retention" terms.
- Bot accounts linked through a one-time code from inside the app (never by phone number alone).
