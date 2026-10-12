# Workflow editor p7/ex1 — step 1: catalog, instantiation and snapshot contracts

Commit: agdevworld `54ac34b` (documentation only).

## What was inspected

- **Code and release.** The editor is agdevworld `cde57cd`, which is also the installed release (`wfe-release.sh status`: current `cde57cd`, previous `cb65490`, no open attempt). The executor is agautolab `d31f654`. The editor composes the execution brief (`server/exec.ts` `brief`). autolab only prepends its common guide and the project's guides (`wfexec.py` `prompt`). So delivering a run's catalog needs no autolab protocol change: only the common guide's facts change.
- **Operating area.** The area is the registry's directory. Its configuration sits beside `registry.json`: `gitea.json`, `system.json`, `zulip.json`, `present.json`. Every `wfe` invocation chooses the registry explicitly, and agents in runs reach the same area through `bin/wfe`.
- **The run-004 trial** (`rts-vs-bot` `improve-game/run-004`, completed and accepted 2026-10-12). The braindump asked for Web research on RTS formation movement before implementing. `plan.md` has a "事前調査（web）" section of four approaches, with no sources, and says details "will also be in `_report.implement`". `report-implement.md` contains no research. The graph stayed `implement → verify → present → human-check → cleanup`, plus an r2 correction after review. No study node was proposed. This is the evidence the plan cites, and it is left as it is.
- **Records and processes.** All runs are completed (`rts-vs-bot` run-001…004, `ppt` 4). There is no open attempt and no live presentation.
- **Mechanisms reused.** p7 appearance tokens and presets (`NODE_PRESETS`); run adjustments (`run.adjust`, source kinds answer, instruction, agreement, person; `adjustRun` file rules); the definition bundle hash check; `attempt.begin` `settings` (contract and guides by hash).
- **Policies read:** `devpolicy/styles.md` (agent guides: facts, not procedures; usage in `--help`; trial references move with their paragraphs), `devpolicy/terms.md` (Tool Giving, no Tool Implantation), `localrule.md`.

## Contracts written

- `workflow_editor/docs/catalog.md` (new): where the catalog lives, the schema, read and write states, stale edits, initialization, examples, instantiation, provenance and digests, run snapshots, attempts, what is delivered, request-sourced adjustments, and the interfaces.
- `docs/adjustments.md`: source `{kind: "request", input, sha256}`, and when to use it versus `agreement`.
- `docs/contract.md`: *Template provenance*, and provenance listed among what the digests exclude.

## Decisions

| Question | Decision |
| --- | --- |
| Storage | `<area>/node-catalog.json`, `ag.node-catalog.v1`. User data beside the configuration, outside workspaces and releases. Portable content only (no host facts, paths or repository names) |
| Template identity | Stable `id`, never reused after deletion (`deleted` list). Integer `revision`, +1 only when content changes (name, type, description, expectation, appearance) |
| Types | `study`, `do`, `talk`. `delegate` is refused, so the catalog cannot suggest delegate execution. Templates have no type-specific fields. `repositories` are chosen at instantiation from the target workflow's declared keys only (no new access). `workflow` and `confirms` are never set by instantiation |
| Read states | `ok` (an empty list is still `ok`) or `unavailable` (with the problem). A missing file is initialized. An unavailable file refuses writes and is never overwritten |
| Writes | One shared module (`server/catalog.ts`) for HTTP, CLI and run creation: lock, validate, atomic replace. `expectRevision` on update and delete (`template-stale`) |
| Presets | Moved into the catalog at a one-time initialization recorded in the file (`initialized.seeded`). The file existing is what prevents re-seeding. The palette loses its built-in presets |
| Example | *Preliminary research* is offered, never added automatically |
| Instantiation | A pure shared function produces an ordinary node: type, name, description and appearance copied; `template: {id, revision, expectation}` as provenance; a unique node id; optional `after` and `before` edges |
| Digest and approval | Provenance is outside both digests and the compiled graph. A provenance-only change is `adjust-nothing`, and the provenance of started work cannot be rewritten. Copied semantic fields keep their usual effect |
| Run snapshot | At creation, through every entrance: `<run>/context/node-catalog.json` and `run.json` `context.catalog` `{state, file, sha256, generation, templates}`. Empty means an explicit empty snapshot. Unavailable means `{state: unavailable, problem}`, and the run is not refused. The hash is checked on read, like the bundle |
| Attempts | Every attempt gets the run's snapshot (never the live catalog), recorded in `attempt.begin.settings.catalog` |
| Delivery | The brief gives the plan's statement and each template's id, name, type and expectation, then names `wfe run nodes`, `wfe nodes instantiate --run --file` and `wfe run help adjust`. Usage stays in help |
| Request-sourced adjustment | `{kind: request, input, sha256}`. `run.create` records `input.sha256` from p7/ex1 on. The file layer checks the file still has it. Older runs cannot use this source |

## Schema and cutover

| Artifact | Change | Version |
| --- | --- | --- |
| `node-catalog.json` | new area file | `ag.node-catalog.v1` |
| Workflow files | optional `nodes.<id>.template` | stays `ag.workflow.v2`. Older releases preserve the unknown field and compute the same digests |
| Compiler | none | `ag.wfe-compile.v2` |
| `run.json` | additive: `input.sha256`, `context.catalog`, `attempt.begin.settings.catalog`, adjust source `request` | stays `ag.workflow-run.v2`. Existing runs replay unchanged and are not converted |
| Stores | none planned | exec 1, notify 3, intake 1, present 1 |
| autolab | the common guide's facts only | no protocol change |

**Rollback limit.** A release before ex1 reads runs carrying `context.catalog` (the reducer stores context as given). It cannot read a run that recorded a `request`-sourced adjustment: it refuses the unknown source kind, and the run becomes unreadable but undamaged. Presets would come back to the old palette, and the catalog file would simply be unused.

## Next

Step 2: `server/catalog.ts` and `shared/catalog.ts`, `wfe nodes`, the HTTP routes and the dashboard's User nodes editor.
