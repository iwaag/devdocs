# scheduled_routine p7 — Plan

Braindump: `braindump.md`. Run **five arXiv investigations in one autolab
session** in the `studyarxiv` project and find out whether Claude Code's
harness-native parallel execution (the `Task` subagent tool) works inside an
in-system autolab run. Parallelism is **explicitly instructed** this time —
whether it would happen unprompted is a different, later experiment. Per the
braindump, the workplan separates **selecting** the five papers from
**investigating** them: selection is one work, the five investigations are one
other work executed in a single session with parallel subagents.

Experimental, non-public environment; destructive phase, no backward
compatibility. Only the **MUST NOT** lines are prohibitions — everything else
is advice the implementer may override with reason.

## Background the implementer should know

- Project: `agautolab/.local/projects/studyarxiv/`, channel `#pj-studyarxiv`,
  repo `autodev/studyarxiv`. `main/papers/INDEX.md` has **4 covered papers**
  (2608.23552, 2608.21156, 2608.23283, 2608.20430) — the five picks must all
  be new rows. Layout per paper: `papers/<id>/summary.md` + INDEX row, the
  same shape the `papers` routine writes (p4 Step 1 standing text is the
  reference for what one summary contains).
- All autolab roles run profile `sonnet` = harness `claude_code`
  (`agautolab/agents.toml`). The harness invocation is
  `claude -p --output-format … --model … --allowedTools <role grant>`
  **without** `--dangerously-skip-permissions`
  (site-packages `agag/harness.py:build_argv`).
- **`Task` is not in any role's `allowed_tools` list.** In recent Claude Code
  the subagent tool may not need an explicit grant (subagents inherit the
  parent's allowedTools for their own tool calls), but this exact question is
  what Step 1 settles empirically. If the tool is denied, the fix is one
  line — add `"Task"` to `roles.supercoder` (and `superdirector` if planning
  wants it) — followed by the p2 deploy ritual: commit, push to GitHub, pin
  bump, `launchctl kickstart -k gui/$(id -u)/com.agdev.agautolab-zulip`.
  Never repoint anything at gitea or a local path (`localrule.md`).
- `WORK_TIMEOUT_SECONDS = 1200` (`zulip_listener.py:170`). One summary fits
  easily (p5 evidence); five **sequential** ones probably do not — parallel
  execution is precisely what should make five fit in one task. If the live
  run still times out, raising the constant for this phase is an acceptable
  one-line evidence-driven change; note it as a finding either way.
- **Write-collision hint** for the parallel task: per-paper folders
  `papers/<id>/` are disjoint, so subagents can write summaries freely, but
  `INDEX.md` is one file — instruct the session to have the **parent** merge
  the five INDEX rows (and make the commit) after the subagents return,
  rather than letting five subagents edit one file concurrently.
- **Evidence of parallelism** does not come from the answer text. Each serving
  leaves `transcript.jsonl` beside its `chatlog.md`; parallel Task launches
  appear there as multiple `Task` tool_use blocks in one assistant message,
  and the proof is overlapping subagent activity plus wall-clock duration
  well under 5× a single investigation (p5: one summary ≈ minutes). The
  result meta (`duration_ms`, `num_turns`, `cost_usd`) is captured per run —
  record it; `localrule.md` says count cost, never economise on it.
- Known trap (memory): the Omni harness's project instructions have leaked
  into in-system agent runs before and killed them on permission denials. If
  the workrun dies oddly, check the transcript for tool denials before
  blaming the mission text.
- Trend signal, paper choice, and subagent prompt wording are the agent's, as
  in p4/p5 — the mission names the outcome (5 new papers, 5 summaries, INDEX
  rows, commit+push), not the method beyond "run the five investigations in
  parallel with subagents in this one session".
- The schedule/Front leg is **not** part of this phase: post the workplan by
  hand as the Developer in `#pj-studyarxiv`. Whether the `routine-papers`
  standing text should ask for 5-in-parallel routinely is a phase-report
  question, answered by this run's evidence.

**MUST NOT**: commit credentials or private machine/cluster facts; work
around the permission classifier (stop and report, `localrule.md`); run
`localtest` work in this phase (local resource contention — braindump).

## Step 1 — prove the Task tool under the role grant

Cheapest honest probe first, no listener involved: from the studyarxiv main
clone, run the harness the way the listener does —

```sh
claude -p --output-format json --model claude-sonnet-5 \
  --allowedTools "<the exact roles.supercoder grant string>" \
  "Use the Task tool to run two subagents in parallel; each reads one file in
   this directory and reports one line. Reply with both lines."
```

— and read the JSON for subagent activity vs a denial. If denied, make the
one-line grant change and deploy (ritual above), then re-probe **through a
real throwaway workrun** (a trivial `workrun-` task in a scratch project such
as `runsmoke2`) to confirm the deployed listener path also serves it.

Report `report1.md`: probe command, verbatim denial or success evidence, any
grant diff and deploy commits.

## Step 2 — the two-work mission

As the Developer in `#pj-studyarxiv › workplan-…`, request one mission with
two explicitly separated tasks:

1. **Select** — pick **five** recent trending arXiv papers (public signal
   named, none already in `INDEX.md`), and record the selection with the
   signal evidence somewhere in `main/` that task 2 will read (the agent
   chooses the file; a short `papers/SELECTION-<date>.md` is a fine shape).
   Commit.
2. **Investigate** — in **one session**, run the five investigations **in
   parallel using subagents (the Task tool)**: each subagent reads its paper
   and writes `papers/<id>/summary.md` in the established shape (problem,
   method, results, why it trends, `runnable:` verdict); the parent then
   merges the five INDEX rows, commits, and reports which parts ran in
   parallel. Say in the mission that parallel subagent execution is itself
   the experiment, so the session should state plainly if it could not do it.

Approve the plan (check it kept the two-work split and put all five papers in
one task, not five tasks — if the planner splits them five ways, say so and
have it replan), `start.flag`, and watch both workruns end to end without
steering unless a run breaks.

Report `report2.md`: mission text, plan as proposed, timeline of both tasks,
frictions and any fixes.

## Step 3 — read the evidence

From task 2's `transcript.jsonl` and result meta, answer:

- Did parallel Task launches actually happen (multiple Task tool_use blocks
  in flight at once)? How many concurrently?
- Wall clock of task 2 vs a plausible per-paper serial estimate; `num_turns`,
  `cost_usd` for both tasks.
- Output integrity: five new `papers/<id>/summary.md`, five new INDEX rows,
  no duplicate of the four existing papers, no INDEX corruption from
  concurrent edits, `main` committed **and pushed** to Gitea.
- Quality skim: the Developer reads two of the five summaries and says
  whether parallel production degraded them versus the p4/p5 singles.

Report `report3.md`: the table of five papers, the parallelism evidence
quoted (timestamps), the numbers, the Developer's skim verdict.

## Step 4 — phase report

`report.md`:

- The braindump's question answered plainly: does harness-native parallel
  execution work in an in-system autolab session, and what (if anything) had
  to change to allow it (grant, timeout, nothing)?
- Cost/time of 5-in-one-session vs five routine fires, from recorded numbers.
- Whether `routine-papers` should adopt a parallel multi-paper form, with the
  evidence for or against — recommend only; changing the standing text is a
  later act.
- Any Deus Ex Machina interventions, each with its one-line handoff note.

## Out of scope unless a live run forces it

`localtest` runs; schedule/dispatcher/Front changes; testing *unprompted*
parallelism; manual.md writing for the five papers; changes to the study
pattern; fixing unrelated drift or the p6 rework-route revalidation.
