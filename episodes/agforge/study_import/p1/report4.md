# study_import p1 — step 4: forge's access to knowledge, and the hand-over into runs

Date: 2026-09-20. agforge `a7297bc` (pj-agdev `16f6617`), listener restarted
on it at 11:24Z. Implemented by the Omni Agent — **did the forge-side code
for agent autolab? No: this is forge's own repository and there is no
in-system agent that edits agforge, so no hand-off candidate is recorded.**

## What forge can reach now

`agforge knowledge`, four verbs, all behind the one CLI every role already
has (`Bash(agforge:*)`), so no Read grant outside the workspace and no
`--add-dir` is needed on any harness — the plan's "reachable under the real
role's tool grant and PATH" was checked by running the bare `agforge` from
`role_run.tool_environment()`'s PATH in a scratch directory:

| verb | what it prints |
|---|---|
| `list` | every source with kind, git revision and **every** `INDEX.md` row, whole — unverified rows with their state, nothing pre-selected |
| `show <source>/<path>` | one file, or a directory's entries; a reference cannot climb out of its source; `.local/` is never listed |
| `search <terms…>` | `source/path:line: text` for lines holding every term, bounded at 200 with the bound said |
| `path <source>[/<path>]` | the absolute location, so a localised script is run in place with `uv run --with …` |

Sources come from the ignored `.local/knowledge.toml`
(`knowledge.example.toml` committed): `mediagen` (`general`, `main/`) and
`localize` (`local`). The distinction the plan asked for — general
knowledge, local knowledge, project requirement, per-run instruction — is
made in the guides: the planner's guide names the four and the order they
win in (chat > `required_items.md` > local > general), the front quotes the
requester's requirement into `required_items.md` with its origin, and the
run's guide says `chatlog.md` overrides `plan.md` where it is more specific.

## Revisions recorded

- `record_plan` writes `[selfnote][knowledge] mediagen@<rev>, localize@<rev>`
  beside `[tools]` every time a plan is posted; `Request.knowledge` reads it
  back (`None` = predates the record, `""` = no source configured). The
  plan's registration line says it too: `recorded a… (toolsets: …;
  knowledge: mediagen@c415b0c, localize@0bc3e90)`.
- `assetrun` writes `knowledge.md` into the run workspace: planned-against
  stamp, current stamp, and a "moved since the plan" line per source whose
  revision changed. Unadopted sources are not pinned — the run reads live
  and is told to say what it used.
- The plan guide asks the planner to cite what it relies on as
  `<source>/<path>` and whether it is verified here, so the plan post itself
  carries the references a later run resolves.

## Async

Unchanged: `pending.json` / `watching.json` / `collected.txt` and the
notifier already keep a submitted job's plan and tools frozen while
collecting; `knowledge.md` is written only on a non-collecting refresh, so a
collecting run keeps the file its job was submitted with.

## Tests

`uv run pytest -q`: 260 passed. New `tests/test_knowledge.py` (15 tests:
config, listing, containment, search bound, CLI, stamp round trip,
`knowledge.md` wording). The assetplan and assetrun fixtures now point
`knowledge.CONFIG_PATH` at nothing, so the suite does not depend on this
host's config.

## Left as is, on purpose

- Toolset documents are still copied into `tools/`; the `speech` toolset is
  still an empty body. `list` makes the state of *knowledge* visible; the
  state of a *toolset* is a separate gap (step 6 notes it).
- No index of forge's own toolsets is merged into `list`; `agforge toolsets
  --list` stays the front's call, as before.
