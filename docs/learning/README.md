# Learning mode: build Cue *and* learn from it

> Status: Living · Last updated: 2026-10-15 · Related: [concept map](concept-map.md), [note template](notes/_template.md), [AGENTS.md](../../AGENTS.md), decision D-47

## Why this exists
Cue is a fun, personal project. It is also a **career project**. The owner is a Node.js developer, and Cue covers
backend and system design, React and React Native, Android and Kotlin, and solid engineering practice.

The coding agent writes the code, but the owner must always know **what was built, why, and how it works** (D-47).

## The task loop (every feature)
```
 1. BRIEF    agent explains in chat: what (feature IDs), the approach and why, 1–2 rejected alternatives,
             new concepts with Node.js analogies. Big pieces → wait for the owner's "go".
 2. BUILD    small steps, one feature ID at a time; code comments explain WHY (+ D-xx / feature IDs).
 3. EXPLAIN  agent walks through what changed: the flow, the key files, how to run and test it.
 4. TRY IT   1–3 small optional exercises: predict, trace, run a test, change one value. Never "write the module".
 5. NOTE     agent writes docs/learning/notes/<milestone>-<feature-id>-<slug>.md from the template
             and ticks the concepts in concept-map.md.
```

## Where things are
| What | Where |
|---|---|
| All concepts this project teaches, with progress checkboxes | [concept-map.md](concept-map.md) |
| One note per feature or task (read in milestone order M1 → M6) | `docs/learning/notes/` |
| Note template | [notes/_template.md](notes/_template.md) |
| The "why" behind every big choice | [decisions.md](../02-product/decisions.md) |
| How the whole system fits together | [05-system-design](../05-system-design/README.md) |

Note file naming: `M1-SEC-06-recovery-codes.md`, `M3-REM-04-android-exact-alarms.md`, and so on.

## Asking for more depth (prompts that work well)
- "Explain this like I know Express and Node, but not Workers."
- "Draw the request flow for this feature step by step."
- "What would break if we removed this line?"
- "How would a big company do this differently at scale?"
- "Quiz me on this note."
- "Walk me through this Kotlin file line by line."
