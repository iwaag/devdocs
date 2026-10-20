# retire_old_ui p1 — Step 1: dependency and data boundaries

Read-only investigation, 2026-10-20. agdevworld `075500c`, agautolab as
deployed, relay and WFE service as running on the coordinator host.

## Frontend classification (`agdevworld/src/`)

Front Desk closure, traced from `frontDesk.ts` imports and the scene's calls:

| Keep (Front Desk / Arguing Room) | Network calls |
|---|---|
| `frontDesk.ts`, `scenes/FrontDeskScene.ts`, `scenes/roomTint.ts` | — |
| `roomState.ts` (desk adapter → WFE `api/conversations…`; argue adapter → relay `/argues…`) | WFE, relay |
| `roomDemo.ts`, `frontDeskPlayback.ts`, `frontDeskInput.ts`, `textLayout.ts`, `frontDeskSettings.ts`, `settingsState.ts` | relay `/settings…` |
| `contextPanel.ts`, `contextState.ts` | relay `/contexts…` |
| `progressPanel.ts`, `progressState.ts` | relay `/progress` |
| `frontDeskClosePanel.ts`, `postMeaning.ts` | — (desk close goes to WFE; argue close to relay `/argues/<a>/close[-plan]`) |
| `completionState.ts` — only `relayCompletionAt`, the plan types, `planLines` words and `announceCompleted` are used by the rooms | relay `/argues/<a>/close…` |
| `endpoints.ts` | — |

The Arguing Room (`/?view=argue`) is the same scene with another adapter.
Its writes have been refused by the relay since project_migration p8; it reads
argue history and can close one. It is not among the screens this phase
retires and it holds historical conversations, so it stays as it is.

Retired UI only (no import from the Front Desk closure):

- World views: `worldViews.ts`, `views.ts`, `viewSwitcher.ts`,
  `scenes/PanelGridScene.ts`, `detailPopup.ts`, `clusterState.ts`,
  `agentRoomState.ts`, `opsState.ts` (relay `/agents`, `/work`, `/ops`,
  `/ops/confirm`).
- Operation room dashboard and graph page: `operationDashboard.ts`,
  `operationParts.ts`, `operationParts.css`, `sessionGraph.ts`,
  `routineState.ts` (relay `/routines…`, `/inflight/…`, `POST /chat`,
  `POST /routines/<n>/start`), `chatPanel.ts`, `completionPanel.ts` and the
  `relayCompletion` / `loadCompletionHistory` parts of `completionState.ts`
  (relay `/complete…`).
- Cost gauge page: `gaugePanel.ts`, `gaugePanel.css`, `costState.ts`,
  `budgetState.ts` — its useful content moves to the dashboard (Step 2).
- Assets and fallbacks: `public/*.sample.json`, `public/cluster/` (ignored
  live snapshots), `scripts/fetch-cluster-state.mjs` and its test, the
  `cluster:fetch` npm script. `public/frontdesk/` and `favicon.svg` stay.
- `index.html`'s 340 px right inset exists for the world views' chat overlay
  (the Front Desk resets it).

Phaser stays: only the Front Desk scene needs it after the removal.

## Relay classification (`agentroom/`)

External callers: none. A search of every project (Python, TypeScript,
shell, launchd, configs) found no caller of relay routes outside
`agdevworld/src/`. pyagag `agag.progress` is a library the relay calls, and
the Observer watchdog left the relay in observer p3.

| Route | Consumer after p1 | Decision |
|---|---|---|
| `/healthz`, `/` | operators | keep |
| `/settings…`, `/contexts…` (GET/POST) | Front Desk | keep |
| `/progress` | Front Desk progress panel | keep |
| `/argues`, `/argues/<a>`, `/argues/<a>/close-plan`, `POST /argues/<a>/close` | Arguing Room | keep |
| `/budget` | Usage (through the WFE service) | keep |
| `/cost` | historical pre-WFE agent records only | keep without roster/routine attribution, shown as history |
| `/agents`, `/work` | world views | remove |
| `/ops`, `POST /ops/confirm` | operation room | remove |
| `/routines…`, `/inflight/…`, `POST /routines/<n>/start` | operation dashboard | remove |
| `POST /chat` | operation dashboard (already refused since p8) | remove |
| `/complete/plan`, `/complete/history`, `POST /complete` | operation dashboard completion panel | remove |
| `/frontdesk…` (GET and POST) | none — the desk talks to WFE since p8 | remove |
| `POST /argues`, `/argues/<a>/post`, `/argues/<a>/render` | refused since p8 | remove |

The ops engine (`ops.py`) has no thread of its own: it re-derives from the
mirror when a reader asks. After the removal its readers are the completion
door (`Closer.topics` → `held_topics`, for argue close), the progress board
(its mirror) and the desk object (`developer_id`, roster names). The routine,
in-flight, board-snapshot and confirmation parts lose all readers and go;
`routines.py` and `inflight.py` go except the agent-roots parsing `/cost`
still uses. The mirror and its subscription keeper stay (Front Desk panels,
argues).

## Usage evidence

- **Identity.** One execution attempt = one row of the coordinator's exec
  store (`<area>/.local/exec/exec.sqlite`, table `attempts`, primary key `id`)
  with workspace, workflow, run, worker, backend label, lane, began/ended.
  407 attempts from 2026-10-11 to 2026-10-20, all ended at the time of
  reading; 403 on worker `studio`, 4 on worker `gpupc`.
- **Records.** Each attempt's harness stream is `<area>/.local/exec/logs/<id>.jsonl`.
  Every attempt row has its log on the coordinator. Remote workers ship their
  log to the coordinator at offsets (`workerApi.ts`), so the coordinator copy
  is the one to read and the worker copy is never read: no double counting
  across copies. 116 MB in total; the largest log is 1.7 MB.
- **Fields.**
  - claude_code (stream-json): the final `result` event carries `usage`
    (token counts), `total_cost_usd`, `modelUsage` and `session_id`.
    `modelUsage.<model>.costBasis` is `list`: the USD is the API list price.
    The budget reader's own note says the same: these runs are on a Max
    subscription, so USD here is API-equivalent, not a charge.
  - codex (`exec --json`): `thread.started` gives the thread; each
    `turn.completed` carries `usage` (input incl. cached, cached, output,
    reasoning). No USD; codex runs on a ChatGPT Plus login (subscription).
  - The relay's `normalize_usage` (Python) maps both into
    input/cached/cache-write/output/reasoning; it is reused (ported to the
    service) rather than re-invented.
- **Per attempt or cumulative.** The one resumed claude session in the store
  (two attempts on session `6cc4…`) shows `usage` per process (output 737,
  then 1107) while `total_cost_usd` and `modelUsage` are cumulative for the
  session (0.136 then 0.242; modelUsage output 1844 = 737 + 1107). So tokens
  are taken from `usage` as they are, and a resumed attempt's USD is its
  total minus the same session's previous attempt's total. codex: 61
  attempts, one turn each, no thread seen twice; turns are summed per
  attempt, and an attempt continuing a thread seen before is flagged.
- **Missing.** 11 claude_code logs have no `result` event (interrupted
  attempts): their usage is unknown, never zero. An attempt still running
  has no result yet: in progress, not zero.
- **Retries and resumed sessions** are separate attempts with separate ids,
  so attribution by attempt id cannot double count; the cumulative USD rule
  above is the only adjustment needed.
- **Historical.** Pre-WFE agents (`ag.agent-run.v1` under the relay's agent
  roots: the retired Front, arxivsage, agecho) stay readable through the
  relay's `/cost`, which loses its ops-engine attribution and is presented as
  history, apart from current activity.
- **Subscription windows.** The relay's `/budget` reads this host's CLIs
  (Keychain, `codex app-server`, `agy`) and caches 60 s per provider. The
  remote worker's own harness account is not read by it: its windows are not
  observed, and its USD is likewise API-equivalent.

## Chosen collection path

The WFE service reads its own exec store and the coordinator's attempt logs
(bounded tail reads, cached by file size and mtime) and answers
`GET api/usage` on the dashboard's own origin. For subscription windows and
history it reads the relay on the same host (`/budget`, `/cost`) with its own
timeout, so a relay or provider failure becomes a marked section, not an empty
page. No new telemetry, no executor change, no new process.

## Entrance and routing facts

- Listeners: nginx `:8090` (static site only) and `:8093` (static site +
  `/wfe/` → service `:8098`, `x-wfe-route` set); the service also serves the
  dashboard directly on `:8098`.
- Slack links point at `<origin>/?view=frontdesk&conv=<id>`
  (`integrations/slack/src/config.ts`); they must keep reaching the Front Desk.
- WFE views hard-code `http://localhost:8093/` for their "← agdevworld"
  link outside `/wfe/`.
