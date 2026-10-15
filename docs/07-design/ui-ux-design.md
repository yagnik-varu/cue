# Cue: UI/UX design (single source of truth)

> Status: Ready for build · Last updated: 2026-10-16 · Decisions: D-50 (UI kits), D-51 (visual style), D-52 (UX refinements) · Covers PLAT-02 (same model everywhere) and PLAT-03 (Light/Dark/System themes, accessible text sizes) plus every UI feature in [features.md](../02-product/features.md)
> Related: [mockups.html](mockups.html) (visual reference, open in a browser) · [features](../02-product/features.md) · [build plan](../06-build-plan/README.md) · [ux-principles](../02-product/ux-principles.md) (short summary)
>
> **For coding agents:** read §0 first. For a build task, read the sections listed for it in the §11.6 map, and copy the look from `mockups.html`. Never invent colours, sizes, components or copy.

## Contents
- §0 How to use this doc · §1 Design principles · §2 Brand
- §3 Design tokens · §4 Content and microcopy · §5 Component library
- §6 Mobile app (Android) · §7 Web app · §8 Browser extension
- §9 States checklist · §10 Accessibility checklist
- §11 Implementation guide (libraries, tokens pipeline, setup, folders, Do/Don't, **task → sections map**) · §12 References

## 0. How to use this doc

This doc is **the source of truth for every pixel of Cue's UI** on Android, web and the browser extension. It implements decisions **D-50** (UI kits), **D-51** (visual style) and **D-52** (UX refinements).

**Rules for agents (read before any UI task):**
1. **Use only the tokens in §3.** Don't type a hex colour, font size, spacing, radius, shadow or duration that isn't listed there. Use the token name (Tailwind class or CSS variable), never the raw value.
2. **Use only the components in §5.** If a screen needs something new, compose it from these components. Never invent a new visual style.
3. **Use only the words in §4.** Copy strings exactly. The vocabulary in §4.1 is mandatory.
4. **Every screen must pass the §9 States checklist and the §10 Accessibility checklist** before its task is done.
5. **Find your sections:** each build-plan task card maps to the sections it needs in the **§11 Screen → task map**. Read those sections; skip the rest.
6. **If something isn't specified:** pick the closest existing pattern (e.g. a new list uses NoteRow + SectionHeader + EmptyState), keep one primary action, and write the choice in your learning note. If it's a real gap, stop and ask the owner. Don't invent colours or sizes.
7. **Mobile first in spirit:** if unsure, choose the calmer, simpler option with fewer visible controls.

Abbreviations: **dp/sp** = Android density-independent units (React Native uses these as plain numbers). **px** = web CSS pixels. "Mobile" = the Android app. "Web" = the web PWA. "Ext" = the Chrome extension (popup + side panel use the **web** sizes).

---

## 1. Design principles

| # | Principle | What it means on screen |
|---|---|---|
| 1 | **Capture first** | The capture action (FAB on mobile, `Q` / quick-add bar on web, popup on ext) is always one tap or key away. Capture asks for **nothing** but text. |
| 2 | **One primary action per screen** | Exactly one filled `primary` button (or the FAB) per screen. Everything else is ghost / outline / text. |
| 3 | **Progressive disclosure** | Secondary actions live in a long-press sheet, an overflow menu (`⋮`) or on hover. Never show a toolbar of 6 icons. |
| 4 | **Undo, not confirm** | Archive, delete, done, move and triage all happen immediately and show a Toast with **Undo**. A confirmation Dialog is only for irreversible things: delete forever, revoke device, regenerate recovery codes. |
| 5 | **One time concept** | Only **Reminder**. No due date, start date or deadline anywhere. |
| 6 | **One organising concept** | **Groups** organise. Tags are secondary (shown small, managed in the editor). There are no folders, labels or projects. |
| 7 | **Same words everywhere** | The §4.1 vocabulary is used identically on Android, web, extension and in notifications. |
| 8 | **Never show empty chrome** | No empty filter bars, empty toolbars or "0" badges. Empty lists show one EmptyState (§5.15). |
| 9 | **Calm by default, expressive at key moments** | Neutral greys + one indigo accent. Motion and haptics only for FAB press, swipe commit and completing a todo. |
| 10 | **Accessibility is not optional** | 48 dp targets, AA contrast, labels on every icon button, reduce-motion support (§10). |

---

## 2. Brand

### 2.1 Name and tagline
- **Name:** Cue. Always capital C, never "CUE" or "cue" in UI text (code identifiers may be lowercase).
- **Tagline:** "The right note at the right moment." Used on the onboarding welcome, the web login screen and the store/README. Never in app chrome.
- **App title / window title:** `Cue`. Web tab title: `Cue · Inbox`, `Cue · Today`, etc.

### 2.2 Logo and app icon
- **Mark:** a lowercase "c" drawn as an open ring (stroke ≈ 18% of the icon size, round caps), with a **filled dot** sitting in the ring's opening at the 2 o'clock position. The dot is the "cue". Geometric, no gradients.
- **Colours:**
  - Light usage: ring and dot in `primary` `#4F46E5` on `#FFFFFF`.
  - Dark usage: ring and dot in `#818CF8` on `#09090B`.
  - Monochrome: `foreground`.
- **Android adaptive icon:**
  - Foreground layer: the white mark (ring + dot `#FFFFFF`) centred in the 66 dp safe zone of a 108 dp canvas.
  - Background layer: solid `#4F46E5`.
  - Monochrome layer (Android 13+ themed icons): the mark alone.
- **Favicon / extension icons:** the same mark on a `#4F46E5` rounded square (radius 22%), exported at 16, 32, 48 and 128 px. At 16 px, drop the dot gap detail if it blurs.
- **Notification small icon (Android):** the white mark only, on transparency (24 dp).
- **Wordmark:** "Cue" in Inter Semibold, letter-spacing −0.01em, placed to the right of the mark with a gap of 0.4× the mark height. Use it only on onboarding and login.

### 2.3 Voice and tone
- **Professional, calm, brief.** Short sentences. Present tense. No exclamation marks, no emoji in UI copy (emoji only as icons inside chips if a Lucide icon is unavailable, e.g. "⏰" in examples here stands for the `bell` icon).
- Speak to the user as "you". Never blame ("You forgot…" ✗ → "Still pending: …" ✓).
- **Notification titles start with "Cue:"**, e.g. `Cue: Pay electricity bill · 6:00 PM`.
- Full do / don't table: see [ux-principles §Voice and tone](../02-product/ux-principles.md#voice-and-tone). §4 below lists the exact strings.

---

## 3. Design tokens

**Where tokens live:** `packages/ui-tokens`. They're exported as CSS variables for web and extension (Tailwind v4 `@theme`) and as a NativeWind preset for mobile. Setup is in §11. The names below are the variable names: `--background` in CSS, `bg-background` in Tailwind / NativeWind.

### 3.1 Colours

**Base (shadcn names):**

| Token | Light | Dark | Used for |
|---|---|---|---|
| `background` | `#FFFFFF` | `#09090B` | App / page background |
| `foreground` | `#18181B` | `#FAFAFA` | Main text, icons |
| `card` | `#FFFFFF` | `#18181B` | Cards, sheets, panels, rows on elevated surfaces |
| `card-foreground` | `#18181B` | `#FAFAFA` | Text on card |
| `popover` | `#FFFFFF` | `#18181B` | Menus, popovers, command palette |
| `popover-foreground` | `#18181B` | `#FAFAFA` | Text in popovers |
| `muted` | `#F4F4F5` | `#27272A` | Subtle fills: input bg on mobile, skeletons, tag chips, hover on neutral |
| `muted-foreground` | `#63636B` | `#A1A1AA` | Secondary text: timestamps, captions, placeholders, section titles |
| `border` | `#E4E4E7` | `#27272A` | Dividers, card / input borders |
| `input` | `#E4E4E7` | `#27272A` | Input borders |
| `primary` | `#4F46E5` | `#818CF8` | The one primary action, FAB, active tab, links, focus accents |
| `primary-foreground` | `#FFFFFF` | `#1E1B4B` | Text / icons on `primary` |
| `secondary` | `#F4F4F5` | `#27272A` | Secondary button fill |
| `secondary-foreground` | `#18181B` | `#FAFAFA` | Text on secondary |
| `accent` | `#EEF2FF` | `#1E1B4B` | Hover / selected row bg, date highlight bg, conflict banner bg |
| `accent-foreground` | `#3730A3` | `#C7D2FE` | Text on accent |
| `destructive` | `#DC2626` | `#DC2626` | Destructive buttons (delete forever, revoke) |
| `destructive-foreground` | `#FFFFFF` | `#FFFFFF` | Text on destructive |
| `ring` | `#6366F1` | `#818CF8` | Focus ring (`focus-visible`) |
| `sidebar` | `#FAFAFA` | `#0F0F12` | Web sidebar bg, extension side-panel header |

**Cue semantic tokens:**

| Token | Light | Dark | Used for |
|---|---|---|---|
| `overdue` | `#DC2626` | `#F87171` | Overdue reminder chip text / icon, "Overdue" section title |
| `reminder` | `#4F46E5` | `#818CF8` | Reminder chip icon + text (not overdue) |
| `important` | `#B45309` | `#FBBF24` | Important flag icon, "Missed reminders" banner text / icon |
| `success` | `#15803D` | `#4ADE80` | Checked todo fill, "Done" confirmations |
| `swipe-todo` | `#4F46E5` | `#4F46E5` | Swipe-right background in Inbox |
| `swipe-done` | `#15803D` | `#15803D` | Swipe-right background in Todos |
| `swipe-archive` | `#52525B` | `#52525B` | Swipe-left background |
| `swipe-foreground` | `#FFFFFF` | `#FFFFFF` | Icon + label on all swipe backgrounds |

**Group colour dots** (the only extra colours; used **only** as an 8 px dot, never as text or a background):

| Name | Light / dark hex |
|---|---|
| `group-indigo` | `#6366F1` |
| `group-sky` | `#0EA5E9` |
| `group-teal` | `#14B8A6` |
| `group-green` | `#22C55E` |
| `group-amber` | `#F59E0B` |
| `group-rose` | `#F43F5E` |
| `group-violet` | `#8B5CF6` |
| `group-zinc` | `#71717A` |

The default for a new group is `group-indigo`.

**Contrast (WCAG 2.2, verified):**

| Pair | Mode | Ratio | Passes |
|---|---|---|---|
| `foreground` on `background` | light | 17.7:1 | AAA |
| `muted-foreground` on `background` | light | ≈5.9:1 | AA |
| `primary-foreground` on `primary` | light | 6.29:1 | AA |
| `primary` text on `background` | light | 6.29:1 | AA |
| `overdue` on `background` | light | 4.83:1 | AA |
| `important` on `background` | light | 5.02:1 | AA |
| `accent-foreground` on `accent` | light | 8.88:1 | AAA |
| `foreground` on `background` | dark | 19.1:1 | AAA |
| `muted-foreground` on `background` | dark | 7.76:1 | AAA |
| `primary-foreground` on `primary` | dark | 5.36:1 | AA |
| `overdue` on `background` | dark | 7.19:1 | AAA |
| `swipe-foreground` on `swipe-todo` / `swipe-done` / `swipe-archive` | both | 6.29 / 5.02 / 7.73 | AA |

**Colour rules:**
- **Red is used only for** overdue items and destructive actions. Never for errors in forms (use `destructive` text for those, which is the same red, and always with an icon plus a message).
- **Colour is never the only signal.** Overdue = red + the word "Overdue" + an icon. Important = flag icon + the word "Important" in the sheet. Swipe = colour + icon + label.
- **Indigo (`primary`) appears at most once as a filled surface per screen** (the primary button or the FAB). Elsewhere it's used for text, icons and the active state.
- **Dark mode is a token swap.** Never hand-tune per-component dark colours.

### 3.2 Typography
**Font:** **Inter** (SIL OFL), weights 400 regular, 500 medium, 600 semibold. **Monospace:** `ui-monospace, "JetBrains Mono", monospace` for recovery codes, the approval number and TOTP codes. Tabular numbers (`font-variant-numeric: tabular-nums`) for times and counts.

**Mobile (sp, size/line-height):**

| Style | Size / LH | Weight | Used for |
|---|---|---|---|
| `largeTitle` | 28 / 34 | 600 | Onboarding headings, empty screen title (rare) |
| `title` | 20 / 28 | 600 | Screen titles in TopBar ("Inbox", "Todos") |
| `headline` | 17 / 24 | 600 | Sheet titles, settings group headers |
| `body` | 16 / 24 | 400 | Note text, row titles, inputs, buttons' secondary text |
| `bodyStrong` | 16 / 24 | 500 | Row titles of todos, button labels |
| `label` | 14 / 20 | 500 | Chips, tab labels, section headers, settings row values |
| `caption` | 13 / 18 | 400 | Timestamps, helper text, captions |
| `micro` | 11 / 14 | 500 | Count badges, tiny meta |

**Web and extension (px):**

| Style | Size / LH | Weight | Used for |
|---|---|---|---|
| `pageTitle` | 20 / 28 | 600 | List header ("Inbox", "Today") |
| `sectionTitle` | 13 / 20 | 600, colour `muted-foreground`, **not uppercase** | "Overdue", "Today", "Upcoming", settings groups |
| `ui` | 14 / 20 | 400 | Default UI text |
| `uiStrong` | 14 / 20 | 500 | Buttons, active nav, emphasised UI |
| `listTitle` | 14 / 20 | 400 (500 for unread Inbox rows) | Row titles |
| `small` | 12 / 16 | 400 | Meta, chips, shortcuts, captions |
| `noteBody` | 16 / 26 | 400 | Note editor text |

**Rules:**
- Note text max line length is **720 px** on web and extension.
- Truncate with an ellipsis. Never shrink the font to fit.
- Letter-spacing: −0.01em for sizes ≥ 20, otherwise 0.
- Respect the user's font scale (mobile up to 200%, browser zoom on web). Layouts must reflow (§9).

### 3.3 Spacing
4-pt scale; use **only** these values:

| Token | 0 | 0.5 | 1 | 2 | 3 | 4 | 5 | 6 | 8 | 10 | 12 | 16 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| value (px/dp) | 0 | 2 | 4 | 8 | 12 | 16 | 20 | 24 | 32 | 40 | 48 | 64 |

These match the Tailwind / NativeWind spacing classes, e.g. `p-4` = 16 and `gap-3` = 12.

- **Mobile:**
  - screen horizontal padding **16**
  - gap between rows **0** (rows have internal padding + a hairline divider inset 16)
  - gap between sections **24**
  - sheet padding **16** (top **8** after the drag handle)
- **Web:**
  - content padding **24**
  - sidebar padding **8** (items have 8/12 inner padding)
  - gap between sections **24**
  - list row inner padding **8 / 12**
- **Extension:** popup padding **12**, side-panel padding **12**.

### 3.4 Radii

| Token | Value | Used for |
|---|---|---|
| `sm` | 6 | Web inputs (inner), kbd hints, small badges |
| `md` | 10 | **Mobile buttons**, inputs on mobile, menu items |
| `lg` | 14 | **Mobile cards**, popovers, extension popup inner cards |
| `xl` | 20 | **Bottom sheet top corners**, Dialogs on mobile |
| `full` | 9999 | Chips, round checkbox, FAB, CountBadge, avatar |

**Web buttons and inputs use 8** (shadcn `--radius: 0.5rem`). Web cards and panels use 12.

### 3.5 Elevation
Light mode uses shadows. **Dark mode replaces shadows with a lighter surface (`card` `#18181B`) plus a 1 px `border`.**

| Token | CSS (light) | Used for |
|---|---|---|
| `shadow-sm` | `0 1px 2px rgba(24,24,27,0.06)` | Inputs on focus (web), hovered cards |
| `shadow-md` | `0 4px 12px rgba(24,24,27,0.08), 0 1px 3px rgba(24,24,27,0.06)` | Popovers, menus, Toast, FAB (mobile uses elevation 3) |
| `shadow-lg` | `0 12px 32px rgba(24,24,27,0.12), 0 2px 6px rgba(24,24,27,0.08)` | Dialogs, command palette, web detail panel (when overlaying), bottom sheet |

- **Android:** `elevation` 1 (sm), 3 (md), 8 (lg), with `shadowColor: #18181B`.
- **Sheet / Dialog scrim:** `rgba(9,9,11,0.40)` in light and `rgba(0,0,0,0.60)` in dark.

### 3.6 Motion

| Token | Value | Used for |
|---|---|---|
| `fast` | 120 ms | Hover, pressed, checkbox tick, chip remove |
| `base` | 200 ms | Sheets/panels small moves, toast in/out, tab change |
| `slow` | 300 ms | Bottom sheet open, web detail panel slide-in, page transitions |
| `ease-standard` | `cubic-bezier(0.2, 0, 0, 1)` | Default for everything |
| `ease-emphasized` | `cubic-bezier(0.05, 0.7, 0.1, 1)` | Things entering: sheet open, panel slide-in |
| `spring-fab` | Reanimated `withSpring`, damping 15, stiffness 220 | FAB press (scale 0.94 → 1) |
| `spring-complete` | damping 12, stiffness 180 | Checkbox fill + check draw on completing a todo |

- **Completing a todo:** check fills (fast) → title strikes through → wait 1 s → row collapses height (base) → Toast "Done · Undo".
- **Reduce motion** (Android "Remove animations", `prefers-reduced-motion`): replace all movement with a 120 ms cross-fade. No springs, no swipe fling animation (snap instead).
- **Never animate** list re-sorts, text changes or colours over 200 ms.

### 3.7 Breakpoints and layout
**Web breakpoints (min-width):** `sm` 640, `md` 800, `lg` 1100, `xl` 1400.

| Width | Web layout |
|---|---|
| ≥ 1100 (`lg`) | Three panes: **Sidebar 232** · **List** (min 360, flexible) · **Detail panel 440–520** (480 default) when a note is open |
| 800–1099 (`md`) | Sidebar 232 + List. The detail panel **overlays** the list from the right (480, `shadow-lg`, scrim none) |
| < 800 | Sidebar collapses to a 56 px icon rail (tooltips on hover). Detail opens as a full page. Toggle the sidebar with `M` |
| < 640 | Not a target (use the Android app), but must not break: single column, sidebar as a sheet |

**Web:** content max width (note editor) is **720**. The list column has no max, but the row title truncates.

**Extension:**
- **Popup:** **380 px** wide, auto height, **max 520** (Chrome's limit is 800×600).
- **Side panel:** design for a minimum width of **320** (Chrome's default is ≈ 320–360, and the user can resize).

**Mobile:**
- Top app bar **56** + status bar inset.
- Bottom nav **64** + gesture-nav inset.
- **FAB 56** placed 16 from the right and 16 above the bottom nav. The large 72 FAB is **not used**.
- All touch targets ≥ **48 × 48 dp**, with ≥ 8 dp between targets.
- The app is **edge-to-edge**: handle insets with `react-native-safe-area-context`.

### 3.8 Z-index

| Layer | Web z-index | Mobile |
|---|---|---|
| Base content | 0 | — |
| Sticky headers (section headers, top bar) | 10 | native order |
| Sidebar / detail panel | 20 | — |
| Dropdown / Popover / Tooltip | 50 | portal |
| Banner (sticky) | 30 | under TopBar |
| FAB | — | above list, below sheets |
| Sheet / Dialog scrim + surface | 60 | portal (BottomSheetModal) |
| Command palette | 70 | — |
| Toast | 80 | above everything except system UI |

### 3.9 Icons
**Library:** **Lucide**. Use `lucide-react` (web/extension) and `lucide-react-native` (mobile).
- **Stroke width:** 1.75 everywhere.
- **Sizes:**
  - **16:** web inline (chips, row meta)
  - **20:** web UI buttons, mobile inline (chips, row meta, sheet rows)
  - **24:** mobile bottom nav, TopBar actions, FAB icon
- **Colour:** icons inherit text colour (`currentColor`): `foreground` by default, `muted-foreground` for meta, and a semantic colour only as listed in §3.1.

| Concept | Lucide icon | Concept | Lucide icon |
|---|---|---|---|
| Inbox | `inbox` | Search | `search` |
| Todos | `list-checks` | Settings | `settings` |
| Notes | `notebook` | Voice | `mic` |
| Add / Capture | `plus` | Pin | `pin` |
| Reminder | `bell` | Undo | `undo-2` |
| Important | `flag` | Today | `sun` |
| Group | `folder` | Upcoming | `calendar` |
| Tag | `tag` | Missed reminders | `alarm-clock-off` |
| Locked | `lock` | Conflict | `copy` |
| Archive | `archive` | Devices | `smartphone` / `laptop` |
| Trash | `trash-2` | Office hours | `briefcase` |
| Done (check) | `check` | QR | `qr-code` |
| More (overflow) | `ellipsis-vertical` (mobile) / `ellipsis` (web) | Recovery | `key-round` |
| Close | `x` | Vault | `shield` |
| Back | `arrow-left` | Export | `download` |
| Open link | `external-link` | Command | `command` |

**Source icons** (shown in source chips and Inbox row meta):

| Source (`source_kind`) | Lucide icon |
|---|---|
| web page (`extension` with a URL, `web` link) | `globe` |
| Android share (`android_share`) | `share-2` |
| widget (`widget`) | `layout-grid` |
| tile (`tile`) | `zap` |
| voice (`voice`) | `mic` |
| extension popup (`extension`, no URL) | `puzzle` |
| web app (`web`) | `monitor` |

> If an import fails, check the exact name at lucide.dev/icons. Names occasionally get aliases; use the closest current name with the same glyph.

### 3.10 Haptics
Mobile only, via `expo-haptics`. Always respect the system haptics setting. Never fire haptics on scroll or typing.

| Event | Haptic call |
|---|---|
| Swipe crosses the commit threshold (once per gesture, both directions) | `Haptics.selectionAsync()` |
| Todo completed (checkbox or swipe Done) | `Haptics.notificationAsync(NotificationFeedbackType.Success)` |
| Long-press opens the actions sheet | `Haptics.impactAsync(ImpactFeedbackStyle.Medium)` |
| Error (save failed, wrong approval number, biometric failed) | `Haptics.notificationAsync(NotificationFeedbackType.Error)` |
| FAB press | `Haptics.impactAsync(ImpactFeedbackStyle.Light)` |

---

## 4. Content and microcopy

### 4.1 Vocabulary
**Always use exactly these words, with capitals as shown when they're a UI label:**

| Use | Meaning |
|---|---|
| **Inbox** | Where every capture lands (CAP-01) |
| **Note** | Any captured text |
| **Todo** | A note with a done state (ORG-01) |
| **Group** | The one organising container (ORG-05) |
| **Tag** | A secondary label `#name` |
| **Reminder** | The only time concept (REM-xx) |
| **Important** | A flag; important reminders repeat until done (REM-07) |
| **Locked** | An end-to-end encrypted note (SEC-07) |
| **Archive** | Hidden but kept and searchable |
| **Trash** | Deleted; restorable for 30 days |
| **Done** | Completing a todo / acknowledging a reminder |
| **Snooze** | Delay a reminder |
| **Capture** | The act of saving a thought quickly |
| **Missed reminders** | Reminders that fired but weren't handled (REM-08) |
| **Device** | A phone, browser or extension install |
| **Office hours** | A device's notification window |

**Never use in UI copy:** task, item, folder, label, due date, deadline, alarm, project, list (as a noun for groups), sync error codes.

**Verbs:** Capture, Save, Archive, Delete, Restore, Done, Snooze, Remind me, Move to group, Lock, Unlock, Approve, Revoke.

### 4.2 Standard strings
Keep these in each app's `strings.ts` (D-45). The keys are suggestions; the text is exact.

| Key | Text |
|---|---|
| `capture.placeholder` | `What's on your mind?` |
| `capture.saved` | `Saved to Inbox.` |
| `capture.savedOffline` | `Saved on this device. Will sync when you're back online.` |
| `capture.savedOfflineExt` | `Saved offline. Will sync.` |
| `action.undo` | `Undo` |
| `action.save` | `Save` |
| `action.cancel` | `Cancel` |
| `action.done` | `Done` |
| `action.snooze` | `Snooze` |
| `action.archive` | `Archive` |
| `action.delete` | `Delete` |
| `action.deleteForever` | `Delete forever` |
| `action.restore` | `Restore` |
| `action.makeTodo` | `Make todo` |
| `action.makeNote` | `Make note` |
| `action.remindMe` | `Remind me` |
| `action.moveToGroup` | `Move to group` |
| `action.lock` | `Lock` / `Unlock` |
| `toast.archived` | `Archived.` |
| `toast.deleted` | `Moved to Trash.` |
| `toast.done` | `Done.` |
| `toast.madeTodo` | `Moved to Todos.` |
| `toast.moved` | `Moved to {group}.` |
| `toast.reminderSet` | `Reminder set for {when}.` |
| `toast.snoozed` | `Snoozed until {when}.` |
| `toast.restored` | `Restored.` |
| `reminder.later` | `Later today` |
| `reminder.evening` | `This evening` |
| `reminder.tomorrow` | `Tomorrow morning` |
| `reminder.nextWeek` | `Next week` |
| `reminder.pick` | `Pick date & time` |
| `reminder.repeat` | `Repeat` |
| `reminder.important` | `Important` |
| `reminder.importantHelp` | `Keeps reminding until done.` |
| `reminder.remove` | `Remove reminder` |
| `notif.title` | `Cue: {title} · {time}` |
| `notif.repeat` | `Still pending: {title}` |
| `notif.locked` | `Cue: 🔒 Locked reminder` (the lock glyph is allowed here because notifications can't show Lucide icons) |
| `notif.group` | `Cue: {n} reminders` |
| `approval.title` | `Sign-in request` |
| `approval.instruction` | `Select the number shown on your other screen.` |
| `approval.notMe` | `This wasn't me` |
| `approval.webInstruction` | `Check your phone and select {n}.` |
| `approval.useQr` | `Use QR code instead` |
| `approval.expired` | `This request expired. Try again.` |
| `approval.denied` | `Sign-in was denied on your phone.` |
| `approval.approved` | `Signed in.` |
| `newDevice.alert` | `New sign-in: {device} · {city}` |
| `recovery.title` | `Save your recovery codes` |
| `recovery.body` | `If you lose this phone, a code is the only way back into your account. Each code works once.` |
| `recovery.saved` | `I saved these codes` |
| `recovery.retype` | `Type one code to confirm` |
| `conflict.banner` | `Edited on two devices · 2 versions` |
| `conflict.keepThis` | `Keep this` |
| `conflict.keepOther` | `Keep other` |
| `conflict.merge` | `Merge` |
| `missed.banner` | `{n} missed reminders` |
| `locked.title` | `Locked note` |
| `locked.unlock` | `Unlock to view` |
| `search.placeholder` | `Search notes` |
| `settings.title` | `Settings` |

### 4.3 Date and time formats
- **Time zone:** the user's `timezone` setting, default **Asia/Kolkata**. Format with `Intl.DateTimeFormat` (`en-IN` locale, `hour12: true`).
- **Time:** `6:00 PM`, `9:30 AM`. Always minutes, no leading zero on the hour, a space before AM/PM.

| Situation | Format | Example |
|---|---|---|
| Today | `Today {time}` | `Today 6:00 PM` |
| Tomorrow | `Tomorrow {time}` | `Tomorrow 9:00 AM` |
| Within the next 6 days | `{Ddd} {time}` | `Tue 6:00 PM` |
| Further, same year | `{D} {Mmm}, {time}` | `21 Oct, 6:00 PM` |
| Different year | `{D} {Mmm} {YYYY}, {time}` | `3 Jan 2027, 9:00 AM` |
| Overdue | `Overdue · {formatted}` in `overdue` + `bell` (or `alarm-clock-off` in Missed reminders) | `Overdue · Mon 6:00 PM` |
| Repeating | append ` · {rule}` | `Tue 9:00 AM · Weekly` |
| Created (lists) | `now` (< 1 min), `{m}m` (< 60 min), `{h}h` (< 24 h, same or prior day), `Yesterday`, then `{D} {Mmm}` | `5m`, `2h`, `Yesterday`, `14 Oct` |
| Full timestamp (detail meta, conflict) | `{Ddd} {D} {Mmm}, {time}` | `Thu 16 Oct, 10:42 AM` |
| Office hours | `Mon–Fri · 9:00 AM–6:00 PM` | |

- **Date-picker "Repeat" labels:** `None`, `Daily`, `Weekdays`, `Weekly`, `Monthly`, `Custom…`.
- **Numbers:** plain digits; counts over 99 show `99+`. Use Indian grouping only for large numbers in settings (e.g. storage `1,00,000` isn't needed; show `12 MB`).

### 4.4 Empty-state copy
Each empty state has: a 48 px muted Lucide icon, one **title** line (bodyStrong / uiStrong), one **hint** line (caption / small, muted), and at most one action.

| Screen | Icon | Title | Hint (mobile / web) |
|---|---|---|---|
| Inbox | `inbox` | `Inbox clear.` | `Capture anything with the + button.` / `Press Q to capture.` |
| Todos (none) | `list-checks` | `No todos.` | `Swipe right on an Inbox note to make it a todo.` / `Press T on a note to make it a todo.` |
| Today (web) / Todos Today section | `sun` | `Nothing for today.` | `Enjoy the calm.` |
| Overdue | — | (hide the section entirely) | — |
| Notes | `notebook` | `No notes yet.` | `Notes you keep show up here.` |
| Group | `folder` | `This group is empty.` | `Move notes here with Move to group.` |
| Archive | `archive` | `Nothing archived.` | `Archived notes stay searchable.` |
| Trash | `trash-2` | `Trash is empty.` | `Deleted notes stay here for 30 days.` |
| Search (no results) | `search` | `No notes match "{q}".` | `Try fewer words.` |
| Missed reminders | `alarm-clock-off` | `No missed reminders.` | — |
| Ext "This page" | `globe` | `No notes for this page.` | `Select text and right-click → Save to Cue.` |
| Devices | — | (never empty: always at least the current device) | — |

### 4.5 Error and status messages
Always say what happened and what to do next. Never show raw error codes (log them instead).

| Situation | Message | Where |
|---|---|---|
| Mobile offline | `Offline. Saved on this device and will sync when you're back online.` | Banner (muted), once per offline period |
| Web offline | `You're offline. Changes can't be saved right now.` | Banner (muted); editing disabled |
| Web save failed (online) | `Couldn't save. Try again.` + `Retry` | Toast |
| Sync conflict | `Edited on two devices · 2 versions` | Banner (accent) on the note |
| Missed reminders | `{n} missed reminders` + `View` | Banner (important) on Todos / web Today |
| Notifications off (mobile) | `Notifications are off, so reminders can't reach you.` + `Fix` | Banner (destructive outline) on Todos |
| Battery restricted (mobile) | `Battery settings may block reminders.` + `Fix` | Banner (destructive outline) on Todos |
| Session expired | `Please sign in again.` | Web/ext login screen |
| Device revoked | `This device was signed out.` | Login screen |
| Rate limited | `Too many attempts. Try again in {m} minutes.` | Inline under the field |
| Wrong approval number | `That number didn't match. Sign-in was blocked.` | Phone, inline + error haptic |
| Biometric failed | `Couldn't verify. Try again or use your PIN.` | Inline |
| Vault PIN wrong | `Wrong PIN.` | Inline under the field |
| Voice not available | `Voice input isn't available on this device.` | Toast |
| Generic | `Something went wrong. Try again.` | Toast (log details) |

---

## 5. Component library

> If a mapped library is unavailable, build the component with NativeWind classes using §3 tokens; behaviour stays the same.

**Mapping key:**
- **Web/ext** = the shadcn/ui component (Radix-based, Tailwind v4) in `apps/web/src/components/ui` (copied by the shadcn CLI), shared with the extension.
- **Mobile** = the React Native Reusables component (shadcn-style, NativeWind) in `apps/mobile/src/components/ui`, or **custom with NativeWind**.
- Cue-specific wrappers live in `components/cue/` on each platform.

### 5.1 Button
- **Purpose:** trigger an action.
- **Variants:**

  | Variant | Look | Used for |
  |---|---|---|
  | `primary` | `primary` fill, `primary-foreground` text | The one main action per screen |
  | `secondary` | `secondary` fill | Alternatives ("Save file") |
  | `outline` | transparent, 1 px `border` | Neutral actions in dialogs |
  | `ghost` | transparent, hover `accent` | Toolbar / inline actions |
  | `link` | `primary` text, underline on hover | "Use QR code instead" |
  | `destructive` | `destructive` fill | Delete forever, revoke |

- **Sizes:**
  - Mobile: default height **48**, padding-x 16, radius `md` (10), text `bodyStrong`; `sm` height 40.
  - Web/ext: default height **36**, padding-x 12, radius 8, text `uiStrong`; `sm` 32; `lg` 40 (login only).
- **Anatomy:** `[icon 20/16]? label [kbd hint]?`. Icon gap 8. Full-width on mobile sheets and onboarding.
- **States:**

  | State | Look |
  |---|---|
  | hover (web) | primary darkens 8% (`primary/90`); ghost gets `accent` bg |
  | pressed | opacity 0.9 + scale 0.98 (fast) |
  | focus | 2 px `ring` outline, offset 2 (`focus-visible` only) |
  | disabled | opacity 0.5, no pointer |
  | loading | label stays, spinner replaces the icon, button disabled, width fixed |

- **A11y:** the label is the accessible name. Icon-only → use IconButton.
- **Mapping:** Web/ext `Button` (shadcn). Mobile `Button` (RN Reusables).

### 5.2 IconButton
- **Purpose:** an icon-only action (TopBar actions, row hover actions, close).
- **Sizes:**
  - Mobile: 48 × 48 touch, 24 icon (TopBar) or 20 icon (inline).
  - Web/ext: 32 × 32, 16–20 icon.
- **Variants:** `ghost` (default), `outline`.
- **States:** as Button. Hover bg `accent` (web).
- **A11y:** **always** set `aria-label` / `accessibilityLabel` (e.g. "Search", "More actions"). On web, show a Tooltip with the label + shortcut after 500 ms.
- **Mapping:** Web/ext `Button size="icon"` + `Tooltip`. Mobile `Button size="icon"` (RN Reusables) with a Lucide icon.

### 5.3 FAB
- **Purpose:** the single capture action on mobile (CAP-01). It's present on all three tabs.
- **Anatomy:** a 56 × 56 circle with a `primary` fill and a 24 px `plus` icon in `primary-foreground`, at elevation 3 (`shadow-md`).
- **Position:** bottom-right, 16 from the right edge and 16 above the BottomNav. It sits above the list and moves up when a Toast is shown.
- **Behaviour:**
  - **Tap:** light haptic, `spring-fab`, then opens the Capture screen (§6).
  - **Long-press:** opens the Capture screen with voice listening (`?voice=1`).
  - It **doesn't** hide on scroll (stays stable for muscle memory). There's no speed-dial and no menu.
- **States:** pressed = scale 0.94 (spring). It is never disabled.
- **A11y:** label "Capture". Hint: "Long press to capture by voice".
- **Mapping:** Mobile **custom with NativeWind** (`Pressable` + reanimated). Not used on web or ext.

### 5.4 Input and Textarea
- **Purpose:** single-line and multi-line text entry (search, username, vault PIN, editor fields).
- **Look:**
  - Mobile: height 48, radius `md`, bg `muted`, no border, text `body`, placeholder `muted-foreground`.
  - Web/ext: height 36, radius 8, 1 px `input` border, bg `background`, text `ui`.
- **States:**

  | State | Look |
  |---|---|
  | focus | web: border `ring` + 2 px ring at 30% opacity; mobile: 1.5 px `ring` border |
  | error | 1 px `destructive` border + a message below in `small` / `caption`, `destructive` colour, with an icon |
  | disabled | opacity 0.5 |

- **Labels:** a visible label above (`label` / `uiStrong`) except for search and capture, which use a placeholder plus an accessible label.
- **Textarea:** auto-grows. Min 3 lines, max 12 lines, then it scrolls.
- **Mapping:** Web/ext `Input`, `Textarea`, `Label`. Mobile `Input`, `Textarea`, `Label` (RN Reusables).

### 5.5 CaptureField
- **Purpose:** the capture text box with live reminder detection (CAP-01, REM-01). Used in mobile Capture, the web quick-add bar, the ext popup and the side-panel capture box.
- **Anatomy:**
  ```
  ┌──────────────────────────────────────────┐
  │ Pay electricity bill [tomorrow 6pm]      │  ← detected text highlighted
  │                                          │
  └──────────────────────────────────────────┘
  ( bell  Tue 21 Oct, 6:00 PM   × )             ← preview chip (Chip/reminder)
  ```
- **Highlight:** the matched date text gets an `accent` background, `primary` text, radius 4, padding-x 2. On web it's an overlay `<mark>` layer behind a transparent textarea. On mobile it's styled `Text` spans in a `TextInput` with children (or a mirrored overlay).
- **Preview chip:** appears below the field within 150 ms of typing stopping. The text is the formatted time per §4.3 (+ ` · Weekly` if it repeats).
- **Behaviour:**
  - Detection runs `parseWhen(text, tz)` debounced at 150 ms.
  - **Tapping the chip's × or the highlighted text removes the detection** (and remembers the dismissal for that text span).
  - Saving creates the reminder **only if the chip is visible**. Nothing is ever applied silently (REM-01 AC2, PLAT-04 Suggestion).
- **Sizes:**
  - Mobile Capture screen: full-screen, text `body`, no border.
  - Web quick-add: single line that grows to 3 lines, height 40, radius 8.
  - Ext popup: 4-line textarea.
- **A11y:** when a reminder is detected, announce politely: "Reminder detected: Tuesday 6 PM. Double tap the chip to remove."
- **Mapping:** Cue component built on `Textarea` (web/ext) / `TextInput` (mobile) + Chip.

### 5.6 Chip
- **Purpose:** compact metadata on rows and in the editor.
- **Sizes:**
  - Web/ext: height **24**, padding-x 8, text `small`, icon 14.
  - Mobile: height **28**, padding-x 10, text `label` (14), icon 16.
  - Radius `full`. Gap 4 between icon and text, 6 between chips.
- **Variants:**

  | Variant | Look | Content |
  |---|---|---|
  | `reminder` | bg transparent, text + icon `reminder` | `bell` + `Today 6:00 PM` |
  | `reminder-overdue` | text + icon `overdue` | `bell` + `Overdue · Mon 6:00 PM` |
  | `group` | bg `muted`, text `foreground` | 8 px group-colour dot + group name |
  | `tag` | bg `muted`, text `muted-foreground` | `#name` |
  | `source` | text `muted-foreground` | source icon (§3.9) + domain (`example.com`) or app name |
  | `important` | text + icon `important` | `flag` (icon only on rows; `flag` + "Important" in the editor) |
  | `removable` | any variant + `x` (16) at the end | in the editor / CaptureField |

- **States:** pressable chips (in the editor) get hover `accent` and pressed opacity 0.9. Chips on rows are not interactive (the row is).
- **A11y:** row chips are part of the row's accessible label (see §5.8). Removable chips: label "Remove reminder".
- **Mapping:** Web/ext `Badge` (variants `secondary` / `outline`). Mobile `Badge` (RN Reusables) or custom NativeWind `View` + `Text`.

### 5.7 Checkbox
- **Purpose:** complete a todo (ORG-02). Also used for "I saved these codes" (that one is square).
- **Todo checkbox:** **ROUND**.
  - Visual size: web **20**, mobile **24**.
  - Unchecked: 1.5 px `muted-foreground` border.
  - Hover (web): border `success`.
  - Checked: `success` fill + white `check` icon.
- **Touch target:** 48 × 48 dp on mobile (the visual is centred); 32 × 32 on web.
- **Completing:** spring fill (`spring-complete`) + success haptic → the title gets a strike-through and `muted-foreground` → after **1 s** the row collapses → Toast `Done.` + `Undo` (8 s) → the item moves to **Completed**. Undo reverses it all.
- **Square checkbox** (forms): 18/20, radius 4, `primary` fill when checked.
- **A11y:** role checkbox. Label `Mark '{title}' done` / `Mark '{title}' not done`. Space toggles it on web.
- **Mapping:** Web/ext `Checkbox` (shadcn, styled `rounded-full` for todos). Mobile `Checkbox` (RN Reusables), custom-styled round.

### 5.8 NoteRow and TodoRow
- **Purpose:** the list row used in every list (Inbox, Todos, Notes, Group, Archive, Trash, Search, side panel).
- **Mobile anatomy** (min height **56**, padding 12 vertical / 16 horizontal, divider inset 16):
  ```
  ┌──────────────────────────────────────────────────┐
  │ (○)  Pay electricity bill before Friday      2h  │  ← TodoRow: round checkbox; NoteRow: no checkbox
  │      [bell Tue 6:00 PM] [● Home] [globe x.com]   │  ← max 1 line of chips, overflow hidden
  └──────────────────────────────────────────────────┘
  ```
  - Title: `bodyStrong` (todo) / `body` (note), **max 2 lines**.
  - NoteRow shows the first line of the body as the title and the second line as a muted 1-line preview **only if there are no chips**.
  - Time: `caption`, `muted-foreground`, right-aligned, top.
  - Important: a `flag` icon (16, `important`) before the time.
- **Web/ext anatomy** (height **36**, single line, padding 8/12):
  ```
  │ (○) Pay electricity bill   [bell Tue 6:00 PM] [● Home]     2h │ hover → [bell][folder][archive] │
  ```
  - Hover shows 3 IconButtons on the right (Remind `H`, Move `M`, Archive `E`), replacing the time.
  - The side panel uses the same row at 2 lines (title + chips) because of its width.
- **States:**

  | State | Look |
  |---|---|
  | default | bg `background` |
  | hover (web) | bg `muted` at 60% |
  | pressed (mobile) | bg `muted` |
  | selected / open | bg `accent` (web: a 2 px `primary` bar on the left) |
  | keyboard focus | 2 px `ring` inset |
  | completed | title strike-through + `muted-foreground` |
  | unread in Inbox (web) | title `uiStrong` |

- **Locked note:** the title is replaced by `Locked note` with a `lock` icon (16/20), and no preview. Chips show only the reminder (as `Locked reminder`) and the group.
- **Conflict copy:** a `copy` icon + the `Conflict` chip (accent).
- **Behaviour:**
  - Tap/click → open (mobile: the editor screen; web: the detail panel). Long-press (mobile) → the actions sheet.
  - Mobile rows are wrapped in a SwipeRow (§5.10) in Inbox and Todos.
- **A11y:** one accessible label per row, e.g. `"Pay electricity bill. Todo. Reminder Tuesday 6 PM. Group Home. 2 hours ago."` TalkBack custom actions: Todo/Done, Archive, Remind me, Move to group.
- **Mapping:** Cue component. Web uses a `<li>` with the shadcn `Checkbox` + `Badge`; mobile uses `Pressable` + NativeWind, inside FlashList.

### 5.9 SectionHeader
- **Purpose:** group a list: Overdue / Today / Upcoming / No date / Completed, Pinned / Notes.
- **Look:**
  - Mobile: `label` (14/20 medium), `muted-foreground`, padding 16 horizontal, 24 top / 8 bottom, sticky.
  - Web: `sectionTitle` (13/20 semibold, muted, **not uppercase**), padding 24/12 top/bottom 8.
  - **Overdue** uses the `overdue` colour plus the text "Overdue".
- **Right side (optional):** a count in muted (`3`). The Overdue header has a `ghost sm` button "Reschedule" that opens the ReminderPicker for all overdue items.
- **Completed:** collapsed by default, written as `Completed (12)` with a chevron. Tap to expand.
- **Rule:** never render a header for an empty section (except Completed when > 0).
- **Mapping:** Cue component (Text + optional Button).

### 5.10 SwipeRow
- **Purpose:** one-swipe triage on mobile (ORG-04).
- **Actions:**

  | Screen | Swipe right (leading) | Swipe left (trailing) |
  |---|---|---|
  | Inbox | **Todo**: bg `swipe-todo`, icon `list-checks`, label "Todo" | **Archive**: bg `swipe-archive`, icon `archive`, label "Archive" |
  | Todos | **Done**: bg `swipe-done`, icon `check`, label "Done" | **Archive** |
  | Notes / Group | (none) | **Archive** |
  | Archive / Trash / Search | no swipes (use long-press) | — |

- **Behaviour:**
  - The background + icon + label (white, `label` 14) are revealed from the first pixel. The icon sits 24 from the edge.
  - **Commit threshold: 40% of the row width.** Crossing it fires a selection haptic, the icon scales 1.0 → 1.15, and the label stays.
  - Releasing past the threshold flings the row off (base duration), runs the action and shows a Toast with Undo. Releasing before it springs back.
- **Alternatives:** every swipe action is also in the long-press sheet and is a TalkBack custom action (never gesture-only).
- **Reduce motion:** no fling; the row fades out.
- **Mapping:** Mobile **custom**: `react-native-gesture-handler` `ReanimatedSwipeable` + `react-native-reanimated`. Not used on web (keyboard / hover actions instead).

### 5.11 BottomSheet
- **Purpose:** every secondary choice on mobile: the long-press actions, ReminderPicker, GroupPicker, filters, the share-save sheet.
- **Look:**
  - Surface `card`, top radius `xl` (20), drag handle 36 × 4 in `border` colour, centred 8 from the top.
  - Scrim per §3.5. Title `headline`, left-aligned, padding 16.
  - **Action rows:** height 52, a 20 icon + `body` label, `destructive` text for Delete.
- **Snap points:** content height (auto), max 90% of the screen. It is keyboard-aware.
- **Behaviour:**
  - Opens with `slow` + `ease-emphasized`.
  - Closes on swipe down, scrim tap or back (predictive back supported).
  - Focus moves into the sheet on open and returns to the trigger on close.
- **Long-press actions sheet (Inbox / Notes):**
  - Make todo / Make note
  - Remind me…
  - Move to group…
  - Pin / Unpin
  - Important on / off
  - Lock
  - Select
  - Archive
  - Delete

  Only the relevant items show (e.g. no "Make todo" on a todo).
- **Mapping:** Mobile **custom**: `@gorhom/bottom-sheet` `BottomSheetModal` styled with NativeWind. Web/ext equivalents: `Popover` or `DropdownMenu` for small choices, `Sheet` only for mobile-width web.

### 5.12 Dialog
- **Purpose:** **only** irreversible confirmations: Delete forever, Empty trash, Revoke device, Regenerate recovery codes, Disable app lock. Also the web shortcut cheat-sheet (`?`) and the vault-PIN entry.
- **Look:**
  - Web/ext: centred, width 440, radius 12, `shadow-lg`.
  - Mobile: centred card, width the screen minus 48, radius `xl`.
  - Title `headline` / `pageTitle`, body `body` / `ui`, buttons right-aligned (web) or stacked full-width (mobile). The destructive button is **last** (right / bottom); Cancel comes first.
- **Copy pattern:** title `Delete forever?`, body `This note will be permanently deleted. This can't be undone.`, buttons `Cancel` · `Delete forever`.
- **Behaviour:** Esc / back / scrim click = Cancel. Focus starts on **Cancel** for destructive dialogs.
- **Mapping:** Web/ext `AlertDialog` (confirmations), `Dialog` (others). Mobile `AlertDialog` / `Dialog` (RN Reusables).

### 5.13 Toast (Snackbar)
- **Purpose:** confirm an action and offer **Undo** (Principle 4).
- **Look:**
  - Surface `foreground` bg with `background` text (an inverted, high-contrast pill), radius `lg` (mobile) / 8 (web), `shadow-md`.
  - Text `body` / `ui`, max 2 lines.
  - Action: a text button in `primary` colour (in dark mode, which inverts the surface, use `#4F46E5`).
- **Position:**
  - Mobile: bottom, full width minus 32, above the FAB / BottomNav (the FAB moves up).
  - Web: bottom-left, 24 from the edges, max width 420.
  - Ext: bottom of the popup / panel.
- **Timing:** **5 s**, or **8 s** when there's an Undo. Hovering (web) pauses the timer.
- **Rules:**
  - **One at a time.** A new toast replaces the old one.
  - Max 1 action.
  - **Never** shown for routine background saves or sync. The only "saved" toast is the capture confirmation (`Saved to Inbox.`).
- **Shortcut:** `Z` on web triggers the latest Undo.
- **A11y:** `role="status"` / `aria-live="polite"`; on Android, `accessibilityLiveRegion="polite"`.
- **Mapping:** Web/ext `Sonner` (shadcn `sonner`). Mobile `sonner-native`, or a custom snackbar (NativeWind + reanimated) if unavailable.

### 5.14 Banner
- **Purpose:** a persistent, screen-level status that needs attention but doesn't block.
- **Look:**
  - Full width under the TopBar (mobile) or above the list (web), padding 12/16, `caption` / `small` text.
  - A 16/20 icon on the left and an optional `link`-style action on the right. Not dismissible unless noted.
- **Variants:**

  | Variant | Look | Example |
  |---|---|---|
  | `offline` | bg `muted`, text `muted-foreground`, icon `cloud-off` | the §4.5 offline copy |
  | `conflict` | bg `accent`, text `accent-foreground`, icon `copy` | `Edited on two devices · 2 versions` + `Compare` |
  | `missed` | bg transparent, 1 px `border`, text + icon (`alarm-clock-off`) `important` | `3 missed reminders` + `View` |
  | `permission` | 1 px `destructive` border, text `foreground`, icon `destructive` | `Notifications are off…` + `Fix` |

- **Rules:** max **one** banner per screen (priority: permission > offline > conflict > missed). Dismissible: `missed` only (it reappears for new misses).
- **Mapping:** Web/ext `Alert` (shadcn). Mobile custom NativeWind `View` (or the RN Reusables `Alert`).

### 5.15 EmptyState
- **Purpose:** explain an empty list and the next action (copy in §4.4).
- **Anatomy:**
  ```
            [ 48 px muted icon ]
              Inbox clear.
   Capture anything with the + button.
  ```
- Vertically centred in the list area (mobile: ~30% from the top). Title `bodyStrong` / `uiStrong`, hint `caption` / `small` in `muted-foreground`. Optional single `ghost` button.
- **Never** show filters, toolbars or headers alongside an empty state.
- **Mapping:** Cue component.

### 5.16 Skeleton
- **Purpose:** a loading placeholder **only when a full list load takes > 1 s** (most loads are instant on mobile because data is local).
- **Look:** bars in `muted`, radius `sm`, shaped like NoteRows (a title bar 60% wide plus a chip bar 30%), 6 rows. Pulse opacity 0.6 ↔ 1, 1.2 s. Reduce motion: static.
- **Rules:**
  - Below 1 s, show nothing (keep the previous content).
  - Never show a spinner over a list.
  - Use a spinner only inside a Button that is loading.
- **Mapping:** Web/ext `Skeleton`. Mobile `Skeleton` (RN Reusables) or a custom `View`.

### 5.17 BottomNav
- **Purpose:** the 3 top-level destinations on mobile.
- **Tabs (exactly 3):** **Inbox** (`inbox`), **Todos** (`list-checks`), **Notes** (`notebook`). Settings is **never** a tab.
- **Look:**
  - Height 64 + inset, bg `background`, 1 px top `border`.
  - Icon 24 + label `micro`/`label` (12–14) **always shown**.
  - Active: icon + label in `primary`, with a 56 × 32 `accent` pill behind the icon (M3 style).
  - Inactive: `muted-foreground`.
- **Badges:** Inbox only. A CountBadge with the number of Inbox notes (hidden at 0, `99+` cap).
- **Behaviour:** tapping the active tab scrolls the list to the top. State is preserved per tab.
- **A11y:** role tab, label `Inbox, 4 notes`.
- **Mapping:** Mobile Expo Router `Tabs` with a custom `tabBar` (NativeWind). Not used on web or ext.

### 5.18 TopBar
- **Purpose:** the screen title + up to 2 actions (mobile) / list header (web).
- **Mobile (height 56 + status inset):**
  ```
  │ Inbox                                   [search] [avatar] │
  ```
  - Title `title` (20/28 semibold), left, 16 padding.
  - Right: **Search** IconButton (`search`) and an **Avatar** button (a 32 circle with the username's initial on `accent`), which opens Settings.
  - Detail screens: back `arrow-left` + title (`headline`) + at most 2 actions + overflow `ellipsis-vertical`.
  - On scroll: a 1 px bottom `border` appears; no colour change.
- **Web list header (height 56):** `pageTitle` + count (muted) on the left; on the right a view option `ghost` button (e.g. "Show completed") and nothing more. The quick-add bar sits directly under it.
- **Mapping:** Mobile custom (NativeWind) via the Expo Router `Stack` header `header` prop. Web Cue component.

### 5.19 Sidebar
- **Purpose:** web navigation (≥ 800 px). The extension side panel doesn't use it.
- **Width:** 232. Bg `sidebar`, right 1 px `border`. Collapses to a **56** icon rail below 800 or with `M`.
- **Contents (top → bottom):**
  ```
  [Cue mark] Cue                      [⌘K]
  ─────────────────────────────────────
  inbox      Inbox                  12
  sun        Today                   3
  calendar   Upcoming
  notebook   All notes
  flag       Important
  ─────────────────────────────────────
  Groups                              +
  ● Home                              5
  ● Work                              8
  ─────────────────────────────────────
  archive    Archive
  trash-2    Trash
  (spacer)
  settings   Settings        [avatar u]
  ```
- **Item:** height 32, radius 6, padding 8/12, a 16 icon + `ui` text. The count is right-aligned, `small`, muted (Inbox and Today only; Groups show counts).
  - Hover: bg `muted`.
  - **Active:** bg `accent`, text `accent-foreground`, weight 500.
- **"Dimmed" rule:** sidebar text uses `muted-foreground` and only the active item uses full contrast, so content comes first.
- The `Groups +` button creates a group inline (an input appears in place).
- **Missed reminders:** when > 0, an item `alarm-clock-off Missed reminders 3` appears under Upcoming in the `important` colour.
- **Mapping:** Web: a shadcn `Sidebar` block, or a custom `nav` + `Button variant="ghost"` + `ScrollArea` + `Tooltip` (for the icon rail).

### 5.20 SearchField
- **Purpose:** keyword search (ORG-07).
- **Mobile:** opened from the TopBar `search` icon as a full screen. The TopBar becomes `[arrow-left] [Input "Search notes" autofocus] [x]`. Results (NoteRows) update as you type (debounced 150 ms), with the matched text bolded in the preview.
- **Web:** `/` focuses a search input in the list header (width 280, expands to 400), or use the Command palette. Results replace the list, with the title `Results for "{q}"`.
- **Ext side panel:** an input at the top of the Inbox tab.
- **States:** empty query → recent searches (max 5) + nothing else. No results → EmptyState (§4.4). Locked notes are never shown in results unless the vault is unlocked.
- **Mapping:** Web/ext `Input` with a `search` icon. Mobile `Input` (RN Reusables).

### 5.21 CommandPalette
- **Purpose:** keyboard access to every function on web (and in the ext side panel).
- **Open:** `Ctrl+K` (and `⌘K` on Mac) from anywhere. Pressing it again closes it and **restores focus** to where it was. Also opens from the sidebar `⌘K` button.
- **Look:**
  - Centred, width 560, top 20% of the viewport, radius 12, `shadow-lg`, bg `popover`.
  - Input height 48 with the placeholder `Type a command or search…`.
  - Max 8 visible rows (the 8th row half-cut to hint at scrolling).
- **Groups:**
  - **Recent** (last 5 commands, shown when the query is empty)
  - **Actions**: Capture `Q`, Make todo `T`, Done `E`, Remind me `H`, Move to group `M`, Important `I`, Lock note, Delete `#`, Undo `Z`
  - **Go to**: Inbox `G I`, Today `G T`, Upcoming `G U`, All notes `G N`, Archive, Trash, Missed reminders, a group…
  - **Settings**: Office hours, Devices, Time zone, Vault, Export, Theme
- **Row:** a 16 icon + label + an alias in muted (`Done (Archive)`), and **the shortcut as `kbd` on the right** (`small`, bg `muted`, radius 4).
- **Matching:** fuzzy and case-insensitive. Context-boosted (note actions first when a note is focused). Irrelevant commands are hidden (e.g. "Restore" outside Trash).
- **Mapping:** Web/ext shadcn `Command` (cmdk) inside a `Dialog`.

### 5.22 ReminderPicker
- **Purpose:** set, change or remove a reminder (REM-02, REM-07).
- **Mobile (BottomSheet):**
  ```
  ┌─────────────────────────────────────────┐
  │                 ────                    │
  │  Remind me                              │
  │  ┌────────────┐ ┌──────────────┐        │
  │  │ Later today│ │ This evening │        │
  │  │  1:30 PM   │ │   6:00 PM    │        │
  │  └────────────┘ └──────────────┘        │
  │  ┌──────────────────┐ ┌────────────┐    │
  │  │ Tomorrow morning │ │ Next week  │    │
  │  │    9:00 AM       │ │ Mon 9:00 AM│    │
  │  └──────────────────┘ └────────────┘    │
  │  calendar  Pick date & time          >  │
  │  repeat    Repeat                None >  │
  │  flag      Important             [ o ]  │
  │            Keeps reminding until done.  │
  │  ──────────────────────────────────     │
  │  Remove reminder   (only if one is set) │  ← destructive text
  └─────────────────────────────────────────┘
  ```
- **Quick options:** a 2 × 2 grid of tiles, radius `md`, bg `muted`, label `label` + time `caption` muted.
  - **Later today** = now + 3 h rounded up to :00/:30 (hidden after 9:00 PM).
  - **This evening** = 6:00 PM, or the end of office hours if set (hidden after 6:00 PM, replaced by "Tonight 9:00 PM").
  - **Tomorrow morning** = 9:00 AM tomorrow.
  - **Next week** = next Monday 9:00 AM.

  Tapping a quick option sets it and closes the sheet immediately (Toast `Reminder set for {when}.`).
- **Pick date & time:** the native date picker, then a time picker with keyboard input as the default and a dial toggle (Android: `@react-native-community/datetimepicker`).
- **Repeat:** a sub-sheet: None, Daily, Weekdays, Weekly, Monthly, Custom… (Custom = every N days/weeks, on chosen weekdays).
- **Important:** a Switch. When on, the sheet shows `Keeps reminding until done.` (REM-07).
- **Web/ext:** a `Popover` anchored to the trigger (width 320) with the same order: 4 quick options as a list with their times on the right (`kbd` 1–4), "Pick date & time" (shadcn `Calendar` + a time input), Repeat (`Select`), Important (`Switch`), Remove.
- **Permission primer:** the first time a reminder is set on mobile, the sheet is followed by the in-context notification primer (D-52, §6).
- **Mapping:** Mobile BottomSheet + `Button` + `Switch` (RN Reusables) + datetimepicker. Web/ext `Popover`, `Calendar`, `Select`, `Switch`.

### 5.23 GroupPicker
- **Purpose:** move a note to a group (ORG-05).
- **Mobile (BottomSheet):** a search Input `Find or create group` at the top, then rows: a group-colour dot + name + count. The current group shows a `check`. The last row is `None (Inbox / ungrouped)`. Typing a new name shows `Create "{name}"` as the first row.
- **Web/ext:** a `Popover` with a `Command` list (same content), opened by `M`.
- **Behaviour:** choosing one moves the note immediately → Toast `Moved to {group}.` + Undo.
- **Create:** opens an inline colour row (the 8 group-colour dots), default `group-indigo`.
- **Mapping:** Mobile BottomSheet + `Input` + list. Web/ext `Popover` + `Command`.

### 5.24 NumberApproval
- **Purpose:** password-less sign-in with number matching (SEC-01, SEC-02).
- **Phone side** (full screen, opened from a high-priority notification, `FLAG_SECURE`):
  ```
  ┌──────────────────────────────────────────┐
  │  [x]                                     │
  │           shield icon (48, primary)      │
  │          Sign-in request                 │
  │                                          │
  │   laptop  Chrome on Windows              │
  │   globe   Ahmedabad, IN   ← example only │
  │   clock   Today 10:42 AM                 │
  │                                          │
  │   Select the number shown on your        │
  │   other screen.                          │
  │                                          │
  │    ┌──────┐   ┌──────┐   ┌──────┐        │
  │    │  17  │   │  42  │   │  88  │        │  ← 3 buttons ≥ 72×72, mono 28 semibold
  │    └──────┘   └──────┘   └──────┘        │
  │                                          │
  │            This wasn't me                │  ← ghost button, destructive text
  └──────────────────────────────────────────┘
  ```
  - The device, location and time rows are `body`, with a 20 muted icon each. The location is whatever the API sends (`{city}, {country}`). **Never hardcode a city.**
  - Number buttons: ≥ **72 × 72** (min 64), radius `lg`, 1 px `border`, `card` bg. Number in monospace 28/34 semibold.
  - After a tap → BiometricPrompt ("Confirm sign-in") → success: check animation + `Signed in.` + auto-close. Wrong number → error haptic + `That number didn't match. Sign-in was blocked.`
  - "This wasn't me" denies the request and shows `Request denied. Consider reviewing your devices.` + a link to Devices.
  - **Expired:** `This request expired.` + Close.
- **Web/ext side** (login page, or an extension tab page):
  ```
            [Cue mark] Cue
       The right note at the right moment.

     Username  [ yagnik            ]  [Continue]

  ── after Continue ─────────────────────────────
              Check your phone and select

                     ┌────────┐
                     │   42   │      ← 48 px mono semibold, in a ring
                     └────────┘      ← 120 s countdown ring around it (primary)

               Waiting for approval…  1:54
               Use QR code instead
  ```
  - The countdown ring is a 2 px `primary` stroke that drains clockwise, with the time `small` muted below.
  - On approve → `Signed in.`, redirect to Inbox. On deny/expire → the error message + `Try again`.
- **A11y:** the web announces "Select 42 on your phone" via aria-live. The phone buttons are labelled `Number 42`.
- **Mapping:** Cue components. Mobile `Button` (RN Reusables) + `expo-local-authentication`. Web/ext `Input`, `Button`, an SVG ring.

### 5.25 QRBlock
- **Purpose:** the QR fallback (SEC-02).
- **Web/ext:** a 200 × 200 QR (`qrcode` lib rendering SVG) in a `card` with 16 padding, radius 12, 1 px `border`, **always black on white even in dark mode** (scanners need it).
  - Below it: `Scan with Cue on your phone: Settings → Devices → Scan QR` in `small` muted, and a "Refreshes in 0:42" countdown. It auto-refreshes every 60 s.
  - Link: `Use number instead`.
- **Phone scanner:** a full-screen camera with a 240 × 240 rounded (radius 20) cut-out frame. The scrim outside is `rgba(0,0,0,0.6)`. Text: `Point at the QR code on your computer.` After a scan → the NumberApproval details screen (without numbers) + "Approve" (primary) + "This wasn't me" + the location-mismatch warning, if any, in `important`.
- **Mapping:** Web/ext Cue component (SVG). Mobile `expo-camera` + a NativeWind overlay.

### 5.26 RecoveryCodesGrid
- **Purpose:** show the 10 recovery codes once (SEC-06, onboarding step 2).
- **Anatomy:**
  ```
  Save your recovery codes
  If you lose this phone, a code is the only way back
  into your account. Each code works once.

  ┌───────────────────┬───────────────────┐
  │ ABCD-EFGH-IJKL    │ MNOP-QRST-UVWX    │   ← mono 16, tabular, 2 columns, 5 rows
  │ ...               │ ...               │      card bg, radius lg, padding 16
  └───────────────────┴───────────────────┘
  [ Copy all ]  [ Save file ]  [ Share ]      ← secondary buttons
  [ ] I saved these codes                     ← enabled after any of the 3 buttons is used
  Type one code to confirm  [__________]      ← appears after the checkbox
  [ Continue ]                                ← primary, disabled until the typed code matches
  ```
- **Behaviour:**
  - "Save file" writes `cue-recovery-codes.txt`.
  - "Copy all" copies the codes newline-separated, and a Toast says `Copied.`
  - **Screenshots blocked** (`FLAG_SECURE`).
  - Codes are never shown again after Continue.
- **Mapping:** Cue component using `Button`, `Checkbox`, `Input`.

### 5.27 SettingsRow and Switch
- **Purpose:** all settings screens.
- **SettingsRow:**
  - Mobile: height ≥ 56, padding 16, a 20 muted icon + label `body` + optional description `caption` muted (max 2 lines), then a trailing element.
  - Web: height 48, `ui`.
- **Trailing elements:**
  - **Switch** (toggle)
  - **value + chevron** (`Asia/Kolkata  >`)
  - **Button** (`Revoke`, outline sm)
  - nothing (navigational rows always show a chevron)
- **Groups:** headed by SectionHeader, separated by 24, rows divided by a 1 px `border` inset.
- **Switch:** a 52 × 32 track (mobile) / 36 × 20 (web). On = `primary` track, white thumb; off = `muted` track with a `border`. The whole row is the touch target.
- **Destructive rows** (Sign out, Delete account later): text in `destructive`, no icon colour change, always followed by a Dialog.
- **Mapping:** Web/ext `Switch`, `Separator`, `Button`. Mobile `Switch` (RN Reusables) + a custom row.

### 5.28 CountBadge
- **Purpose:** a small count on the Inbox tab, sidebar items and the ext toolbar badge.
- **Look:**
  - Min width 18, height 18, radius full, padding-x 5, `micro` (11) semibold, tabular.
  - Tab / nav badges: `primary` bg + `primary-foreground`.
  - Sidebar counts: plain muted text, no pill.
- **Rules:**
  - **Hidden at 0** (never show "0").
  - `99+` above 99.
  - **Ext toolbar badge:** text `1`–`99`, then `99+`. Background `#4F46E5`, text white. Tooltip `Cue: {n} notes on this page`.
- **Mapping:** Web/ext `Badge`. Mobile custom `View` + `Text`. Ext: `chrome.action.setBadgeText` / `setBadgeBackgroundColor`.

### 5.29 ConflictCompare
- **Purpose:** resolve "edited on two devices" (DATA-01).
- **Layout:**
  - **Web:** two columns 50/50 with a 1 px `border` between, max width 960, in the detail panel expanded to full page.
  - **Mobile:** stacked, with segmented tabs `This device` | `Other device` above a single text view (to save space) **plus** a toggle "Show both" that stacks them.
- **Column header:** `This device · edited 10:42 AM` / `Other device · edited 10:40 AM` (`label` / `uiStrong`, muted time). The text below is read-only, `noteBody` / `body`.
- **Actions (bottom bar):**
  - `Keep this` (primary)
  - `Keep other` (secondary)
  - `Merge` (ghost): opens the editor with both texts concatenated, separated by `\n\n---\n\n`, to edit manually
- **Behaviour:** keeping one soft-deletes the other (it goes to Trash, restorable) → Toast `Kept this version.` + Undo.
- **Mapping:** Cue component using `Button`, `Tabs` (mobile segmented), `ScrollArea`.

---

## 6. Mobile app (Android)

> Platform: React Native (Expo SDK 57) + NativeWind + React Native Reusables (see §5 and §11). Every screen below must
> also pass the §9 states checklist and the §10 accessibility checklist.
>
> **Golden rule for mobile:** one main thing per screen. If you're unsure whether to show something, **don't show it**.
> Put it behind ⋮, long-press, or a sheet.
>
> Wireframe notes:
> - The wireframes are about 34 characters wide.
> - `( + )` = FAB, `[ ]` = button, `○` = round checkbox, `▸` = chevron, `⋮` = overflow menu.
> - Icon names come from §3.9.

### 6.1 Navigation map

```
                       ┌──────────────────────────────┐
  App start ─────────▶ │  (tabs)  Inbox │ Todos │ Notes │ ◀── default: Inbox
                       └──────┬─────────┬───────┬──────┘
            FAB (+) on all ──▶│ /capture (full screen)
            tap row ─────────▶│ /note/[id]  ─▶ ⋮ menu / sheets
            top-bar 🔍 ──────▶│ /search
            top-bar avatar ──▶│ /settings  ─▶ /settings/devices, /settings/device/[id],
                              │               /settings/reminders, /settings/security,
                              │               /settings/vault, /settings/appearance, …
            Todos banner ────▶│ /missed
            notification ────▶│ /approve/[requestId]   (sign-in request)
            Settings ▸ Link ─▶│ /scan                  (QR sign-in)
  First launch ──────────────▶ /onboarding/username → /onboarding/recovery → /onboarding/ready
  First reminder set ────────▶ /permissions/notifications → /permissions/battery
  Widget / QS tile / Share ──▶ /capture?source=widget|tile|share
  Escalated reminder ────────▶ AlarmActivity (native screen, 6.20)
```

**Rules:**
- Only **3 tabs**. Never add a 4th in MVP. (Future "Chat" would be the 4th, D-24; leave nothing visible for it.)
- **Settings is never a tab.** It's reached only from the top-bar avatar.
- Every screen other than the tabs uses a back arrow (top-left). The system back and predictive back do the same thing.
- Modals and sheets close with a swipe down, the scrim tap, or back.

### 6.2 App shell (top bar, bottom nav, FAB)

**Purpose:** a consistent frame on the 3 tab screens. (PLAT-02)

```
┌──────────────────────────────────┐
│ Inbox                    🔍  (V) │  ← TopBar
│──────────────────────────────────│
│                                  │
│        (screen content)          │
│                                  │
│                                  │
│                           ( + )  │  ← FAB, 16 dp from right/bottom edge of content
│──────────────────────────────────│
│  📥 Inbox ³    ✅ Todos   📓 Notes │  ← BottomNav
└──────────────────────────────────┘
```

**TopBar (tab screens):**
| Element | Spec |
|---|---|
| Title | The screen name ("Inbox", "Todos", "Notes"), type `title` (§3.2), colour `foreground`. Left-aligned, 16 dp start padding. It doesn't collapse into a large header. |
| Search | IconButton `search` (§3.9), 48 dp target. Opens `/search`. a11y label "Search". |
| Avatar | A 32 dp circle with the user's first initial in `primary-foreground` on `primary`. 48 dp target. Opens `/settings`. a11y label "Settings". |
| Height | 56 dp + the status-bar inset (edge-to-edge). Background `background`. A 1 px bottom border (`border`) appears only when the content is scrolled. |

**BottomNav:**
| Element | Spec |
|---|---|
| Items | Exactly 3, always labelled: **Inbox** (`inbox`), **Todos** (`list-checks`), **Notes** (`notebook`). |
| Active item | Icon + label in `primary`, with a pill indicator behind the icon (64×32 dp, `accent` background). |
| Inactive item | Icon + label in `muted-foreground`. |
| Badge | A CountBadge **only on Inbox**, showing the item count (1–99, then "99+"). It's hidden at 0. |
| Height | 64 dp + the navigation-bar inset. Background `card`, top border `border`. |
| Behaviour | Tapping the active tab scrolls its list to the top. |

**FAB:**
| Element | Spec |
|---|---|
| Look | A 56 dp circle, `primary` background, `plus` icon 24 dp in `primary-foreground`, shadow `md` (§3.5). |
| Position | Bottom-right, 16 dp from the right edge, 16 dp above the BottomNav. It stays above Toasts: a Toast shifts up above the FAB. |
| Meaning | **Only Capture.** It opens `/capture?source=app`. No speed-dial or menu. |
| Motion | Press → scale 0.94 (expressive moment, §1). Reduce-motion → no scale. |
| Hidden | It's hidden in multi-select mode and while the keyboard is open. |
| a11y | Label "New note". |

**Pull to refresh:** every tab list supports pull-to-refresh, which runs `sync()`. Use a small spinner in `primary`, with no text.

**Offline indicator:** when the device is offline, a 6 dp `muted-foreground` dot sits next to the TopBar title, and long-pressing it shows "Offline. Changes will sync later." No banner: offline is normal on mobile (CAP-09).

Patterns borrowed from: Material 3 navigation bar (3 labelled tabs), Things 3 (one Magic Plus button).

### 6.3 Inbox

**Purpose:** everything captured lands here; the user triages it with one swipe. Route `/(tabs)/inbox`. Features CAP-01, ORG-04, CAP-10, PLAT-01.

```
┌──────────────────────────────────┐
│ Inbox                    🔍  (V) │
│──────────────────────────────────│
│ Pay electricity bill             │
│ ⏰ Today 6:00 PM                 │
│                          2m ago  │
│──────────────────────────────────│
│ Call Ravi about the flat deposit │
│ and the agreement copy           │
│ 📱 WhatsApp              1h ago  │
│──────────────────────────────────│
│ Book Diwali train tickets        │
│ 🌐 irctc.co.in         Yesterday │
│──────────────────────────────────│
│ 🔒 Locked note        Mon 9:12 AM│
│──────────────────────────────────│
│                           ( + )  │
│──────────────────────────────────│
│  📥 Inbox ⁴    ✅ Todos   📓 Notes │
└──────────────────────────────────┘
```

**Swipe state (in progress, right swipe past 40%):**
```
│██ ✓ Make todo ████│Pay electrici…│   ← background `swipe-todo`, white icon+label
```
**Swipe state (left swipe past 40%):**
```
│…lectricity bill│████ Archive 🗄 ██│   ← background `swipe-archive`, white icon+label
```

**NoteRow (Inbox variant):**
| Part | Spec |
|---|---|
| Title | The first line of the note, max **2 lines**, ellipsis. Type `body` (§3.2), `foreground`. |
| Meta line | Up to **2 chips** in this priority: a reminder chip (if any), then a source chip (`source_kind` icon + app/site name) or a group chip. Type `caption`. |
| Time | Relative time on the right of the meta line, `caption`, `muted-foreground` (§4 date rules). |
| Important | A small `flag` icon (16 dp, `important`) before the title. |
| Todo | Inbox items that are already todos show a round Checkbox on the left (rare; most become todos when they leave the Inbox). |
| Locked | Lock icon + "Locked note" + time. No preview. |
| Conflict | A `copy` icon (16 dp, `destructive`) before the title. |
| Spacing | Row padding 12 dp vertical, 16 dp horizontal, min height 64 dp. A divider (1 px `border`) inset 16 dp. |
| Tap | Opens `/note/[id]`. |

**Interactions:**
- **SwipeRow right → Make todo:**
  - Background `swipe-todo`, icon `list-checks` + label "Make todo".
  - Commit at 40% of the row width. A haptic `selection` fires when the threshold is crossed (§3.10).
  - On commit the row slides out and a Toast says "Moved to Todos · Undo".
- **SwipeRow left → Archive:** background `swipe-archive`, icon `archive` + label "Archive". Toast "Archived · Undo".
- **Long-press (400 ms):** haptic `impactMedium`, then the **Actions sheet** opens (6.10).
- **Select mode:** from the Actions sheet → "Select".
  - The TopBar becomes a contextual bar: `✕   3 selected   [Todo] [Archive] ⋮`.
  - Checkboxes appear on every row, the FAB hides, and the BottomNav stays.
  - ⋮ holds Move to group…, Remind me…, Delete.
  - Back or ✕ exits select mode.
- **Toast Undo:** shown 5 s, one at a time, positioned above the FAB. Undo restores the item in place.

**States:**
- **Empty:**
  ```
  │            ( inbox icon )         │
  │           Inbox clear.            │
  │ Capture anything with the + button.│
  ```
  EmptyState: icon `inbox` 48 dp `muted-foreground`, title `title`, body `body` `muted-foreground`. Nothing else, no buttons (the FAB is the action).
- **Loading (first run only):** 4 Skeleton rows. After the first sync, the local DB loads instantly, so there's never a skeleton again.
- **Error:** none. The data is local. Sync errors show only in Settings → "Sync" status.
- **Offline:** works normally (CAP-09). Only the offline dot (6.2).

Patterns borrowed from: Superhuman / Things (inbox zero), Todoist (swipe actions), Material 3 (snackbar Undo).

### 6.4 Capture screen

**Purpose:** thought → saved in under 2 seconds. Route `/capture?source=app|widget|tile|share&voice=0|1`. Features CAP-01, CAP-02, CAP-03, CAP-04, CAP-05, CAP-10, REM-01, PLAT-01, SEC-04 AC2.

```
┌──────────────────────────────────┐
│ ✕                                │  ← close (= save)
│                                  │
│ Pay electricity bill tomorrow 6pm│
│                    ‾‾‾‾‾‾‾‾‾‾‾‾‾‾│  ← detected date highlighted
│ ┌──────────────────────┐         │
│ │ ⏰ Tue 21 Oct, 6:00 PM ✕│        │  ← reminder preview Chip
│ └──────────────────────┘         │
│                                  │
│                                  │
│──────────────────────────────────│
│ 🎤   🔔   ☑ Todo            [ ✓ ]│  ← keyboard accessory toolbar
│──────────────────────────────────│
│ q  w  e  r  t  y  u  i  o  p     │  ← system keyboard (already open)
└──────────────────────────────────┘
```

**Opened from Share (pre-filled):**
```
│ ✕                                │
│ https://irctc.co.in/booking…     │
│ ┌────────────────┐               │
│ │ 🌐 Chrome     ✕ │               │  ← source Chip (CAP-10), removable
│ └────────────────┘               │
```

**Elements:**
| Element | Spec |
|---|---|
| Close ✕ | Top-left IconButton `x`, a11y "Close and save". Same as back: **saves** if the text is non-empty. |
| CaptureField | Full-width multiline, **autoFocus**, placeholder "What's on your mind?", type `body-lg` (§3.2: 18/28), no border, no title field. It grows to fill the screen. |
| Date highlight | When `detectReminder()` finds a date, the matched words get an `accent` background and `accent-foreground` text (radius sm). Tapping the highlight removes the detection (it stays plain text). |
| Reminder preview Chip | Shown under the text when a date is detected: `bell` icon + formatted date (§4). ✕ removes it. Tapping the chip opens the ReminderPicker pre-filled. |
| Source Chip | Only when `source=share`: the app name or URL host, ✕ removes it. |
| Toolbar | Sits on top of the keyboard, height 48 dp, background `card`, top border `border`. |
| 🎤 Mic | IconButton `mic`. **Hold to talk**: while held, the icon is `primary` with a pulsing ring (reduce-motion: static ring) and the live transcript streams into the field. Releasing stops. A single tap shows the Toast "Hold to talk". |
| 🔔 Bell | IconButton `bell` → ReminderPicker sheet (6.9). When a reminder is set, the icon is filled `primary`. |
| ☑ Todo | A toggle chip `list-checks` + "Todo". When ON it's `accent` background and `primary` text, and the item saves as a todo (`is_todo = 1`). |
| ✓ Save | Right side, a 40 dp circle in `primary` with a `check` icon. It's disabled (`muted`) when the text is empty. Saves and closes. |

**Behaviour:**
- **The keyboard is up on first frame.** Don't import sync or search on this screen (PLAT-01, M2-T02).
- **Save happens on:**
  - ✓
  - ✕
  - system back or predictive back
  - the app going to background

  Empty text → discard silently, with no Toast.
- **After saving:**
  - Close the screen.
  - Show the Toast **"Saved to Inbox · Undo"**, or "Saved to Todos · Undo" if the Todo toggle is ON.
  - Haptic `success`.
- **`voice=1`:** start listening immediately (as if the mic were held). Show a hint above the toolbar, "Listening… tap to stop". In this mode a tap stops listening.
- **App lock ON (SEC-04):** the capture screen opens and saves **without unlocking**. After saving, the Toast says "Saved". It shows no other data and doesn't navigate into the app.
- **Mic permission:** on the first mic hold, explain in a BottomSheet: "Cue turns your voice into text on this phone." → [Continue] → system prompt. If denied: Toast "Microphone is off. You can allow it in Settings."

**States:**
- **Offline:** works (it saves locally).
- **Error saving:** this should never happen locally. If it does: Toast "Couldn't save. Your text is still here." and stay on the screen.

Patterns borrowed from: Drafts (open to a blank page), Todoist (inline date highlight), Things (one capture button).

### 6.5 Todos

**Purpose:** what to do now and next. Route `/(tabs)/todos`. Features ORG-01, ORG-02, REM-09, REM-08 AC4.

```
┌──────────────────────────────────┐
│ Todos                    🔍  (V) │
│──────────────────────────────────│
│ ┌──────────────────────────────┐ │
│ │ 🔕 2 missed reminders · View  │ │  ← Banner (only when any)
│ └──────────────────────────────┘ │
│ OVERDUE                Reschedule│
│ ○ Pay electricity bill           │
│   ⏰ Yesterday 6:00 PM  (red)    │
│ TODAY                            │
│ ○ Call Ravi about deposit        │
│   ⏰ 4:30 PM · Home              │
│ ○ 🚩 Submit GST return           │
│   ⏰ 5:00 PM                     │
│ UPCOMING                         │
│ ○ Book Diwali train tickets      │
│   ⏰ Fri 24 Oct                  │
│ NO DATE                          │
│ ○ Buy new phone cover            │
│                                  │
│ ▸ Completed (5)                  │
│                           ( + )  │
│──────────────────────────────────│
│  📥 Inbox ⁴    ✅ Todos   📓 Notes │
└──────────────────────────────────┘
```

**Elements:**
| Element | Spec |
|---|---|
| Missed Banner | Shown only when missed reminders exist. Banner variant `info`: `bell-off` icon + "{n} missed reminder(s)" + text button "View" → `/missed`. Not dismissible (it disappears when the list is empty). |
| SectionHeader | "OVERDUE", "TODAY", "UPCOMING", "NO DATE". Type `overline` (§3.2: 12/16, semibold, letter-spacing 0.5), `muted-foreground`. Sticky while scrolling. A section is **hidden when empty**. |
| Reschedule | Text button on the OVERDUE header, `primary`. It opens the ReminderPicker for **all** overdue todos at once. The Toast reads "Rescheduled 2 todos · Undo". |
| TodoRow | A round **Checkbox** (24 dp visual, 48 dp target) on the left, title (2 lines max) `body`, and a meta line with a reminder chip + group chip (max 2). An important todo shows `flag` in `important` before the title. |
| Overdue time | The reminder chip text uses colour `overdue`. **The colour plus the word ("Yesterday") must both show it's overdue**: colour is never the only signal. |
| Completed | A collapsed row "▸ Completed (n)". Tapping expands it to the last 20 completed, shown struck through in `muted-foreground`. |

**Interactions:**
- **Checking the box:**
  1. Checkbox fill animation 200 ms + haptic `success`.
  2. The title is struck through.
  3. After **1 s** the row collapses out.
  4. Toast "Done · Undo".
- **SwipeRow:**
  - Right = **Done**: background `success`, icon `check`, label "Done".
  - Left = **Archive**: as in Inbox.
- **Tap a row** → `/note/[id]`. **Long-press** → Actions sheet (6.10) without "Make todo" (it shows "Make note" instead).

**States:**
- **Empty (no todos at all):** EmptyState with icon `list-checks`, "No todos yet.", "Swipe right on an Inbox item to make it a todo."
- **All done today:** under TODAY show a single line "All done for today 🎉" in `muted-foreground` (an expressive moment).

Patterns borrowed from: Todoist (Overdue + Reschedule, Today/Upcoming), Things (collapsed logbook), Microsoft To Do (round checkbox).

### 6.6 Notes

**Purpose:** everything kept, organised by groups. Route `/(tabs)/notes`, with filters via query `?group=<id>|pinned|archive|trash`. Features ORG-05, ORG-06, DATA-02.

```
┌──────────────────────────────────┐
│ Notes                🔍  ⋮  (V)  │
│──────────────────────────────────│
│ (All) (Pinned) (Home) (Work) (+ G│  ← chip filter row, horizontal scroll
│──────────────────────────────────│
│ 📌 Wi-Fi password for Papa's flat │
│    Home               3 Oct      │
│──────────────────────────────────│
│ Ideas for the weekend trip       │
│ Lonavala or Alibaug?   Yesterday │
│──────────────────────────────────│
│ Meeting notes – sprint review    │
│ Work                   Mon       │
│──────────────────────────────────│
│                           ( + )  │
│──────────────────────────────────│
│  📥 Inbox ⁴    ✅ Todos   📓 Notes │
└──────────────────────────────────┘
```

**Elements:**
| Element | Spec |
|---|---|
| ⋮ (TopBar) | A menu: **Archive**, **Trash**, **Manage groups**. These open the same screen filtered (`?group=archive` / `?group=trash`) or `/settings/groups`. |
| Filter chips | Chip variant `filter`, single-select: **All** (default), **Pinned**, then one chip per group (alphabetical), then "**+ Group**" (opens the GroupPicker in create mode). Horizontal scroll, 8 dp gaps, 16 dp side padding. |
| Rows | NoteRow (same as Inbox), but the meta line shows the **group chip** (when the filter is All) + a short body preview (1 line) + date. Pinned rows show `pin` before the title and always sort first. |
| Swipes | **The same as Inbox** (right = Make todo, left = Archive), for consistency. |

**Archive view** (`?group=archive`): TopBar title "Archive" with a back arrow. Rows swipe left = **Delete** (`destructive`) and right = **Unarchive**. Empty: "Nothing archived."

**Trash view** (`?group=trash`):
```
│ ← Trash                          │
│ Notes in Trash are deleted after │
│ 30 days.                         │
│ Old shopping list      [Restore] │
│ Deleted 3 days ago               │
│        [ Empty Trash ] (red text)│
```
- Each row has a "Restore" text button. Long-press → "Delete forever".
- **"Empty Trash"** is the **one place that uses a confirm Dialog**, because it's irreversible: "Delete 4 notes forever? This can't be undone." [Cancel] [Delete forever (destructive)].
- Empty: "Trash is empty."

**States:**
- **Empty (no notes at all):** EmptyState with icon `notebook`, "No notes yet.", "Everything you capture and keep shows up here."
- **Empty group:** "Nothing in Home yet."

Patterns borrowed from: Google Keep (filter by label), Bear (pinned first).

### 6.7 Note editor

**Purpose:** read and edit one note or todo, and set everything about it. Route `/note/[id]`. Features ORG-01, ORG-02, ORG-05, ORG-06, REM-02, REM-07, CAP-10, DATA-01, SEC-07.

```
┌──────────────────────────────────┐
│ ←              ☑ Todo   🚩   ⋮   │
│──────────────────────────────────│
│ Call Ravi about deposit          │  ← title (first line), headline
│ (⏰ Today 4:30 PM) (Home) (#flat) │  ← metadata chips
│ (📱 WhatsApp)                     │
│                                  │
│ He said the owner wants two      │
│ months' deposit. Ask if 1.5 is   │
│ okay and get the agreement copy. │
│ ☐ Ask about deposit              │
│ ☑ Get agreement copy             │
│                                  │
│──────────────────────────────────│
│ ☐ Checklist   🔔 Remind   🎤      │  ← keyboard toolbar (only while typing)
└──────────────────────────────────┘
```

**TopBar (editor):**
| Element | Spec |
|---|---|
| ← Back | Saves (already auto-saved) and returns. |
| ☑ Todo | A toggle chip "Todo". ON = `accent`. Turning it ON makes this a todo (ORG-01), which shows a Checkbox before the title. Toast "Moved to Todos". |
| 🚩 Important | IconButton `flag`. ON = filled `important`. a11y "Mark important". If it has a reminder, Important makes it nag (REM-07); the first time, show the Toast "Important reminders repeat until done." |
| ⋮ Menu | Move to group…, Tags…, Pin / Unpin, Lock / Unlock, Archive, Delete, Show source (only if a source exists). Delete → Trash + Toast "Moved to Trash · Undo". |

**Body:**
| Element | Spec |
|---|---|
| Title | The **first line of the text** is shown as the title: type `headline` (§3.2: 22/30, semibold). There's no separate title input. |
| Metadata chips | Under the title, wrapping, 8 dp gaps:<br>• reminder chip (tap → ReminderPicker)<br>• group chip (tap → GroupPicker)<br>• tag chips `#flat` (tap → Tags sheet)<br>• source chip (tap → open the URL in the browser / open the app)<br>No chips = no row. A **"+ Add"** chip at the end opens a small sheet: Reminder, Group, Tag. |
| Done checkbox | For todos only: a Checkbox before the title. Checking it → title struck through + Toast "Done · Undo", and the user stays on the screen. |
| Body | Plain text, type `body-lg` (18/28) for reading comfort. A line starting with `[ ] ` / `[x] ` renders as a checklist item with a tappable box. |
| Toolbar | Shown only while the keyboard is open:<br>• **☐ Checklist**: turns the current line into a checklist line<br>• **🔔 Remind**: ReminderPicker<br>• **🎤**: hold to dictate at the cursor |
| Auto-save | 600 ms after the last keystroke (debounced) → `localWrite`. No save button. |

**States:**
- **Conflict (DATA-01):** a Banner `warning` at the top: "Edited on two devices: 2 versions · Compare" → ConflictCompare (6.17).
- **Locked note:** opening needs biometric (6.18). After 5 min, or when the app goes to background, it re-locks and the body is replaced by "🔒 Locked · Tap to unlock".
- **In Trash:** a read-only body with the Banner "This note is in Trash. [Restore]".
- **Deleted on another device while open:** Banner "This note was deleted on another device. [Restore] [Close]".
- **Offline:** normal.

Patterns borrowed from: Bear / Tot (plain text, minimal toolbar), Superlist (note ↔ todo in one tap), Things (metadata under the title).

### 6.8 Search

**Purpose:** find anything by any word, offline. Route `/search`. Feature ORG-07.

```
┌──────────────────────────────────┐
│ ← ┌────────────────────────────┐ │
│   │ 🔍 elec                 ✕  │ │  ← SearchField, autofocus
│   └────────────────────────────┘ │
│ TODOS                            │
│ ○ Pay **elec**tricity bill       │
│   ⏰ Today 6:00 PM               │
│ NOTES                            │
│ Meter reading – **elec**tricity  │
│ board visit…           12 Sep    │
└──────────────────────────────────┘
```

| Element | Spec |
|---|---|
| SearchField | Full width, autofocus, placeholder "Search notes and todos", ✕ clears. A search runs 150 ms after typing stops (debounced). |
| Results | Grouped under SectionHeaders **TODOS**, then **NOTES** (each hidden if empty), max 20 per group plus "Show all (n)". Matched text is **bold + `accent` background**. |
| Rows | The normal TodoRow / NoteRow. Tap → `/note/[id]`. |
| Recent | When the query is empty: SectionHeader "RECENT" + the last 5 searches (tap to reuse, ✕ to remove). If there are none, show nothing. |
| Locked notes | Not searched unless the vault is unlocked. While it's locked, the footer says "Locked notes aren't searched." |
| No results | "No matches for "{q}"." in `muted-foreground`. |
| Offline | Works (FTS5 on the device). |

### 6.9 Reminder sheet

**Purpose:** set when something comes back. A BottomSheet holding the **ReminderPicker**. Opened from capture, the editor, the Actions sheet, and Reschedule. Features REM-01, REM-02, REM-07, D-28.

```
┌──────────────────────────────────┐
│            ───                   │  ← drag handle
│ Remind me                        │
│ ┌────────────┐ ┌────────────────┐│
│ │ Later today│ │ This evening   ││
│ │  9:30 PM   │ │  7:00 PM       ││
│ └────────────┘ └────────────────┘│
│ ┌────────────┐ ┌────────────────┐│
│ │ Tomorrow   │ │ Next week      ││
│ │  9:00 AM   │ │  Mon 9:00 AM   ││
│ └────────────┘ └────────────────┘│
│ 📅 Pick date & time            ▸ │
│ 🔁 Repeat                Never ▸ │
│ 🚩 Important (repeat until done)⬤│  ← Switch
│──────────────────────────────────│
│ [Remove reminder]       [ Set ]  │
└──────────────────────────────────┘
```

| Element | Spec |
|---|---|
| Quick chips | A 2×2 grid of large chips (min height 64 dp), each with a label + the computed time in `caption`:<br>• **Later today** = now + 3 h, rounded to :00 or :30; hidden after 9 PM<br>• **This evening** = 7:00 PM; hidden after 6 PM, when it's replaced by **Tomorrow evening** at 7:00 PM<br>• **Tomorrow** = 9:00 AM<br>• **Next week** = next Monday 9:00 AM<br>All times are in the user's time zone (default IST, D-28). Tapping a chip **selects** it (`accent` fill); it doesn't save yet. |
| Pick date & time | A row → the system date picker, then the system time picker (12-hour, follows the device setting). After that, the row shows the chosen value. |
| Repeat | A row → a sub-sheet: **Never**, **Daily**, **Weekdays (Mon–Fri)**, **Weekly on {day}**, **Monthly on the {n}th**, **Custom…** (every N days/weeks). The selected value shows on the right. |
| Important | A Switch row with the helper text "Repeats every 10 min until done". It reflects and sets the note's `important` flag (REM-07). |
| Footer | **Set** (Button primary, disabled until a time is chosen) and **Remove reminder** (text button `destructive`, shown only when editing an existing one). |
| After Set | The sheet closes. The source screen shows the reminder chip. Toast "Reminder set for Tue 6:00 PM". |
| Pre-filled | When opened from a detected date, that date is pre-selected under "Pick date & time". |

**First reminder ever:** after Set, if the notification permission isn't granted yet, open the **Permission primer** (6.14) before saving the system alarm. The reminder is saved either way.

Patterns borrowed from: Google Keep (quick picks), TickTick (constant reminder / Important).

### 6.10 Actions sheet and group picker

**Actions sheet:** a BottomSheet opened by long-press on any row (or ⋮ in select mode). It's the **visible alternative to every gesture** (§1, 6.21).

```
┌──────────────────────────────────┐
│            ───                   │
│ Pay electricity bill             │  ← item title (1 line)
│ ☑  Make todo                     │
│ 🔔 Remind me…                    │
│ 📁 Move to group…                │
│ 🚩 Mark important                │
│ 📌 Pin                           │
│ 🔒 Lock                          │
│ 🗄  Archive                       │
│ 🗑  Delete                (red)   │
│ ☐  Select                        │
└──────────────────────────────────┘
```
- Rows are 56 dp high, icon 20 dp `muted-foreground`, label `body`.
- Labels toggle with state: "Make note" / "Make todo", "Unpin", "Remove important", "Unlock".
- "Delete" uses `destructive` text → Trash + Toast Undo (no confirm).
- "Select" enters multi-select mode.
- The order is fixed as above; hidden items depend on the context (e.g. no "Make todo" in Todos).

**GroupPicker:** a BottomSheet.
```
│ Move to group                    │
│ 🔍 Find or create group          │
│ ○ No group                       │
│ ● Home                           │
│ ○ Work                           │
│ ○ Shopping                       │
│ + Create "Bills"                 │  ← appears when the typed name doesn't exist
```
- Single choice with radio rows. Choosing closes the sheet immediately + Toast "Moved to Home".
- Search filters the list. "+ Create" makes a new group (the name is trimmed, 1–30 characters) and selects it.

**Tags sheet** (from ⋮ → Tags): the same pattern but **multi-select** with checkboxes + a "Done" button. Tags are secondary (§1), so they're **never** shown in the filter row.

### 6.11 Approve sign-in

**Purpose:** approve a sign-in on the web or extension with number matching and biometric. Route `/approve/[requestId]`. Feature SEC-01. The screen uses **FLAG_SECURE** (no screenshots).

**Entry:** a high-priority notification on the "Sign-in requests" channel:
- Title: "Sign-in request"
- Text: "Chrome on Windows · Ahmedabad, IN"
- Tapping it opens this screen. If app lock is on, it unlocks first.

```
┌──────────────────────────────────┐
│ ✕                       1:54 left│  ← countdown (120 s)
│                                  │
│      🖥  Sign in to Cue?          │
│                                  │
│  Chrome on Windows               │
│  Near Ahmedabad, IN              │
│  Today 10:42 AM                  │
│                                  │
│  Select the number shown on      │
│  your computer                   │
│                                  │
│  ┌──────┐  ┌──────┐  ┌──────┐   │
│  │  17  │  │  42  │  │  88  │   │  ← NumberApproval, each ≥ 64 dp
│  └──────┘  └──────┘  └──────┘   │
│                                  │
│        This wasn't me            │  ← destructive text button
└──────────────────────────────────┘
```

| Element | Spec |
|---|---|
| Countdown | Top-right, `caption`, `muted-foreground`, format "m:ss left". |
| Header | Icon `monitor` (or `puzzle` for the extension) 32 dp, then the title "Sign in to Cue?" in `headline`. |
| Details | 3 lines in `body`, `muted-foreground`: device + browser ("Chrome on Windows" / "Cue extension · Chrome"), approximate place ("Near {city}, {country}"), time. |
| Instruction | "Select the number shown on your computer", `body`, `foreground`. |
| NumberApproval | 3 square buttons, min 64×64 dp (use 88×72), number in `display` type (32/40, semibold), outline style (`border`), radius lg. Order is random; exactly one is correct. |
| This wasn't me | A text button in `destructive`, centred, 48 dp target. |

**Flow:**
1. The user taps a number. Haptic `impactLight`, then the button fills `primary`.
2. A BiometricPrompt opens with title "Confirm it's you" and subtitle "Approve sign-in on Chrome on Windows". It allows device-credential fallback.
3. Success leads to the result screen: a ✓ in `success`, "Signed in. You can go back to your computer.", and a [Done] button.
4. A wrong number leads to the **blocked** result screen:
   ```
   │        ⛔                         │
   │   Sign-in blocked                │
   │ If this wasn't you, nothing else │
   │ is needed. Your account is safe. │
   │            [ Done ]              │
   ```
   It has a `destructive`-tinted icon. All trusted devices get the alert "A sign-in to Cue was blocked."
5. "This wasn't me" blocks the request immediately, with no confirm, and shows the same blocked screen.
6. When the countdown hits 0: "This request expired. Start again on your computer." [Done].
7. If the biometric is cancelled, the user stays on the screen and can try again.

**States:** offline shows "You're offline. Connect to approve." and the numbers are disabled. Already-handled requests show "This request was already handled."

Patterns borrowed from: Microsoft Authenticator (number matching), Google prompt (device / place / time, "No, it's not me").

### 6.12 QR scan

**Purpose:** a fallback sign-in made by scanning the QR code on the computer. Route `/scan`. It's reached from Settings → Devices → "Link a computer" or from the web's "Use QR instead" hint. Feature SEC-02.

```
┌──────────────────────────────────┐
│ ←  Scan to sign in               │
│ ┌──────────────────────────────┐ │
│ │                              │ │
│ │      ┌──────────────┐        │ │  ← camera viewfinder, dimmed outside frame
│ │      │              │        │ │
│ │      │              │        │ │
│ │      └──────────────┘        │ │
│ │                              │ │
│ └──────────────────────────────┘ │
│ Point at the QR code on your     │
│ computer.                        │
└──────────────────────────────────┘
```
- **Camera permission:** ask the first time, with the explanation "Cue uses the camera only to scan sign-in codes." If it's denied, show "Camera is off. [Open settings]".
- **After a valid scan:** haptic `success`, then the same details panel as 6.11 (device, place, time), **without the numbers**, plus a Banner `warning`: **"Only approve if you are looking at this QR on your own screen right now."**
- **Buttons:** [Approve] (primary, opens the BiometricPrompt) and "This wasn't me" (destructive text).
- **Location-mismatch variant:** when the requester's city differs from the phone's, the Banner turns `destructive` with the text "This computer seems to be in Mumbai, IN, but you're in Ahmedabad, IN. Approve only if you're sure."
- **Invalid or expired QR:** Toast "This code expired. Refresh it on your computer."

### 6.13 Onboarding

**Purpose:** create the account in 3 screens, with no permissions asked. Routes `/onboarding/username` → `/onboarding/recovery` → `/onboarding/ready`. Features D-33, SEC-06, D-52.

**Screen 1: Welcome + username**
```
┌──────────────────────────────────┐
│                                  │
│          ( Cue logo )            │
│        Welcome to Cue            │
│ The right note at the right      │
│ moment.                          │
│                                  │
│ Pick a username                  │
│ ┌────────────────────────────┐   │
│ │ vinod                   ✓  │   │
│ └────────────────────────────┘   │
│ 3–20 letters, numbers or _       │
│                                  │
│ [        Continue          ]     │
└──────────────────────────────────┘
```
- **The input:** autofocus, lowercase-only (input is auto-lowercased), no autocorrect. An availability check runs 400 ms after typing stops:
  - ✓ in `success` = "Available"
  - ✗ in `destructive` = "Taken" or "Use 3–20 letters, numbers or _"
- **Continue:** a primary Button, disabled until ✓. It creates the account (`POST /auth/signup`).
- **Offline:** Banner "You need internet to create your account."

**Screen 2: Recovery codes**
```
┌──────────────────────────────────┐
│ Save your recovery codes         │
│ If you lose this phone, these    │
│ codes are the only way back in.  │
│ ┌──────────────────────────────┐ │
│ │ ABCD-EFGH-IJKL  MNOP-QRST-UVWX│ │  ← RecoveryCodesGrid, mono font
│ │ … (10 codes, 2 columns)       │ │
│ └──────────────────────────────┘ │
│ [Copy]   [Save file]   [Share]   │
│ ☐ I saved these somewhere safe   │
│ Type the 3rd code to confirm     │
│ ┌────────────────────────────┐   │
│ │                            │   │
│ └────────────────────────────┘   │
│ [        Continue          ]     │  ← disabled until all 3 done
└──────────────────────────────────┘
```
- **RecoveryCodesGrid:**
  - A 2-column monospace grid, `card` background, `border`.
  - The codes are selectable.
  - FLAG_SECURE is **off** on this screen, so the user may screenshot it.
- **Buttons:**
  - **Copy:** copies to the clipboard and shows Toast "Copied".
  - **Save file:** saves `cue-recovery-codes.txt` through the system file picker.
  - **Share:** opens the Android share sheet.
- **Continue is enabled only when all of these are true:**
  - (a) at least one of Copy / Save / Share was used
  - (b) the checkbox is ticked
  - (c) the re-typed code matches. The code to re-type is a random one (e.g. "Type the 3rd code"); case and dashes are ignored.
- **Back is disabled** on this screen, because the codes are shown only once.

**Screen 3: You're ready**
```
┌──────────────────────────────────┐
│             ✓                    │
│        You're ready              │
│                                  │
│  Tap ( + ) anytime to catch a    │
│  thought. Cue sorts it later.    │
│                                  │
│ [      Start capturing      ]    │
└──────────────────────────────────┘
```
- "Start capturing" opens the Inbox **and then the capture screen immediately**, so the first note gets written.
- **No notification, battery or mic permission requests here** (D-52). Those come later, in context (6.14).

Patterns borrowed from: GitHub/GitLab recovery codes (forced save), "ask in context" from Android permission guidance.

### 6.14 Permission primer and battery guide

**Purpose:** make reminders reliable, asked **only when the user first sets a reminder**. Routes `/permissions/notifications` → `/permissions/battery`. Features REM-04, REM-05, D-46, D-52.

**Step 1: Notifications**
```
┌──────────────────────────────────┐
│             🔔                    │
│ Let Cue remind you               │
│ Cue needs notifications to show  │
│ your reminders on time.          │
│                                  │
│ [      Allow notifications  ]    │
│          Not now                 │
└──────────────────────────────────┘
```
- "Allow notifications" opens the system POST_NOTIFICATIONS prompt, then goes to Step 2.
- "Not now" saves the reminder anyway. Settings then shows the Banner "Notifications are off · Turn on".

**Step 2: Battery guide for the phone's brand** (detected brand: Oppo, Vivo, Realme, OnePlus; others get a generic list)
```
┌──────────────────────────────────┐
│ ←                         Skip   │
│ Make reminders reliable on Oppo  │
│ Oppo phones close apps in the    │
│ background. These 4 steps keep   │
│ your reminders on time.          │
│ ☐ Allow auto-launch   [Open ▸]   │
│ ☐ Battery: Unrestricted [Open ▸] │
│ ☐ Allow background activity      │
│                       [Open ▸]   │
│ ☐ Lock Cue in recent apps        │
│   (open Recents, tap ⋮ on Cue,   │
│    choose Lock)                  │
│ [   Test reminder (10 s)    ]    │
│ [          Done             ]    │
└──────────────────────────────────┘
```
- **Each step row:**
  - A Checkbox the user ticks themselves (the app can't detect most of these), plus an "Open" button that deep-links to the brand's settings screen when possible. Otherwise it opens the app's system settings and shows a Toast describing where to go.
  - "Battery: Unrestricted" uses `REQUEST_IGNORE_BATTERY_OPTIMIZATIONS` when available. Once granted, the box ticks automatically.
- **Test reminder (10 s):** schedules a real alarm in 10 s and shows Toast "Lock your phone. A test reminder arrives in 10 seconds." The notification says "Cue: Test reminder ✓".
- **Done / Skip:** both return to the screen that started the flow. If any step is unticked, Settings → Reminders shows the Banner `warning` **"Reminders may be delayed · Fix"**, which reopens this guide.
- **Brand texts:** use the same 4 steps for all 4 brands. Only the brand name changes, plus the intro "{Brand} phones close apps in the background." The deep-link targets live in a table in the code, built from dontkillmyapp.com.

### 6.15 Settings

**Purpose:** everything that isn't daily use. Route `/settings` (a grouped list) plus sub-routes. Features REM-08, SEC-03, SEC-04, SEC-05, SEC-06, SEC-07, D-28.

```
┌──────────────────────────────────┐
│ ←  Settings                      │
│ ACCOUNT                          │
│ (V) vinod                        │
│ REMINDERS                        │
│ Repeat important every   10 min ▸│
│ Reliability check      ⚠ Fix   ▸ │
│ Send test reminder               │
│ DEVICES & OFFICE HOURS           │
│ This phone (Oppo)      Always  ▸ │
│ Work PC – Chrome  Office hours ▸ │
│ Cue extension     Office hours ▸ │
│ Link a computer (QR)           ▸ │
│ Time zone           India (IST)▸ │
│ SECURITY                         │
│ App lock                    ⬤    │
│ Lock after          5 minutes  ▸ │
│ Authenticator app        Off   ▸ │
│ Recovery codes                 ▸ │
│ Locked notes vault             ▸ │
│ APPEARANCE                       │
│ Theme                  System  ▸ │
│ DATA                             │
│ Export your notes              ▸ │
│ Audio storage        12 / 100 MB │
│ ABOUT                            │
│ Version 1.0.0 · Licenses       ▸ │
│                                  │
│ Sign out of this device  (red)   │
└──────────────────────────────────┘
```

| Section | Rows (SettingsRow / Switch) |
|---|---|
| ACCOUNT | The username with an avatar. Tapping it shows "Username can't be changed in this version." |
| REMINDERS | **Repeat important every**: a picker with 5 / 10 / 15 / 30 min, default 10. **Reliability check**: shows "✓ Good" or "⚠ Fix" and opens 6.14 Step 2. **Send test reminder**: fires in 10 s. |
| DEVICES & OFFICE HOURS | One row per device: name + its schedule mode, opening the schedule editor below. **Link a computer (QR)** opens `/scan`. **Time zone** opens a searchable list, default **India (IST)**. |
| SECURITY | **App lock**: a Switch, then **Lock after** (Immediately / 1 / 5 / 15 min). **Authenticator app**: Off/On; enrolment shows a QR + a 6-digit confirm. **Recovery codes**: "Generate new codes" (needs biometric). **Locked notes vault**: 6.18. |
| APPEARANCE | **Theme**: System / Light / Dark (radio sheet). |
| DATA | **Export your notes**: a sheet saying "Export is done on the web app: Settings → Export. Open cue-api.<you>.workers.dev on your computer." with [Copy link]. **Audio storage**: shows usage, and a row "Delete all kept audio". |
| ABOUT | Version and open-source licenses. |
| Bottom | **Sign out of this device** in `destructive` text. Confirm Dialog: "Sign out? Notes stay safe on your other devices." [Cancel] [Sign out]. |

**Device schedule editor** (`/settings/device/[id]`):
```
│ ←  Work PC – Chrome              │
│ Show reminders on this device    │
│ ( Always ) (●Office hours) (Never)│  ← segmented control
│ Days                             │
│ (Mon)(Tue)(Wed)(Thu)(Fri) Sat Sun│  ← toggle chips, selected = accent
│ From  9:00 AM ▸    To  6:00 PM ▸ │
│ Outside these hours, reminders go│
│ to your phone only.              │
│ Last seen today 10:40 AM · Pune  │
│ [Revoke this device]  (red)      │
```
- **Defaults:** a new computer or extension starts on Office hours, Mon–Fri, 9:00 AM – 6:00 PM. The phone defaults to Always.
- **Revoke:** BiometricPrompt, then a confirm Dialog: "Revoke Work PC – Chrome? It will be signed out right away." [Cancel] [Revoke]. After that, Toast "Device revoked".

**Banners on the Settings home** (top, only when relevant):
- "Notifications are off · Turn on"
- "Reminders may be delayed · Fix"
- "Save your recovery options: set up an authenticator app or a second phone · Set up" (SEC-06 nag)

### 6.16 Missed reminders

**Purpose:** reminders that fired while you couldn't see them, or that went undelivered on a device outside its hours. Route `/missed`. Feature REM-08 AC4.

```
┌──────────────────────────────────┐
│ ←  Missed reminders   Clear all  │
│ Pay electricity bill             │
│ Missed · Yesterday 6:00 PM       │
│ [Done]  [Snooze ▾]               │
│──────────────────────────────────│
│ Call Ravi about deposit          │
│ Missed · Today 9:00 AM           │
│ [Done]  [Snooze ▾]               │
└──────────────────────────────────┘
```
- **Each row:**
  - The title, then "Missed · {time}" in `caption` with `overdue` colour.
  - Two small buttons: **Done** (secondary) and **Snooze ▾**, which opens the quick sheet: 1 hour, This evening, Tomorrow 9:00 AM, Pick…
- **Removal:** a row disappears after Done or Snooze. Tapping the title opens the note.
- **Clear all:** marks every missed reminder as seen without completing them. Toast "Cleared · Undo".
- **Empty:** "No missed reminders. 🎉" When it's empty, the Todos Banner disappears.

### 6.17 Conflict compare

**Purpose:** resolve "edited on two devices", and never lose either version. It opens from the editor Banner "Compare". Route `/note/[id]/conflict`. Feature DATA-01.

```
┌──────────────────────────────────┐
│ ←  2 versions                    │
│ ┌──────────────────────────────┐ │
│ │ THIS PHONE · 10:40 AM        │ │
│ │ Call Ravi about deposit.     │ │
│ │ Ask if 1.5 months is okay.   │ │
│ │        [ Keep this ]         │ │
│ └──────────────────────────────┘ │
│ ┌──────────────────────────────┐ │
│ │ WORK PC – CHROME · 10:42 AM  │ │
│ │ Call Ravi about deposit.     │ │
│ │ Owner wants 2 months.        │ │
│ │        [ Keep this ]         │ │
│ └──────────────────────────────┘ │
│ [   Keep both as separate notes ]│
└──────────────────────────────────┘
```
- **ConflictCompare (stacked on mobile):** each version is a card (`card`, `border`, radius lg) with a header of device name + time (`overline`) and the full text (scrollable, max 40% of the screen height each).
- **Keep this:** soft-deletes the other version (it goes to Trash, so nothing is lost) and shows Toast "Kept this version · Undo".
- **Keep both:** removes the conflict link, and both stay as separate notes. Toast "Kept both".
- **No merge editor in MVP:** the user can copy text from one card before choosing.

### 6.18 Locked notes and vault

**Purpose:** end-to-end encrypted notes that only the user can open. Feature SEC-07.

**Row (anywhere):** a `lock` icon + "Locked note" + time. No preview and no chips except the reminder chip, which shows only the time (e.g. "⏰ 6:00 PM").

**Opening:** tap → BiometricPrompt "Unlock locked notes" (device credential allowed). Success opens the editor, and the vault stays unlocked for 5 min. Failure or cancel keeps the user on the list.

**First time locking a note** (⋮ → Lock, when there's no vault yet): `/settings/vault/setup`, 3 steps in one screen with progress dots.
```
┌──────────────────────────────────┐
│ ←  Locked notes          ● ○ ○   │
│ 🛡  Only you can read locked notes│
│ They're encrypted on this phone. │
│ Cue's server can't read them.    │
│ Set a vault PIN (for the web)    │
│ ┌────────────────────────────┐   │
│ │ ● ● ● ● ● ●                │   │
│ └────────────────────────────┘   │
│ At least 6 digits                │
│ [          Next            ]     │
└──────────────────────────────────┘
```
- **Step 1:** explanation + set the vault PIN (≥ 6 digits) + confirm the PIN.
- **Step 2: vault recovery key.**
  - Shown in a mono box as `XXXX-XXXX-XXXX-XXXX-XXXX-XXXX`, with [Copy] [Save file].
  - The text: "If you forget your PIN and lose this phone, this key is the only way to open locked notes. Cue can't recover them."
  - Next is enabled only after Copy or Save + the checkbox "I saved my vault recovery key".
- **Step 3:** "Vault ready" ✓ → returns to the note, which is now locked. Toast "Note locked".

**Auto-lock:** after 5 minutes, or immediately when the app goes to background. An open locked note shows "🔒 Locked · Tap to unlock".

**Notifications** for locked notes always say **"Cue: 🔒 Locked reminder"** with the time only.

**Vault settings** (`/settings/vault`): Change vault PIN, Show recovery key (needs biometric), "Locked notes are excluded from search and export unless unlocked."

### 6.19 Home widget and Quick Settings tile

**Widget** (Jetpack Glance, 4×2 default, resizable to 4×1 and 4×3). Feature CAP-02.
```
┌──────────────────────────────────┐
│ Cue          [ + New ] [ 🎤 Voice ]│
│ ○ Call Ravi about deposit  4:30PM│
│ ○ Submit GST return        5:00PM│
│ ○ Pay electricity bill   ⚠ 6:00PM│
│ +2 more                          │
└──────────────────────────────────┘
```
| Element | Spec |
|---|---|
| Container | Radius 20 dp, background `card` (follows system light/dark), 12 dp padding. |
| Header | "Cue" in `label` semibold `foreground`. **+ New** = a primary filled pill → `cue://capture?source=widget`. **🎤 Voice** = a tonal pill (`accent`) → `cue://capture?voice=1&source=widget`. |
| Today list | Max 3 rows: round checkbox + title (1 line) + time on the right (`caption`). An overdue time uses the `overdue` colour. |
| "+n more" | When there are more than 3, opens the app at Todos. |
| Empty | "Nothing due today." in `muted-foreground`. |
| 4×1 size | Header only (the + New and Voice buttons). |
| Tick | Tapping the checkbox ticks it at once in the widget (a pending action) and syncs on the next app open (M2-T08). |

**Quick Settings tile:** label **"Quick note"**, icon a `notebook-pen`-style monochrome vector drawable. Tapping it collapses the shade and opens `cue://capture?source=tile`. The tile state is always "inactive" (it's a launcher, not a toggle). Feature CAP-03.

**App icon:** see §2 (an indigo rounded square with a white "c" mark), plus an adaptive icon and a monochrome themed-icon layer.

### 6.20 Notifications and alarm screen

**Channels** (created on first launch; the user can change them in Android settings):
| Channel id | Name | Importance | Sound / vibration |
|---|---|---|---|
| `reminders` | Reminders | High | Default sound, short vibration |
| `important` | Important reminders | High | Distinct sound, long vibration |
| `alarm` | Alarm-style reminders | Max | Alarm audio usage, loops until handled |
| `signin` | Sign-in requests | High | Default sound |
| `system` | Sync & app status | Low | Silent (used rarely) |

**Normal reminder (REM-02):**
```
┌──────────────────────────────────┐
│ 🔔 Cue · now                      │
│ Cue: Pay electricity bill        │
│ 6:00 PM                          │
│ [Done] [Snooze 10 min] [1 h] [Tomorrow]│
└──────────────────────────────────┘
```
- Small icon: a monochrome Cue mark. Accent colour `primary`.
- The actions work **without opening the app** (M3-T05). Tapping the body opens `/note/[id]`.

**Important reminder (REM-07):** the same layout, plus the `important` accent colour and a 🚩 before the title: "🚩 Cue: Submit GST return". It **repeats every 10 min** (the user's setting) until Done or Snooze. Each repeat updates the text to "Still pending · 2nd reminder".

**Alarm-style screen (after 3 repeats, D-44):** a native AlarmActivity, full-screen when allowed, otherwise the `alarm` channel heads-up.
```
┌──────────────────────────────────┐
│                                  │   background: zinc-950 (always dark)
│           🚩                      │
│    Submit GST return             │   display type, white
│    5:00 PM · Still pending       │   muted
│                                  │
│                                  │
│  ┌────────────────────────────┐  │
│  │           Done             │  │   primary, 72 dp tall
│  └────────────────────────────┘  │
│  ┌────────────────────────────┐  │
│  │          Snooze            │  │   outline, 72 dp tall
│  └────────────────────────────┘  │
│   10 min · 1 hour · Tomorrow     │   text buttons when Snooze tapped
└──────────────────────────────────┘
```
- It shows over the lock screen. Alarm sound + vibration loop until Done or Snooze; the volume buttons silence the sound only.
- **Done:** haptic `success`, then "Done ✓" for 1 s and the screen closes.
- **Snooze:** reveals the 3 options in place. The default if the screen is dismissed is 10 min.
- **If the full-screen permission is missing:** a heads-up notification on the `alarm` channel with the same title and actions. Settings → Reminders shows "Full-screen alerts are off · Allow".

**Sign-in request:** "Sign-in request" / "Chrome on Windows · Ahmedabad, IN". Tapping it opens 6.11. It's dismissed automatically when it expires (120 s).

**Locked note reminder:** title **"Cue: 🔒 Locked reminder"**, text = the time only. The actions are the same.

**Test reminder:** "Cue: Test reminder ✓" / "Reminders are working."

**Grouping:** when there are more than 3 Cue notifications, Android groups them under the summary "Cue · {n} reminders".

### 6.21 Gestures and haptics

| Gesture | Where | Result | Visible alternative |
|---|---|---|---|
| Swipe right ≥ 40% | Inbox, Notes rows | Make todo (`swipe-todo`) | Actions sheet → Make todo; editor ☑ Todo |
| Swipe right ≥ 40% | Todos rows | Done (`success`) | Checkbox |
| Swipe left ≥ 40% | All lists | Archive (`swipe-archive`); in Archive view = Delete | Actions sheet → Archive / Delete |
| Long-press 400 ms | Any row | Actions sheet (6.10) | ⋮ in the editor |
| Pull down | Tab lists | Sync now | (automatic sync; Settings shows the sync status) |
| Hold | Mic button | Voice dictation | `?voice=1` from the widget |
| Swipe down / back | Sheets, capture | Close (capture **saves**) | ✕ button |
| Predictive back | Every screen | Preview of the previous screen; capture saves | ← button |

**Swipe details:**
- Under 40% the row springs back.
- The background colour and the icon + label appear **from the first pixel** of movement.
- The label fades in at 20%.
- The icon scales to 1.1× at the threshold (reduce-motion: no scale).
- No double-direction confusion: a row only moves horizontally after 10 dp of horizontal intent.

**Haptics** (expo-haptics, §3.10; all skipped if the system haptics setting is off):
| Moment | Haptic |
|---|---|
| Swipe crosses the 40% threshold | `selection` |
| Todo completed (checkbox or swipe) | `notification success` |
| Note saved from capture | `notification success` |
| Long-press opens the Actions sheet | `impact medium` |
| Number tapped on approve | `impact light` |
| Error (blocked sign-in, invalid code) | `notification error` |

### 6.22 Mobile accessibility

- **Touch targets:** at least **48×48 dp** for everything tappable, including checkboxes, chips with ✕, and icon buttons. Visual size can be smaller, but the hit area can't.
- **Font scaling up to 200%:**
  - Rows grow in height and never clip.
  - Chips wrap to new lines.
  - The BottomNav labels may truncate with an ellipsis, but the icons stay.
  - Test every screen at 100%, 130% and 200% (§9).
- **TalkBack:**
  - **Rows:** one focusable element, with the label "{title}, {reminder}, {group}". **Custom actions:** Make todo / Make note, Archive, Remind me, Move to group, Delete.
  - **Checkbox:** label "Mark '{title}' done" (checked state "'{title}' done").
  - **FAB:** "New note". Avatar: "Settings". BottomNav: "Inbox, 4 items", "Todos", "Notes".
  - **Number buttons:** "Number 42".
  - **Swipe backgrounds** are not focusable (the actions are reachable via custom actions).
- **Colour:** never the only signal.
  - Overdue = red **and** the word ("Yesterday", "Overdue").
  - Important = 🚩 icon **and** the colour.
  - Swipes show icon + label.
- **Contrast:** all text pairs meet WCAG AA (§3.1 is already verified). Don't place `muted-foreground` text on `accent` backgrounds.
- **Reduce motion** (the system "Remove animations"): replace slides, scales, springs and pulsing with a 150 ms fade.
- **Edge-to-edge:**
  - Draw behind the system bars.
  - Pad the TopBar by the status-bar inset, and the BottomNav and FAB by the navigation-bar inset.
  - Sheets respect the keyboard inset (the capture toolbar sits on top of the keyboard).
- **Focus order:** TopBar → content → FAB → BottomNav. Opening a sheet moves focus to its title, and closing it returns focus to the trigger.
- **Language:** English only (D-45). Every string comes from `strings.ts`; none are inline.

## 7. Web app

> **Platform:** desktop browser, React + Vite PWA served from the Worker (D-41). **Online-only** (D-39).
> **Feel:** calm and minimal (Linear, Things). Zinc neutrals, indigo primary, Inter, Light/Dark/System (§3).
> **Rule for builders:** use only §3 tokens and §5 components. When something isn't specified here, copy the nearest
> pattern on this page. Don't invent new colours, sizes or components.

### 7.1 Layout and breakpoints
**Purpose:** one stable, predictable frame for every view. *Patterns borrowed from: Linear ("inverted-L"), Things (Mac).*

**Wireframe (≥ 1100 px, an item selected):**
```
┌──────────────────────┬───────────────────────────────────────┬──────────────────────────────────────┐
│ ● Cue        [+ Q]   │ Inbox 12                       ⌕  ⋯   │ ☐  ★  ⏰  ▣  🔒  ⋯              ✕     │
│ ⌕ Search         /   │ ┌───────────────────────────────────┐ │                                      │
│                      │ │ + Add to Inbox…                   │ │ Call electrician about fan           │
│ ▣ Inbox          12  │ └───────────────────────────────────┘ │ ⏰ Today 6:00 PM · ● Home · 🌐 Chrome │
│ ☀ Today           3  │ ○ Call electrician about fan  ⏰ 6 PM │ ─────────────────────────────────────│
│ ▦ Upcoming           │ ▢ Weekend trip idea          2h ago   │ He said the capacitor might be bad.  │
│ ★ Important          │ ▢ Book: Deep Work            Yday     │ Ask for quote first.                 │
│ ▤ All notes          │ ○ Pay electricity bill   ⏰ Tue 9 AM  │                                      │
│                      │ ▢ Gift ideas for Mom         Mon      │                                      │
│ GROUPS               │                                       │                                      │
│ ● Home               │                                       │                                      │
│ ● Work               │                                       │                                      │
│ + New group          │                                       │                                      │
│                      │                                       │                                      │
│ ▢ Archive            │                                       │                                      │
│ 🗑 Trash              │                                       │ Created Tue 21 Oct · Edited 2m ago · │
│ ⏰ Missed          2  │                                       │ from Chrome extension      Saved     │
│ ⚙ Settings   (V)  ◐  │                                       │                                      │
└──────────────────────┴───────────────────────────────────────┴──────────────────────────────────────┘
   Sidebar 232 px          List (min 360 px, flex)                 Detail panel (440–520 px)
```

**Spec:**
| Element | Spec |
|---|---|
| App shell | Full-viewport CSS grid. Columns are `232px minmax(360px, 1fr) [detail]`. Page background is `--background`. |
| Sidebar | `Sidebar` component. Background is `--sidebar` (dimmed), with a right border `--border`. Full height, and it scrolls on its own. See §7.2. |
| List pane | Background `--background`. Header (sticky, 56 px): page title (§3 `pageTitle`) + a muted count, e.g. `Inbox 12`. Then a search IconButton and a ⋯ view menu (Sort: Newest / Oldest; Show completed). |
| Detail panel | Background `--card`, left border `--border`, width `clamp(440px, 34vw, 520px)`. It slides in from the right over 200 ms (§3 motion `base`, standard easing) when a row is opened. **Esc** closes it and returns focus to the row. |
| Note text width | Editor body `max-width: 720px` (in the full-page editor it is centred). |
| Full-page editor | **Cmd/Ctrl+Enter** on a focused or open note goes to `/note/:id`. Single column centred at 720 px, a TopBar with "← Back" (Esc) and the same icon row as the detail panel. |

**Breakpoints (§3 breakpoints):**
| Width | Behaviour |
|---|---|
| ≥ 1100 px | Sidebar + list + detail panel side by side. |
| 800–1099 px | Sidebar + list. The detail opens as an **overlay sheet** from the right (same width, `--shadow-lg`, a dim scrim at 20% black). Clicking the scrim or pressing Esc closes it. |
| < 800 px | The sidebar collapses to a 56 px **icon rail**. `[` or the hamburger expands it as an overlay. The list is full width. The detail opens as a **full page** (`/note/:id`). |

**States:** first load over 1 s → Skeleton rows in the list (§7.15). Offline → Banner on top, editing disabled (§7.15).
**Routes:** `/inbox` (default after login), `/today`, `/upcoming`, `/important`, `/notes`, `/groups/:id`, `/archive`, `/trash`, `/missed`, `/note/:id`, `/settings/*`, `/login`. Selecting an item in a list updates the URL to `?open=<id>` so the open item survives a refresh.

---

### 7.2 Sidebar
**Purpose:** reach every area in one click or two keys. *Borrowed from: Linear (dimmed sidebar), Todoist (Inbox/Today/Upcoming).*

```
┌──────────────────────┐
│ ● Cue        [+ Q]   │  ← logo + Quick add Button (ghost, shows "Q" kbd hint)
│ ⌕ Search         /   │  ← SearchField trigger (shows "/" kbd)
│                      │
│ ▣ Inbox          12  │  ← CountBadge (only when > 0)
│ ☀ Today           3  │  ← count = overdue + today (red if any overdue)
│ ▦ Upcoming           │
│ ★ Important          │
│ ▤ All notes          │
│                      │
│ GROUPS            +  │  ← SectionHeader + "+" IconButton
│ ● Home          (8)  │  ← colour dot; count shown on hover only
│ ● Work               │
│ + New group          │
│                      │
│ ▢ Archive            │
│ 🗑 Trash              │
│ ⏰ Missed          2  │  ← only rendered if missed > 0
│                      │
│ ⚙ Settings   (V)  ◐  │  ← settings, user initial avatar, theme toggle
└──────────────────────┘
```

**Spec:**
- Item: 32 px tall, 8 px horizontal padding, radius `--radius-sm`, icon 16 px (lucide, §3 icon map), label is §3 `ui` (14 px).
  - Default text `--muted-foreground`.
  - Hover background `--accent` at 60%.
  - **Active** item: background `--accent`, text `--accent-foreground`, icon `--primary`.
- CountBadge: right-aligned, §3 `small` (12 px), `--muted-foreground`. Today's count turns `--overdue` if anything is overdue.
- **Groups:**
  - Collapsible SectionHeader "GROUPS" (12 px, uppercase, tracking +0.04em, `--muted-foreground`).
  - Each group: 8 px colour dot (group colour from §3 group palette) + name. Double-click to rename.
  - "+ New group" opens an inline Input (Enter creates it, Esc cancels).
  - Right-click (or ⋯ on hover) shows Rename / Change colour / Delete group. Delete opens a Dialog: "Delete group 'Home'? Notes stay, they just lose this group."
- **Missed:** rendered only when the missed count is > 0.
- **Bottom row:**
  - Settings link (`G S`).
  - Avatar: a circle with the user's initial, `--primary` background, white text. Clicking it opens a menu: "Signed in as vinod", Settings, Sign out of this browser.
  - Theme toggle IconButton cycles Light → Dark → System (tooltip "Theme: System").
- Drop target: dragging list rows onto a group moves them (optional nice-to-have; the keyboard `M` is the primary way).

**Keyboard:**
- `G I` Inbox, `G T` Today, `G U` Upcoming, `G A` All notes, `G S` Settings. `[` toggles the sidebar (collapsed or rail).
- `G` sequences have a 1 s timeout. While waiting, show a tiny hint in the bottom-left: `G → I Inbox · T Today · U Upcoming · A All · S Settings`.

---

### 7.3 List views and row anatomy
**Purpose:** scan many items calmly and act fast. *Borrowed from: Todoist (row, Overdue + Reschedule), Linear (hover actions, density).*

**Row anatomy (36 px):**
```
┌───────────────────────────────────────────────────────────────────────────────────────────────┐
│ ○  Pay electricity bill    ⏰ Tue 9:00 AM  ● Home  #bills  🌐        2h   [⏰][▣][▢][⋯] │
│ ↑  ↑ title (1 line, ellipsis)  ↑ chips (muted, max: reminder, group, 2 tags, source)  ↑ time ↑ hover │
│ Checkbox (todo) or note icon                                                                  │
└───────────────────────────────────────────────────────────────────────────────────────────────┘
```
| Part | Spec |
|---|---|
| Lead | **TodoRow:** round `Checkbox` 16 px (border `--border`, checked = `--primary` fill + check, strikethrough title fades to `--muted-foreground` over 300 ms). **NoteRow:** `file-text` icon 16 px `--muted-foreground`. Locked note: `lock` icon. |
| Title | §3 `ui` 14 px, `--foreground`, one line with ellipsis. If empty, use the first line of the body. |
| Chips | §5 Chip, size `sm` (20 px tall, 12 px text). **Reminder chip:** `⏰ Tue 9:00 AM`; it turns `--overdue` (text + icon) when overdue; repeating adds `↻`. **Group chip:** a dot + name. **Tags:** up to 2, `#name`, then `+1`. **Source:** an icon only (globe / extension / phone / telegram-future), with a tooltip "From Chrome extension · example.com". **Important:** `★` in `--important`. Each chip is hidden when there's no data. |
| Time | Right-aligned, §3 `small`, `--muted-foreground`, relative (§4 formats: "now", "5m", "2h", "Yday", "Mon", "21 Oct"). Hidden while hover actions show. |
| Hover actions | IconButtons (28 px, ghost): Remind `H`, Move `M`, Archive `E`, More `⋯`. Each has a tooltip with its shortcut, e.g. "Remind (H)". |
| States | Hover background is `--muted` at 50%. **Selected (bulk):** `--accent` background + a checked selection box in the lead slot. **Focused (keyboard):** a 2 px `--ring` bar on the left edge plus a `focus-visible` outline. **Open in detail:** `--accent` background. |

**Bulk selection:** `X` toggles selection on the focused row, Shift+click selects a range, and Ctrl/Cmd+click toggles. A bulk bar replaces the list header:
```
┌───────────────────────────────────────────────────────────────────────────────────────────────┐
│ ✕  3 selected        [Make todo]  [Remind]  [Move]  [Archive]  [Delete]                       │
└───────────────────────────────────────────────────────────────────────────────────────────────┘
```
`Esc` clears the selection. Actions apply to everything selected, then show one Toast "3 archived · Undo".

**Views:**
| Route | Content | Sections / empty state |
|---|---|---|
| `/inbox` | State = inbox, newest first. An always-visible **"Add to Inbox…"** Input is pinned on top (§7.5). | Triage hint "3 left" next to the title. Empty: §7.6. |
| `/today` | Todos with a reminder due today, plus overdue ones. | SectionHeaders: **Overdue** (red count + a "Reschedule all" ghost Button that opens the ReminderPicker for all of them) then **Today**. Empty: "Nothing due today." + "Press Q to add something." |
| `/upcoming` | Items with future reminders, grouped by day. | Headers: "Tomorrow", "Wed 22 Oct", "Thu 23 Oct"… (14 days ahead, then "Later"). Empty: "Nothing scheduled. Add a reminder with H." |
| `/important` | Items with important = true. | Empty: "No important items. Mark one with I." |
| `/notes` | All active (non-archived, non-deleted) notes and todos, pinned first, then by updated time. | A filter row: All · Notes · Todos · group Select. Empty: "No notes yet. Press Q to capture your first thought." |
| `/groups/:id` | Items in the group. Title = group name + colour dot. | Empty: "Nothing in Home yet. Move items here with M." |
| Completed todos | In any list, completed todos are hidden. A muted "Show 4 completed" link at the bottom expands them (greyed, strikethrough). | — |

---

### 7.4 Detail panel and editor
**Purpose:** read and edit one item without losing the list. *Borrowed from: Linear (side panel), Bear (plain editor).*

```
┌──────────────────────────────────────────────┐
│ [☐] [★] [⏰] [▣] [🔒] [⋯]                  ✕ │  ← icon toolbar (IconButton 32 px)
│                                              │
│ Call electrician about fan                   │  ← Title (pageTitle, 20 px, semibold)
│ [⏰ Today 6:00 PM ↻] [● Home] [#repairs] [+] │  ← metadata Chips
│ [🌐 example.com · Open page ↗] [Jump to passage ↗] │
│ ──────────────────────────────────────────── │
│ He said the capacitor might be bad.          │  ← Body Textarea (noteBody 16/26)
│ Ask for a quote first.                       │
│ - [ ] buy capacitor                          │
│                                              │
│                                              │
│ ──────────────────────────────────────────── │
│ Created Tue 21 Oct · Edited 2m ago ·         │  ← footer (small, muted)
│ from Chrome extension                Saved   │
└──────────────────────────────────────────────┘
```
| Element | Spec |
|---|---|
| Toolbar | IconButtons, ghost, 32 px, each with a tooltip and shortcut. **Todo toggle** (`T`, `square-check`): pressed state = `--accent` background. **Important** (`I`, `star`): when on, the icon fills `--important`. **Remind** (`H`, `alarm-clock`): opens the ReminderPicker popover (§7.10). **Move to group** (`M`, `folder`): opens the GroupPicker popover. **Lock** (`lock`): turns the note into a locked note (§7.17); asks for the vault PIN if not set up. **⋯** menu: Pin (`P`), Tags…, Archive (`E`), Delete (`#`), Copy link, Open full page (`Ctrl+Enter`). **✕** closes (Esc). |
| Title | A borderless Input, §3 `pageTitle` (20/28 semibold). Placeholder "Untitled". Enter moves focus to the body. |
| Metadata row | Chips (§5): reminder chip (click → picker; × on hover removes it), group chip (click → picker), tag chips (× removes), "+" Chip adds a tag (inline Input with autocomplete). **Source chip** only when the note has a source: icon + domain/app, then "Open page ↗" (opens `source_url` in a new tab). If `source_fragment` exists, add "Jump to passage ↗" (opens the URL with `#:~:text=`). |
| Body | A borderless Textarea, §3 `noteBody` (16/26), auto-growing, max-width 720 px. Plain text; lines starting with `- [ ] ` render as checkboxes in read view (optional). Placeholder "Write anything…". |
| Auto-save | Debounced **600 ms** after the last keystroke → PATCH with `base_version`. The footer status shows "Saving…" while the request runs, then "Saved" (muted, fades after 2 s), or "Couldn't save · Retry" in `--destructive` on error. |
| Footer | §3 `small`, `--muted-foreground`: `Created Tue 21 Oct · Edited 2m ago · from Chrome extension`. |
| Conflict | If the note has a conflict copy, a Banner sits at the top of the panel (§7.14). |

**Keyboard inside the panel:** the toolbar shortcuts work when focus isn't in a text field. `Esc` in a text field blurs it; a second `Esc` closes the panel. **Cmd/Ctrl+Enter** opens `/note/:id` (full page).
**States:** loading (Skeleton: title bar + 3 lines), deleted elsewhere ("This note was deleted on another device." + Restore Button), offline (all inputs read-only + Banner).

---

### 7.5 Quick add
**Purpose:** capture from anywhere in about 2 seconds (CAP-06, REM-01). *Borrowed from: Todoist Quick Add (inline date highlight), Things Quick Entry.*

```
                  ┌──────────────────────────────────────────────────────────┐
                  │ Quick add                                              ✕ │
                  │ ┌──────────────────────────────────────────────────────┐ │
                  │ │ Pay electricity bill [tomorrow 9am]                  │ │ ← CaptureField; the date is highlighted
                  │ │                                                      │ │
                  │ └──────────────────────────────────────────────────────┘ │
                  │ [⏰ Wed 22 Oct, 9:00 AM ✕]                                │ ← preview chip (accept by saving; ✕ removes the parse)
                  │ [☐ Todo]  [▣ Group ▾]  [★ Important]          [Save ↵] │
                  └──────────────────────────────────────────────────────────┘
                     Dialog 560 px, top offset 15vh, scrim 20%
```
| Element | Spec |
|---|---|
| Open | **`Q`** (when focus isn't in a text field) or **`Alt+N`** (works everywhere, including inside inputs). The sidebar "Quick add" button and the palette "Quick add" command open it too. |
| Dialog | §5 Dialog, 560 px wide, positioned 15vh from the top, radius `--radius-lg`, `--shadow-lg`. Title "Quick add" (`ui` semibold). |
| CaptureField | Autofocus, 3 rows that grow to 8, placeholder "What's on your mind? Try 'call mom tomorrow 6pm'". Detected date text gets an inline highlight: `--accent` background, `--accent-foreground` text, radius 4. Clicking the highlight removes the parse (the text stays plain). |
| Preview chip | Shown when a date is detected: `⏰ Wed 22 Oct, 9:00 AM` (+ ` ↻ Every Monday` for repeats), with a × to discard. |
| Options row | Toggle Buttons (outline, sm): **Todo** (`square-check`), **Group ▾** (opens the GroupPicker), **Important** (`star`). The primary Button **Save** shows a `↵` kbd hint. |
| Keys | `Enter` saves. `Shift+Enter` adds a new line. `Esc` closes, keeping a draft for 10 minutes; reopening restores it. |
| After save | The dialog closes, then a Toast "Saved to Inbox · Undo" (or "Saved · Reminder Wed 9:00 AM · Undo"). The item appears at the top of the Inbox. |

**Inline "Add to Inbox…" input** (top of `/inbox`): a single-line Input with a `+` icon and placeholder "Add to Inbox… (press Enter)". It behaves like the CaptureField (date highlight + preview chip below while typing). Enter saves and keeps focus, so you can type the next item straight away.
**Offline:** the Save Button is disabled, and an inline line says "You're offline. Not saved." with a Retry link (D-39).

---

### 7.6 Inbox triage
**Purpose:** reach Inbox zero quickly with the keyboard (ORG-04). *Borrowed from: Superhuman (one decision per item, focus moves on).*

```
┌──────────────────────────────────────────────────────────────────┐
│ Inbox 3   · 3 left                          ⌕  ⋯                 │
│ + Add to Inbox…                                                  │
│▌○ Call electrician about fan              ⏰ 6 PM   [⏰][▣][▢][⋯]│ ← focused row (left ring)
│ ▢ Weekend trip idea                       2h                     │
│ ▢ Book: Deep Work                         Yday                   │
│                                                                  │
│ T todo · E archive · H remind · M move · I important · # delete  │ ← hint bar (muted, small)
└──────────────────────────────────────────────────────────────────┘
```
**Keys (focus on a row):**
| Key | Action | After |
|---|---|---|
| `J` / `K` (↓ / ↑) | Next / previous row | — |
| `T` | Make todo (state → active, is_todo = true) | Focus moves to the next row. Toast "Moved to Todos · Undo" |
| `E` | Archive | Next row. Toast "Archived · Undo" |
| `H` | Open the ReminderPicker. On pick, the item leaves the Inbox (becomes active with a reminder) | Next row |
| `M` | Open the GroupPicker. On pick, the item leaves the Inbox | Next row |
| `I` | Toggle important (stays in the Inbox) | Stays |
| `#` | Delete (to Trash) | Next row. Toast "Moved to Trash · Undo" |
| `Enter` | Open in the detail panel | — |
| `Z` | Undo the last action | Focus returns to the restored row |

- **Progress hint:** next to the title, "· 3 left" (muted). It is hidden when the Inbox is empty.
- **Hint bar:** a one-line muted legend at the bottom of the list (shown only on `/inbox`; dismissible ×, remembered).
- **Empty state (EmptyState):** icon `inbox` (48 px, `--muted-foreground`), heading **"Inbox zero. Nice."**, body **"Press Q to capture something."**

---

### 7.7 Command palette
**Purpose:** every action and place, reachable by typing (all features). *Borrowed from: Superhuman, Linear (Cmd+K).*

```
            ┌────────────────────────────────────────────────────────────┐
            │ ⌕  Type a command or search…                               │
            │────────────────────────────────────────────────────────────│
            │ RECENT                                                     │
            │   ⏰ Set reminder                                        H │
            │ ACTIONS                                                    │
            │ ▸ + Quick add                                            Q │ ← highlighted row
            │   ☐ Make todo                                            T │
            │   ▣ Move to group…                                       M │
            │   ★ Mark important                                       I │
            │   ▢ Archive                                              E │
            │   🗑 Delete                                              # │
            │   ◐ Toggle theme                                           │
            │   ⇩ Export…                                                │
            │ GO TO                                                      │
            │   ▣ Inbox                                              G I │
            │   ☀ Today                                              G T │
            │ NOTES                                                      │
            │   ▤ Call electrician about fan                             │
            └────────────────────────────────────────────────────────────┘
               CommandPalette, 640 px, top 12vh, max 8 rows visible
```
- **Open/close:** `Ctrl/Cmd+K` (works in inputs too). Pressing it again or `Esc` closes it and returns focus where it was.
- **Groups:**
  - **Recent:** last 3 commands used.
  - **Actions:** Quick add, Make todo, Set reminder, Move to group, Mark important, Pin, Archive, Delete, Toggle theme, Export…, Sign out.
  - **Go to:** Inbox, Today, Upcoming, Important, All notes, every group by name, Archive, Trash, Missed reminders, Settings: Account / Devices and office hours / Notifications / Time zone / Security / Appearance / Export.
  - **Notes:** when the query is ≥ 2 characters, the top 5 matching notes by title/body appear (server search, §7.9). Enter opens the note.
- **Context:** item actions (Make todo, Archive…) apply to the focused or open item, or to the selection. They are hidden when nothing is focused.
- **Row:** icon 16 + label (`ui`) + a right-aligned kbd hint (§5 Kbd style: `--muted` background, 11 px mono, radius 4). The highlighted row has an `--accent` background.
- **Matching:** fuzzy and case-insensitive (cmdk default). Aliases: "done" → Mark done, "remind/later/snooze" → Set reminder, "folder" → Move to group.
- **Empty:** "No results for 'xyz'. Press Enter to add it to Inbox." Enter then quick-adds the typed text.

---

### 7.8 Keyboard map
**Purpose:** the single, final keymap (D-52; supersedes the older lists in ux-principles).

| Key | Action | Where |
|---|---|---|
| `Q` | Quick add | anywhere (not while typing) |
| `Alt+N` | Quick add | anywhere, even while typing |
| `Ctrl/Cmd+K` | Command palette | anywhere, even while typing |
| `/` | Search | anywhere (not while typing) |
| `?` | Shortcuts overlay | anywhere (not while typing) |
| `J` / `K` (↓ / ↑) | Next / previous item | lists |
| `Enter` | Open item | lists |
| `Ctrl/Cmd+Enter` | Open full-page editor | list or detail |
| `Esc` | Close / back / clear selection | everywhere |
| `X` | Select / deselect for bulk | lists |
| `T` | Toggle todo | item |
| `D` | Mark done (todos) | item |
| `E` | Archive (everywhere) | item |
| `H` | Set reminder | item |
| `M` | Move to group | item |
| `I` | Toggle important | item |
| `P` | Pin / unpin | item |
| `#` | Delete (to Trash) | item |
| `Z` | Undo last action | anywhere |
| `G I` / `G T` / `G U` / `G A` / `G S` | Go to Inbox / Today / Upcoming / All notes / Settings | anywhere |
| `[` | Toggle sidebar | anywhere |

- **Rule:** single-key shortcuts are **disabled while focus is in an input/textarea/contenteditable**. Only `Esc`, `Ctrl/Cmd+K`, `Alt+N` and `Ctrl/Cmd+Enter` work there.
- **`?` overlay:** a Dialog (640 px) with a search Input on top ("Search shortcuts…") and grouped tables: General, Navigation, Lists, Items. Each line shows the action + the kbd.
- **Tooltips** on every IconButton show the shortcut, e.g. "Archive (E)".

---

### 7.9 Search
**Purpose:** find any note by any word (ORG-07). *Borrowed from: Linear, Todoist (`/`).*

```
┌───────────────────────────────────────────────────────────────────────┐
│ ⌕ [ electr                                   ✕ ]                      │ ← SearchField (list header)
│ [Group ▾] [Has reminder] [Todo] [Include archived]                    │ ← filter Chips (toggle)
│                                                                       │
│ TODOS · 1                                                             │
│ ○ Pay [electr]icity bill               ⏰ Tue 9 AM  ● Home            │
│ NOTES · 2                                                             │
│ ▢ Call [electr]ician about fan                         2h             │
│   …capacitor might be bad. Ask the [electr]ician for…                 │ ← snippet (muted)
└───────────────────────────────────────────────────────────────────────┘
```
- **Open:** `/` focuses the sidebar Search, which turns the list pane into search results (route `/notes?q=…`, back with Esc).
- **Query:** results update 250 ms after the last keystroke (debounced), calling `GET /api/search?q=` (server FTS5). Minimum 2 characters.
- **Results:** two SectionHeaders, **Todos · n** and **Notes · n**, using the same NoteRow/TodoRow plus a 1-line snippet below the title. Matches are highlighted with an `--accent` background and `--accent-foreground` text.
- **Filters:** toggle Chips for Group (GroupPicker dropdown), Has reminder, Todo, Include archived.
- **States:**
  - Typing / under 2 characters: "Type at least 2 letters."
  - Loading: a thin progress line under the field, no skeleton.
  - No results: EmptyState "No results for 'electr'." + "Try fewer words or include archived."
  - Locked notes: never matched; hint row "Locked notes aren't searched."

---

### 7.10 Reminder picker
**Purpose:** set a reminder in one or two clicks (REM-01/02/07). *Borrowed from: Google Keep (quick chips), Todoist.*

```
┌──────────────────────────────────────────┐
│ Remind me                                │
│ [Later today 9:00 PM]  [This evening 7 PM]│ ← quick Chips (2×2)
│ [Tomorrow 9:00 AM]     [Next Mon 9:00 AM]│
│ ──────────────────────────────────────── │
│ ┌ Oct 2026 ───────────── ‹ › ┐ Time      │
│ │ Mo Tu We Th Fr Sa Su       │ [09:00 AM]│ ← Calendar + time Input
│ │        1  2  3  4  5       │           │
│ │ ...  [22] ...              │           │
│ └────────────────────────────┘           │
│ Repeat  [Never ▾]                        │ ← Select: Never, Daily, Weekdays, Weekly, Monthly, Custom…
│ Important (repeat until done)   [  ○]    │ ← Switch
│ ──────────────────────────────────────── │
│ [Remove reminder]              [Set ↵]   │
└──────────────────────────────────────────┘
   ReminderPicker in a Popover (320 px), anchored to its trigger
```
- **Quick chips** (computed in the user's time zone, default IST):
  - **Later today:** now + 3 h, rounded to :00/:30. Hidden after 9 PM.
  - **This evening:** 7:00 PM. Hidden after 6 PM.
  - **Tomorrow:** 9:00 AM.
  - **Next week:** next Monday, 9:00 AM.
  - Clicking a chip **sets the reminder immediately** and closes the picker.
- **Custom:** pick a date in the Calendar and type a time (accepts "9", "9pm", "21:30"), then **Set** (Enter).
- **Repeat:** a Select with Never / Daily / Weekdays / Weekly (on chosen day) / Monthly (on date) / Custom… (Custom opens a small form: every N days/weeks/months).
- **Important:** a Switch with the helper line "Repeats every 10 min until you mark it done."
- **Remove reminder:** a ghost Button, shown only if a reminder already exists.
- **Keyboard:** arrow keys move between chips; Enter applies; Esc closes. In triage, `H` opens the picker with focus on the first chip.
- **Copy:** title "Remind me". Toast after setting: "Reminder set · Wed 9:00 AM · Undo".

---

### 7.11 Sign-in
**Purpose:** passwordless sign-in by approving on the phone (SEC-01), with QR (SEC-02) and authenticator (SEC-03) fallbacks. *Borrowed from: Microsoft Authenticator (number matching), Google prompts, WhatsApp Web (QR).*
**Route:** `/login`. **Layout:** a centred card, 400 px wide, on `--background`, `--card` background, radius `--radius-lg`, `--shadow-md`, 32 px padding.

**Step 1: username**
```
                    ┌──────────────────────────────────────┐
                    │                 ● Cue                │
                    │           Sign in to Cue             │
                    │                                      │
                    │  Username                            │
                    │  [ vinod                         ]   │
                    │                                      │
                    │  [            Continue             ] │
                    │                                      │
                    │  New to Cue? Create your account in  │
                    │  the Android app.                    │
                    └──────────────────────────────────────┘
```
- An Input (autofocus, lowercase, `[a-z0-9_]`, 3–20 characters) with an inline error: "Usernames use a–z, 0–9 and _ (3–20 characters)."
- **Continue** (primary, full width, Enter) → `POST /auth/login/start`.

**Step 2: approve on phone (NumberApproval, web side)**
```
                    ┌──────────────────────────────────────┐
                    │                 ● Cue                │
                    │        Check your phone              │
                    │                                      │
                    │               ┌──────┐               │
                    │               │  42  │               │ ← 64 px mono, --primary
                    │               └──────┘               │
                    │  Open the notification on your phone │
                    │  and select 42.                      │
                    │                                      │
                    │  Expires in 1:58                     │ ← countdown, muted
                    │                                      │
                    │  Use QR code instead                 │ ← link Button
                    │  Can't use your phone? Use           │
                    │  authenticator code                  │
                    │  ← Use a different username          │
                    └──────────────────────────────────────┘
```
- The number in §3 mono, 64 px, `--primary`, letter-spacing 0.05em, inside a 120×88 box with a `--accent` background.
- **Countdown:** 120 s, as `m:ss`, muted. Status is polled every 2 s.

**QR alternative (QRBlock)**
```
                    ┌──────────────────────────────────────┐
                    │        Scan with the Cue app         │
                    │           ┌────────────┐             │
                    │           │ ▓▓ ▓ ▓▓ ▓  │             │
                    │           │ ▓ ▓▓▓  ▓▓  │  200×200    │
                    │           │ ▓▓  ▓ ▓ ▓▓ │             │
                    │           └────────────┘             │
                    │  Open Cue on your phone → Settings → │
                    │  Devices → Scan QR.                  │
                    │  Refreshes in 0:47                   │
                    │  Use number instead                  │
                    └──────────────────────────────────────┘
```
- The QR auto-refreshes every **60 s** (a new request). The QR sits on a white background even in dark mode, for scannability.

**Authenticator fallback (TOTP, SEC-03):** title "Enter authenticator code", 6 segmented boxes (each 44×52, mono 24 px). It auto-advances, accepts paste, and auto-submits when all 6 are filled. Below it: "Use number instead". Error: "That code didn't work. Check your authenticator app and try again." Locked out: "Too many attempts. Try again in 15 minutes."

**States and copy:**
| State | Copy | Actions |
|---|---|---|
| Waiting | "Check your phone" | as above |
| Expired | "This request expired. Try again." | Button "Try again" (new request) |
| Denied | "Sign-in was blocked from your phone." | "Try again", "← Use a different username" |
| Wrong number picked | "Sign-in was blocked from your phone." (same as Denied, for safety) | same |
| Rate limited | "Too many attempts. Wait a few minutes and try again." | — |
| Network error | "Can't reach Cue. Check your connection." | "Retry" |
| Success | A check icon + "Signed in" | Redirect to `/inbox` after 600 ms |

- **No signup on the web** (D-33): the footer always says "New to Cue? Create your account in the Android app."

---

### 7.12 Settings
**Purpose:** all configuration in one calm place. **Route:** `/settings/<section>`. *Borrowed from: Linear (settings nav).*

```
┌──────────────────────┬─────────────────────┬───────────────────────────────────────────────────────┐
│ (app sidebar)        │ SETTINGS            │ Devices and office hours                              │
│                      │ Account             │ Notifications on each device follow its schedule.     │
│                      │▌Devices & office hrs│                                                       │
│                      │ Notifications       │ ┌───────────────────────────────────────────────────┐ │
│                      │ Time zone & dates   │ │ 📱 Pixel 8 (this phone)   Trusted · 2m ago · Pune │ │
│                      │ Security            │ │    Always                     [Edit schedule]     │ │
│                      │ Appearance          │ ├───────────────────────────────────────────────────┤ │
│                      │ Export              │ │ 💻 Work PC – Chrome (this browser) · now · Pune   │ │
│                      │ Danger zone         │ │    Mon–Fri 9:00 AM–6:00 PM    [Edit schedule] [Revoke]│
│                      │                     │ ├───────────────────────────────────────────────────┤ │
│                      │                     │ │ 🧩 Chrome extension · 1h ago · Pune               │ │
│                      │                     │ │    Mon–Fri 9:00 AM–6:00 PM    [Edit schedule] [Revoke]│
│                      │                     │ └───────────────────────────────────────────────────┘ │
└──────────────────────┴─────────────────────┴───────────────────────────────────────────────────────┘
```
- The settings nav is 200 px. Content max width 720 px. Each section uses SettingsRow (label + description + control on the right).

**Sections (in this order):**
1. **Account** (`/settings/account`): Username (read-only) and "Signed in on this browser since 21 Oct".
2. **Devices and office hours** (`/settings/devices`, SEC-05, REM-08):
   - A device list: kind icon, name (+ "this browser" / "this phone"), Trusted badge, last seen, city, and schedule summary. Schedule formats: "Always", "Mon–Fri 9:00 AM–6:00 PM", "Never".
   - **Edit schedule** opens a Dialog:
     - mode radio: Always / Only during office hours / Never
     - day toggles (M T W T F S S)
     - Start and End time Inputs
     - preview line "Notifications on Mon–Fri, 9:00 AM–6:00 PM IST"
     - Save
   - **Revoke** opens a Dialog: title "Revoke 'Work PC – Chrome'?", body "It will be signed out immediately.", buttons Cancel / **Revoke** (destructive). Revoking a trusted phone isn't allowed from the web; the note says "Revoke phones from a trusted phone."
3. **Notifications** (`/settings/notifications`): Switch "Desktop notifications in this browser". Turning it on triggers the browser permission prompt. A "Send test notification" Button. Status text: "Allowed" / "Blocked in browser settings" (with a how-to link).
4. **Time zone and dates** (`/settings/time`): a time-zone Select, default **"India (IST, UTC+05:30)"**. Time format: 12-hour (default) / 24-hour. Week starts on Monday (default) / Sunday.
5. **Security** (`/settings/security`):
   - Authenticator app: status + "Set up on your phone" (enrolment happens on the trusted phone).
   - Recovery codes: "Regenerate on your phone" (info only).
   - Vault: "Change vault PIN" (needs the current PIN).
   - Security log: a table of the last 20 events.
6. **Appearance** (`/settings/appearance`): a Theme segmented control (Light / Dark / System) and a Density radio (Comfortable, default; Compact = 32 px rows).
7. **Export** (`/settings/export`): §7.18.
8. **Danger zone** (`/settings/danger`): "Sign out of this browser" (outline destructive), body "You'll need your phone to sign in again."

---

### 7.13 Missed reminders, Archive, Trash
**Missed reminders** (`/missed`, REM-08 AC4): reminders that fired while no device could show them (e.g. outside office hours).
```
┌──────────────────────────────────────────────────────────────────────┐
│ Missed reminders 2                                    [Clear all]    │
│ ○ Pay electricity bill     was due Tue 8:00 PM       [Done] [Snooze▾]│
│ ○ Call Ravi                was due Mon 7:30 PM       [Done] [Snooze▾]│
└──────────────────────────────────────────────────────────────────────┘
```
- Per row: Done (primary sm) and a Snooze ▾ menu (1 hour, Tomorrow 9:00 AM, Pick…).
- "Clear all" only dismisses them from this list (the reminders stay done/undone as they are). Toast "Cleared · Undo".
- Empty: "No missed reminders."

**Archive** (`/archive`): the normal list (read-only feel). Row hover shows "Unarchive" (`E` toggles it back). Empty: "Nothing archived."

**Trash** (`/trash`, DATA-02):
- A muted info line at the top: **"Deleted notes are removed after 30 days."**
- Row hover shows **Restore** and **Delete forever**.
- Delete forever opens a Dialog: "Delete forever? This can't be undone." with Cancel / **Delete forever** (destructive).
- Header Button "Empty trash" opens a Dialog: "Delete 14 notes forever? This can't be undone."
- Empty state: "Trash is empty."

---

### 7.14 Conflict resolution
**Purpose:** never lose an edit made on two devices (DATA-01). *Pattern: "keep both", then the user decides.*

**Banner in the detail panel and on the row:**
```
┌──────────────────────────────────────────────┐
│ ⚠ Edited on two devices.          [Compare] │ ← Banner (warning tone), top of the detail panel
└──────────────────────────────────────────────┘
```
The row shows a small `copy` icon chip, "2 versions".

**ConflictCompare Dialog (900 px):**
```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│ Edited on two devices                                                                ✕ │
│ ┌───────────────────────────────────────┐  ┌───────────────────────────────────────┐   │
│ │ THIS VERSION · Web · 10:42 AM         │  │ OTHER VERSION · Pixel 8 · 10:40 AM    │   │
│ │ Call electrician about fan            │  │ Call electrician about fan            │   │
│ │ Ask for quote first.                  │  │ Ask for quote first. Budget ₹1500.    │   │
│ │ - [ ] buy capacitor                   │  │                                       │   │
│ └───────────────────────────────────────┘  └───────────────────────────────────────┘   │
│                         [Keep this]   [Keep other]   [Merge…]                          │
└────────────────────────────────────────────────────────────────────────────────────────┘
```
- Two columns. Lines that differ get a subtle highlight (`--accent` background).
- **Keep this:** keeps the left version and soft-deletes the other. **Keep other:** the reverse.
- **Merge…:** one editable Textarea prefilled with both bodies (separated by `--- other version ---`), then **Save merged**.
- After resolving, a Toast "Conflict resolved · Undo".

---

### 7.15 Feedback: toasts, banners, loading, offline
| Pattern | Spec |
|---|---|
| **Toast** (sonner) | Bottom-left, 16 px from the edges, max width 420 px, **one at a time** (a new one replaces the old). Shown for 5 s, or **8 s when it has Undo**. `Z` triggers Undo while it's shown. Copy pattern: "<Result> · Undo", e.g. "Archived · Undo", "3 moved to Home · Undo". Errors use the destructive tone, e.g. "Couldn't save. Retry". Never toast routine saves. |
| **Offline Banner** (D-39) | A slim (32 px) full-width bar above the app, `--muted` background, `wifi-off` icon, text **"You're offline. Changes can't be saved right now."** While it shows, every input is `readonly`, action buttons are disabled (50% opacity, tooltip "You're offline"), and Quick add's Save is disabled. It disappears automatically on reconnect, with the Toast "Back online". |
| **Loading** | **No spinner or skeleton for anything under 1 s.** For the first page load over 1 s: 8 Skeleton rows (36 px, `--muted` pulse). Detail panel: a title bar + 3 lines. Export: a progress bar. Never a full-page spinner. |
| **Inline load error** | Inside the list pane: icon `cloud-alert` + "Couldn't load your notes." + Button "Retry". |
| **Session expired** | Redirect to `/login`, then the Toast **"Please sign in again."** |
| **Permission banner** | A one-time Banner at the top of `/inbox`: "Get reminders on this computer during office hours · [Enable] [✕]" (§7.16). |
| **Empty states** | EmptyState: 48 px icon, heading (`ui` semibold) + one helpful line with a shortcut. Copy is listed per view in §7.3, §7.6 and §7.13. |

---

### 7.16 Desktop notifications
**Purpose:** reminders on the PC, **only during office hours** (REM-06, REM-08).
- **Asking:** never auto-prompt on page load. Ask only from:
  - Settings → Notifications (the Switch), or
  - the one-time Banner on `/inbox`: "Get reminders on this computer during office hours · Enable". Enable triggers the browser prompt. ✕ hides the Banner forever.
- **Format (Web Push → service worker):**
```
┌───────────────────────────────────────┐
│ ● Cue: Pay electricity bill           │
│ 6:00 PM                               │
│ [Done]  [Snooze 1 h]                  │
└───────────────────────────────────────┘
```
  - Title "Cue: <title>" (locked notes: "Cue: 🔒 Locked reminder"). Body is the time ("6:00 PM"), plus " · Important" when flagged.
  - Actions: **Done**, **Snooze 1 h**. Clicking the body opens `/note/:id` in the existing tab.
  - Tag = `reminderId:fire_at` (replaces duplicates).
- **Grouping:** when 2 or more are due in the same minute → one notification, "Cue: 3 reminders", with body "Pay bill, Call Ravi, +1". Clicking it opens `/today`.
- **Office hours:** the server only pushes inside this browser's schedule. Anything outside goes to Missed (§7.13). The PWA does not show a notification on its own outside office hours either.
- **Important items:** `requireInteraction: true` (they stay until handled). Normal ones auto-dismiss.

---

### 7.17 Locked notes on web
**Purpose:** see E2EE notes only after unlocking (SEC-07).
- **Row:** the title is replaced by **"Locked note"** with a `lock` icon. No chips except the reminder (shown as "🔒 Reminder Tue 9:00 AM"). Never shown in search results.
- **Opening it:** the detail panel shows a centred lock state:
```
┌──────────────────────────────────────────────┐
│                    🔒                         │
│            This note is locked               │
│     Enter your vault PIN to view it.         │
│            [ • • • • • • ]                   │
│               [Unlock]                       │
│   Forgot PIN? Use your vault recovery key.   │
└──────────────────────────────────────────────┘
```
  - A PIN Input (6+ digits, masked) and **Unlock**.
  - Wrong PIN: "Wrong PIN. Try again." After 5 tries: "Too many tries. Wait 1 minute."
- **Unlocked:** the normal editor shows a `lock-open` indicator in the toolbar. Content stays visible **until you leave the note or after 5 minutes** (whichever is first). Then it re-locks and is cleared from memory.
- **Locking a note:** toolbar Lock → if no vault exists, a Dialog says "Locked notes are encrypted on your devices. Set up the vault on your phone first."; otherwise it asks for the PIN.

---

### 7.18 Export
**Purpose:** take all data out at any time (DATA-03; zip built in the browser, D-48). **Route:** `/settings/export`.
```
┌───────────────────────────────────────────────────────────┐
│ Export                                                    │
│ Download everything as Markdown files plus a JSON file.   │
│                                                           │
│ [☐] Include locked notes (unlock vault)                   │
│                                                           │
│ [ Download all notes (.zip with Markdown + JSON) ]        │
│                                                           │
│ Preparing 1,240 notes…  ███████████░░░░░░  64%            │ ← while running
└───────────────────────────────────────────────────────────┘
```
- The **Include locked notes** Checkbox: checking it opens the vault PIN Dialog (§7.17). If you cancel, it unchecks.
- Button (primary). While running, it is disabled and a progress bar shows "Preparing 1,240 notes…" (pages fetched / zipped). Then the browser downloads `cue-export-YYYY-MM-DD.zip`.
- Success Toast: "Export ready · 1,240 notes". If locked notes were skipped, add the line "Locked notes were skipped."
- Error: "Export failed. Try again." + Retry.
- Offline: Button disabled, "You're offline."

---

## 8. Browser extension

> **Platform:** Chrome MV3 via WXT + React, Tailwind + shadcn using the **same tokens as the web** (§3).
> Compact UI: `ui` is 14 px, `small` is 12 px. Light/Dark follows the browser/OS.

### 8.1 Principles
1. **A companion, never an interruption.** It doesn't open by itself, never notifies about saves or sync, and stays quiet outside office hours.
2. **Capture first.** The popup opens straight into a focused text field with the current page attached.
3. **Never lose a draft.** The popup closes whenever it loses focus, so the draft is saved to `chrome.storage.local` **on every keystroke** and restored on the next open (until saved or cleared).
4. **Same look and words as the web.** Same tokens, Chips, CaptureField and Toast copy.
5. **Works offline for capture.** Saves are queued (CAP-09) and synced later.
*Patterns borrowed from: Todoist for Chrome, Raindrop (saved-page indicator), Glasp/Readwise (jump to highlight), Notion Web Clipper (sign in outside the popup).*

### 8.2 Popup
**Purpose:** a quick note about the current page in about 2 s (EXT-01, CAP-07, CAP-09, CAP-10, REM-01).
**Size:** width **380 px**, height auto, **max 520 px** (inner scroll if longer). Padding 12 px.

**Signed in, typing:**
```
┌────────────────────────────────────────┐
│ ● Cue        Inbox 12 ↗   ⏰ 2     ⋯ │ ← header: logo, Inbox link, missed (if >0), menu
│────────────────────────────────────────│
│ ┌────────────────────────────────────┐ │
│ │ Read this later [tomorrow 7pm]     │ │ ← CaptureField (autofocus, 4 rows)
│ │                                    │ │
│ │                                    │ │
│ └────────────────────────────────────┘ │
│ [🌐 How sleep works – example.com ✕]  │ ← source Chip (removable)
│ [⏰ Wed 22 Oct, 7:00 PM ✕]             │ ← date preview Chip (if detected)
│ [☐ Todo] [▣ Group ▾] [★]               │ ← options row
│ [              Save ↵              ]   │ ← primary Button, full width
└────────────────────────────────────────┘
```
| Element | Spec |
|---|---|
| Header | 40 px tall. ● Cue logo (16 px mark + "Cue" in `ui` semibold). **"Inbox 12 ↗"** link opens the web app `/inbox` in a new tab. **⏰ 2** (only when there are missed reminders) opens `/missed`. **⋯** menu: Open side panel, Open Cue web app, Settings (opens `/settings`), Sign out. |
| CaptureField | Autofocus, 4 rows (grows to 8), placeholder **"Note about this page…"**, inline date highlight like web §7.5 (click the highlight to un-parse). |
| Source Chip | A favicon (or `globe`) + page title truncated to 1 line + `· domain`. × removes the attachment (then the note has no source). Hidden on restricted pages (§8.9). |
| Date preview Chip | `⏰ Wed 22 Oct, 7:00 PM` with ×. Shown only when a date is detected. |
| Options | Toggle Buttons (sm, outline): **Todo**, **Group ▾** (GroupPicker as a dropdown inside the popup), **★** (Important; tooltip "Important"). |
| Save | Primary, full width, label "Save", with a `↵` kbd hint. Disabled while the field is empty. |

**Keys:** `Enter` saves, `Shift+Enter` adds a new line, `Esc` closes the popup (the draft is kept).

**After saving:**
```
┌────────────────────────────────────────┐
│                                        │
│                 ✓                      │ ← check icon, scale-in animation 200 ms (--success)
│           Saved to Inbox               │
│                                        │
└────────────────────────────────────────┘
```
The popup auto-closes after **800 ms**. The draft is cleared. If a reminder was set: "Saved · Reminder Wed 7:00 PM".

**Offline save (CAP-09):** the same confirmation with an icon `cloud-off` and text **"Saved offline. Will sync."** The item goes to the capture queue.

**Signed out:**
```
┌────────────────────────────────────────┐
│ ● Cue                                  │
│                                        │
│   Capture thoughts from any page.      │
│                                        │
│   [         Sign in to Cue         ]   │ ← opens the sign-in tab page (§8.3)
│                                        │
│   New? Create your account in the      │
│   Cue Android app.                     │
└────────────────────────────────────────┘
```

### 8.3 Sign-in tab page
**Purpose:** sign the extension in by approving on the phone (EXT-03, SEC-01/02).
**Why a tab:** a popup closes as soon as you look at your phone or click elsewhere (D-52). This page is an extension page in a full tab: `chrome-extension://<id>/signin.html`. It keeps the **extension's own device key** (non-extractable WebCrypto in IndexedDB).
- **Layout and flow:** identical to web §7.11: username → NumberApproval (big 2-digit number, countdown) → "Use QR code instead" (QRBlock) → authenticator fallback. All states and copy are the same as §7.11.
- **Page title** (browser tab): "Sign in · Cue".
- **On success:**
```
┌──────────────────────────────────────┐
│                 ✓                    │
│      Signed in as vinod.             │
│   You can close this tab.            │
└──────────────────────────────────────┘
```
  The tab auto-closes after **2 s**. The next time you open the popup, it's signed in.

### 8.4 Side panel
**Purpose:** notes for the current page, the Inbox, and capture, next to what you're reading (EXT-05, EXT-04).
**Width:** designed for a **minimum of 320 px** (Chrome lets the user resize it). It follows the active tab and updates when you switch tabs or when the URL changes.

```
┌────────────────────────────────────────┐
│ ● Cue                        ⌕    ⋯    │ ← header
│ [This page (2)] [ Inbox ]              │ ← Tabs
│────────────────────────────────────────│
│ How sleep works – example.com          │ ← current page (muted, 1 line)
│                                        │
│ ▢ Key idea: 90-min cycles        2h    │ ← NoteRow (compact)
│   "Sleep happens in cycles of…"        │ ← selection snippet (italic, muted)
│   Jump to passage ↗                    │
│ ○ Read rest of article  ⏰ Tonight 9 PM │
│                                        │
│                                        │
│────────────────────────────────────────│
│ ┌────────────────────────────────────┐ │
│ │ Note about this page…              │ │ ← CaptureField pinned at the bottom (2 rows)
│ └────────────────────────────────────┘ │
│ [☐][▣][★]                     [Save ↵] │
└────────────────────────────────────────┘
```
| Element | Spec |
|---|---|
| Header | Logo, a search IconButton (opens an inline SearchField on top, server search §7.9, compact results), and a ⋯ menu (Open Cue web app, Sign out). |
| Tabs | **"This page (n)"** \| **"Inbox"**, using the shadcn Tabs (underline style). The default tab is "This page" when n > 0, otherwise "Inbox". |
| This page | Notes whose `page_key` matches the current page or site, newest first. Each row: compact NoteRow/TodoRow (32 px). If the note came from a selection: a 2-line snippet of the saved text + **"Jump to passage ↗"** (opens the text-fragment URL in the current tab). Clicking a row opens the note in the web app (new tab). |
| Empty (This page) | EmptyState (small): icon `file-plus`, **"No notes for this page yet."**, "Write one below or select text and right-click → Save to Cue." |
| Inbox tab | The 20 most recent Inbox items (compact rows). The footer link "Open Inbox in Cue ↗". Empty: "Inbox zero." |
| Bottom capture | **Pinned** to the bottom: a CaptureField (2 rows, grows to 6, placeholder "Note about this page…"). The current page is **auto-attached** (a small muted line above: "Attached: example.com", with × to detach). An options row of icon toggles (Todo, Group, Important) + Save (`Enter`). After saving, the item appears in "This page" instantly, the field clears, and an inline "Saved" fades after 1.5 s. |
| Future chat | A reserved area for a future chat tab (AI-01). **Not rendered** while `ai.enabled = false`. |

**States:** signed out (same as popup, §8.2); loading (show cached data instantly; refresh silently); offline (a Banner "Offline. Notes will sync later.", and capture still works via the queue); restricted page (§8.9).

### 8.5 Context menu
**Purpose:** save the selected text or the page without opening anything (EXT-02).
| Context | Menu title (exact) | Result |
|---|---|---|
| Text selected | `Save "%s" to Cue` | Note body = the selection. Source = page URL + title. `source_fragment` = a text-fragment link. Goes to the Inbox. |
| Page (no selection) | `Save page to Cue` | Note body = the page title. Source = URL. Goes to the Inbox. |
| Toolbar icon (right-click) | `Open side panel` · `Open Cue web app` | — |

- Chrome truncates `%s` itself.
- **Feedback:** **no system notification.** The toolbar badge flashes **"✓"** for **2 s** (`chrome.action.setBadgeText`, background `--success` #15803D, white text), then returns to the page count (§8.6).
- **Offline:** saved to the capture queue, and the badge flash shows **"…"** (grey #52525B) for 2 s.

### 8.6 Toolbar badge
**Purpose:** a quiet signal that you have notes on this page (EXT-04).
- **Text:** the number of notes linked to the current page or site: `1`…`99`, then `99+`. **Hidden at 0** (empty string), never "0". It is set per tab (`tabId`).
- **Colours:** background `--primary` **#4F46E5**, text **#FFFFFF** (`setBadgeTextColor`).
- **Tooltip** (`setTitle`): `Cue: 3 notes on this page` (singular: `Cue: 1 note on this page`). With no notes: `Cue`.
- **Missed reminders do NOT go on the badge** (to keep it calm). They show in the popup header (⏰ 2).
- **Temporary flashes:** ✓ after a save (§8.5), "…" while queued offline. Each lasts 2 s.

### 8.7 Notifications
**Purpose:** reminders on this computer, only during this device's office hours (REM-06, REM-08).
```
┌───────────────────────────────────────┐
│ ● Cue: Pay electricity bill           │
│ 6:00 PM                               │
│ [Done]  [Snooze 1 h]                  │
└───────────────────────────────────────┘
```
- Shown only **within this extension device's office-hours schedule**. Outside it, **nothing** is shown and the reminder goes to Missed (the popup header shows ⏰ n).
- **Grouping:** 2 or more due in the same check → one notification **"Cue: 3 reminders"** with body "Pay bill, Call Ravi, +1". Clicking it opens the web `/today`.
- **Buttons:** **Done**, **Snooze 1 h** (`chrome.notifications` buttons). Clicking the body opens the note in the web app.
- `requireInteraction: true` **only for Important** reminders. Others auto-dismiss.
- Locked notes: "Cue: 🔒 Locked reminder".
- **Never** notify about saves, sync, sign-in or errors.
- Remember what was shown (`chrome.storage`) so the same reminder occurrence is never shown twice. Clear a notification when it's acknowledged on another device (next tick).

### 8.8 Keyboard shortcuts
| Shortcut | Action |
|---|---|
| `Alt+Shift+N` | Open the popup (`_execute_action`) |
| `Alt+Shift+S` | Open the side panel |
| `Enter` (in popup / side panel capture) | Save |
| `Shift+Enter` | New line |
| `Esc` | Close the popup (the draft is kept) |

- Users can change shortcuts at `chrome://extensions/shortcuts`. Mention this in the popup ⋯ menu as "Keyboard shortcuts…" (opens that page).
- The popup shows the hint "Alt+Shift+N" in its empty-state tooltip on first use.

### 8.9 States
| State | Popup | Side panel | Copy |
|---|---|---|---|
| **Signed out** | The "Sign in to Cue" screen (§8.2) | Same message + Button | "Capture thoughts from any page." |
| **Loading** | No skeleton. The field is usable immediately; header counts fill in when ready | Cached lists show instantly and refresh silently | — |
| **Offline** | Save works and is queued | Banner + capture works | "Saved offline. Will sync." / "Offline. Notes will sync later." |
| **Save error (online)** | An inline red line under Save | Inline under the field | **"Couldn't save. Retry"** (Retry is a link) |
| **Session expired** | The signed-out screen | Same | "Please sign in again." |
| **Rate or quota limit** (server 429) | Inline | Inline | "Cue is busy. Your note is queued and will be saved soon." (queued) |
| **Restricted page** (`chrome://`, Chrome Web Store, PDF viewer, `file://` without access) | Source Chip hidden; a muted line | "This page" tab shows the message | **"Cue can't attach this page, but you can still write a note."** |
| **Long title / long selection** | The Chip truncates to 1 line (ellipsis), with a full-text tooltip | Snippet clamped to 2 lines | — |
| **Dark mode** | Follows `prefers-color-scheme` using the §3 dark tokens | Same | — |

## 9. States checklist

**Every screen and component must handle all of these before its task is "done":**

| State | What to do |
|---|---|
| **Loading** | Show nothing for the first 1 s (keep previous content). After 1 s, show a Skeleton (§5.16) shaped like the content. Never a full-screen spinner. Buttons show their own spinner. |
| **Empty** | Show the §4.4 EmptyState for that screen, and hide headers, filters and toolbars. Never show "0". |
| **Error** | Inline message near the cause (forms) or a Toast with `Retry` (actions). Log the details; show the §4.5 copy. Never show raw error text or codes. |
| **Offline** | **Mobile:** everything keeps working; show the offline Banner once; writes go to the outbox. **Web:** offline Banner + **editing disabled** (inputs read-only, action buttons disabled with the tooltip `You're offline`). **Ext:** captures are queued with `Saved offline. Will sync.` (D-39). |
| **Very long text** | Titles truncate (2 lines on mobile, 1 line on web) with an ellipsis. Chips never wrap the row (overflow hidden). The editor scrolls. URLs truncate in the middle (`example.com/…/page`). Group names max 32 characters. |
| **200% font scale** | Rows grow taller, chips wrap to a 2nd line, BottomNav labels may truncate but never overlap, sheets scroll. Test on Android at the largest font + display size. Nothing gets clipped. |
| **Dark mode** | Only token swaps (§3.1). Shadows → border + `card` surface. Check every screen in both modes. QR codes stay black on white. |
| **Locked notes** | Show `Locked note` + `lock`, with no preview. Never decrypt in lists, search results, notifications or widgets. Opening one requires an unlock (biometric / vault PIN). |
| **No network at first launch** | Mobile onboarding needs the network for signup. Show `Connect to the internet to create your account.` + `Try again`, and keep the typed username. Web/ext login shows the offline Banner and a disabled Continue. |

---

## 10. Accessibility checklist

Target: **WCAG 2.2 AA** on web and extension, and the equivalent Android accessibility guidelines.

- [ ] **Contrast:** text ≥ 4.5:1 (≥ 3:1 for text ≥ 18 px regular / 14 px bold and for icons/UI boundaries). Use only §3.1 pairs.
- [ ] **Focus:** a visible **`focus-visible` 2 px ring** in `ring`, offset 2, on every interactive element. The focus is never hidden by sticky headers or banners (WCAG 2.4.11). No focus ring on mouse click.
- [ ] **Targets:** mobile ≥ **48 × 48 dp** with 8 dp spacing. Web ≥ **44 × 44 px** for primary touch-capable targets, and ≥ 24 × 24 px for dense desktop controls (WCAG 2.5.8).
- [ ] **Labels:** every icon-only button has `aria-label` / `accessibilityLabel`. Inputs have labels (visible or `aria-label`).
- [ ] **Rows** expose one combined label (e.g. `"Pay bill. Todo. Reminder Tuesday 6 PM."`). Checkboxes are labelled `Mark 'Pay bill' done`.
- [ ] **Gestures have alternatives:** swipe actions are also in the long-press sheet **and** TalkBack custom actions. Drag is never required.
- [ ] **Colour is never the only signal** (overdue = colour + word + icon; swipes = colour + icon + label).
- [ ] **Reduce motion:** honour Android "Remove animations" and `prefers-reduced-motion` (cross-fades only, no springs or fling).
- [ ] **Font scaling:** support Android font scale up to 200% and browser zoom up to 200% with no clipping or overlap. Never set `allowFontScaling={false}`.
- [ ] **Logical focus order:** web — sidebar → list header → quick-add → list → detail panel. Opening the detail panel moves focus to its title; closing it returns focus to the row. Sheets and dialogs trap focus and restore it on close.
- [ ] **Toasts and live updates** are announced politely (`aria-live="polite"` / `accessibilityLiveRegion="polite"`). Errors that block an action use `assertive`.
- [ ] **Keyboard:** everything on web and in the ext is reachable by keyboard. Shortcuts never fire while typing in an input (except `Ctrl+K` and `Esc`). The `?` cheat sheet is available.
- [ ] **Screen readers:** headings use real heading roles (`h1` page title, `h2` sections). Lists use list semantics. The BottomNav uses the tab role with the count.
- [ ] **Haptics** are supplemental only; they never carry meaning alone. Respect the system setting.
- [ ] **Secure screens** (`FLAG_SECURE`: approval, recovery codes, vault) remain readable by TalkBack.

---

**Sources (Part A):**
- Material 3 components and guidelines:
  - [FAB](https://m3.material.io/components/floating-action-button/guidelines)
  - [Navigation bar](https://m3.material.io/components/navigation-bar/guidelines)
  - [Snackbar](https://m3.material.io/components/snackbar/guidelines)
  - [Bottom sheets](https://m3.material.io/components/bottom-sheets/guidelines)
  - [Motion](https://m3.material.io/styles/motion/overview)
  - [Date pickers](https://m3.material.io/components/date-pickers/guidelines)
- Material 3 Expressive research: https://design.google/library/expressive-material-design-google-research
- NN/g:
  - [Empty states](https://www.nngroup.com/videos/empty-states-in-application-design-guidelines/)
  - [Skeleton screens](https://www.nngroup.com/articles/skeleton-screens/)
- Todoist:
  - [Natural-language dates](https://www.todoist.com/help/articles/introduction-to-dates-and-time-q7VobO)
  - [Keyboard shortcuts](https://todoist.com/help/articles/use-keyboard-shortcuts-in-todoist-Wyovn2)
- Superhuman, command palette rules: https://blog.superhuman.com/how-to-build-a-remarkable-command-palette/
- Linear:
  - [UI redesign](https://linear.app/now/how-we-redesigned-the-linear-ui)
  - [Design refresh](https://linear.app/now/behind-the-latest-design-refresh)
- Number matching:
  - [Microsoft](https://learn.microsoft.com/en-gb/entra/identity/authentication/how-to-mfa-number-match)
  - [Google prompts](https://support.google.com/accounts/answer/7026266)
- Google Keep reminder presets: https://support.google.com/keep/answer/3187168
- Recovery codes UX: [GitHub 2FA recovery](https://docs.github.com/en/authentication/securing-your-account-with-two-factor-authentication-2fa/configuring-two-factor-authentication-recovery-methods)
- Accessibility:
  - [WCAG 2.2 what's new](https://tetralogical.com/blog/2023/10/05/whats-new-wcag-2.2)
  - [Android accessibility](https://developer.android.com/guide/topics/ui/accessibility/apps)
- Chrome extension limits:
  - [action (popup size, badge)](https://developer.chrome.com/docs/extensions/reference/api/action)
  - [sidePanel](https://developer.chrome.com/docs/extensions/reference/api/sidePanel)

## 11. Implementation guide for low-tier models

> Library facts were verified on 2026-10-16 from the npm registry, the Expo SDK 57 `bundledNativeModules.json`, and official docs (D-50).
> **On mobile, always install with `npx expo install <pkg>`.** The npm "latest" is often newer than what Expo SDK 57 supports.

### 11.1 Libraries per platform

| Purpose | Web app + extension | Android app (Expo SDK 57) |
|---|---|---|
| Styling engine | **Tailwind CSS v4** (`tailwindcss` + `@tailwindcss/vite`) | **NativeWind 4.2.x** (uses **Tailwind 3.4**; v5 is still an RC, so don't use it yet) |
| Component kit | **shadcn/ui** (CLI v4), created with the **Radix** base. ⚠ The shadcn CLI defaults to *Base UI* since July 2026, so choose **Radix** explicitly | **React Native Reusables** (shadcn for RN), using its **`nativewind`** registry |
| Icons | `lucide-react` | `lucide-react-native` (+ `react-native-svg`, pinned by Expo) |
| Toasts | `sonner` (shadcn `Sonner`) | `sonner-native` |
| Command palette | `cmdk` (shadcn `Command`) | — |
| Bottom sheet | shadcn `Sheet` (side panels on web only) | `@gorhom/bottom-sheet` v5 (test after every Reanimated upgrade) |
| Gestures and animation | CSS transitions | `react-native-gesture-handler` **~2.32** (`ReanimatedSwipeable`, **not** the legacy `Swipeable`, **not** v3) + `react-native-reanimated` **4.5** (Expo pins) |
| Lists | plain lists (virtualise only if more than 500 rows) | `@shopify/flash-list` v2 (Expo pin) |
| Date/time picker | shadcn `Calendar` + time `Input` | `@react-native-community/datetimepicker` (Expo pin) |
| Haptics | — | `expo-haptics` |
| Font | **Inter** via `@fontsource-variable/inter` (bundled locally; never a CDN in the extension because of the MV3 CSP) | **Inter** via `@expo-google-fonts/inter`, embedded with the `expo-font` config plugin |
| Utilities | `clsx` + `tailwind-merge` (`cn()` from shadcn) | the same `cn()` from Reusables |

**Rejected (don't use):**
- Tamagui (v3 is brand new, with its own styling system)
- gluestack v5 (needs NativeWind v5 RC)
- React Native Paper (Material look; doesn't match the web)
- Material You dynamic colour (D-51)
- the `web-push` npm package (Node-only)

### 11.2 Design tokens pipeline (one source → all platforms)
```
packages/ui-tokens/
├─ src/tokens.ts        ← the ONLY place token values are written (copy them exactly from §3)
├─ scripts/build.ts     ← generates the three files below (run: pnpm --filter @cue/ui-tokens build)
├─ dist/web.css         ← shadcn-style :root{…} and .dark{…} with HEX values  → imported by web + extension
├─ dist/native.css      ← :root{…} and .dark:root{…} with HSL triples ("239 84% 67%") → NativeWind global.css
└─ dist/preset.js       ← Tailwind 3 preset for NativeWind: colors: { background: 'hsl(var(--background))', … },
                          borderRadius, fontFamily (Inter_400Regular / Inter_500Medium / Inter_600SemiBold)
```
- **Web/extension:** `@import "@cue/ui-tokens/web.css";` then map the variables with Tailwind v4 `@theme inline { --color-background: var(--background); … }`. This is the shadcn v4 pattern.
- **Mobile:** NativeWind `tailwind.config.js` uses `presets: [require('nativewind/preset'), require('@cue/ui-tokens/preset')]`, and `global.css` imports `native.css`.
  - Add the shared package paths to `content`, and the workspace root to Metro's `watchFolders`.
- **Colours are stored as HEX** in `tokens.ts`. Don't use `oklch()`, because NativeWind v4 can't parse it reliably. The build script converts hex to HSL for native.
- **Fonts on Android:** load each static weight and map `font-normal` / `font-medium` / `font-semibold` to `Inter_400Regular` / `Inter_500Medium` / `Inter_600SemiBold` in the preset. `fontWeight` alone won't choose the right font file.
- **Dark mode:**
  - Web: the `class` strategy (`.dark` on `<html>`), with System by default.
  - Mobile: NativeWind's `useColorScheme()`, with System by default.

### 11.3 Setup steps (done once, in build task M2-T01)
1. **ui-tokens:** create the package from §11.2 and write `tokens.ts` from §3.1–§3.6 exactly. Write a test that checks the WCAG contrast pairs in §3.1 (≥ 4.5:1 for text).
2. **Web:**
   - Install `tailwindcss @tailwindcss/vite`.
   - Run `npx shadcn@latest init` and **choose Radix** as the base, with the "neutral" base colour. Then replace the generated colour variables with `@cue/ui-tokens/web.css`.
   - Add components: `npx shadcn@latest add button input textarea badge checkbox dialog sheet popover calendar command sonner skeleton switch tabs tooltip dropdown-menu separator scroll-area alert`.
3. **Extension:**
   - In `wxt.config.ts`, set `vite: () => ({ plugins: [tailwindcss()] })`.
   - Import the same tokens CSS and fontsource Inter.
   - Copy (don't re-generate) the shadcn components from `apps/web/src/components/ui`, or move them into a shared `packages/ui-web` if duplication grows.
   - Popup: `body { width: 380px }`.
   - Side panel: `h-screen`, flexible width (min 320).
4. **Mobile:**
   - `npx expo install nativewind react-native-reanimated react-native-safe-area-context react-native-svg`, plus `tailwindcss@3.4` as a dev dependency.
   - Follow the NativeWind v4 install guide (babel preset, Metro `withNativeWind`, `global.css`).
   - Add Reusables components through its CLI with the **nativewind** registry: button, input, textarea, checkbox, badge, switch, separator, dialog, text.
   - `npx expo install @gorhom/bottom-sheet react-native-gesture-handler @shopify/flash-list expo-haptics @react-native-community/datetimepicker sonner-native lucide-react-native @expo-google-fonts/inter expo-font`.
5. **Verify:**
   - One sample screen per platform shows Button, Input, Chip and a NoteRow in light and dark.
   - Compare it side by side with `docs/07-design/mockups.html`.

### 11.4 Component folders
```
apps/web/src/components/ui/        ← shadcn components (generated; edit only their classes, not their behaviour)
apps/web/src/components/cue/       ← Cue components from §5 built on top: CaptureField, Chip, NoteRow, TodoRow,
                                      ReminderPicker, GroupPicker, NumberApproval, QRBlock, ConflictCompare, EmptyState…
apps/extension/components/{ui,cue}/ ← same structure (copied or shared)
apps/mobile/components/ui/         ← Reusables components
apps/mobile/components/cue/        ← Cue components: FAB, SwipeRow, BottomNav (Expo Router Tabs), CaptureField,
                                      NoteRow, TodoRow, ReminderSheet, ActionsSheet, NumberApproval, RecoveryCodesGrid…
packages/shared/src/ui-logic/      ← platform-free logic used by both: detectReminder(), formatWhen() (§4.3),
                                      relativeTime(), pageKey() display, string tables (§4.2)
```
- **One component = one file**, named exactly as in §5 (e.g. `NoteRow.tsx`).
- Props are typed. No inline colours or sizes: use Tailwind classes that map to tokens (`bg-primary`, `text-muted-foreground`, `rounded-lg`, `p-4`).

### 11.5 Do / Don't for agents
**Do:**
- Use only token classes (`bg-background`, `text-foreground`, `border-border`, `text-overdue`, …).
- Copy the structure and spacing from `mockups.html`.
- Use the exact copy in §4.2 / §4.4 / §4.5.
- Handle every state in §9.
- Test in light and dark mode and at 200% font size (Android).
- Add `aria-label` (web) and `accessibilityLabel` (RN) to icon-only buttons.
- Use `cn()` to merge classes.
- Keep screens thin: data hooks in `hooks/`, UI in `components/`.

**Don't:**
- Hex colours in components.
- New font sizes.
- New icons outside §3.9.
- Spinners for loads under 1 s.
- Confirmation dialogs for normal deletes (use Undo).
- Extra tabs or buttons not in the spec.
- Material/Paper components.
- `oklch()` on mobile.
- Shadow DOM for popup or side panel.
- Loading fonts from a CDN in the extension.
- Installing native libraries with `npm i` on mobile.

### 11.6 Task → design sections map
When building a UI task, read these sections first (plus §0, §3 and §9 for every UI task).

| Task | What | Read |
|---|---|---|
| M0-T07 | Web skeleton / PWA manifest | §2.2, §3.1 |
| M0-T08 | Mobile skeleton / app icon and splash | §2.2, §3.1 |
| M0-T09 | Extension skeleton / icons | §2.2 |
| M1-T11 | Onboarding (3 screens) | §6.13, §5.26, §5.1, §5.4, §4.2 |
| M1-T13 | Notes list + conflict banner | §6.3, §6.17, §5.8, §5.14, §5.29 |
| **M2-T01** | **UI foundation** (tokens, kits, fonts, icons) | **§3 (all), §11.1–§11.5, §5 (install list)** |
| M2-T02 | Tabs, fast capture, Inbox | §6.1–§6.4, §5.3, §5.5, §5.8, §5.15, §5.17, §5.18 |
| M2-T03 | Editor, todos, groups, trash | §6.5–§6.7, §5.6, §5.7, §5.9, §5.23, §4.4 |
| M2-T04 | Swipe triage | §6.3, §6.10, §6.21, §5.10, §5.11, §5.13 |
| M2-T05 | Voice capture | §6.4, §5.5 |
| M2-T06 | QS tile | §6.19 |
| M2-T07 | Share target | §6.4, §5.6 (source chip) |
| M2-T08 | Home widget | §6.19 |
| M2-T10 | Phone-approval login | §6.11, §5.24, §7.11 |
| M2-T11 | QR login | §6.12, §5.25, §7.11 |
| M2-T13 | Web app | §7.1–§7.9, §7.15, §5.19, §5.20, §5.21, §4.2 |
| M3-T03 | Date suggestions | §5.5, §6.4, §7.5 |
| M3-T05 | Notification actions | §6.20 |
| M3-T06 | Nag + alarm screen | §6.20 |
| M3-T07 | Permission primer + battery guide | §6.14 |
| M3-T08 | Web Push | §7.16 |
| M3-T10 | Reminder UI everywhere | §6.5, §6.9, §6.15, §6.16, §7.10, §7.12, §7.13, §5.22 |
| M4-T01 | Extension sign-in tab | §8.3, §7.11 |
| M4-T02 | Popup + capture queue | §8.2, §8.8, §8.9 |
| M4-T03 | Context menu | §8.5 |
| M4-T04 | Badge | §8.6 |
| M4-T05 | Side panel | §8.4 |
| M4-T06 | Extension notifications | §8.7 |
| M5-T02 | Mobile search | §6.8, §5.20 |
| M5-T03 | Web/extension search | §7.9, §8.4 |
| M6-T01 | App lock | §6.15 (Security), §6.18 |
| M6-T02 | Devices + revoke | §6.15, §7.12 |
| M6-T03 | TOTP | §7.11 (authenticator fallback), §6.15 |
| M6-T04 | Recover account | §6.13 (same layout; screen "Recover account": username + one recovery code), §5.26 |
| M6-T05 | Locked notes vault | §6.18, §7.17 |
| M6-T06 | Export | §7.18 |
| M6-T08 | Keep audio | §6.4, §6.15 (Data) |
| M2-T14, M3-T11, M4-T07, M6-T09 | Acceptance | §9, §10 |

### 11.7 Definition of done for any UI task
- [ ] It matches the wireframe and spec in its sections, and the mockup if one exists.
- [ ] It uses only §3 tokens and §5 components. No hard-coded colours or sizes.
- [ ] Every state in §9 is handled: loading, empty, error, offline, long text, dark mode.
- [ ] Accessibility (§10): labels, focus ring (web), 48 dp targets (mobile), 200% font (mobile).
- [ ] The copy is exactly from §4.
- [ ] It was tested on the owner's Android phone (Oppo/Vivo/Realme/OnePlus) and in Chrome.

## 12. References

- **UI libraries (verified 2026-10-16):**
  - [Expo SDK 57 changelog](https://expo.dev/changelog/sdk-57) · [Expo SDK 57 pinned versions](https://github.com/expo/expo/blob/sdk-57/packages/expo/bundledNativeModules.json)
  - [NativeWind install](https://www.nativewind.dev/docs/getting-started/installation) · [NativeWind v5 (RC)](https://www.nativewind.dev/v5)
  - [React Native Reusables](https://github.com/founded-labs/react-native-reusables)
  - [shadcn/ui changelog](https://ui.shadcn.com/docs/changelog) · [shadcn Tailwind v4](https://ui.shadcn.com/docs/changelog/2025-02-tailwind-v4)
  - [WXT FAQ](https://wxt.dev/guide/resources/faq.html)
  - [@gorhom/bottom-sheet](https://github.com/gorhom/react-native-bottom-sheet)
  - [Uniwind (alternative)](https://uniwind.dev)
- **Mobile UX research:**
  - [Drafts design guide](https://blakecrosley.com/guides/design/drafts) · [Things Magic Plus](https://culturedcode.com/things/support/articles/2803582)
  - [Todoist dates](https://www.todoist.com/help/articles/introduction-to-dates-and-time-q7VobO) · [Todoist swipe actions](https://todoist.com/help/articles/360016892459)
  - [Proximus swipe actions](https://dsdev.proximus.com/pagesDS/20__Components/Swipe_actions/)
  - [NN/g empty states](https://www.nngroup.com/videos/empty-states-in-application-design-guidelines/)
  - [Google Keep reminders](https://support.google.com/keep/answer/3187168)
  - [Microsoft number matching](https://learn.microsoft.com/en-gb/entra/identity/authentication/how-to-mfa-number-match) · [Google prompts](https://support.google.com/accounts/answer/7026266)
  - [Android notification permission](https://developer.android.com/develop/ui/views/notifications/notification-permission) · [Android 16 behaviour changes](https://developer.android.com/about/versions/16/behavior-changes-16) · [Android accessibility](https://developer.android.com/guide/topics/ui/accessibility/apps)
  - [M3 Expressive research](https://design.google/library/expressive-material-design-google-research)
  - [M3 FAB](https://m3.material.io/components/floating-action-button/guidelines) · [M3 navigation bar](https://m3.material.io/components/navigation-bar/guidelines) · [M3 snackbar](https://m3.material.io/components/snackbar/guidelines) · [M3 bottom sheets](https://m3.material.io/components/bottom-sheets/guidelines)
  - [GitHub recovery codes](https://docs.github.com/en/authentication/securing-your-account-with-two-factor-authentication-2fa/configuring-two-factor-authentication-recovery-methods)
- **Web and extension UX research:**
  - [Linear UI redesign](https://linear.app/now/how-we-redesigned-the-linear-ui) · [Linear design refresh](https://linear.app/now/behind-the-latest-design-refresh)
  - [Superhuman command palette](https://blog.superhuman.com/how-to-build-a-remarkable-command-palette/)
  - [Todoist keyboard shortcuts](https://todoist.com/help/articles/use-keyboard-shortcuts-in-todoist-Wyovn2) · [Things features](https://culturedcode.com/things/features/)
  - [NN/g skeleton screens](https://www.nngroup.com/articles/skeleton-screens/) · [WCAG 2.2 overview](https://tetralogical.com/blog/2023/10/05/whats-new-wcag-2.2)
  - [chrome.action](https://developer.chrome.com/docs/extensions/reference/api/action) · [Add a popup](https://developer.chrome.com/docs/extensions/develop/ui/add-popup) · [chrome.commands](https://developer.chrome.com/docs/extensions/reference/api/commands) · [chrome.sidePanel](https://developer.chrome.com/docs/extensions/reference/api/sidePanel) · [chrome.contextMenus](https://developer.chrome.com/docs/extensions/reference/api/contextMenus)
  - [Notification requireInteraction](https://developer.chrome.com/blog/notification-requireInteraction/)
  - [Notion Web Clipper](https://www.notion.com/help/web-clipper) · [Raindrop extension](https://help.raindrop.io/browser-extension)
