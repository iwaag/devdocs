# Workflow editor p8 — step 4: a Run view organised around the current task

Changed in agdevworld `workflow_editor/`:

- `src/runView.ts`: header, title band, run details, graph card.
- `src/runSide.ts`: the focus card, and the homes of the persistent forms.
- `src/runCommon.ts`: `runTitle`, `topoOrder`, `descendants`.
- `src/canvas.ts`: the canvas takes its height from its graph.
- `src/styles.css`.
- Checks, for the new behavior: `runInteraction.ts`, `runs.ts`, `review.ts`, `docsteps.ts`.

## Layout

- **Header**: the slim dark bar. It holds the breadcrumb (Projects / project / Run / ref, plus an "as committed at" pill in a historical view), the connection pill (● live / not live / history) and Refresh.
- **Title band**: the run's name, a one-line status, and a subline with the workflow, run ref and "fixed snapshot" or "adjusted, r2 in force". On the right are three separate facts:
  - the recorded execution state ("✓ Completed", "In progress", "Nothing can proceed" for a stalled record, "Cancelled");
  - the human decision ("Accepted", "Accepted · cleanup pending", "Rejected", or "No decision" on a completed run);
  - executor activity ("Executor: executing · process alive", "idle, waiting for a person", "held (stopped)", …).

  Connection health is the fourth, in the header. The band also has **Open current definition** and **Run details**.
- **Run details** opens a panel with what used to fill the dark header: input and author, executor and backend, created / started / ended, last update, separate work and documentation counts, policy, the definition in force with the comparison to the current file, and the open or last attempt. It also has the full Active / Waiting / Ready / Failed / Blocked / Cancelled / Decision lines and the note on what "running" means.
- **Main column**: the **focus card**, then the **graph card**.
  - The graph card heading shows "Run workflow", "Fixed snapshot · rN in force", "Read-only", and "Work 3/5 completed · Documentation 4/7 completed".
  - The canvas is sized to the graph, between 300 and 620 px tall, once, when first framed. Its initial view centres on the current position (step 3).
  - The legend lists only the states present in this run.
- **Side pane**: "Recording as", the selected node (read-only, now with its recorded outcome first and "Added in revision r2" where it applies), and the records sections. Step 5 makes these collapsible.

## Run title and summaries

Only recorded data is used, and no model is called.

- **Title**: the latest presentation's title when the run has one ("Night mode for the settings page"); otherwise the workflow name.
- **Subline**: the workflow name and the run reference are always there.
- **Status line**: built from the execution: what is in progress, what waits for whom, what failed, then "work n of m completed · documentation n of m".
- **Recorded outcomes**: each one is labelled with its step and listed in dependency order. When the workflow has a result-confirmation node, the outcomes of its descendants appear apart under "After Human check". The cleanup's "Stopped the review server" therefore never reads as the delivered result, even though it is the latest event.
- **No suitable summary**: the card shows the status and the links (Original request, Reports & files, Read final report when `report.md` exists).

## The focus card, by situation

The kind is chosen from the record in this order. Every condition the chosen kind does not cover still gets a line: "also held", "Failed: …".

| Situation (fixture run) | Heading and content | Controls |
| --- | --- | --- |
| Historical view (`run-working/at/<commit>`) | "A recorded state": as committed at …; recorded outcomes | none (no forms anywhere in the view; host facts are not read) |
| Cancelled (`run-cancelled`) | Reason, who, when; recorded outcomes; nodes started then cancelled ("may need cleaning up by hand") | **Stop p1** for a presentation still recorded available |
| Open plan / ordinary question (`run-plan`, `run-branch`) | "A question waits for your answer" / "2 questions wait…"; each question with its document link (`plan.md`) | each question's one **answer form** |
| Open result confirmation (`run-review`) | "Ready for your review"; **Open the review: <title>** with availability; the question with its presentation link | its answer form. The lead says the answer is recorded and that the executor records the decision when it takes the answer up |
| Answer recorded, not taken up (`run-answered`) | "Your answer is recorded"; the answer receipt and "Next move: Sample Agent" | "Record another answer" opens the same form |
| Held (`run-held`) | "Held (stopped)": who, when, why; "nothing runs until resumed with an instruction" | the **Resume** form (instruction required) |
| Failed / blocked (`run-failed`) | "Verify failed": each failure with its reason; the blocked work by name (documentation steps counted) | **Adjust future work** (opens the side form), **Cancel run** |
| Accepted, cleanup pending (`run-accepted`) | "Accepted — cleanup is still running": from which question; Clean up in progress; the executor line; recorded outcomes. Not labelled completed | **Stop** |
| Completed (`run-completed`, `run-tidy`) | "Completed and accepted" (who, from which question and answer) or "Completed — no decision recorded"; the stopped presentation with its title; recorded outcomes | **Read final report**, Original request. Without a decision: the evidence field and **Accept / Reject / Cancel run** |
| Work in progress (`run-working`) | Active steps with their latest note; the executor line from this host ("attempt at-0002: process 4242 alive"); "Nothing is needed from you now" | **Stop** |
| Not started / idle (`run-large`) | Waiting for the executor; what is ready | — |
| No usable record (`run-broken`) | "No usable record"; the banner says why; graph and details hidden | — |

A question answer is still only an answer: it does not decide or resume. Resume needs its instruction and a hold, and Accept needs a completed execution. Every server-side precondition is unchanged. No Pause, "run this node only", or automatic presentation restart was added.

## Persistent controls and their homes

The answer forms, the resume instruction and the decision evidence are still created once (`Owned`, p7/ex2). The answer forms sit inside the question items. Their homes now depend on the state:

- **Questions**:
  - open and answered questions are in the focus card, oldest first;
  - taken-up and withdrawn questions are records in the side pane, newest first, under "Questions (n resolved)".

  An answered question shows its receipt. Its form comes back on "Record another answer", or stays while it holds a draft.
- **Resume form**: in the focus card while held. The Execution section links to it.
- **Decision form**: in the focus card when a completed run has no decision; otherwise in the side Result section.

A control is moved only while it holds no draft and does not contain the focus. Moving a focused element would blur it, and a draft must stay where it is being written. A form whose operation stopped applying stays put, read-only after any composition ends, with its text and the reason.

## Checks

| Check | Result |
| --- | --- |
| `npm run check` | pass (211 tests) |
| `checks/runInteraction.ts` | **pass, 39 assertions** (p7/ex2 had 38). New: after a recorded answer clears its draft, the question shows its receipt instead of an empty form, and "Record another answer" brings back the same control, focused and empty. The historical view counts controls across the whole run body (focus card included), not only the side pane |
| `checks/runs.ts --repeat 5` | pass. It reopens the answered question's form before the stale-view answer, and reads "does not wake the executor" anywhere in the run body |
| `checks/review.ts` | pass. After the stop, the "no longer served" line is in the focus card |
| `checks/docsteps.ts` | pass (the coordination title gained a state suffix) |
| `checks/conversation.ts` | pass |

The fixture runs were captured at 1440 × 1000 and looked at state by state, along with `run-branch` at 1280 × 800. That review led to five fixes:

- blocked lists show authored work only;
- a recorded reason that already ends with a stop no longer gets a second period;
- a stalled execution reads "Nothing can proceed", not "Stopped", which collided with "Held (stopped)";
- the cancelled card no longer shows the presentation notice twice;
- a global `[hidden]` rule was added. The actor row's `display: flex` had overridden `hidden` in historical and broken views.

Left for step 5: the side sections are still always expanded, so the side pane is long. Their labels already carry counts and problems (presentations live / not served / stopped, files missing, publication problems, coordination needing attention).
