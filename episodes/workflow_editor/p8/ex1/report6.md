# Workflow editor p8/ex1 — step 6: validating the reduction and the continuity

**Source**: agdevworld `5d4519c`. **Deployment**: none. The installed release is `8cda5c9`, with `a571ae2` as the previous release and no open attempt (`wfe-release.sh status`).

## Checks

All runs were against private fixtures and services. None touched the operating registry.

| Check | Result |
| --- | --- |
| `npm run check` (tsc, unit tests, vite build) | **pass, 211 tests** |
| `checks/displayMode.ts` (new, Simple) | **pass, 96 assertions** |
| `checks/simplerUi.ts` (p8, Detail) | **pass, 71**. Three expectations were updated for intended changes (report4). |
| `checks/runInteraction.ts` | **pass, 39** |
| `checks/runs.ts` | **pass, 33** |
| `checks/docsteps.ts` | **pass, 35**. The check reads a card and its report icon again when a background redraw lands between the two reads. |
| `checks/appearance.ts` | **pass, 15** |
| `checks/review.ts` | **pass, 22** |
| `checks/nodes.ts` | **pass, 20** |
| `checks/conversation.ts` | **pass, 12** |
| `checks/exec.ts`, `checks/backend.ts` | **pass, 20 and 5** |
| `checks/agdev.ts` (Dashboard and Project Editor against a disposable Gitea in Docker) | **pass, 13** |
| `checks/e2e.ts` through `checks/ui.ts` (dev service on 8095 and Vite on 5175, both started for the run and stopped after; reseeds only the ignored dev fixture) | **pass, 45** |
| `checks/setup.ts` | **fails at its first assertion**, the same way on an untouched `8cda5c9` worktree. It is a pre1 check of the old home page; p4's dashboard replaced that page, and the check was not in p8's list either. Not changed. |

**How the existing checks run now.**

- **Detail by default.** `checks/lib.ts` opens the existing checks in Detail, because a fresh browser starts in Simple. Only an unset preference is filled, so a check's own switch survives navigation.
- **`docsteps.ts`.** It failed on the untouched `8cda5c9` as well. The cause was the check: a box read across the canvas's redraw returns null. That is the "null bounding box" noted in p6 and p7. The check now retries the read and passes, which confirms the cause.
- **Gitea container.** One leftover Gitea test container, `wfe-test-gitea-69299-59874`, has been running since 2026-10-12 21:24. It predates this session and was left alone.

## Captures

[image_after/](image_after/) holds 96 captures: 24 screens in Simple and in Detail, at 1440 × 1000 and at 1280 × 800. [image_after/README.md](image_after/README.md) gives the conditions and pairs each screen with its p8 capture.

- **Screens covered**: Dashboard, Project, a project with a long Japanese and English name, the definition with and without a selection, a definition with long names, a large graph, and User nodes.
- **Run states covered**: working, open question, two questions, a Japanese question, live review, answer awaiting take-up, accepted while cleaning up, completed, completed without a decision, held, failed, cancelled, historical and disconnected.

## The visual reduction, measured

The same states were measured with p8 (`8cda5c9` built in a worktree) and with this source, at 1440 × 1000. The counts cover text visible on the first screen. A graph's text counts at its rendered size.

| Screen | Characters (p8 → Simple) | Characters below 13.5 px (p8 → Simple) | Text blocks below 13.5 px (p8 → Simple) | Visible controls (p8 → Simple) |
| --- | --- | --- | --- | --- |
| Dashboard | 2214 → 420 | 1930 → 106 | 38 → 4 | 25 → 10 |
| Project | 1417 → 812 | 1072 → 221 | 48 → 20 | 20 → 20 |
| Workflow definition | 955 → 374 | 841 → 60 | 43 → 6 | 29 → 24 |
| Run, completed | 2092 → 526 | 1203 → 145 | 70 → 28 | 43 → 26 |
| Run, working | 1531 → 585 | 1147 → 136 | 64 → 26 | 35 → 26 |
| Run, live review | 2014 → 619 | 1327 → 144 | 69 → 27 | 38 → 27 |
| Run, failed | 1631 → 684 | 1090 → 241 | 63 → 27 | 36 → 27 |
| User nodes | 1826 → 542 | 1274 → 317 | 18 → 5 | 8 → 8 |

- **Detail keeps p8's content.** It measures within 1–14 % of p8 on every screen. The small increases are the toggle and the sticky header.
- **What Simple still has in small text** is the graph's state pills, form labels, and the side pane's hint.

## The three acceptance criteria

1. **Purpose, situation and next action without small technical text.**
   - Every Simple screen names its object in a 22–32 px heading.
   - The purpose is 15–16 px text: the project's intent, the workflow's first paragraph, or the run's title.
   - The situation is a large icon with a heading: the focus card, the project tiles' status icons, the run badges, the workflow state.
   - The next action is a primary button or form: Open project, Request run, Record answer, Resume, Accept result, Read final report, Open the review, Add node and Save.
   - Commit ids, remotes, branches, paths, backend names, exact times, run references and record sections are absent. `displayMode.ts` asserts this on the project, a completed run and a historical run.
2. **More prominent titles, icons and nodes; fewer small blocks and routine controls.**
   - Small text falls by 75–95 % on every screen, and small text blocks by 55–90 %.
   - Graph cards carry a 34 px icon and a 17 px name. 6 of 8 cards are readable on the completed run, against p8's 4 of 8; the large graph shows 14 of 18.
   - Routine controls are Detail's: Refresh, the workspace switcher, Run details, Open current definition, Compact/Mini, Auto-arrange, the policy chip and the nine side sections.
3. **Detail one click away; a switch keeps the work and the reading context.**
   - The toggle is the last item of a sticky top bar on every screen. It works with one click, with Tab and Space, and with Enter.
   - Its state is announced through `aria-pressed` in a labelled group.
   - Drafts, panels, sections, window and side-pane scroll, file offsets, the selection and the graph's place all survive a switch. `displayMode.ts` asserts this (report5).

## Keyboard, toggle state, IME

- **Keyboard.** Tab reaches the toggle, and Space or Enter switches. A hidden focused control hands the keyboard to the toggle. Contextual entries put the keyboard on their target: the project form, the template form, the inspector heading, the first record section, the adjustment form. All of this is asserted.
- **Toggle state.** `aria-pressed` on both buttons, inside a `group` named "Display". Asserted.
- **Synthetic composition.** A switch requested during a synthetic `compositionstart`/`compositionend` waits for the composition to end. Asserted.
- **Native macOS Japanese IME: not verified.** It needs a person at the keyboard: compose Japanese in an answer, a resume instruction or the adjustment reason, click the toggle mid-composition, then let a background update arrive. This was not done. The synthetic result above is not a substitute for it.
