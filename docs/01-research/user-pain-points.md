# Real user pain points (ranked)

> Status: Final · Research date: 2026-10-15 · Related: [research index](README.md), [features](../02-product/features.md)
>
> Sources: Hacker News, the Obsidian, Apple and Microsoft forums, Reddit (via mirrors and quotes), Product Hunt, and blogs.
> Product Hunt and blog authors are often selling a fix. Treat their posts as evidence that a problem exists, not of how common it is.

| Rank | Problem | Intensity | Our answer (feature IDs) |
|---|---|---|---|
| 1 | Notes are captured but never seen again (the "note graveyard") | Very high | AI-01, EXT-04, REM-07; Future: FUT-02 digest, FUT-03 resurfacing |
| 2 | Capture is too slow, so the thought is lost | Very high | CAP-01…CAP-09, PLAT-01 |
| 3 | A reminder is dismissed once and then forgotten | Very high (ADHD), high (all) | REM-07, REM-02 |
| 4 | Reminders don't fire, or fire late | High | REM-04, REM-05, REM-06 |
| 5 | Search can't find the old note | High | AI-01, AI-02 |
| 6 | A hurried note loses its meaning | Medium-high | CAP-10; Future: FUT-04 AI expand |
| 7 | Organising takes over (tag and folder fatigue) | High | ORG-03, ORG-04, P6 |
| 8 | Sync conflicts and data loss | High (rare, catastrophic) | DATA-01, DATA-02, CAP-09 |
| 9 | Lock-in, how long the app will last, pricing changes | Medium-high | DATA-03 |
| 10 | Fear of AI reading or training on notes | Medium-high, rising | SEC-07, AI-06 |
| 11 | Slow, online-only apps | Medium | CAP-09, PLAT-01 |
| 12 | Voice memos pile up and are never replayed | Medium | CAP-05 |
| 13 | Reminders scattered across apps; features removed | Medium | One app for notes and reminders; Future: FUT-01 location |
| 14 | Web clippers that are unreliable or limited | Medium-low | EXT-01…EXT-04 |
| 15 | Lockouts (lost encryption password or 2FA) | Low frequency, catastrophic | SEC-05, SEC-06 |

---

## 1. Notes are captured but never seen again (the "note graveyard")
- People capture a lot and almost never read it back. Reviewing notes feels like studying.
- Quotes:
  - HN, *nighttrooper*: "I almost never go back to them… sitting down to 'review my notes' feels like studying." Reply: "It's a famous problem… never saw any real solution." ([HN](https://hn.svelte.dev/item/47791772))
  - Product Hunt: "My notes don't die. They become digital fossils." Another: notes "die because traditional apps act as passive storage, not active partners." ([PH](https://www.producthunt.com/p/buddy-ai-note/where-do-your-notes-go-to-die))
  - "The archive kept growing; the part I could reach kept shrinking." ([PH](https://www.producthunt.com/p/ovandor/everything-you-ve-ever-captured-is-still-there-that-s-the-problem-august-3))
  - JA Westenberg deleted about 10,000 notes: "An idea never re-encountered might as well be forgotten entirely." ([blog](https://westenberg.beehiiv.com/p/i-deleted-my-second-brain))
- **Users want:** the app to bring notes back on its own ("on this day", tied to their current context), notes that turn into actions, and review that doesn't feel like work.

## 2. Capture is too slow, so the thought is lost
- Unlock the phone, find the app, wait for it to load, pick a folder or title: by then the thought is gone.
- Quotes:
  - Obsidian forum: "Launching is so slow that I don't even bother using it for notes on the go." Users measured 4.8 s and about 15 s. ([forum](https://forum.obsidian.md/t/mobile-startup-reduce-the-time-until-the-user-can-write/57450))
  - HN on Keep: "it loads so fast, so I don't forget what I'm writing before I write it." ([HN](https://hn.algolia.com/api/v1/items/41228758))
  - "Losing my best ideas in the four seconds it takes to open a notes app." ([PH](https://www.producthunt.com/p/jot-13/i-kept-losing-ideas-in-the-four-seconds-it-takes-to-open-a-notes-app-so-i-built-jot))
- **Users want:** capture in under a second from the lock screen, a widget, a hotkey, the share sheet or voice. No title or folder required. The editor loads first and everything else loads later.

## 3. A reminder is dismissed once and then forgotten
- One notification is easy to swipe away without thinking, and most apps never remind you again.
- Quotes:
  - Marco Arment built *Unforgetful* (Aug 2026) because Apple Reminders never re-alerts after you dismiss it: "A hidden task is a forgotten task." ([marco.org](https://marco.org/2026/08/))
  - ADHD coaching on swiping notifications away "without a thought". ([substack](https://adhdbigbrother.substack.com/p/how-to-murder-your-notifications))
  - Users of the Due app say repeating alerts are the only thing that works. ([reviews](https://appshunter.io/ios/app/due-reminders-and-timers/id390017969/reviews))
- **Users want:** reminders that repeat until marked done or snoozed, custom snooze times, and fewer notifications overall so the important ones stand out.

## 4. Reminders don't fire, or fire late
- Android battery savers (especially Samsung) kill background apps. Some apps only fire after a sync. Recurring reminders get lost.
- Sources:
  - Microsoft To Do users report reminders that never showed overnight. ([MS community](https://techcommunity.microsoft.com/discussions/to-do/to-do-task-reminders-not-always-working/2082879/replies/3641923))
  - The Tasks.org troubleshooting guide covers manufacturer restrictions; see dontkillmyapp.com. ([tasks.org](https://tasks.org/docs/troubleshooting_notifications))
  - Apple Reminders notification fixes. ([guidingtech](https://www.guidingtech.com/ways-to-fix-apple-reminders-not-sending-notifications))
- **Users want:** alarms scheduled on the device itself (they work offline), help with battery-saver exemptions during setup, and backup channels.

## 5. Search can't find the old note
- Keyword search needs the exact word you used. People remember the *meaning*, not the wording.
- Quotes:
  - "Search only works if you know the thing exists." ([PH](https://www.producthunt.com/p/ovandor/everything-you-ve-ever-captured-is-still-there-that-s-the-problem-august-3))
  - On Apple Notes: "typing word variations into the search bar is frustrating as hell." ([substack](https://bryancollins.substack.com/p/how-apple-notes-is-secretly-sabotaging))
- **Users want:** search by meaning, and the ability to ask questions of their notes and get answers that cite the source note.

## 6. A hurried note loses its meaning ("what did I mean by this?")
- Quotes:
  - "I forget intent, not information." "I completely lost why it mattered." ([PH](https://www.producthunt.com/p/buddy-ai-note/where-do-your-notes-go-to-die))
  - Writers call this "context rot". ([concept](https://concepts.dsebastien.net/concept/context-rot/))
  - "…a note for myself but then been unable to understand what I was trying to remember." ([blog](https://www.mickmel.com/write-notes-for-future-you/))
- **Users want:** context saved automatically (source page or app, time), and AI help to expand the note while keeping the original wording.

## 7. Organising takes over (tag and folder fatigue)
- Quotes:
  - A Notion setup with "four levels of nested pages, nineteen tags" lasted six weeks. Evernote tags were rebuilt twice. ([PH](https://www.producthunt.com/p/ovandor/everything-you-ve-ever-captured-is-still-there-that-s-the-problem-august-3))
  - "Spending more time tweaking plugins … than thinking and writing." ([Android Police via Yahoo](https://tech.yahoo.com/apps/articles/reason-obsidian-works-better-without-171611617.html))
  - Tiago Forte, "Tagging is broken". ([fortelabs](https://fortelabs.com/blog/tagging-is-broken/))
  - ADHD users prefer simple apps because they stop them from "endlessly tweaking settings". ([r/ADHD mirror](https://cal1.lr.ggtyler.dev/r/ADHD/comments/1ervz6i/what_adhd_apps_do_you_use/li1mfkj))
- **Users want:** no setup, AI auto-grouping, power features kept hidden.

## 8. Sync conflicts and data loss
- Examples:
  - Obsidian iPhone sync erased a daily note. ([forum](https://forum.obsidian.md/t/obsidian-sync-on-iphone-overwrites-newer-data-causing-data-loss/85214))
  - Apple Notes lost years of notes after an update. ([Apple](https://discussions.apple.com/thread/255048193))
  - Google Keep lost notes after an account switch. ([justuseapp](https://justuseapp.com/en/app/1029207872/google-keep-notizen-listen/problems))
- Quote: "Sync has to never lose data due to conflicts." ([HN](https://hn.algolia.com/api/v1/items/43217003))
- **Users want:** no silent overwrites, both copies kept on a conflict, easy backups.

## 9. Lock-in, how long the app will last, pricing changes
- Evidence:
  - Evernote's free tier was capped at 50 notes. ([The Register](https://www.theregister.com/2023/11/30/evernote_free_tier_changes/))
  - "My personal notes are valuable. I want them to last a decade." ([HN](https://hn.algolia.com/api/v1/items/43217003))
- **Users want:** one-click export to open formats and stable pricing.

## 10. Fear of AI reading or training on notes
- Evidence:
  - Evernote reversed a 2016 policy that would have let staff read notes. ([Computerworld](https://www.computerworld.com/article/3151152/evernote-backs-off-from-privacy-policy-changes-says-it-messed-up.html))
  - "I don't need yet another AI trained on my notes." ([HN](https://hn.algolia.com/api/v1/items/43217003))
  - Cloud AI needs plaintext, so it can't coexist with E2EE on the same note. ([calmara](https://calmara.app/blog/why-ai-note-apps-arent-end-to-end-encrypted))
- **Users want:** a "never used for training" promise, opt-in AI, and E2EE for sensitive notes.

## 11. Slow, online-only apps
- Notion added offline mode only in Aug 2025, and it has limits. ([Android Authority](https://www.androidauthority.com/notion-offline-mode-vs-obsidian-3590238))
- **Users want:** local-first apps where everything works offline.

## 12. Voice memos pile up
- "47 untranscribed voice memos and no idea what's in any of them." ([Mem guide](https://get.mem.ai/guides/voice-notes-that-get-used))
- **Users want:** automatic transcription, tagging, and conversion into tasks.

## 13. Reminders scattered across apps; features removed
- Keep's reminders moved to Tasks, and location reminders and Wear OS reminder creation were dropped. ([TechRadar](https://www.techradar.com/computing/websites-apps/your-google-keep-reminders-are-now-moving-to-google-tasks-and-power-users-will-find-this-very-confusing))

## 14. Web clippers that are unreliable or limited
- Evernote's clipper hangs. Notion's clipper can't tag. ([Evernote help](https://help.evernote.com/hc/en-us/articles/209004837), [Notion help](https://notion.com/help/web-clipper))
- **Users want:** one-click save, highlight plus note, and an attached reminder ("read Friday").

## 15. Lockouts (lost encryption password or 2FA)
- Evidence:
  - Forgotten Apple Notes lock passwords can't be recovered. ([Apple](https://discussions.apple.com/thread/256221714))
  - Notesnook's last-resort reset wipes data. ([Notesnook](https://notesnook.com/help/recovering-your-account))
  - USENIX 2024: only 14.8% of users stored their recovery code in more than one place. ([paper](https://usenix.org/system/files/usenixsecurity24-holtervennhoff.pdf))
- **Users want:** guided recovery that is tested at signup, and multiple ways to recover.
