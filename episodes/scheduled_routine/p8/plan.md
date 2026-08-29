# scheduled_routine p8 — Plan

Braindump: `braindump.md`. Two improvements to the arXiv trend study
workflow in `studyarxiv`:

1. **Graded test records** — local test results get written to a `test.md`
   with an explicit test **level** (the braindump says "3段階ぐらい" but
   lists four; use the four it lists):
   - **L1** — system built locally, most basic function confirmed working.
   - **L2** — a workflow described in the paper completed end to end.
   - **L3** — a small, minimal *original* verification experiment measured
     performance (not a paper reproduction).
   - **L4** — a reproduction of the paper's own experiments, or beyond.
2. **`publish/` copy routine** — an **independent** routine (explicitly not
   self-censorship inside the paper/localtest workflow) that reviews reports
   in `main/`, **edits each one into compliance** with the conditions below,
   and **copies** the result into `publish/`. `main/` is never moved from or
   emptied: it stays the working record; `publish/` holds the sanitized
   derivative. The conditions, verbatim from the braindump:
   1. no local-environment or secret facts; environment info limited to
      generic experimental conditions (processor model, specs);
   2. the paper's version is clearly stated;
   3. no long copyright-scale quotations, no fabricated quotations; avoid
      direct transcription of quoted text in general.

Experimental, non-public environment; destructive phase, no backward
compatibility. Only the **MUST NOT** lines are prohibitions — everything
else is advice the implementer may override with reason.

## Background the implementer should know

- Workspace: `agautolab/.local/projects/studyarxiv/`. `main/papers/INDEX.md`
  has 9 papers; exactly two have completed local tests, both `verified`:
  `2608.23552` (Prime Agent) and `2608.23283` (Apodex 1.1). Their raw run
  logs live in `localtest-<id>/report.md` + `localtest.yaml` (local Gitea
  repos `autodev/studyarxiv-localtest-<id>`).
- `publish/` is already a clone of the **public GitHub repo**
  `https://github.com/iwaag/study-arxiv-trend.git`, one commit (`fbf6517`),
  containing only `LICENSE`. The study pattern
  (`agautolab/agent/project_pattern.md`) says: moved on developer approval,
  and **the developer pushes `publish/` by hand — the agent never pushes
  it**. Keep that push rule; it is the single hard gate that makes the rest
  of this phase safe to run loosely.
- Where `test.md` goes is the implementer's call, but the natural spot is
  `main/papers/<id>/test.md`, beside `summary.md`/`manual.md` — publish
  material is drawn from `main/`, and the localtest repos are local-Gitea
  workspaces that will never be public. Treat `localtest-<id>/report.md` as
  the raw log and `test.md` as the distilled, publishable result: level
  reached, upstream revision tested, generic environment, evidence, what
  would raise the level. Mind relative links: a `test.md` that links into
  `../../../localtest-…/` breaks when copied to `publish/`.
- Both existing verified tests are honest **L1** (one-shot smoke commands
  returning `OLLAMA_PRIME_OK` / `FRONTIER_OLLAMA_OK`). Backfilling them as
  L1 is the expected, informative outcome — do not inflate.
- `INDEX.md` has a `localtest` column with values `no`/`verified`. This is a
  destructive phase: repurposing that column to carry the level (`no`, `L1`…
  `L4`, plus whatever in-progress marker is useful) is a fine one-file
  change. The `papers` routine only needs the row to *exist* for dedup, so
  column semantics are free.
- Condition 2 (paper version) is already nearly met by the summary shape —
  `summary.md` carries a "First posted …; current version vN" line. The
  publish review should verify per paper, not re-invent the field.
- Standing routines are latest-post standing texts in `#front` topics
  (`routine-papers`, `routine-localtest` message 2525 — see p6
  `report2.md`), fired via `rtschedule` one-shot events through the
  production dispatcher (p6 used `r10`/`e40`). Follow the same shape for
  `routine-publish`, and post workplans as the Developer in
  `#pj-studyarxiv` if the Front leg adds no value to a given step (p7
  precedent).
- In-system changes to `main/` and `publish/` go through autolab missions,
  not direct Omni edits; record any direct intervention (role-boundary
  memory). Harness-side edits (pattern doc, CLI templates, standing texts)
  are ordinary developer work here.
- p7 learnings that apply: have missions quote their own result JSON
  (`cost_usd`, `num_turns`, `duration_ms`) in the Zulip report — p7 lost
  its cost comparison to this gap; the `main` clone's push to bare-hostname
  `agstudio` needed a one-time `git remote set-url` to `agstudio.local` —
  expect the same friction and just fix the remote if it bites;
  `WORK_TIMEOUT_SECONDS = 1200` fits these documentation-sized tasks
  easily. If a workrun dies oddly, check its transcript for leaked-CLAUDE.md
  permission denials before blaming the mission text.
- Copy, not move (developer's decision): `main/` keeps its files, INDEX
  rows, and localtest links untouched. The publish routine produces an
  **edited copy** — it rewrites local hostnames/paths into generic
  environment facts, adds a missing version line, trims or paraphrases long
  quotations, fixes links that would dangle outside `publish/` — rather than
  rejecting a report for a fixable defect. Rejection is reserved for what
  editing cannot honestly fix (e.g. a claim whose only evidence is private,
  a quotation that cannot be verified against the paper). The diff between
  the `main/` original and the `publish/` copy is itself evidence of what the
  gate does; keep it readable and report it.

**MUST NOT**: push `publish/` to GitHub (the developer publishes by hand
after review — this is the whole point of the gate); commit credentials or
private machine/cluster facts to any repo that leaves `main/` for
`publish/`; work around the permission classifier (stop and report,
`localrule.md`).

## Step 1 — the test.md / level contract

Define the four-level scale and the `test.md` shape in
`agautolab/agent/project_pattern.md` (extend the existing localtest
section; a few sentences and the level table, not a framework). While
there, rewrite the study pattern's `publish/` line from "moved" to "an
edited copy, produced by the publish routine; `main/` stays intact; the
developer pushes by hand". If `init-localtest` templates deserve a pointer
to `test.md`, add it. Update
the `routine-localtest` standing text in `#front` so future local tests end
by writing/updating `test.md` with the level reached. Commit and push
agautolab per `localrule.md`; no deploy ritual is needed unless listener
code changed (doc/standing-text edits take effect on the next run).

Report `report1.md`: the contract as written, diffs, standing-text new
message id.

## Step 2 — backfill the two verified papers

One autolab mission in `#pj-studyarxiv`: write
`main/papers/2608.23552/test.md` and `main/papers/2608.23283/test.md` from
the existing localtest reports, assign levels honestly (expect L1), update
the INDEX `localtest` column to the level form, commit and push `main`.
Have the mission quote its result-JSON cost numbers in its report.

Report `report2.md`: mission text, the two test.md files as landed, INDEX
diff, cost numbers, frictions.

## Step 3 — the publish routine, defined and fired once

Write the `routine-publish` standing text in `#front › routine-publish`.
It should tell Front to ask autolab for one mission in the study project
that:

- reviews `main/papers/` for papers whose reports are worth publishing (at
  minimum the two with test.md; the agent decides whether summary-only
  papers qualify too);
- checks each candidate against the three braindump conditions — grep-level
  sanitization for local facts (hostnames like `agstudio`, `*.local`,
  `home.arpa`, local paths, ports, credentials; environment reduced to
  generic specs such as "Apple-silicon Mac"), version line present, quote
  hygiene (no long verbatim passages, nothing quoted that isn't in the
  paper);
- **edits each candidate into compliance and copies it into `publish/`**
  (never moves; `main/` stays as is) in a per-paper layout of the agent's
  choosing, adds/updates a `publish/README.md` index, commits `publish/`
  locally, and **does not push it**;
- reports per paper: copied with a summary of the edits made, or rejected
  with the condition that editing could not honestly satisfy — a rejection
  is a valid, useful result, not a failure;
- states plainly that final publication is the developer's manual push.

The review must run as its own mission — never as a self-check appended to
a papers/localtest run (the braindump's one structural demand). Add a
one-shot `rtschedule` event and let the production dispatcher fire it, p6
style; this also revalidates the Front routing that p6 left unproven after
its rework-route repair.

Report `report3.md`: standing text, schedule request/event ids, dispatch
timeline, the publish commit's file list, per-paper verdicts with the
main→publish edit summary, cost numbers.

## Step 4 — developer review, hand push, phase report

The Developer reviews the `publish/` commit against the three conditions
independently (this is the review the pattern always promised), pushes by
hand if it passes, and notes anything the routine's own check missed — each
miss is a concrete improvement for the standing text.

`report.md`:

- both braindump features answered: is the level scale usable and honest,
  and did the independent publish gate catch what it should?
- what the developer's review caught that the routine didn't (and vice
  versa — over-rejection is also a finding);
- costs and timings from the quoted result JSONs;
- whether `routine-publish` should recur on a schedule or stay
  manual-fire — recommend only;
- any Deus Ex Machina interventions, each with its one-line handoff note.

## Out of scope unless a live run forces it

Automating the GitHub push or changing the push-by-hand rule; moving or
deleting anything in `main/`; new local
tests or re-running the verified ones; `manual.md` publication policy
beyond what the conditions already imply; a recurring publish schedule;
fixing unrelated drift; dispatcher/schema changes.
