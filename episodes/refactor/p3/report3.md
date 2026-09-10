# refactor p3 step 3 — the obsolete UI and the shared client are removed

## What was asked

Delete the `tasks / plane` view, its navigation entry, `planeState.ts` and its
dispatch code; trace callers before touching gateway code and remove only the
Plane-specific parts; remove `agag.plane` once nothing needs it, with its
tests, dependencies and configuration; update package pins and deployment
templates so reinstalling an agent cannot restore Plane credentials; and
inspect older deployed checkouts rather than assuming the local tree is every
deployment.

## What changed

### The `tasks / plane` view, and what tracing its callers found

`src/planeState.ts`, `tasksViewConfig`, its `worldViews.ts` scene and its
`VIEW_KEYS` entry are deleted. The cycle is six views:
`nodes → workspaces → autolab → agentroom → ops → routines`. `autolab`'s
`switchTo` named `tasks` and now names `agentroom`, so the cycle closes.

**Tracing the callers is what made the scope obvious, and it is the finding
worth keeping.** The view called `/api/plane/states`, `/api/plane/issues`,
`PATCH /api/plane/issues/<id>` and `POST /api/autolab/<node>/window`. Every
one of those went through the assistant gateway that `modernize_agdevworld`
p1 deleted in August; `nginx.conf` serves static files and nothing else, and
there is no `/api` proxy in the dev server either. So this was not a working
feature being retired for Plane's sake — it was a screen whose every action
had been failing for a month, and the plan's warning that "the existence of
this frontend code does not establish that its backend route still works" was
exactly right. There was no gateway left to clean up, so nothing else was
touched: the conversational windows, evidence reads and service functions the
plan asked to retain are the relay's, and they are untouched.

`autolabState.ts` calls the same dead gateway and was **not** removed: it is
not Plane-specific, three live views read it, and its scope is another
episode's.

### The relay's last Plane residue

`WorkTarget` carried `project_id`, `project_name` and `issue_id` as
permanently empty strings, kept in p2 so a consumer reading `issue_id` got an
honest "there is none". With Plane gone there is nobody left to be honest to,
and a field nothing can ever fill invites a reader to look for the system
behind it. They are removed, along with the unused `_issue_label` helper;
`WorkTarget.key` is the label alone. Nothing in the frontend read them.

### `agag.plane` is gone

p1 moved autolab's record into the conversations, p2 forge's, p3 step 1
cagent's. A sweep of every project for `agag.plane` / `from agag import
plane` found **no product consumer** — only pyagag's own
`tests/test_plane.py` and one assertion in `tests/test_document.py`. The
module and its test are deleted. `agag.document`, which is what the client
carried that was never about Plane, stays. `agag init`'s human checklist no
longer tells anybody to create a Plane account for a new agent.

The three `test_plane_independence` / `test_no_plane` suites are kept. They
were about one agent's restraint while the module existed; now they are what
stops it coming back, and their docstrings say so.

### Reinstalling an agent cannot restore Plane

`roles/agag_agent/templates/plane.env.j2`, the
`agag_agent_plane_credentials_source` default and
`autolab_node_plane_credentials_source` / `autolab_node_plane_config_file` are
deleted, with the `AUTOLAB_NODE_PLANE_CREDENTIALS_SOURCE` instructions in
`ansible_agdev/README.md`.

A role that merely stops writing a file leaves the old one in place, so a
task was **added**: `Remove the retired Plane configuration` deletes
`.local/plane.env` on the node. That is what makes the claim true of the
nodes that already have one, not only of the next install.

Every agent project's `uv.lock` was moved to pyagag `0d26145` — agautolab,
agforge, agfront, agentroom, arxivsage, agecho and cagent — so a fresh
install cannot resolve a pyagag that still contains the client.

### The older deployed checkout

`agautolab1` — the node p1 and p2 deliberately left alone — was inspected
over Ansible rather than assumed:

- `~/agautolab` is at `5675943` (`agent_standardize` p3 step 2), months
  behind;
- its venv's `agag/` still contains `plane.py`, and so does agecho's on the
  same node;
- `~/agautolab/.local/plane.env` exists and holds `PLANE_URL`,
  `PLANE_API_KEY`, `PLANE_WORKSPACE_SLUG`.

It is therefore a real remaining Plane-dependent path. It is **not** repaired
in this step: pulling the credential out from under a checkout whose code
still imports the client would break its listener rather than free it. The
redeploy — new checkout, new pin, and the removal task above — is step 4's
deployment work, where it belongs beside stopping the service.

## Verification

- **Frontend build passes**: `npm run build` (`tsc && vite build`) clean, and
  the image rebuilt and redeployed to `:8090` (`HTTP/1.1 200 OK`).
- **Navigation has no Plane task screen.** In the shipped bundle
  `dist/assets/worldViews-D-4aM360.js` the view cycle reads
  `nodes, workspaces, autolab, agentroom, ops, routines` — six keys, no
  `tasks` — and `/api/plane`, `planeIssue` and `dispatchPlane` are absent
  from it. Screenshots of the running `:8090` build are in
  `agdevworld/.local/shots/p3*/`.
- **Suites.** pyagag 515 passed. agentroom 267. agautolab 237. agforge 221.
  agfront 103. cagent 198.
- **The search, and what it deliberately kept.** Active source,
  configuration, startup jobs and deployment templates were searched for
  `plane`. What remains is historical text (old reports, migration files,
  docstrings that explain what a module used to be) and the unrelated sense
  of the word — `cagent status`'s "health of the control plane itself".
  Neither is a dependency.

## Limitations

- `agautolab1` still holds an old checkout, an old pyagag with `plane.py`,
  and a `.local/plane.env`. Step 4 redeploys it.
- The Plane stack itself is still running and still declared as an expected
  service. Step 4 stops and removes it.
- `autolabState.ts` still calls the deleted `/api/autolab/*` gateway. That is
  not Plane's, and repairing or removing it is out of this phase's scope.
