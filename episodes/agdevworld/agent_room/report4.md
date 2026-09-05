# Step 4 — the view

The agent room is the **fifth entry in the existing view cycle**, not a new
component: `PanelGridScene` already renders a grid of rectangular cards from a
config, which is the shape the braindump asked for, and four views share it.
`nodes → workspaces → autolab → tasks → agentroom → nodes` is the ⇄ / V cycle.

New files: `src/agentRoomState.ts` (the two reads and their helpers). Changed:
`src/views.ts` (the config), `src/viewSwitcher.ts` (the key), `src/main.ts`
(the scene and the ask handler), `src/detailPopup.ts` (two renderers).

## Two modes

Chips switch between them, the way the autolab view switches projects/jobs.

**agents** — one card per agent: the instance, its own channel, the first
prose line of its introduction, and a badge counting the open topics in that
channel (`3 OPEN`, or `NOTHING OPEN` in green). Clicking it opens the
introduction **as posted**, unabridged, then that channel's open topics, then
the earlier introductions in a collapsed `<details>`. An introduction is a
contract other agents act on, so the popup summarises nothing.

**open work** — the flat list: one card per unresolved topic, showing the
topic name as posted and the channel, filed under its project or its agent.

The headline says how many channels were swept, and if any channel failed it
names them instead. A shorter list and a quieter realm look identical
otherwise.

## A change to `/work` this step made

Step 3's `/work` swept project channels only, and the badge the plan asks for
had nothing honest to count: **project work is not attributable to an agent**.
A `pj-` channel is the project's board and no topic name says whose task it
is — reading `workplan-`/`workrun-` to guess would be exactly the per-agent
vocabulary this episode refuses to depend on.

So the sweep now also covers **each agent's own channel**, which the plan
leaves to the implementer ("どのagent/プロジェクトの分まで巡回対象にするかは
実装者の裁量"). That is where the work an agent actually owns lives — forge's
`assetplan-`/`assetrun-` topics, and every question put to an instance — and
none of it appears in any `pj-` channel. Rows gained `kind`
(`project`/`agent`) and `group`; the badge counts `kind === "agent"` rows for
that instance, and the popup says in as many words that project work is listed
under its project.

The sweep went from 41 channels / 58 open topics to 47 / 94.

## Reaching the relay

`http://localhost:8094` by default, overridable at build time with
`VITE_AGENTROOM_URL`. No nginx or vite proxy was added: the page is served
from `:5173` or `:8090` depending on how it was started, the relay answers
`Access-Control-Allow-Origin: *`, and a proxy through the nginx container
would have needed `host.docker.internal` for no gain — the relay's whole
surface is a read the browser's own user can already do.

A dead relay is named, not swallowed: `the agentroom relay is not answering on
http://localhost:8094`. The other four views do not touch it.

## Evidence

`npm run build` (tsc + vite) is the frontend's only check and it passes. The
production image was rebuilt and the shipped bundle carries the view:

```
$ docker compose up --build -d web && curl -sI http://localhost:8090/
HTTP/1.1 200 OK
$ curl -s http://localhost:8090/assets/index-DE4unwB6.js | grep -o "agent room / zulip now"
agent room / zulip now
```

The browser's own module was then compiled with `tsc` and run under Node
against the live relay — the same `loadRoomAgents` / `loadRoomWork` /
`agentWork` / `introHeadline` the cards are built from:

```
agents: 6  open topics: 94  errors: 0
  agecho-agautolab1    | agecho-agautolab1    |  1 open | This instance is a minimal agag agent with no work of its ow…
  agecho-agstudio1     | agecho-agstudio1     |  2 open | This instance is a minimal agag agent with no work of its ow…
  agforge-agstudio1    | agforge-agstudio1    | 15 open | This instance makes media assets to order — images, video, m…
  agping-agstudio1     | agping-agstudio1     |  1 open | This instance is a minimal agag agent with no work of its ow…
  arxivsage-agstudio1  | arxivsage-agstudio1  |  6 open | I answer questions about the arXiv papers published in study…
  autolab-agstudio1    | autolab-agstudio1    | 11 open | This instance develops software projects. Give it a mission …
first open-work rows:
  [project] pj-ghtrends / pj-ghtrends / workplan-trend3
  [project] pj-ghtrends / pj-ghtrends / workplan-review1
  [project] pj-ghtrends / pj-ghtrends / workplan-trend2
selfnote leaked into the browser payload: false
```

The relay's failure path was checked too: pointed at a dead Zulip, `/healthz`
still answers `200` and `/agents` answers **`502`** with
`ZulipError: GET streams -> <urlopen error [Errno 61] Connection refused>` —
which is what the view repeats instead of drawing an empty room.

**Not verified at the time this report was written: the rendered pixels.**
What was proven here is the bundle, the data path and the card contents; what
was not is Phaser's layout of them.

*Corrected in step 5.* The claim above that this Mac has no headless browser is
wrong — `~/.cache/puppeteer` holds a working Chrome for Testing, and the check
that produced the claim only looked for a `playwright` binary on `PATH` and the
Python module. The view was screenshotted in step 5, and the layout it showed
was **not** right. See `report5.md`.

## Constraints, at the end

1. **No snapshot file** — nothing is written; the relay reads per request, and
   caches 30 s in memory only.
2. **`[selfnote]` hidden** — stripped in the relay, so no consumer can forget.
   Confirmed absent from the browser payload above; proven by
   `agentroom/tests/test_room.py`, because live `intro-` topics contain none.
3. **`✔ ` is the only open/closed rule** — `agag.zulip.RESOLVED_TOPIC_PREFIX`
   is imported, never re-implemented, and no topic name is interpreted.
4. **No harness, model or backend** — nothing is added to what agents posted;
   grepped absent from the live payload.
5. **No credential committed** — `AGENTROOM_ZULIP_ENV` points at an ignored
   file; the host, the realm URL and the account appear only in the ignored
   `pj-agdev/.local/devenv.md`, which gained a section for this service.

## Handoff note

Deus ex machina: the Omni Agent built this for agdevworld — a frontend whose
work would ordinarily go to autolab in a `workplan-` topic of a `pj-` channel.
agdevworld has no `pj-` channel today, which is itself the reason. Handoff
candidate.
