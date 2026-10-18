# p13 Step 1 — sources and the resolver

## Inventory: what an agent is given today (before p13)

Traced from `server/exec.ts` (claim, brief, continuation), autolab
`wfexec.py` (prompt) and the authoring templates.

| Path | Direct text | Referenced | Generated facts | Loaded by the harness, not WFE |
| --- | --- | --- | --- | --- |
| Bootstrap (new session) | autolab's common contract `wfrun/guide.md` (host-owned, offered by hash); each `execution.guides` file appended as `# Project guide: <path>` | the request file, `plan.md` | the brief: identity, copy, why, pending answers and instructions in full, presentations, shares, delegations, catalog, documentation steps | Claude Code: `CLAUDE.md` of cwd ancestors, user settings and hooks; codex: `AGENTS.md` up the tree |
| Continuation (resumed session) | none | — | the continuation: delta, compact state, pending input, new steps | as above (the conversation keeps the bootstrap) |
| Reconstruction (compacted) | the whole bootstrap again | as bootstrap | as bootstrap | as above |
| Authoring (IDE) | `AGENTS.md` of a `wfe setup` area | `docs/contract.md`, `docs/runs.md` | `wfe status` on demand | the IDE's own configuration |

Findings:

- Project intent, goals and workflow intent were never given to a run; the
  agent found them only by reading files.
- `agent/guide.md` is a project guide in seven projects (agsfx, archsage,
  cagent, forge, jevlab, playprey, ppt) but is copied into every workflow's
  `execution.guides`; agops' `agent/guide.md` belongs to `recover` only, and
  rts-vs-bot uses cross-project guides of archsage in one workflow.
- No `CLAUDE.md`/`AGENTS.md` sits in any ancestor of the run copies today
  (projects directory, `pj-agdev`, the area, the workspaces), and
  `~/.claude/CLAUDE.md` does not exist. User `~/.claude/settings.json` is
  loaded by every claude_code run. The operating area has no authoring entry.
- WFE controls the brief, the continuation and (since p13) the context
  bundle. It does not control the contract's text, the harness's own loading
  or an IDE session. docs/context.md says so; the bundle is never called the
  whole prompt.

## What was built (agdevworld `workflow_editor/`)

- **Model** `shared/context.ts` (`ag.wfe-context.v1`): sources with scope,
  category, applicability, origin, provenance, delivery and sha256; the
  fingerprint over effective sources (no run input, no contract, no
  metadata); rendering; per-source differences.
- **Project guides**: `project.yaml` `guides` (model, YAML shape,
  validation `project-guide-path`/`-duplicate`). Basic workflows (`freework`)
  no longer copy `agent/guide.md`; their notes name the inherited project
  guides.
- **Override store** `server/contextStore.ts`: `<area>/.local/contexts/`,
  per-scope Markdown plus `<file>.revisions.jsonl` (author, time, sha256,
  exact content, removal), `base`-checked writes (`409 context-conflict`),
  hand edits kept as external revisions, validated ids, 64 KiB limit,
  content-addressed snapshots.
- **Resolver** `server/context.ts`: one function for execution and
  authoring. Run defaults from the run's copy and its definition in force,
  cross-project guides from committed main with the commit recorded,
  overrides from the area. One source per guide with every declaration;
  `absent` for a missing override; `error` for an unreadable configured
  source.
- **CLI** `wfe context show|get|set|clear|history|run|list` with the
  scope/storage table, source selection, applicability and delivery modes in
  `--help`; human and `--json` output; `--rendered` prints the exact text.
- **API** `GET api/context`, `GET/PUT api/context/override`,
  `GET api/context/overrides`, `GET …/runs/<wf>/<run>/context`.
- **Contract** `workflow_editor/docs/context.md`; contract.md, runs.md,
  sessions.md and README updated.

## Checks

- `test/context.test.ts` (6): store revisions/conflict/removal/hand edit/id
  validation; resolver scopes, provenance, guide deduplication, project
  isolation, authoring applicability, error sources; fingerprint (content vs
  metadata, clearing restores the inherited bundle); claim snapshot equals
  the handed text and the record has no override text or host path, a later
  edit reaches only the next attempt, missing snapshot reported unavailable;
  session comparison; delegation scope.
- Whole WFE suite after the change: see report2 (run together with Step 2).

## Deferred to Step 5

Managed definitions (`project.yaml guides: [agent/guide.md]` and removing it
from each workflow) are changed right after the release is deployed: the
installed release ignores `project.yaml guides`, so changing them earlier
would drop the guide from runs claimed in between.
