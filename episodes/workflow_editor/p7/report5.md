# Workflow editor p7 — step 5: recorded adjustments of future run work

Commit: agdevworld `cb65490`.

## Changes

- **Record** (`shared/run.ts`).
  - A new operation, `run.adjust`, carries the whole revision:
    - `revision` (`r2`, `r3`, …) and `parent`;
    - `file` (`definition/r<n>/<workflow>.yaml`) with `sha256` and `definitionDigest`;
    - the complete compiled graph `effective`;
    - `reason`, `source`, and the `diff` it applies.
  - The state keeps `definition.revisions` and `definition.active`. Both are absent until first used, so old records replay unchanged.
  - `definition.effective` stays the original (r1). `effectiveOf()` gives the graph in force.
  - `replay` switches graphs at each `run.adjust`, so the original history replays across revisions. History readers (`notableEvents`) look node ids up across all revisions.
- **Rules in the reducer.**
  - The adjustment is a node boundary:
    - no node running (`mid-node`);
    - no unacknowledged instruction (`instructions-pending`);
    - not completed or cancelled;
    - `parent` must be the revision in force (`revision-outdated`), checked under the run's lock with `expectSeq`.
  - While an attempt is open only that attempt adjusts (fenced); a person's adjustment waits (`attempt-open`).
  - Every node that is not `pending` keeps its effective node and incoming edges (`adjust-started`, `adjust-generated`, `adjust-question`, `adjust-dependencies`).
  - The graph is acyclic, has known edges and no delegate (`adjust-graph`); the documentation policy is unchanged; there is something executable to change (`adjust-nothing`).
  - Authorization (`adjust-unauthorized`):
    - an answer that was taken up;
    - an acknowledged instruction;
    - an agreement whose question's document is this exact proposal (same hash);
    - a person.
  - New nodes start `pending`. Removed (pending) nodes leave the state. Questions, answers, instructions, presentations and decisions are untouched.
  - The decision rule now takes the latest question from any result-confirmation node in force, so a correction's new confirming talk supersedes the old one.
- **File layer** (`server/runs.ts` `adjustRun`, under the run's lock).
  - Parses the whole edited definition. It refuses (`adjust-access`) changes of `repositories`, `execution`, `intake` or `documentation`, and edits of any authored field (type, name, description, repositories, workflow, confirms) of non-pending nodes.
  - Validates the definition. A second `confirms: result` is allowed only when the extra one is completed.
  - Compiles with the current compiler and runs the reducer on a copy, so an inspection lists every refusal.
  - `dryRun` writes nothing. `propose` writes `definition/proposals/<name>.yaml` for an agreement question (`wfe run ask … --about-adjustment <name>` records it by hash).
  - Applying writes the revision file, then `run.json`, and removes the file if the record refuses.
  - Reading checks every revision file's hash, workflow id and graph (and, for the current compiler, exactly what it compiles to). A changed file makes the record unusable (`bundle`).
- **Interfaces.**
  - `wfe run adjust <run> --definition <file> --reason … --source answer:<q>[:<n>]|instruction:<i>|agreement:<q>|person [--dry-run] [--propose <n>] [--expect-revision r<n>]`.
  - `wfe run revisions <run> [--show r<n>] [--diff r<a> r<b>]`.
  - `POST runs/<wf>/<run>/adjust`.
  - In the run view: a *Definition (r<n> in force)* section with the original and each revision (reason, source, who, added/removed work), and *Adjust future work*: the definition in force in an editor, a reason, *Preview* (the difference and refusals), and *Apply* enabled only after a successful preview. The canvas draws the revision in force.
  - The executor brief names the revision in force. `wfe run show` prints the adjustments.
  - Coordination announces `adjusted` ("Plan adjusted (r2) by … — added …; removed …").
- The reusable workflow file in `devdocs/workflows/` is never touched.

## Verification

`test/adjust.test.ts`, 6 tests:

| Required evidence | Test |
| --- | --- |
| An adjustment preserves past state and replays | removing pending cleanup and adding `lint`: r2 with parent r1; completed `implement` byte-for-byte equal; the original snapshot is unchanged; `checkRecord` replays the whole history from the original graph; work continues on the new graph (`lint` becomes ready); the `adjusted` event and its message |
| Stale, cyclic, mid-node, question-destroying, access-expanding changes are rejected | `revision-outdated`, `adjust-graph` (cycle), `adjust-started` (edit and removal of completed work), `adjust-dependencies`, `adjust-access` (new binding; policy change), `adjust-nothing` (appearance only), `mid-node`, `adjust-question` (rewriting a node waiting on a question), `attempt-open`. None changes the record or leaves a revision file |
| Pending instructions cannot be bypassed | `instructions-pending` on the adjustment; the last completion of a run is refused while an instruction is pending |
| Authorization | an answer index not taken up; agreement to a different proposal (`exact definition`); an applied agreement to the proposal asked about |
| A correction leads to new work and a new review without resetting completed nodes or reusing the prior decision | after the person's "make them faster" answer to the confirmation of p1: the adjustment adds repair → reverify → represent → recheck and moves cleanup after `_report.recheck`; completed nodes stay completed; the earlier question, answer and p1 remain; no decision. The old answer cannot be decided once p2 exists; the new confirmation's answer records the single decision, bound to p2 |
| Incomplete writes | a leftover unreferenced `definition/r2/` does not affect the run, and the next adjustment replaces it; a tampered revision file makes the record unusable instead of being recompiled |

- Full suite (`npm run check`: `tsc`, tests, build): **190/190 pass**.
- Browser, `checks/review.ts` (now 23 checks, all pass): the person opens *Adjust future work*, sees the definition in force, previews ("work added archive … new edges cleanup → archive") and applies. The view says "r2 in force", the canvas draws `archive`, and completed work stays completed. `checks/appearance.ts` still passes 15/15.
- During this step `npm run build` failed once because of an unused variable in a test (a type error). The browser check had silently run the previous build. This was noticed when the new panel was missing, then fixed and re-run. Lesson: read the build output before a browser check.

## Limitations

- Concurrent proposals are serialized by the lock and `parent`/`expectSeq`. The run view's preview is not re-validated automatically when another adjustment lands: *Apply* sends `expectRevision` and is refused if stale.
- Proposals (`definition/proposals/`) stay in the run folder after use, as evidence of what was asked.
- The run view's editor is plain YAML text, not the graph editor. Editing graphically within a run is not part of p7.

## Next

Step 6: the RTS `improve-game` workflow with presentation and cleanup, `confirms: result`, the review guide, and the executor's common contract.
