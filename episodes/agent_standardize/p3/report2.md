# p3 step 2 — agautolab vocabulary (and agfront's fixtures)

`mission-` → `workplan-`, `run-` → `workrun-`, `create-asset_` →
`assetplan-asset_`. Code and tests only; no listener was reloaded, so the
live realm still speaks the old vocabulary until Step 3.

Commits, all pushed:

- `agautolab 5675943` — the rename.
- `agfront cc36fd0` — intro-text fixtures and two doc comments.
- `pj-agdev 78c2382` — submodule bumps.

## agautolab

Prefix constants in `zulip_listener.py`:

| old | new |
|---|---|
| `MISSION_TOPIC_PREFIX = "mission-"` | `WORKPLAN_TOPIC_PREFIX = "workplan-"` |
| `RUN_TOPIC_PREFIX = "run-"` | `WORKRUN_TOPIC_PREFIX = "workrun-"` |
| `CREATE_TOPIC_PREFIX = "create-"` | `ASSETPLAN_TOPIC_PREFIX = "assetplan-"` |

`BMINING_TOPIC_PREFIX`, `PROJECT_CHANNEL_PREFIX` and `WORK_CHANNEL_PREFIX`
are untouched, as the plan says.

Derived and adjacent names:

- `RUN_TOPIC_NAME` → `WORKRUN_TOPIC_NAME`,
  `^run-task(\d+)-(.+)$` → `^workrun-task(\d+)-(.+)$`. Planning therefore
  opens `workrun-task<N>-<label>` topics from now on.
- `mission.ASSET_TOPIC_PREFIX = "create-asset_"` →
  `"assetplan-asset_"`. The `asset_<issue>` substructure is kept, as the
  plan scoped it; collapsing it further stays deferred.
- `handle_run` → `handle_workrun`, `handle_create` → `handle_assetplan`,
  and the helper `run_supercoder()` → `workrun_supercoder()`.
- The sweep tuple in `main()`, so the startup log line now prints the new
  prefixes.

Guide folders renamed (`git mv`; their *contents* needed no edit — they
carry no prefix vocabulary, confirming the plan's survey):

| old | new |
|---|---|
| `agent/guides/mission_superdirector/` | `agent/guides/workplan_superdirector/` |
| `agent/guides/run_supercoder/` | `agent/guides/workrun_supercoder/` |
| `agent/guides/create_answer_superdirector/` | `agent/guides/assetplan_answer_superdirector/` |

Plus docstrings and comments in `mission.py`, `zulip_listener.py`,
`role_run.py`, `agents.toml`, `README.md`, and ~50 test references.

**Deliberately not renamed.** "Mission" survives as a *domain noun* — a
mission is still the thing a `workplan-` topic plans. So `mission.py`,
`mission_directory()`, `MISSION_DIR_TITLE_CHARS`, the `mission:` field in
the `work-` channel binding, and phrases like "a mission-level cancel" stay
as they are. Only the *topic prefix* moved. The `[AUTO]` / `FORGEAUTO` Plane
markers are likewise not prefixes and are untouched.

## agfront

Three intro-text fixtures (`test_agents_md.py`, `test_zulip_listener.py`,
`test_role_run.py`) now say ``Open an `assetplan-…` topic``, and two doc
comments in `src/agfront/zulip_listener.py` were reworded. **No code
change** — criterion 2 requires Front to reach an `assetplan-` topic from
the harvested intro alone, so nothing in agfront may teach it the prefix.
One of those comments described the retired p2 behaviour ("posted it into a
`create-` topic in `#general`"); it now says "an asset-request topic … whose
name it derived itself", which keeps the history true without keeping the
dead word alive.

## Evidence

- `agautolab`: `uv run pytest -q` → **170 passed**.
- `agfront`: `uv run pytest -q` → **31 passed**.
- Criterion-3 grep over agautolab `src tests agent README.md agents.toml`
  and agfront `src tests`, `grep -rn 'create-\|runcreate-\|mission-\|"run-'`,
  leaves only two documented false-positive classes:
  - `run-0001.json` / `run-{n:04d}.json` — the `ag.agent-run.v1` record
    filename in `agent/gateway.py` and `tests/test_gateway_window.py`.
    Same non-prefix as pyagag's.
  - `("create-channel", …)` — a fake Zulip client's *call label* in
    `tests/test_zulip_listener.py`, recording that `create_channel` was
    called. Not a topic name. Left alone rather than renamed to flatter the
    grep.
  - `mission-level` / `mission-planning` in two docstrings — English
    compounds on the surviving domain noun, not prefixes.

## Node deploy

- **agautolab1: done.** `nctl render production` (6 hosts, 21 placements,
  all applied) then `setup_autolab_node.yml --limit agautolab1` with
  `AUTOLAB_NODE_PLANE_CREDENTIALS_SOURCE=…/.local/plane/autolab.env`.
  `ok=21 changed=3 failed=0`; the gateway restarted and its health probe
  passed. Verified: `git -C /home/eiji/agautolab rev-parse --short HEAD` →
  `5675943`. As the plan notes, the node runs no Zulip listener — only the
  gateway — so this changes nothing about topic routing.
- **agstudio: not needed, and the playbook cannot run here anyway.**
  `--limit agstudio` fails in `claude_code_agent`, before any autolab task:
  it asserts a user-scoped npm at `/Users/eiji/.local/node/bin/npm`, which
  does not exist on this Mac — node is the Homebrew `v26.6.0` at
  `/opt/homebrew/bin/node`. This is a **pre-existing** placement/environment
  mismatch, unrelated to the rename, and left alone as out of scope. It
  costs nothing here: the agstudio placement's "deployment checkout" *is*
  `pj-agdev/agautolab`, the developer's live working tree, which is already
  at `5675943`. Worth its own ENT episode.

## State after this step

Both repos speak the new vocabulary on disk; both running listeners are
still on old code and still sweeping `create-`/`runcreate-`/`mission-`/
`run-`. The cross-repo asset contract is consistent again — autolab orders
into `assetplan-asset_<issue>` and forge sweeps `assetplan-` — but only
once both listeners restart together, which is Step 3.
