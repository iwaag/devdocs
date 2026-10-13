# Workflow editor p8/ex2 — final report

[braindump.md](braindump.md) · [plan.md](plan.md) · step reports [0](report0.md) [1](report1.md) [2](report2.md)
[3](report3.md) [4](report4.md) [5](report5.md) [6](report6.md) [7](report7.md)

**Implemented and deployed on 2026-10-13:**

- agdevworld **`ce3387d`** (previous `6bdcd3b`);
- agautolab **`2e74777`** (the wfrun guide);
- pj-agdev **`4da29ae`**.

**freework was added to rts-vs-bot, ppt and agsfx** (pushed: `1eb3b7a`, `8c972d0`, `ce6da58`).

## What the braindump asked, and what is there now

| Braindump | Now |
| --- | --- |
| Progress was hard to feel: no fine logs needed, but the bar alone is too slow and weak; e.g. a growing turn count | Under the run's progress, one line: **"Working on Implement · 24 turns this session · last activity 8 s ago"**. The age ticks, and a new turn makes the line pulse once. Waiting, queued, held, a lost process, another host and a lost connection each say what they are instead of a number. Turns are counted from the agent's own log on this host, once, without sending log text. A run whose plan grew shows "3 steps done" without a fraction |
| Creating a new workflow should be visible in Simple | **Create workflow** sits in the workflow list's head in both modes, and an empty list offers it too. It asks for a name; the id, the template and the documents are behind More settings. Errors appear beside their field, and the input survives reloads and mode switches |
| freework: a braindump and one talk, the workflow made and run inside the conversation | Every project has the basic workflow **Free work**: one talk, custom documentation with a final report. The run adds the work agreed in the conversation as ordinary adjustments, does it, and talks again when needed. Nothing branches on its name |

What the plan's agreements required under the hood:

- **Documentation settings chosen one by one** (`policy: custom` with `plan`, `reports`, `final`), shared by the
  compiler (v3), validation, the reducer, the brief and the views. `default` and `manual` are unchanged.
- **An adjustment at a node's completion** (`run.adjust completing`, `wfe run adjust --completing`): the run can
  grow at the moment its last node completes, with both changes recorded or neither, and the history checked
  on every read.
- **One placement mechanism** for new and existing projects (`server/basics.ts`, `wfe workflow basics`). It
  resolves bindings and execution from the project and never overwrites.
- **The common guide** says how a run's work grows from the conversation.

## Evidence

- **Tests:** 255/255 (`npm run check`, also run inside the release). Of these, 38 are new in this episode:
  activity, the activity line, custom policy, growth, basics, creation.
- **Browser:** `checks/freework.ts` passes 68 assertions in Chromium. In WebKit, 3 of 4 runs passed; the first
  WebKit launch ended without its summary, and the failure did not repeat. Nine existing browser checks pass.
  `checks/growth.ts` is a CLI rehearsal across three attempts.
- **A real agent** (autolab, Claude Code) ran three freework runs in a private area, for $2.64.
  - All three completed and were usable, and they exercised both boundary forms.
  - Two findings became fixes, each confirmed once by the next run:
    - the edge format is now in `wfe run help adjust`, and the result notes an added node nothing leads to and
      unknown node fields;
    - the guide now says that agreed work goes in a node of its own.
- **Production:** all 11 runs are `usable` under the new code and the release.

## Defects found and fixed on the way

- **`wfe run revisions <run> --show r1`** printed the file name instead of the original definition. Broken
  since p7; this is the documented start of an adjustment draft.
- **A retransmitted adjustment** left an unreferenced revision file.
- **The run view's answer messages** said the executor would not be woken. Since p6 it is.
- **The request form** defaulted to a workflow with validation errors.
- **The creation form:** a template list arriving late replaced the person's documentation choice.

## For the Developer

- **Try it.** Open a project in agdevworld (Simple). Choose **Free work** in the request form, write what you
  want, and request it. The run's talk will ask you; answer in the run view; it continues by itself.
- **`agent/review.md`.** In rts-vs-bot, freework's resolved guide is `agent/review.md`, the only guide every valid
  workflow lists, and it is `improve-game`'s review guidance. Edit
  `pj-rts-vs-bot/devdocs/workflows/freework.yaml` if it should be different. Your edits are never overwritten.
- **Zulip:** three new channels, `wf-<project>.freework`, made by the existing per-workflow notification. They
  have no intake.
- **Not verified:** native macOS IME on the new forms (synthetic input only). Save as a reusable workflow is
  not offered (out of scope).
- **Rollback:** `wfe-release.sh rollback .local/agdev`, plus agautolab's guide back to `f1c466d`. The details
  and their consequences for the new records are in [report7.md](report7.md).
