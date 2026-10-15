# Vision: the main idea

> Status: Final (v1) · Last updated: 2026-10-15 · Related: [decisions](02-product/decisions.md), [features](02-product/features.md), [pain points](01-research/user-pain-points.md)

## 0. Name
**Cue**, with the tagline **"The right note at the right moment."** (D-29)
A *cue* is a prompt that triggers memory. Cue captures a thought in a second and cues you when it matters, through
reminders, page badges, and (later) answers to your questions.

## 1. The problem (in the owner's words)
> "Sometimes I remember things accidentally, but I don't have notes, so I forget them."
> (from [requiremet.txt](../requiremet.txt))

Research shows this is common, and that existing apps don't solve it (see [user-pain-points.md](01-research/user-pain-points.md)):
- **Capture is too slow.** Opening a note app takes 4–15 seconds, and the thought is gone before you type.
- **Notes die after capture.** They go into a "note graveyard" and are never seen again.
- **Reminders fail.** Users swipe them away once and forget, or Android battery savers stop them from firing.
- **Hurried notes stop making sense.** "Call him re: thing" means nothing a week later.
- **Organising is a chore.** People feel tag and folder fatigue and give up.
- **Users don't trust AI with private notes.**

## 2. The main idea
```
   CAPTURE (<2 s, anywhere)  →  INBOX (one place)  →  SORT (one swipe / key)  →  RETURN (reminders, search, page badges)
   widget · tile · share ·        no title, no          group · todo · reminder ·     nag if important · keyword
   voice · extension · bot        folder needed         done · archive                search · "2 notes on this site"
```
> **AI is Future, not MVP (D-24).** The loop above works without AI. Later, AI plugs into SORT (suggestions) and
> RETURN (chatbot, meaning search). See [future/ai-integration.md](future/ai-integration.md).
1. **Capture is never the bottleneck.** Every surface (Android widget or tile, share sheet, voice, browser
   extension, web hotkey) takes one action. Capture works offline on Android, and the extension queues it.
2. **One Inbox.** All captures land in one place, with no decisions needed at capture time. Context (the source app or
   page) is saved automatically, so hurried notes still make sense later.
3. **Sorting is one action.** Swipe or press one key to make a note a todo, group it, archive it or remind yourself.
   Dates typed in the text ("tomorrow 6pm") are detected automatically. *(Future: AI suggests the sorting and you
   accept it with one tap.)*
4. **Things come back.** Reminders actually fire, nag until done for important items, go to the right device (no PC
   pop-ups after office hours), and important ones escalate to an alarm-style alert on your phone. The extension shows notes you took on the page you're
   visiting again. *(Future: the chatbot answers "what did I note about X?")*
5. **Private by default.** Data is encrypted, and AI never trains on notes. Locked notes are end-to-end encrypted and
   are never seen by AI. Login is approved on your phone, not with a password you'll forget.

## 3. Product principles (use these to break ties)
| # | Principle | Meaning in practice |
|---|---|---|
| P1 | **Speed of capture beats everything** | Opening the editor must never wait for network, sync, or AI. |
| P2 | **Capture now, decide later** | No required title, folder, or tag at capture time. |
| P3 | **Suggest, don't force** | The app (and, later, AI) proposes; the user confirms. Never silently move or delete. |
| P4 | **A reminder that doesn't fire is a bug** | Reliability over features. Schedule on the device, have fallback channels. |
| P5 | **Never lose data** | Offline-first, keep both copies on conflict, trash with restore, full export. |
| P6 | **Simple on the surface** | Power features stay hidden until needed (Keep-simple, not Notion-complex). |
| P7 | **Privacy is visible** | Clear marks show which notes AI can see; locked notes are excluded. |
| P8 | **Same mental model everywhere** | Inbox, Notes, Todos, and Groups behave the same on Android, web, extension, and bot. |
| P9 | **Free to build, free to run** | No paid API, plan, or service. Use open source, self-hosted, on-device, or permanent free tiers ([zero-cost.md](02-product/zero-cost.md), D-21). |

## 4. Target user
- **Now:** the owner, a single user who uses Android and a work PC (browser) and has office hours.
- **Later:** the public. Design for multiple users from the start (accounts, isolating each user's data), but don't build
  billing or teams yet.

## 5. Non-goals (for now)
- Team collaboration or real-time co-editing (Notion/Google Docs style).
- A complex knowledge-graph or whiteboard tool (Obsidian/Heptabase style).
- Our own hardware (AI pendants and pins). If needed, accept imports instead.
- An iOS app in the first phase (Android first).
- Choosing the architecture or tech stack. That's the next discussion ([05-system-design](05-system-design/README.md)).
- **AI in the MVP**: chatbot, AI organising, meaning search, AI server. All are deferred (D-24), but the MVP stays AI-ready (D-25).

## 6. What makes this different (differentiators)
**MVP:**
1. **Capture in under 2 s from anywhere into one Inbox**, with one-swipe triage. Mem and Capacities each do only part of this.
2. **Login approved on your phone** (number match and fingerprint), with a QR fallback. No note app we found does this.
3. **Page-linked notes.** The extension badge reminds you of notes you took on the site you're visiting.
4. **Reminders that actually fire.** Important ones nag until done, and they're routed by device and office hours.
5. **Free and private.** No cost, locked notes are end-to-end encrypted, and you can export everything.

**Future (AI, D-24):**
6. **AI you can trust.** It answers from your notes with citations. Locked notes are excluded and nothing is used for
   training. Encrypted apps have no AI, and AI apps aren't encrypted; we sit in that gap.
7. **Voice to several todos.** One voice ramble becomes separate, dated tasks.
