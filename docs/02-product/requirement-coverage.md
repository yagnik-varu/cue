# Requirement coverage

> Status: Final · Last updated: 2026-10-15 · Source: [requiremet.txt](../../requiremet.txt) · Related: [features](features.md), [decisions](decisions.md)

Each line of the original requirement maps to the decisions and feature IDs that satisfy it.

| # | Original requirement (paraphrased) | Covered by |
|---|---|---|
| R1 | "I remember things accidentally but don't have notes, so I forget." | Vision; CAP-01…CAP-10 (fast capture), REM-xx (bring it back), ORG-07 (search); Future: AI-01 (ask your notes) |
| R2 | One platform connected across mobile, website, and an integrated chatbot | D-02; PLAT-02; Android (A), Web (W), Extension (E). Messaging bot → Future (D-44). **Chatbot: Future** (D-03 scope, D-24 timing); AI-ready via PLAT-04 / D-25 |
| R3 | Authenticated so data is safe | SEC-01…SEC-08 |
| R4 | Reminders for tasks | REM-01…REM-09 (MVP), REM-10 (v2) |
| R5 | Secure, with easy-to-use features | SEC-xx; [ux-principles.md](ux-principles.md); P1–P8 in [vision](../00-vision.md) |
| R6 | Browser extension for quick notes | EXT-01…EXT-05, CAP-07 |
| R7 | Authenticator app to log in on a device with a short username | D-05; SEC-01 (username → phone approval), SEC-02 (QR), SEC-03 (TOTP backup) |
| R8 | Research existing note-takers and how to make ours more useful | [01-research/competitor-survey.md](../01-research/competitor-survey.md) |
| R9 | Which new features we can introduce | [vision §6 differentiators](../00-vision.md#6-what-makes-this-different-differentiators); [emerging-features.md](../01-research/emerging-features.md); REM-08, EXT-04, SEC-01, CAP-05 |
| R10 | Easy to use on web, mobile and extension | [ux-principles.md](ux-principles.md); PLAT-01…PLAT-03 |
| R11 | Hurried notes can later be grouped, marked complete, or converted into todos | CAP-01 (Inbox), ORG-01, ORG-02, ORG-04…ORG-07; Future: ORG-03, AI-04 |
| R12 | Explore web sources first, then review them one by one | Research method in [01-research/README.md](../01-research/README.md) |
| R13 | Find the real problems people face | [01-research/user-pain-points.md](../01-research/user-pain-points.md) |
| R14 | No system architecture yet; gather features first | D-19; [05-system-design](../05-system-design/README.md) is a placeholder |
| R15 | *(Added 2026-10-15 in chat)* PC/browser notifications only during office hours, set by the user; time zone India | D-08, D-28; REM-08 (AC5, AC6) |
| R16 | *(Added 2026-10-15 in chat)* App must be free; no spending | D-21, D-22, D-23; [zero-cost.md](zero-cost.md) |
| R17 | *(Added 2026-10-15 in chat)* MCP support in the future | D-15; FUT-05 |
| R18 | *(Added 2026-10-15 in chat)* Move AI and the AI server to Future; build so AI integrates easily; leave a small doc | D-24, D-25; PLAT-04; [future/ai-integration.md](../future/ai-integration.md) |
