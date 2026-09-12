# routine_tests p2 ex2 — run study-uspolitics again on the repaired system

## Objective

Run the `study-uspolitics` routine once more, now that the three defects
p2 demonstrated have been repaired ([ex1](../ex1/report.md)), and learn two
separate things from that one run:

1. **Do the same problems recur?** The trial is first a re-measurement of
   ex1's repairs on the routine that exposed the defects, with a real
   autolab mission in the loop — something ex1's live sequence did not
   exercise.
2. **Which way does the researcher take the investigation** when the
   request says as little as possible? The guide already leaves every
   research choice to the researcher; this time the request adds nothing,
   and the interesting result is the direction it chooses and why.

Source: [braindump.md](braindump.md) (renamed from `brandump.md`, content
unchanged). This document plans execution; writing it starts nothing.

One run is the deliverable. It is enough to answer question 1 for the
ordinary path and to give one observation for question 2. A second run is
not part of this exercise; if the developer wants a trajectory rather than a
point, that is a further request after the report.

## What is already in place (read-only, 2026-09-13 JST)

Read this plan with `devpolicy/styles.md`, `devpolicy/terms.md`,
`devdocs/README_DEV.md`, both projects' ignored environment notes,
`pj-clusterintent/nctl/README.md`, the p2 [plan](../plan.md) and
[report](../report.md), and ex1's [problem](../ex1/problem.md) and
[report](../ex1/report.md). Paths are relative to the projects workspace;
`main/` and `publish/` are relative to the study workspace.

- `nctl status --json` is `ok: true` (Nautobot reachable and authenticated,
  intent catalog present, one worker with no pending jobs, all submodules
  clean); `nctl drift --json` reports no errors and `converged: 43`. That is
  infrastructure state, not proof of agent readiness. The agent half: the
  launchd listeners for Front, autolab, forge, the arXiv sage, cagent and
  the agentroom relay are all running, and the relay's routines view is
  live.
- **The routine exists and needs no change.** `#routine-study-uspolitics`
  is on the routine board with `retired: false`; its authority is guide
  **v2, message 6393**; both p2 runs are finished and resolved with no open
  run. Do not post a v3. A guide change is justified only by a failure this
  run demonstrates, and none has been.
- **The workspace is as p2 left it.** `README_PROJECT.md` is at the
  workspace root (still tracked nowhere; a later serving depends on it).
  `main/` is at **`ac2f7d2`** and pushed, holding two method notes and one
  report. `publish/` is at **`d467141`**, one commit ahead of the supplied
  GitHub repository, which is still at `569a8e3`: the developer's push is
  still outstanding. `publish/` is not touched by this exercise.
- **Deployed revisions are ex1's:** pyagag `ed65b4e` installed in every
  consumer (`conversation_context` verified importable from Front's
  environment), agfront `842d7b1`, agautolab `b5259cd` on this host.
  The `agautolab1` VM is not redeployed and is not on this path.
- **Four `workplan-` topics in `#pj-studyuspolitics` are unresolved**
  (setup, both study missions, publish), plus the `✔ retired-…-m6371` one.
  Leave them. Tidying them would remove the exact condition under which
  ex1's remaining limit — a run reusing a delegation topic name another run
  already anchored — can show itself.
- The developer obtained an api.data.gov (FEC) key during p2 and keeps it
  in an ignored location. The researcher was never told and this plan does
  not tell it. If a run asks for one, the guide's human-dependency rule is
  working as intended: supply it in the conversation where it was asked,
  as an ordinary human step, and record it as such rather than as a repair.

## Step 1 — define "the same problems" before the run

Write down, before posting anything, what would count as a recurrence and
where each one is read. The p2 and ex1 numbers are the baseline:

| Problem | p2 | ex1 | Read here from |
|---|---|---|---|
| **C** — Front's first serving of the new conversation answers as if it were empty | 2 of 2 requests | 0 of 16 servings | Front's reply text; the `front` run record's turn count and tool calls |
| **B** — the serving that opens the run delegates on the run's behalf | 1 of 1 (`publish`) | 0 of 3 | the delegation's `[selfnote][rootchat]` note: does it name the run or the requesting conversation; which role the plan callback serves |
| **A** — a plan retirement strands the run's anchor | once (m6371) | recovered in a staged replacement | only if autolab retires a plan; then the replacement's `[replaces]` note and the listener's `serves …` line |
| topic-name collision (ex1 limit) | — | 1 of 3, repaired by `agentchat anchor` | the run's delegation topic name against the four unresolved `workplan-` topics |
| stall with no wake signal (p2 first attempt) | once | not exercised | a task that ends its serving while its own work is still running, with nobody left to speak |
| task resolved without the requester's agreement (p2 step 1) | once | not exercised | whether autolab waits for the run's "done" before resolving |

Also record what ex1 did **not** test and this run does: an autolab mission
end to end on pyagag `ed65b4e`, and a real callback into a `routine_run`
serving that carries the plan text in the prompt rather than only in
`threads/` (ex1's "C repaired the chatlog, not the threads" limit — note
whether the run relays what the thread says, or has to open the file).

Confirm the Front and autolab listener logs are being written before
starting, so the `serves …` lines and run records are available
afterwards. Keep raw logs, transcripts and run records in ignored evidence.

## Step 2 — one request, as small as it can be

Post one request at Front's ordinary entrance, `#front`, in a fresh
`front-*` conversation, as the Developer. This is the same entrance p2 used,
so C is measured on the same shape of conversation.

The request names the routine and asks for one run, and nothing else about
the research:

> Run the routine `study-uspolitics` once and report back here. One run
> only.

It does **not** say where the guide is, what the project holds, that the
researcher should continue from earlier work, what to investigate, whether
the method should change, or which backend to use. All of that is either in
the guide already or is precisely the choice under observation. Use current
execution defaults; the developer named no backend, budget threshold or
schedule.

If Front asks a question before opening the run, answer only what it asked,
in the same conversation, and record it. If the first serving answers as if
the conversation were empty, that is a recurrence of C: record it with the
run record, then re-state the request once so the trial can continue, and
count the re-statement as an intervention.

## Step 3 — observe, without steering

Watch through read-only conversation reads and run-record reads. Do not
post progress prompts into any agent's topic: a post is a paid serving and a
change of speaker, and either one changes what is being measured. Front's
supervising run waits inside its own serving; autolab answers Front, not the
Developer, as long as the Developer stays out of the workplan topic.

Distinguish, as p2 did, a run that is waiting on its delegate from one that
has ended: the `ag-routinerun` block ends the run and cannot be an interim
note.

**If a listed problem recurs**, preserve the evidence first (message ids,
listener lines, the run record) and then decide whether the run can finish
without help. Prefer letting an agent use the tool it now has: ex1 showed
Front runs `agentchat anchor` itself from one sentence of context, so a
pointer is the smallest intervention and is recorded as one. Serve a run
directly, or write a note as another agent, only if nothing smaller lets the
run finish, and record it with the `did X for agent Y — handoff candidate`
note. Do not repair code or guides mid-run; a demonstrated defect goes into
the report and to a follow-up at its owner.

**If the researcher meets a human dependency and asks**, answer in the topic
where it asked. That is the guide's rule working, not a defect; record it as
an ordinary step. If it meets one and does not ask — routes around it, or
stalls without asking — that is a recurrence of the p2 first-attempt
behaviour and is recorded as such.

**Do not steer the research.** No hint about population, source, period,
measure or method reaches any agent from the Developer during the run. If
the researcher's own report asks the developer a question about direction,
record the question; answering it is a further request, after this trial.

## Step 4 — evaluate the two questions separately

### 4a. Did the same problems recur?

Fill the step-1 table with this run's numbers, each with its evidence:
message ids, run-record fields, listener log lines. State plainly which
rows were exercised and which were not (A and the stall only occur on
paths the run may never take). One clean run is one observation, not a
reliability result; say so.

Count interventions, with p2's seven and ex1's one as reference, and keep
"needed no help" apart from "was helped and finished".

### 4b. Which way did the researcher go?

The guide gives the goal, the accumulation rule, and freedom over
everything else. p2's two runs took one slice (35 Senators up in 2026,
committee-level FEC totals) and then went one layer finer into it. The
third run, told nothing, could go many ways; classify what it did, from the
committed material and the mission conversation, not from the agents'
summaries:

- **What it read first**, and whether it named the earlier commits and
  findings before choosing. Whether the choice is grounded in something the
  earlier reports said themselves (an admitted limit, a flagged
  uncertainty, a "left for a later run") or comes from elsewhere.
- **The kind of step**: the same slice one layer deeper again; a wider
  population (the rest of the Senate, the House); a new source or record
  level (itemized transactions, PACs, outside spending); a new question
  asked of the same data; a change to the *method* (scripts, reproducible
  retrieval, indexes, a cross-check that runs on its own); or tidying and
  consolidation. Several may apply; say which and in what proportion.
- **Reuse versus change**: what was kept unchanged with a stated reason,
  what was changed and why, and whether an improvement was exercised or
  only proposed. Retaining a working method with evidence still counts.
- **Reliability**: is a new figure cross-checked against something the
  earlier runs established, as run 2's split reproduced run 1's totals? A
  new slice with no bridge to the old one is a weaker step than one with a
  bridge.
- **The report itself**: does it use evidence to explain a political
  question rather than list records; are observation, interpretation and
  uncertainty kept apart; is missing data still missing.
- **What it said about direction**: whether the report states why this
  step and not another, what it left for later, and whether it asked the
  developer anything.

Then verify the numbers independently, as p2 did: recompute a small sample
of reported figures from the raw source files the researcher cites (kept
under `main/.local/` or re-retrieved from the named public source), and
check any bridge to run 1 or run 2 figures against `78a4148`'s material.
Missing or unretrievable data is reported as such, not filled in.

Do not grade the direction against a direction the observer would have
chosen. The question is whether the choice was reasoned, grounded in the
accumulated knowledge, executed, and honestly reported — not whether it was
the "right" next step.

## Step 5 — report and finish

Write `report.md` beside this plan (step reports `report1.md`… only if the
run splits naturally; one report may be enough). Include the request as
posted and its message id; the run topic and guide revision read; the
mission, its plan and task topics; the timeline; the `main` commit(s) and
files under `methods/` and `reports/`; the filled recurrence table with
evidence; the direction classification with the material it rests on; the
independent figure checks; backends and cost from the run records; and every
intervention or human step, kept apart from what ran unassisted.

Separate the conclusions:

1. Which of the p2 problems recurred, which did not, and which this run did
   not exercise.
2. What the researcher chose to do with a minimal request, and how that
   choice was grounded.
3. Whether the run was autonomous, and what one run can and cannot say
   about reliability.

Leave the routine available, its guide at v2 unless a demonstrated failure
says otherwise. Leave `publish/` as it is; publishing run 3's material is a
separate `publish` request, and the developer's push of `d467141` is still
theirs. Do not resolve the project's `workplan-` topics on the researcher's
behalf; record their state.

Keep private transcripts, credentials, service addresses, host names and
absolute local paths in ignored evidence. For episode documentation, check
links, scope and the diff, then commit and push under `localrule.md`,
staging only this exercise's files — including the `brandump.md` →
`braindump.md` rename, which is currently uncommitted.
