# p1 plan — agfront skeleton, assistant removal

Goal (reconcile): the Front agent lives in `pj-agdev/agfront`, answers
`front-*` topics in `#front`, and can dispatch work; `agdevworld` becomes a
pure frontend with no embedded agent. Proof is two conversations:

1. A `front-*` topic saying "please advance the work" → Front posts one
   message into a `run-*` topic in `#general` → the agstudio autolab listener
   picks up one Work.
2. A clearly impossible request → Front replies that it cannot.

Backward compatibility is not required. agdevworld may stop working this
phase; later phases restore it against the new design.

## Facts discovered during planning

- `run-` is a **topic prefix, not a channel**. agautolab's listener
  (`agautolab/src/agautolab/zulip_listener.py`, `dispatch()`) reacts to
  `run-*` topics in **any channel it is subscribed to**. The topic body is
  never read — "a `run-` topic is a button, not a conversation". One eligible
  Work is chosen from Plane and executed. So dispatching = one post, exactly
  the braindump's minimum.
- Target channel: `#general` (stream id 3). Its subscribers already include
  both autolab bots (11 agstudio, 12 agautolab1). Only agstudio runs a Zulip
  listener today (checked 2026-08-17: agautolab1 has no listener service), so
  exactly one node reacts. `#FreeForge` has no autolab bot — not a target.
- No result returns to Front. autolab replies inside the `run-*` topic.
  P1 is fire-and-forget by design; write that into Front's guide so it says
  so honestly instead of promising follow-up.
- No bot loop: Front's sweep filter is `front-` only, so autolab's replies in
  `run-*` topics never match it; autolab's filters (`mission-`, `run-`,
  `create-`) never match `front-*`.
- Almost everything is already in pyagag. `agag.topics.serve_topic` owns the
  whole skeleton (ack → numbered generation workspace → `chatlog.md` → run →
  always reply → re-check for posts that arrived during the run), and
  `agag.zulip.sweep_serve(client, handler, topic_filter=…)` owns the lossless
  pull loop. agfront is a thin caller of both — the same shape as
  `agautolab/src/agautolab/zulip_listener.py`, minus mission/asset/project
  vocabulary. Read agforge's `role_run.py` too; it is the smaller of the two.
- Role name `front` does not collide: `agents.toml` `project` scopes roles,
  and agdevworld + agautolab already both define `[roles.front]`.
- Existing Zulip bots each have an env file under `pj-agdev/.local/zulip/`
  (mode 0600). Realm provisioning helpers live in `.local/zulip-selfhost/`
  (`provision-realm.py` is the reference for making a bot and channel).
- launchd templates live in `pj-agdev/devenv/launchd/*.plist.in`
  (`__PROJECTS_ROOT__` substitution, `launchctl bootstrap`, reload with
  `launchctl kickstart -k`). `com.agdev.agautolab-zulip.plist.in` is the
  closest model.

## Steps

Write `report<N>.md` beside this plan after each step.

### Step 1 — Zulip resources

Create the Front bot, its env file `pj-agdev/.local/zulip/front.env`
(0600, same keys as the sibling files), the `#front` channel with only
Developer (8) and Front subscribed, and subscribe Front to `#general`.
Subscription is the routing decision (pyagag README): Front's only outbound
channel this phase is `#general`, so subscribe it to nothing else.
Use the Zulip admin UI or the `.local/zulip-selfhost/` helpers, whichever is
faster. Verify with the API: `#front` member list, Front's own `whoami`.

### Step 2 — agfront scaffold

`pj-agdev/agfront` currently holds only LICENSE + .git. Create:

- `pyproject.toml` with the pyagag GitHub source (copy agforge's dependency
  spelling), package `agfront`.
- `agents.toml` (`schema = "ag.agent-config.v1"`, `project = "agfront"`)
  with the standard three profiles (`local`, `sonnet`, `stub`) and
  `[roles.front] profile = "sonnet"` — the local rule says don't economize.
- `.local/agents.local.toml` — copy the Claude Code binary glob from a
  sibling's local overlay.
- `agent/guides/front/guide.md` — **one file**, guide and channel table
  together; split only when growth forces it. Content: who Front is (the
  human's relay — it routes, it does not do the work), the dispatch table
  (today one row: "to advance development work, post one message to a
  `run-<something unique>` topic in `#general`; nothing comes back here"),
  and "refuse plainly when no row fits".
- `.gitignore` covering `.local/`, standard Python noise.

### Step 3 — listener, proven with stub

`src/agfront/zulip_listener.py` + `role_run.py`:

- `role_run.py`: agforge's shape. Allowed-tools table for `front` is
  read-only plus whatever the dispatch mechanism needs — Front routes, so it
  gets no working shell. This is role definition, not anxiety.
- Listener: `sweep_serve(client, handler, topic_filter=("front-",))`, handler
  delegates to `serve_topic` with an agfront ack/empty-reply. Workspaces
  under `.local/topics/`, run records under `.local/agent/` — the sibling
  layout.
- The handler runs the `front` role over the chatlog + guide, then must get
  the dispatch out. Two known-good mechanisms — implementer's choice:
  (a) command file: the run writes `dispatch.md` (first line = topic name,
  body = message) and the *handler* posts it — agautolab's `new_mission.md`
  pattern; (b) tool: give the run a small posting CLI. (a) is less machinery
  and easier to test; (b) is purer Tool Giving. Either way the reply into the
  `front-*` topic should say what was dispatched, or why nothing was.
- Prove the wiring with `profile = "stub"` in the local overlay and pytest
  against a fake client (agautolab's tests show the fixtures). Then switch to
  sonnet.

### Step 4 — launchd + live reconcile

- `devenv/launchd/com.agdev.agfront-zulip.plist.in` modeled on the autolab
  one; log to `agfront/.local/out/zulip-listener.log`.
- Run the two reconcile conversations from the Developer account in `#front`.
  Evidence: topic links or screenshots into `.local/`, summarized in
  `report4.md`. For conversation 1, confirm the agstudio autolab listener
  acked the `run-*` topic (its reply appears there; if Plane has no eligible
  Work it answers "no work" — that still proves the route).

### Step 5 — strip agdevworld

Remove `agdevworld/assistant/` as the embedded agent. Full deletion is the
default, but partial removal is fine at the implementer's discretion — keep a
piece when deleting it now costs more than it saves, and say so in
`report5.md`. What must be true afterwards: agdevworld itself hosts no agent,
and no seam left behind pretends a dead route is alive.

- compose: `assistant` service; nginx.conf: `/api` proxy; vite proxy config —
  drop what points at removed routes.
- **The chat panel UI stays.** `src/chatPanel.ts` keeps rendering; its
  backend is gone this phase, so its send path may fail or be visibly
  disabled — either is fine. It becomes a thin `#front` wrapper in a later
  phase, so removal work on it now is wasted.
- `agents.toml`, assistant-related `.env` values, README_DEV's assistant
  sections.
- The frontend views fall back to sample JSON where they read `/api/*`
  (workspaces / autolab / tasks). Acceptable this phase; don't spend effort
  softening it.
- `npm run build` and `docker compose up --build -d web` still succeeding is
  the whole check.

Known loss, deliberately deferred: assistant's project-start route
(Gitea → Plane → Zulip, `projects.py`) duplicated `agautolab/init_project.py`;
after this step project creation is agautolab-side only. Note it in
`report5.md` and move on.

## Out of scope (later phases)

- Rewiring the (kept) chat panel as a thin wrapper over `#front` `front-*`
  topics.
- Any result flowing back from dispatched work to Front.
- More dispatch rows (missions, forge, project starts) in Front's table.
