# Cue: docs index

> Status: Living index · Last updated: 2026-10-15 · Entry point for agents: [../AGENTS.md](../AGENTS.md)

| # | Doc | What's inside | Status |
|---|---|---|---|
| 00 | [00-vision.md](00-vision.md) | Problem, main idea, principles, non-goals | Final (v1) |
| 01 | [01-research/README.md](01-research/README.md) | Research method and key takeaways | Final |
| 01 | [01-research/competitor-survey.md](01-research/competitor-survey.md) | About 30 note and task apps compared; market gaps; patterns to copy | Final |
| 01 | [01-research/user-pain-points.md](01-research/user-pain-points.md) | 15 real user problems, ranked, with sources and our answer to each | Final |
| 01 | [01-research/emerging-features.md](01-research/emerging-features.md) | AI, voice, bots, context reminders, extension ideas, offline sync | Final |
| 01 | [01-research/security-and-auth.md](01-research/security-and-auth.md) | Login patterns, QR/push approval, E2EE vs AI, recovery | Final |
| 02 | [02-product/decisions.md](02-product/decisions.md) | Decision log (what, why, rejected alternatives) | Living |
| 02 | [02-product/features.md](02-product/features.md) | Feature catalogue with IDs, priorities, acceptance criteria | Living |
| 02 | [02-product/zero-cost.md](02-product/zero-cost.md) | **Zero-cost rule**: free alternatives for every paid service | Living |
| 02 | [02-product/ux-principles.md](02-product/ux-principles.md) | "Easy to use" rules per platform | Living |
| 02 | [02-product/requirement-coverage.md](02-product/requirement-coverage.md) | Original requirement → feature IDs | Final |
| 03 | [03-roadmap.md](03-roadmap.md) | MVP → v2 → Future phases | Living |
| 04 | [04-open-questions.md](04-open-questions.md) | Risks and questions for the system design | Living |
| 05 | [05-system-design/README.md](05-system-design/README.md) | System design: architecture, stack, data model, API, sync, auth, reminders, Android, extension, bot, AI hooks, hosting | **Draft, waiting for review** |
| 06 | [06-build-plan/README.md](06-build-plan/README.md) | **Step-by-step build plan**: 71 task cards with files, commands, checks, documentation links; Antigravity setup | Ready |
| 07 | [07-design/ui-ux-design.md](07-design/ui-ux-design.md) | **UI/UX design**: tokens, components, every screen for mobile, web and extension, plus the [mockups](07-design/mockups.html) | Ready |
| L | [learning/README.md](learning/README.md) | **Learning mode**: how the agent teaches while building, the [concept map](learning/concept-map.md) and per-feature notes | Living |
| F | [future/ai-integration.md](future/ai-integration.md) | How to add AI later (deferred by D-24): hooks, provider interface, build order | Future |
| — | [glossary.md](glossary.md) | Terms used across the docs | Living |

## Reading order
Vision → Decisions → Features → UX principles → Roadmap → Open questions. Read the research when you need the evidence behind something.

## How to update these docs
- Each doc starts with a `Status · Last updated · Related` line. Update it when you edit.
- A new decision gets a new row in `decisions.md`. Old rows are marked *Superseded by D-xx*, never deleted.
- A new feature gets the next free ID in its module (`CAP`, `ORG`, `REM`, `AI`, `EXT`, `SEC`, `DATA`, `BOT`, `PLAT`).
- Research is dated (2026-10-15). If you re-check a competitor, add a dated note rather than rewriting history.
