---
name: cue-task
description: Do exactly one Cue build-plan task card (e.g. M1-T05) in learning mode. Use when the user says "/cue-task <TASK-ID>", "do task M2-T06", or asks to continue the build plan.
---

# Do one Cue task card

The user gives a task ID like `M1-T05`. If they didn't, open `docs/06-build-plan/README.md`, find the first unticked
task in the **Progress tracker**, and ask the user to confirm it.

## Steps
1. Follow `AGENTS.md`: learning mode, the zero-cost rule (D-21), no AI (D-24), feature IDs.
2. Open the milestone file: `M0-setup.md`, `M1-foundations.md`, `M2-capture-organise.md`, `M3-reminders.md`,
   `M4-extension.md`, `M5-search-hooks.md` or `M6-hardening.md` in `docs/06-build-plan/`. Find the card
   `### <TASK-ID> · …`. Check that its **Depends on** tasks are ticked in the tracker; if not, tell the user.
3. Read **only** the docs in the card's **Read first** line (plus `docs/06-build-plan/links.md` if you need a link).
   If the task touches UI, also read the sections of `docs/07-design/ui-ux-design.md` that its §11 task map lists for
   this task ID, and use `docs/07-design/mockups.html` as the visual reference. Never invent colours, sizes, components
   or copy. Use only §3 tokens and §5 components.
4. **Brief the user** before coding:
   - what you'll build (feature IDs)
   - the approach and why, and 1–2 rejected alternatives
   - new concepts, explained with Node.js / Express analogies

   If the card title has ⭐, **wait for the user to say "go"**.
5. Do the **Steps** in order, using the exact file paths. Code comments explain *why* and cite D-xx and feature IDs.
   Kotlin gets extra explanatory comments. Don't add anything that isn't in the card.
6. Run every **Done when** check and show the output. If a check fails, fix it, or stop and explain.
7. **Explain** what you built: the flow, the key files in reading order, and how to run and test it.
8. Give 1–3 small, optional **Try it** exercises (predict, trace, tweak). Never ask the user to write a whole module.
9. Write the learning note from `docs/learning/notes/_template.md`, using the file name given on the card. Tick the
   matching concepts in `docs/learning/concept-map.md`.
10. Suggest the commit message `feat(<TASK-ID>): <summary> [<FEATURE-IDS>]`, and tick the task in the tracker in
    `docs/06-build-plan/README.md`.

## If something doesn't fit
If the design doc, the card and reality disagree (a library changed, a version doesn't work), **stop**. Explain the
problem and propose a fix. Only after the user agrees, record it as a new row in `docs/02-product/decisions.md`.
Never change a decision silently.
