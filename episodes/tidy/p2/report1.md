# tidy p2 — Step 1 report: current boundaries and baseline

Date: 2026-10-27 (JST). Published revisions read with `wfe projects list|read` at 2026-10-27T05:36Z.
WFE release in service: agdevworld `2cc9cd9` (tidy p1's last release).

## p1 open findings, re-checked

| Finding | Published state now | Still open? |
| --- | --- | --- |
| agsfx `protoprey_locations` r3 not published | agsfx main `606aeef40a` (unchanged since p1 Step 1). The share has `runs/agsfx/create/run-003/{o1,o2}` and `run-004`, nothing of `create/run-005`; `wfe run outputs create/run-005` (agsfx): none | **Yes** |
| protoprey adoption of `location_sfx` r3 points at agsfx's run copy | protoprey main `2ee7d8f28d`, `data/deliveries.json`: `source` is an absolute path inside `pj-agsfx/.local/wfe/runs/create/run-005/tree/…`, `producer_tree: modified`, `wfe_output: {}` | **Yes** |
| agsfx README does not describe its delivery | README (1116 bytes) is a folder table; no "For other projects" | **Yes** |
| lowerpoly README lost the toolchain-host constraint | lowerpoly main `8d1bf5278d` (unchanged) | Yes (not in p2's scope) |
| agops/cagent root records, projects without README | unchanged | Yes (not in p2's scope) |

The preferred real case of Step 3 (agsfx publishes, protoprey re-adopts by reference) is therefore still
available. Nothing has to be regressed to make a trial.

## Where results are reported and received today

### Producer (the run that did the work)

| Boundary | What the agent sees now | Code |
| --- | --- | --- |
| Brief at attempt start | run folder path; documentation steps and their documents; for a delegated child, who asked, for whom, who accepts | `server/exec.ts` `brief`, `documentationBrief`, `delegationBrief` |
| `wfe run start _final-report` | `Recorded: _final-report started.` | `cli/run.ts` `start` |
| `wfe run complete <step>` with no document | reminder `document-missing` (refuses; `--yes` passes) | `shared/reminders.ts` `evaluate` |
| completion of the last work node / of the run | fix5 reminders (presentation, cleanup, unrecorded activity) | same |
| `_final-report`'s own instructions | "A final report can summarize the delivered work, actual checks and remaining work." (frozen into each run by the compiler) | `shared/compile.ts` |

Nothing asks, at the moment the agent writes its report, whether the project's entry still matches the
change, or which checks ran, or whether a stated path exists. p1's producer defects all sit there:
protoprey run-006 said "Report written" (now caught by `document-missing`), lowerpoly run-004 copied host
facts into the README, protoprey run-007 reported "7 tests pass" for 8 definitions, and protoprey run-006
read `deliveries.json` without running `status`. Only the first is a fact WFE observes.

### Receiver (the run that delegated)

| Boundary | What the agent sees now | Code |
| --- | --- | --- |
| Brief, for each result not taken up | the event text, the manifest line, then `assessmentLines` | `exec.ts` `assessedLines`, `delegationBrief` |
| `wfe delegate take-up` | `assessmentLines` + one "Next:" line | `cli/delegate.ts` |
| `wfe delegate show` | the delegation, then `assessmentLines` | same |
| `wfe delegate dispose satisfied` | refused with `delegated-*` reminders when a check is not ok | `server/delegate.ts` `dispose` |
| `conversation reply --kind result|synthesis`, `announce`, run completion | the same `delegated-*` reminders | `server/reminders.ts` `adviseOrRefuse` (`assess`) |
| a standing conductor's rest | `conversation-unanswered` (CLI only) | `cli/run.ts` `advice` |

`assessmentLines` today prints a header, then every check in fixed order (`presentation-missing`,
`review-unrecorded`, `presentation-serving`, `document-missing`), mostly `ok`, then integrations and "now"
lines. In p1's d3 (lowerpoly) the whole assessment was `ok`: it says nothing about what the child claimed,
which files it changed, or that WFE cannot judge their content. A `MISSING` line, when there is one, sits
below the `ok` lines of the checks before it. In run-048 ponderer judged each result 12–23 s after taking
it up.

### Overlaps and gaps

1. **Claim beside evidence is absent.** The manifest's `outcome` and report path are printed in the event
   lines, the assessment is printed apart, and the child's changed files are used only to classify work.
   Nothing puts "the child says X; its records hold report R (sha, committed?), files F…, outputs O" in one
   place, or says that the content is the receiver's to inspect.
2. **Findings are buried.** A finding is one line among `ok` lines with the same indentation.
3. **A corrected result** is assessed anew (fix6), but its lines do not say that it follows a correction,
   or that an earlier judgement does not cover it.
4. **Reports outside `--kind result|synthesis` are unchecked.** ponderer's #450, #452 and #453 were all
   `reply`, so the fix6 report boundary never ran for them.
5. **The multi-child gap is real and narrower than "no reply".** In run-048 d1 and d3 were both judged
   (20:38:02, 20:37:27) before #453 (20:38:13), which reported d1 only. #452 already answered #451, so
   `conversation-unanswered` could not fire. A rule "results newer than the last post" would not have
   fired either. What would have helped is the list of results changed since the previous post, shown
   where the post is made. A rest after judged results with no post at all is a factual omission that
   `conversation-unanswered` misses whenever an earlier reply exists.
6. **`conversation-unanswered` and `discussion-open` exist only in the CLI.** Agents use the CLI. The
   browser's completion of a conductor's node does not get them.

## Implementation points (smallest change)

One pure module, `shared/evidence.ts`, keeps the questions and the selection in one place. The file layer
gives it observed facts. No new state, schema, compiler version or gate.

1. **Context selection** (`workKinds`): from the run's template provenance (a `tidy` node), its changed
   paths (documentation, tests, delivery manifests, removed files) and its published outputs. A path or
   template selects which *questions* are shown. It never establishes a defect. With no context, only the
   general questions are shown.
2. **Producer guidance** (advisory, printed, never refusing):
   - with `wfe run start <final-report step>`;
   - with the completion that leaves only reporting steps (documentation, present, cleanup, review)
     when the run has no final-report step pending;
   - one line in a delegated child's brief, saying that its requester assesses the claim against its files.

   Ordinary work is asked whether the entry, guide and setup or acquisition instructions still match the
   change, and to correct them through a node rather than assert it. A run that tidied is asked for
   evidence (files and revision, checks run, locations, what remains), not another pass.
3. **Receiver evidence** in `assessmentLines`, so the brief, take-up and `delegate show` (and the HTTP
   view's `assessment`) carry it:
   - a findings headline first;
   - the claim (outcome, report) beside the evidence: report presence and commit, changed files, removed
     files, outputs, and paths the report names that are not in the child's copy;
   - "WFE cannot judge this; inspect" questions for the selected contexts;
   - for a corrected result, the correction it answers and that an earlier judgement does not cover it.
4. **Multi-child account:**
   - every `conversation reply` (any kind) from a run with delegations prints the delegations whose facts
     changed since that run's previous post in the room;
   - at a standing conductor's rest, a factual reminder `results-unreported` when a delegation's latest
     result or judgement is newer than the run's last post there.

   The rest checks (`conversation-unanswered`, `discussion-open`, `results-unreported`) move into one
   server function that the CLI and the HTTP completion both call.
5. **Unchanged:** `document-missing` (step and assessment), `delegated-*` reminders and `--yes` per command,
   optional documentation, frozen catalog snapshots.

## Before → after (what each side will see)

**Sender**, protoprey run-006-like, at `wfe run start _final-report`:

- before: `Recorded: _final-report started.`
- after: the same line, then about six lines. This run has a tidy node, so it asks for evidence, not
  another pass: which files changed and at which commit; which checks actually ran, with results; that
  the report and every path it names exist where stated; what is unverified or left. Because
  `data/deliveries.json` changed, it also asks whether a fresh copy can obtain each adopted delivery
  without the producer's working copy (its status/acquisition check run, not the record read).

**Receiver**, ponderer at the take-up of d3 (lowerpoly run-004, first result):

- before: four `ok` checks, integration lines.
- after:
  - `findings: none recorded — WFE checked records, not content`;
  - the claim: outcome "…", report at path (sha, commit);
  - the evidence: changed `README.md`, …;
  - for documentation changes, questions: does the entry agree with the implementation; do its
    references resolve; is history apart from current state; are machine-specific facts out of portable
    text;
  - how to read the files (`wfe delegate file`);
  - the checks.

**Conductor**, ponderer posting #453 (`reply`):

- before: `Posted: #453 …`
- after: the same, then `Results changed since your previous post #452: d1 → protoprey freework/run-006
  result event 4, judged satisfied; d3 → lowerpoly freework/run-004 result event 4, judged satisfied. A
  final account names each with its disposition and what stays unverified.`

## Next

Step 2: implement in agdevworld (and one fact in the wfrun guide if the existing delegation bullet goes
stale), test, release to studio and agpc.
