# AGENTS.md — Cue · Start here (for AI coding agents and humans)

## What this project is
**Cue** — *the right note at the right moment.* (Product name decided in D-29.)
A secure, personal **notes + reminders platform**, for people who think of something at a random moment and forget it
because they never wrote it down. Core loop: **capture in under 2 seconds from anywhere → it lands in one Inbox →
sort it in one swipe (group / todo / reminder) → reminders bring it back when it matters.**
Clients: **Android app (first)**, **web app**, **browser extension**. (Telegram bot → Future, D-44.) Built for the owner first,
designed so it can go public later. **Free to build and run (D-21).**
**AI (chatbot, AI organising, meaning search) is Future, not MVP (D-24).** The MVP must be AI-ready (PLAT-04,
`docs/future/ai-integration.md`).

## Current status (2026-10-15)
- ✅ Research done → `docs/01-research/`
- ✅ Product decisions and feature catalogue done → `docs/02-product/`
- ✅ Stack decided (D-30…D-38): TypeScript monorepo, React Native (Android) + small Kotlin modules, React + Vite PWA, Cloudflare Workers + D1
- 📝 **System design drafted, waiting for the owner's review** → `docs/05-system-design/README.md`
- ⛔ No code yet. Don't change the stack or architecture without first recording it in `docs/05-system-design/`
  and `docs/02-product/decisions.md`.

## Reading order
1. `docs/00-vision.md`: the problem, main idea, and principles
2. `docs/02-product/decisions.md`: what has been decided and why
3. `docs/02-product/features.md`: the feature catalogue (stable IDs, priorities, acceptance criteria)
4. `docs/02-product/ux-principles.md`: "easy to use" rules for each platform
5. `docs/03-roadmap.md`: build order (MVP → v2 → Future)
6. `docs/04-open-questions.md`: unresolved risks and questions
6b. `docs/05-system-design/README.md`: how it is built (architecture, data model, API, sync, auth, reminders)
7. `docs/02-product/zero-cost.md`: free alternatives for every paid service
8. `docs/future/`: guides for features deferred on purpose (e.g. AI)
8b. `docs/learning/`: learning mode, the concept map and per-feature learning notes
8c. `docs/06-build-plan/`: **the build order. Work from one task card per session** (or use `/cue-task <ID>`)
8d. `docs/07-design/ui-ux-design.md` + `mockups.html`: **the UI/UX source of truth. Any screen you build must follow it exactly**
9. `docs/01-research/`: background evidence (read when you need the "why")

## Learning mode (always on, D-47)
The owner is a **Node.js developer building Cue for fun and to learn** (backend and system design, React and React
Native, Android and Kotlin, engineering practice). You write the code, but the owner must understand **what** you
built, **why**, and **how** it works. Follow this loop for every task (details in `docs/learning/README.md`):
1. **Brief first (in chat).** Cover:
   - the feature IDs
   - the approach and why, with 1–2 rejected alternatives
   - any new concepts, explained with **Node.js analogies** (e.g. a Worker is like an Express handler with no server
     to manage; React Native is React rendering native views; Kotlin types work like stricter TypeScript)

   For big pieces (sync rule, auth flows, every Kotlin module, the reminder engine), **wait for the owner's "go"**.
2. **Build in small steps.** One feature ID at a time.
   - Comments explain **why**, not what, and cite D-xx and feature IDs.
   - Kotlin gets extra explanatory comments, because the owner is new to it.
   - No unexplained magic: say what a library does underneath when it matters.
3. **Explain after.** Walk through the flow, the key files in reading order, and how to run and test it.
4. **Try it (optional, small).** Give 1–3 exercises: predict an output, trace a request, run a test, or change one value.
   **Never ask the owner to write a whole module or feature.**
5. **Write the learning note.** Use the template `docs/learning/notes/_template.md` and save it as
   `docs/learning/notes/<milestone>-<feature-id>-<slug>.md`. Tick the concepts covered in
   `docs/learning/concept-map.md`. Record bugs as lessons (root cause and how it was found), and include the
   interview angle and how it's done at scale.

## Rules for agents
- **Reference feature IDs** (e.g. `CAP-03`, `REM-05`) in task names, commits, PR titles and code comments where useful.
- **Build MVP features first, in task-card order** (`docs/06-build-plan/README.md` tracker). Don't start v2/Future items unless asked.
- **Never change a decision silently.** If something must change, update `docs/02-product/decisions.md` (add a new
  decision row that supersedes the old one) and the affected feature entries.
- **Don't build AI features in MVP**, but always keep the PLAT-04 hooks: events, Suggestion model, search interface,
  `locked`/`ai_visible` flags, command router, `ai.enabled` flag.
- **Locked notes are never sent to AI** (now or in the future), and notes are never used to train models. This is a hard rule (see `SEC-07`, `AI-06`).
- **Zero-cost rule (D-21):** the app must cost nothing to build and run. Never add a paid API, paid plan or a service
  that needs a credit card. Check `docs/02-product/zero-cost.md` before adding any dependency or service.
- **Never silently overwrite user data.** On a sync conflict, keep both copies (see `DATA-01`).
- When you add a feature, give it the next free ID in its module and add acceptance criteria.
- The original raw requirement is `requiremet.txt` (filename typo kept on purpose). Don't edit it.
