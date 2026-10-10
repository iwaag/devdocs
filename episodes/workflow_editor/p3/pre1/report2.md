# workflow_editor p3/pre1 — step 2: run records and CLI

## Result

Runs exist as files, with one set of rules for every route. All paths are in
`pj-agdev/experiments/workflow_editor/`.

| Part | Where | What it does |
| --- | --- | --- |
| Reducer | `shared/run.ts` | `apply` / `replay` / `operate` / `checkRecord`, readiness, blocking, execution summary, holder derivation. Pure, so the browser uses it too. |
| Files | `server/runs.ts` | Creation with snapshot closure and exclusive folder; reading with bundle digest checks, replay check and caching; operations writing one atomic `run.json` replacement; delegation from the parent's bundle; source comparison; reading at a devdocs commit; listing; checks. |
| CLI | `cli/run.ts` (+ `wfe run` in `cli/wfe.ts`) | `create list show check start progress wait complete fail cancel ask answer take-up withdraw delegate attach decide`. Each `--help` states inputs, reads/writes and result. `--json` everywhere. |
| HTTP | `server/api.ts` | `GET runs`, `GET runs/<wf>/<run>[?rev=]`, `GET runs/<wf>/<run>/file?path=`, `POST runs/<wf>/<run>/ops`. Thin over the same functions; `via: browser`; `by` required. |
| Contract | `docs/runs.md` | Updated: the answer records `from` (who gave it) and `by` (who recorded it). |

The CLI does not need the service. When the service runs for the same
registry, the CLI prints the browser link of the run.

## How the record works

- **One operation is one history entry.** The entry holds `seq`, `at`, `by`,
  `via`, `op`, the op's parameters and its `change`, for example
  `{"nodes": {"ask": ["running", "waiting"]}}`. `operate` appends the entry,
  replays it into the state, and the file layer writes both in one atomic
  rename.
- **Reading replays.** The stored state and each entry's `change` must equal
  what the replay produces. A tampered state, a removed entry, an edited
  change or an entry the rules forbid are all reported, each by its code:
  `inconsistent`, `sequence`, or the rule's own code (`not-ready` …).
- **The bundle is checked on every read** against its recorded `sha256`.
- **Problems are visible and never repaired.** A malformed, inconsistent or
  tampered record appears in `run list` / `run check` with its problem. Every
  operation on it is refused, and the file is left byte for byte. The tests
  check all of this.

## Tests

`test/runs.test.ts` has 14 tests: 6 for the reducer, 7 on files and Git, and
1 for HTTP. The whole suite, `npm run check`, passes: type check, **65/65
tests**, and the production build.

These map to the plan's validation criteria as follows:

| Criterion | Test evidence |
| --- | --- |
| 1 authorship, provenance, collisions | A braindump records `author` and `recordedBy`. A request records `requester`, `entrustedBy`, `onBehalfOf` and `original-input.md`, with no invented author. A same-name create is refused (409) and `run.json` is unchanged. A bad name is refused (400). Run numbering continues `run-001` → `run-002`. |
| 2 snapshot closure, later edits | The bundle holds ship + sub + leaf (transitive) as byte copies. Editing, renaming and deleting the sources leaves the run's graph unchanged and is reported as `changed`, `renamed` or `deleted`. A child created **after** its source was deleted runs the parent's copy, and its bundle is self-contained (sub + leaf). |
| 3 branch/join, failure | Both branches show as active. A join is refused until every predecessor completes. A failed or cancelled node blocks the join (`blocked`, then `stopped`) and never counts as success. A failed node resumes only with a reason. |
| 4 talk questions | The node waits on q1. An answer to q2 leaves it waiting, and so does an answer to q1. Take-up resumes the node; it does not complete it. Completing the node is a separate entry. Withdraw closes q1 without an answer. Timestamps 9 months old change nothing. |
| 5 delegation | The child and the parent link each other. Parent completion is refused for a `not-started`, a failed (`stopped`) or a missing child, and allowed for a `completed` one, with the child's state and seq kept as evidence. A three-level chain completes bottom-up. |
| 6 Git | Two devdocs commits, read with `--rev`, each give the fixed workflow and the stage at that commit (seq 2: survey running; seq 6: ask waits on q1). The history between them is kept. |
| Stale views | `--expect-seq` / `expectSeq` refuse an answer based on an outdated sequence (409). |

Criteria 7–9 (UI behavior, latency, the person's route) are steps 3–5.

## CLI example

These commands ran on scratch data with a branch/join workflow:

```text
$ wfe run create ship --braindump bd.md --author "A. Person" --executor "Omni Agent" --backend claude-code
Created run ship/run-001 in devdocs/ship/runs/run-001/
  devdocs/ship/runs/run-001/braindump.md
  devdocs/ship/runs/run-001/definition/ship.yaml
  devdocs/ship/runs/run-001/run.json
Ready: survey
Nothing was committed.
$ wfe run ask run-001 ask --question "Small or large map?" --to "A. Person"
Recorded question q1 to A. Person; ask waits on it.
Run ship/run-001 is in-progress (seq 6); waiting: ask (next move: A. Person)
$ wfe run start run-001 join
wfe run: node "join" is not ready: build is running, ask is waiting
$ wfe run answer run-001 q1 --answer "Small." --by "A. Person"
Recorded an answer to q1. A node waiting on it keeps waiting until the answer is taken up.
Run ship/run-001 is in-progress (seq 7); waiting: ask (next move: Omni Agent (answer recorded, not yet taken up))
$ wfe run show run-001
Run ship/run-001 — in-progress   (devdocs/ship/runs/run-001)
Input: braindump.md, the words of A. Person (recorded by Omni Agent)
Executor: Omni Agent (backend: claude-code)
Definition (fixed): ship in devdocs/ship/runs/run-001/definition/
  ship  approvals at creation: intent unapproved, definition unapproved; current devdocs/workflows/ship.yaml: same
…
Nodes (1 pending, 1 running, 1 waiting, 1 completed):
  ask      waiting   waits: question q1; next move: Omni Agent (answer recorded, not yet taken up)
  build    running   reported started 2026-10-10T10:15:26.969Z; last update … (1 s ago)
  join     pending   waits for predecessors
  survey   completed outcome: Surveyed.
Questions:
  q1 answered  to A. Person (node ask), asked by Omni Agent: Small or large map?
     answer 0 from A. Person via cli: Small.
History (last 5 of 7; --history for all): …
```

## Decisions made while implementing

- **Answers record `from` and `by` separately.** An agent that relays the
  person's reply from VS Code records `from` = the person and `by` = itself.
  Without this, either the agent would appear as the author, or the person
  would appear to have typed it into the tool.
- **A generated child `request.md` says it is generated.** `delegate` without
  `--request` quotes the node description. The text states that no person
  authored it.
- **The child's `definition.workflows[*].source`** is the parent's bundle
  file, for example `devdocs/ship/runs/run-001/definition/sub.yaml`. That is
  where the bytes came from, not `devdocs/workflows/`.
- **Delegation checks the parent first, without writing.** The parent
  operation is validated before anything is written. A refused delegation
  (wrong node type, node not running, outdated sequence) therefore creates no
  orphan child.
- **Artifacts are bounded to the project.** `attach` and `complete
  --artifact` accept a path relative to the current directory, a file name in
  the run folder, or a project-relative path. The path is stored
  project-relative, and a path outside the project is refused.
- **Repository context** records only the project root and the repositories
  the bundled workflows bind: head, branch and dirty count.
