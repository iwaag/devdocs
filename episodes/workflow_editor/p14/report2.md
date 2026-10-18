# p14 Step 2 — publication and detail access for agents

## Operations

`wfe context` (help: `wfe context --help`) and the service's HTTP API take
the same operation through one function (`applyBlockOp`):

| CLI | HTTP |
| --- | --- |
| `list [<scope>] [--all]` | `GET api/context/blocks?scope=&project=&workflow=` · `?all=1` |
| `get <scope> <block>` · `history` | `GET api/context/block?scope=&project=&workflow=&block=` |
| `set` · `check` · `refresh-failed [--partial]` · `remove` | `PUT api/context/block {op, scope, project?, workflow?, block, base, by, run?, attempt?, …}` |
| `detail <context:…> [--out F]` | `GET api/context/detail?ref=` (`&raw=1`) |

- **Explicit targets.** Every write names scope, block and `--base` (the
  content revision read, `new` for a new block); project and workflow
  writes require `--project` — a run copy's own project is never assumed,
  so cagent writing forge's context cannot land in its own project by
  default.
- **Remote writers.** `--service <url>` or `WFE_SERVICE` sends every block
  command (reads included) to that WFE service; the CLI then needs no
  registry or area of its own. Without it, the launcher's area store is
  used. Each result says where it was written (`the context store of the
  WFE area on <host>`, plus the service URL when remote).
- **Structured results** (`--json`): `{ok, changed, written, state}` with
  the block, its revision, content base, size, manager and last editor;
  conflicts are refusals carrying the current state.
- **Provenance.** An executing attempt's `WFE_ATTEMPT` is sent along; the
  service (or the local CLI) resolves it to the attempt's run and records
  it as the editor's `run`; `--run` names it explicitly. The entrance is
  recorded as `via` (`cli`, `http`, `browser`). Run records receive nothing
  from these writes.
- **Idempotent publication**: republishing identical content returns
  `changed: false` without a revision, even with a stale base.
- **Failed refresh**: `refresh-failed` keeps the successful body and
  evidence and records the failure separately (delivered with its day); a
  block with no content refuses it.

## Detail and consultation routes

- **Details** are named documents of a block, stored content-addressed in
  the area's store and addressed logically as
  `context:<scope>/<block>/<name>` — the same reference works from the
  executor (`wfe context detail`, locally or `--service`) and from the
  browser (`api/context/detail?ref=…&raw=1`), with no host path.
- **Consultation**: a block's `consult: <project>/<workflow>` (cagent:
  `cagent/consult`) is delivered as "Ask <manager>: the <workflow> workflow
  (a delegate node, or a request in its channel wf-<project>.<workflow>)".
- Existing routes stay as they are and can be evidence references:
  `wfe projects read|run …` for managed project files and run artefacts
  (cagent's scan records and `snapshot.json`), `shared://` for shares.
- Verifying both references from a consuming project is Step 4's trial.

## Checks

- `test/context.test.ts` 12/12, including two process-level CLI checks:
  - from a directory with no registry or area, `WFE_SERVICE` set: create a
    block with evidence, a detail and a consultation workflow; republish
    unchanged (no revision); a stale base is refused with "changed since it
    was read"; `detail` prints the stored bytes; `get` shows manager and
    last editor; a project write without `--project` is refused;
  - the local CLI with the area's registry records a `check` in the same
    store, shown as `[bookkeeping]` in `history`.
  - HTTP: create, conflict (409), detail raw read, list, an invalid project
    id (400), provenance `run` and `via`.
- autolab `tests/test_wfexec.py`: 21/21 with the updated `wfrun` guide.
- Cross-host write against the deployed service from another PC is
  verified in Step 5.

## Revisions

agdevworld `92678d4` (on `d7dbcf9`), agautolab `60ed2fb` (`wfrun` guide:
blocks, environment information, `wfe context detail`).
