# retire_old_ui p1 — Step 4: obsolete UI and its dedicated runtime removed

agdevworld `dbd974b`; devdocs README_DEV updated. pj-agdev's launchd
template and its README are updated in the working tree and committed with
the submodule pin. Nothing is deployed yet (Step 5).

## Frontend (`src/`, `public/`, `scripts/`)

Deleted:

- World views: `worldViews.ts`, `views.ts`, `viewSwitcher.ts`,
  `scenes/PanelGridScene.ts`, `detailPopup.ts`, `clusterState.ts`,
  `agentRoomState.ts`, `opsState.ts`.
- Operation room dashboard and graph page: `operationDashboard.ts`,
  `operationParts.ts`/`.css`, `sessionGraph.ts`, `routineState.ts`,
  `chatPanel.ts`, `completionPanel.ts`, with their polling loops.
- Cost gauge page: `gaugePanel.ts`/`.css`, `costState.ts`, `budgetState.ts`.
  The useful content moved to the dashboard in Step 2.
- Sample fallbacks `public/*.sample.json`, the ignored live snapshots
  `public/cluster/` (and its `.gitignore` line), and the cluster snapshot
  importer `scripts/fetch-cluster-state.mjs` with its test and the
  `cluster:fetch` script. No other consumer exists (Step 1).
- `completionState.ts` keeps the plan words and the Arguing Room's door.
  Its generic `/complete` source, the history read and the page-wide
  "completed" event lost their callers and are gone.

Kept as they are: the Front Desk and the Arguing Room with their modules,
Phaser, `public/frontdesk/`, and the favicon. The bundle is now two chunks
(entry and room).

## Relay (`agentroom/`)

Routes removed:

- `/agents`, `/work`;
- `/ops`, `POST /ops/confirm`;
- `/routines`, `/routines/<n>`, `POST /routines/<n>/start`;
- `/inflight/<n>`;
- `POST /chat`;
- `/complete/plan`, `/complete/history`, `POST /complete`;
- `/frontdesk…` (GET and POST);
- `POST /argues`, `/argues/<a>/post`, `/argues/<a>/render`. These had
  been refused since p8 and are now plain 404s.

Kept:

- `/healthz`;
- `/settings…`, `/contexts…`, `/progress` (the Front Desk's panels);
- `/argues`, `/argues/<a>`, `/argues/<a>/close-plan`,
  `POST /argues/<a>/close`;
- `/budget`;
- `/cost`, now history only.

Runtime parts removed with them:

- The ops engine's board snapshot, row evidence and provenance, confirmation
  persistence, the routine board and detail, and the in-flight signal
  (`inflight.py`). What is left is the conversation index that the argue
  completion preview walks, derived on demand from the mirror. It has no
  background thread; it never had one.
- The agent room (`Room`), the legacy desk door (`FrontDesk`; its topic
  naming stays for the completion rules), the chat door's routine posts and
  run requests, and the Arguing Room's writes.
- `/cost` loses its roster and routine attribution (`missing`,
  `routines`). The agent-roots parsing moved into `cost.py`.
- `main.py` no longer builds a room or a desk. The progress board's viewer
  is the chat credential's id. `serve.sh check` prints the mirror, chat,
  cost, settings and budget.

The relay process stays: the Front Desk and Usage still read it. No service
was stopped or replaced. `AGENTROOM_STALLED_SECONDS` is dropped from the
launchd template. An installed plist that still sets it is harmless,
because the variable is no longer read.

## Documentation

- `agdevworld/README_DEV.md` is rewritten around what remains:
  - Home = the dashboard;
  - the Front Desk and Arguing Room bundle;
  - the entrance rule;
  - the relay;
  - Usage;
  - the workflow editor.

  The retired screens are named in its head, with their history left to git
  and the episodes.
- `agentroom/README.md` covers the surviving routes. The completion section
  says that only the Arguing Room reaches the door now, and `/cost` is
  described as history.
- `workflow_editor/README.md`: Home and the route header.
- `pj-agdev/devenv/launchd` template and README: the reason each variable
  is still there.
- `devdocs/README_DEV.md`:
  - a new section, *The agdev site (retire_old_ui p1)*;
  - the Opsroom Observer section describes what the mirror still serves;
  - historical mentions of the operation room, ops board, cost gauge,
    in-flight view and `cluster:fetch` are marked retired. Episode files
    are untouched.

## Validation

- `npm run build` (agdevworld bundle): passes.
- `cd agentroom && uv run pytest -q`: 140 passed.
  - Removed: tests of the removed features (`test_ops.py`,
    `test_routines.py`, `test_frontdesk.py`, the agent room tests in
    `test_room.py`, the argue write tests, and the HTTP tests of
    `/complete` and `/frontdesk` close).
  - Updated: the argue HTTP tests, which now expect the old write routes to
    be 404 and not listed, and the cost test, which now has no roster or
    routine attribution.
  - The completion rules' unit tests remain, and argue close is still
    exercised over HTTP.
- `pyflakes` over the changed relay modules: clean, apart from unused
  imports in `close.py`, `closing.py` and `routines.py` that were there
  before this step.
- The WFE check is unchanged from Step 2 except a sentence in Home; it runs
  again in Step 5.

## Not removed, and why

- `routines.py` and the routine kinds in `closing.py`: the completion rules
  still classify historical routine topics when an argue's closure reaches
  them.
- The Arguing Room: it was not among the screens to retire, and it holds
  historical argues (Step 1).
- Progress panel internals (`progress.py`, pyagag `agag.progress`): these
  belong to the Front Desk, which keeps its current behaviour in this phase.
