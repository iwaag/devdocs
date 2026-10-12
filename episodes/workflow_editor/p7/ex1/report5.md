# Workflow editor p7/ex1 — step 5: adjustments grounded in the original request

Commit: agdevworld `6058c0f`.

## Changes

- **Source `request`** (`shared/run.ts`): `{kind: request, input, sha256}`. The reducer accepts it only when:
  - `input` is the run's own input file (`braindump.md` or `request.md`);
  - the run recorded the input's hash at creation (`input.sha256`, from p7/ex1 on);
  - the source's hash equals that recorded hash.
  
  Otherwise it refuses with `adjust-unauthorized` and says which of these failed.
- **File layer** (`adjustRun`): a request source may leave out `input` and `sha256`. The file layer then names the run's input file and hashes it as it is on disk now. A missing file, or content that differs from what the caller named, is `adjust-unauthorized`. The source therefore binds to the actual input identity and to immutable content evidence: the hash recorded with `run.create`.
- Everything else is unchanged and applies to this source as to any other:
  - attempt fencing (`attempt-open`, `attempt-stale`);
  - `revision-outdated`, `mid-node`, `instructions-pending`;
  - `adjust-access`, `adjust-started`, `adjust-dependencies`, `adjust-graph`, `adjust-nothing`.
  
  The record keeps the source with input and hash, so it can be reproduced.
- **Interfaces:**
  - `wfe run adjust … --source request`. Its help says what it means (why, not approval of a graph; `plan.md` names the added nodes and is agreed as usual; not for scope the request lacks, which needs `agreement`). Its definition-file line names `wfe run revisions --show > <draft>` and `wfe nodes instantiate … --run <run> --file <draft>`.
  - The HTTP adjust route takes the same source object.
  - `wfe run revisions` shows `request braindump.md <hash>…`, and the run view shows "source: the original request (braindump.md)".
- **Planning context.** The brief's catalog lines end with the adjust command and `--source request`. The common guide prepared in step 4 states the fact and keeps the `improve-game/run-004` reference. The `_plan` step's frozen instructions are unchanged, so the compiler stays `ag.wfe-compile.v2`.

## Verification

`test/requestadjust.test.ts` (4 tests). Runs are created through `ExecStore.request` with a catalog, and the agent's steps are fenced by a claimed attempt.

| Required evidence | Test |
| --- | --- |
| The formation example | The braindump is run-004's request, and the brief lists `preliminary-research`. While `_plan` runs, the dry run is refused with `mid-node`. After the plan (naming the research node) is agreed, the draft (definition in force plus `wfe nodes instantiate … --run --file --before implement`) is applied with `--source request` as `r2`. The source is recorded with `braindump.md` and its hash. The graph has `_plan → research → _report.research → implement` and no `_plan → implement`. Only `research` is ready. `implement` is `not-ready` until `_report.research` completes, and that completion needs `report-research.md` (`document-missing` first). The original `definition/improve.yaml` is byte-identical, `checkRecord` replays the history from the original graph across `r2`, and the revision file keeps the node's provenance |
| Input identity and content | Another file named as input, a hash that is not the file's, and a braindump changed on disk are each refused (`adjust-unauthorized`). Restoring the file makes the dry run pass. A record without an input hash refuses the source ("recorded no hash") |
| Existing refusals retained | `attempt-open` (no attempt id while one is open), `attempt-stale`, `revision-outdated` (`--expect-revision r2` while r1 is in force), `adjust-access` (an added binding), `instructions-pending` (until `i1` is acknowledged), `adjust-dependencies` (after `implement` started) |
| No keyword expansion, no capability, no mandatory work | A braindump asking for research plus a research template in the catalog still gives exactly the workflow's compiled graph. The template node compiles to `{type: study}` and the bindings are the workflow's. The run completes without the template |

`npm run check`: **211/211**, tsc and build clean.

## Limitations

- The browser's *Adjust future work* form still applies adjustments as the person (`source: person`). The request source is for the executor, who plans from the request.
- Whether a change is "work the request already asks for" or a change of scope is the agent's reading of the request. The record keeps the source, the reason and the agreed `plan.md` so a person can check that reading.
