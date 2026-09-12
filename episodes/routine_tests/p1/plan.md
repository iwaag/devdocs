# Routine tests p1 — publish the backlog, investigate once, publish the result

## Objective

Exercise the system with one composite request through Front's ordinary
entrance, and use the result to find defects in multi-routine coordination:

> Across all study projects, publish everything that has not yet been
> published. After all of that is finished, investigate one arXiv trend
> item, then publish that result too.

The source is [braindump.md](braindump.md). Implement a prerequisite before
the trial only when inspection demonstrates that it is missing. Mere doubt
about an agent's judgment is a reason to observe the trial, not to add a rule.

This document plans the execution. Creating it does not start a routine,
post a request to an agent, or modify a study repository.

## Current contracts and interpretation

Read alongside `devdocs/README_DEV.md`, `devpolicy/styles.md`,
`devpolicy/terms.md`, both projects' ignored environment notes, and
`pj-clusterintent/nctl/README.md`. Paths here are relative to the projects
workspace unless explicitly described as study-workspace paths.

Read-only preparation on 2026-09-12 checked service state through
`nctl status --json` and `nctl drift --json`, and read the live routine
guides and study-channel list. Repeat relevant reads before execution;
Nautobot observations do not prove that an agent can complete this request.
Keep raw service output and machine-specific evidence in ignored files.

The live contracts establish the following:

- `routine-publish/guide`, message **5496**, serves study projects with a
  `publish/` folder. Its current table names `studyarxiv` and
  `studyrealworld`. Discover the actual project set at execution time from
  project records and `README_PROJECT.md`; neither this table nor a channel
  prefix alone proves that the inventory is exhaustive. Deduplicate aliases
  for the same repository.
- **Publish means completing the publication gate and committing
  `publish/` locally.** The guide explicitly reserves its push for the
  developer. Corrections to `main/` are committed and pushed to its own
  origin. This request uses that existing routine meaning; it does not
  silently change publication into an agent push to the public remote.
  Report remote publication as still awaiting the developer.
- `routine-papers/guide`, message **5499**, is the existing arXiv trend
  investigation route. It selects exactly one new paper and writes its
  summary; the same task also writes missing manuals for runnable papers
  already in the index. Those manuals are part of the existing guide, not
  additional paper investigations. No new `arxivtrend` routine is needed.
- The newest full post in each `guide` topic is authoritative. A run has
  its own `routinerun-…` topic, records the guide message it read, and ends
  with an `ag.routinerun-finish.v1` result. Ending and achieving the goal are
  separate facts. Work records are Zulip conversations; Plane and the old
  scheduled dispatcher are retired.

No execution option or usage threshold was requested. Use existing defaults
and record the actual backends; do not inherit p2's `agy` or budget condition
from the earlier `refine_routine` episode.

## Step 1 — establish the baseline and repair demonstrated prerequisites

1. Refresh relevant service and placement reads through Nautobot/nctl.
   Read Front's and autolab's current introductions, the current guides,
   and any active work that could overlap the study repositories. Use the
   ignored environment notes to locate services and credentials without
   copying their values into tracked documentation.
2. Establish a read-only project baseline: study classification, canonical
   project and repository, presence of `main/` and `publish/`, candidate
   paths, source revisions, publication revisions, and pending differences.
   Include changed files and indexes, not only entirely absent reports.
   Classify an unreadable project as unknown, never as already published.
   Preserve pre-existing local edits and record concurrent work separately.
   The executor may inspect this evidence, but Front should still discover
   the scope through the system's ordinary agent interfaces during the trial.
3. Inspect the transition from a finished routine back to the composite
   request. There is a concrete concern in
   `pj-agdev/agfront/src/agfront/zulip_listener.py`: `finish_run()` posts
   the report into the origin using Front's own credential, while the owner
   sweep does not serve topics whose last speaker is Front.
   `start_opened_runs()` starts newly opened runs; it does not by itself
   resume an already served parent after a child finishes. Confirm whether
   another existing path provides that continuation.
4. If continuation is absent, add the smallest generic completion handoff
   that lets Front reconsider a still-active composite request after its
   child result is durably recorded. Keep the next-stage decision in the
   agent's conversation. Cover success and unsuccessful completion, duplicate
   delivery/recovery, resolved names, and restart between delivery and
   continuation. An ordinary one-routine request must not enter a self-reply
   loop. Do not encode this episode's three stages into the listener.
5. Read the Front guides, including the Front Desk role if that entrance is
   used. The routine-run guide currently forbids opening another routine
   run. Prefer sequential sibling runs coordinated from the original Front
   conversation. Adjust guidance only where it demonstrably prevents keeping
   a composite request active across child completions; do not introduce a
   workflow engine, a scheduler, or a new umbrella routine for this trial.

Likely implementation surfaces are `agfront`'s listener, routine helpers,
role guides and `tests/test_routine_run.py`. Change autolab or shared pyagag
only if the evidence locates a defect there. Use `pj-clusterintent` for
state discovery and any necessary deployment through its existing workflows.

For an actual code fix, add focused behavioral regression coverage, run the
owning package's relevant checks, commit and push, and deploy it before the
trial. Verify the deployed revision. Restart an affected listener only after
checking its in-flight work; refresh its introduction only if its contract
changed. Do not preemptively fix possible selection or planning mistakes.

## Step 2 — submit one composite request

Open one fresh conversation at Front's ordinary entrance and submit the
request above as a single piece of work. Make clear that Front is to supervise
the whole sequence through completion under the existing routine guides.
If Front asks for its ordinary proposal acceptance, accept that proposal
within the execution session; do not send the three stages as independent
developer requests.

Record the request message, its conversation, the guide revisions used, and
the start time. Let Front choose project delegations and workplans. Each
routine execution and delegation needs a fresh topic so an old root note
cannot redirect its callbacks to an earlier request.

The expected dependency order is:

| Stage | Work | Evidence required before moving on |
|---|---|---|
| A — existing backlog | Run the publish gate across every study project with unpublished or changed material. | Every discovered project is accounted for; all pending material is successfully processed or the remaining blocker is explicitly reported. |
| B — one investigation | Run `papers` once, after A succeeds. | Exactly one previously unindexed paper has a source-backed summary and index entry; the guide's manual work is accounted for; the source commit is pushed. |
| C — publish new results | Run the publish gate for the study project changed by B. | The new summary, required supporting files and changed index/manuals pass the gate and are committed in `publish/`. |

At A, an empty backlog is a successful no-op and should proceed directly to
B without an empty publication commit. A rejection permitted by the guide
is an honest gate outcome, but it is not proof that the user's request to
publish everything succeeded: do not cross into B while material remains
blocked. Likewise, a failed investigation is not permission to publish an
old item and call it the new result.

Use the baseline revisions to distinguish the existing backlog from B's new
output. Recheck pending differences at the A-to-B boundary; account for
concurrent changes explicitly rather than silently claiming the starting
snapshot is still current. This is a finite request, not an instruction to
keep generating more research or chase unrelated future changes forever.

## Step 3 — observe the run and verify the work

Observe conversations and run records read-only while agents work. Capture
the transition evidence before intervening. A progress question is a new
agent trigger, so do not use repeated posts as a polling mechanism.

Verify these outcomes independently of Front's final claim:

- **Coverage:** every study project is listed with its candidate count,
  processed paths, already-current paths, and rejected or unreadable items.
  Do not omit a project because another one produced a successful report.
- **Order:** message IDs and timestamps show that all A results were
  checked before B was started, and B's completed source revision existed
  before C began. An acknowledgment, opened workplan, or started task is not
  completion. Front performs the task-start and acceptance actions required
  by autolab's introduction.
- **One new investigation:** compare the arXiv index before and after B.
  Verify the new paper's identity, the guide's recency requirement, the
  public trend evidence, and evidence that the paper itself was read.
  Check the runnable verdict and manual outcomes against the guide; a local
  reproduction is not required for publication.
- **Publication correctness:** retain per-item verdicts against the guide's
  four checks, inspect source corrections for lost knowledge, compare each
  published file with its source, and check relative links within the
  published tree. The reader-facing `README.md` is the documented exception
  to byte equality. Check that required indexes and supporting files travel
  with the new result.
- **Repository outcome:** record both source and publication commits, source
  push evidence, and the publication clone's initial and final remote
  relationship. An already-ahead or behind clone is reported without an
  automatic reset, merge, or push. `publish/` remains committed locally for
  developer review.
- **Continuation and closure:** child results return to the right parent;
  the next stage starts without an Omni Agent nudge; each run ends once;
  the final report reaches the original request with all stages accounted
  for. Completing a child leaves the parent available for the remaining
  stages and leaves the routine guide and unrelated runs untouched.

If the workflow stalls, retain the triggering messages, root/served notes,
run records, and relevant code path. Separate a demonstrated defect from a
slow in-flight task, an upstream read failure, or an honest publication
rejection. Fix an observed system defect at its owner, then resume from
verified work or run a clearly identified follow-up. Do not redo already
completed research or publication just to make a clean-looking transcript.

Any manual continuation makes the first attempt assisted, even if its work
eventually finishes. Record it explicitly. If the Omni Agent performs work
belonging to an in-system agent, add the required one-line note:
“did X for agent Y — handoff candidate”. Prove the repaired transition on a
follow-up before describing it as autonomous.

## Step 4 — report and finish

Write `report.md` beside this plan, with step reports only if the evidence
warrants them. Include:

- The submitted request and its interpretation, particularly local
  publication preparation versus the developer's final push.
- The project inventory, backlog before/after, one new paper, file lists,
  commit evidence, and the timeline across A, B and C.
- Prerequisites fixed before the trial, defects actually encountered,
  interventions, regression checks and follow-up outcomes.
- Separate conclusions for work completed, autonomous coordination proven,
  and anything still blocked or unproven; include actual run backends and
  available elapsed-time/usage evidence without inventing measurements.

Keep hostnames, credentials, internal service URLs, absolute local paths,
and raw private transcripts in ignored evidence. Commit and push the episode
documents and any implementation changes according to `localrule.md`;
preserve the publish routine's explicit local-only boundary for study
`publish/` repositories. Stage only files belonging to this work.

The desired success is all three stages completed in order from one request,
with verified artifacts and no manual stage-start nudges. An unsuccessful
trial is still useful evidence, but it must identify the defect and remaining
work rather than relabeling an ended run as a fulfilled request.
