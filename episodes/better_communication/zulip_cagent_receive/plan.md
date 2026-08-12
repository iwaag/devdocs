# Plan: cagent receives Zulip DMs and records failures

## Goal

A Zulip DM to the `Cagent` bot reaches cagent through a new **unauthenticated
`window` entrance** with its own guide and its own tool permissions. When the
message reports a defect — above all "what cagent said about the cluster turned
out to be wrong" (`devdocs/README_DEV.md`) — the window records it as a local
file and does **not** try to fix it in that turn.

## Non-goals

- No Plane write-back, no ticket lifecycle. A local file is the whole record.
- No streams/mentions. DMs to the Cagent bot only.
- No change to the `:8788` node entrance or the `:8789` human entrance; they
  keep working as they are (`agdevworld`'s `cluster:fetch` depends on `:8789`).

## Decisions already made

1. Window permissions: **read-only cluster access + the incident script**.
   No `desired apply`, no `reconcile`, no free-form writes.
2. The window listener binds like the other cagent listeners — **LAN included**.
3. The Zulip client is **shared through `pyagag`** (`agag.zulip`), lifted out of
   agforge, not copied.
4. Records are **local files only**.

This is an experimental cluster with one user. Do not add auth, approval gates,
or defensive machinery beyond the permission set in Step 3 — the permission
layer is the safety story, prose rules are not.

## Constraints

1. No secrets, tokens, or local host/IP details in git-tracked files.
2. Every window answer leaves a run record naming its backend/model
   (Agent ≠ Model, `devpolicy/agent_records.md`). Reusing cagent's existing
   store/evidence gives this for free.

Everything else is the implementer's call.

## Useful context and hints

### cagent as it stands

- `cagent/src/cagent_api/` is **stdlib-only** and built around
  `server.py` (two listeners, `make_handler(..., authenticate, serve_ui)`),
  `worker.py`, `store.py`, `evidence.py`, `opencode_client.py` (HTTP client to
  an OpenCode `serve` instance). `main.py` wires it; the human listener already
  runs on its own background thread — a third listener follows that pattern.
- `cagent/opencode/start.sh` renders `config.json.template` with
  `__MODEL__` (`CAGENT_OPENCODE_MODEL`, default `openai/gpt-5.6-luna`) and
  `__AGENTS_PATH__`, then runs `opencode serve` on `127.0.0.1:4097`.
- **Permission audit result** (braindump item 2): the existing config already
  allows `.local/**` edits and `bash: {"*": "allow"}` minus destroy-class
  patterns. The main agent needs no new permission to write incidents; the
  point of the window config is to give the unauthenticated door *less*.
- Instructions changes to `opencode/AGENTS.md` need an OpenCode restart;
  `llms.txt` and (by design) the window guide are re-read per request.
- Currently running on agstudio: OpenCode `:4097` and `cagent-api`. `nctl status`
  was healthy at planning time.

### The window pattern to copy

`pj-agdev/agautolab/agent/` is the reference: `GUIDE.md` re-read from disk per
request and served raw at `GET /guide`, `POST /window {"text": str}` as the only
conversational door, one answer at a time, one record per answer, and a
role-specific OpenCode config (`agent/opencode-front.json`) that is a
deny-by-default `bash` allowlist. Copy the shape, not the code.

### Zulip mechanics (already proven in `../zulip_receive`)

- Bot credentials: `pj-clusterintent/.local/zulip/cagent.env` (mode 0600, present).
  Zulip is at `https://agstudio.local:8543`, self-signed TLS; auth is HTTP Basic
  `bot_email:api_key`.
- Receive is `POST /api/v1/register` then long-poll `GET /api/v1/events`.
  `BAD_EVENT_QUEUE_ID` (HTTP 400, must read the body) is normal — re-register.
- **This realm hides real email addresses from events. Key everything on numeric
  user ids.** Self-loop guard on `sender_id` is mandatory or the bot answers
  itself forever.
- Three bugs that cost the last episode a step; the shared code already fixes
  them — do not reintroduce them when moving it: the identity (`whoami`) lookup
  must sit *inside* the retry loop, `http.client.RemoteDisconnected` escapes
  `urlopen` unwrapped, and the listener must survive a Zulip stack restart in
  place (verified: ~20 s of retries).
- History: `GET /api/v1/messages?anchor=newest&num_before=50&narrow=[{"operator":"dm","operand":[<ids>]}]`
  with `apply_markdown=false`. Reply: `POST /api/v1/messages`, `type=direct`.
- agforge's listener is supervised by launchd from
  `pj-agdev/devenv/launchd/com.agdev.agforge-zulip.plist.in`; that template is
  the one to clone into `pj-clusterintent/devenv/`.

### pyagag

`pyagag` (import package `agag`) currently holds `agent_config.py` and
`harness.py`, and is consumed **from GitHub** (`git+https://github.com/iwaag/pyagag.git`,
branch `main`) by agforge and agautolab. So publishing `agag.zulip` needs a
push, and the developer does the pushing — ask, don't push. cagent gains its
first dependency here; that is accepted.

## Steps

Each step ends with a `reportN.md`. Order and tooling inside a step are yours.

### Step 1 — `agag.zulip` + one round trip

Move agforge's `zulip.py` (and whatever of `zulip_listener.py` is generic) into
`pyagag` as `agag.zulip`, keeping it stdlib-only and free of agforge specifics.
Point agforge at it and confirm the running forge listener still answers a DM.
Ask the developer to push pyagag, then add the dependency to cagent.
From a pj-clusterintent shell with `cagent.env`, register a queue, send the
Cagent bot a DM from the developer account, see the event, and reply as the bot.
**Done when**: forge is unregressed and the cagent bot has completed one manual
receive/reply round trip. `report1.md`.

### Step 2 — `incident.py`

`cagent/window/incident.py`, stdlib-only, runnable as
`uv run cagent/window/incident.py -i "description"`. Writes one file per
incident under `.local/cagent/incidents/` (gitignored), prints the path.
Suggested shape — change it if something better emerges, and document whatever
you choose: `<UTC timestamp>-<slug>.md`, frontmatter with id / time / reporter /
source / ref, body = the report verbatim. Add `--reporter`, `--source`, `--ref`
and a `--list` that shows recent incidents so the window can answer "what has
been reported lately".
**Done when**: two incidents recorded by hand, `--list` shows them. `report2.md`.

### Step 3 — the unauthenticated window

- `cagent/window/opencode-window.json` — deny-by-default `bash` with an allowlist
  of read-only `nctl` (`status`, `drift`, `relations`, `actual`, `ops list`,
  `ops show`) plus `incident.py`; `edit`/`write` denied everywhere (the incident
  script is the only write path). Verify the denials empirically rather than
  trusting the pattern syntax — glob shapes here are easy to get wrong.
- `cagent/window/AGENTS.md` and `cagent/window/GUIDE.md`. The guide is the
  capability card, served raw at `GET /guide`, re-read per request. It must
  carry, minimally:
  > When a message reports a defect or a wrong answer, record it with
  > `uv run window/incident.py -i "<description>"`. Do not attempt the repair
  > in this turn. Reply with the fact that it was recorded and where.
  Also state what the window can read, what it cannot do, and roughly what an
  answer costs (see `llms.txt`: cents per turn on `gpt-5.6-luna`).
- `cagent/window/start.sh` — a second `opencode serve` (e.g. `:4098`) with that
  config.
- A third listener in `cagent-api`: **no authentication**, `POST /window`
  `{"text": str}`, `GET /guide`, `GET /healthz`, bound like the others
  (`CAGENT_API_HOST`, new `CAGENT_WINDOW_PORT`). Reuse `store`/`worker`/
  `evidence` with the window OpenCode client; the async
  `202 → GET /requests/{id}` contract is fine — the chat side polls.
**Done when**: `curl` gets an answer through `POST /window`; asking the window to
run a reconcile comes back denied *from the permission layer* (keep that output,
it is the evidence); the run record names harness/model. `report3.md`.

### Step 4 — the listener

`cagent/service/zulip_listener.py` on `agag.zulip` + a launchd template in
`pj-clusterintent/devenv/launchd/`. DM → last ~50 messages as a speaker-labeled
transcript → immediate ack DM → `POST /window` → poll → answer DM. Log to an
ignored path under `.local/`.
**Done when**: a DM reporting a bogus defect ("you told me node X was up, it is
not") produces an ack, an incident file, and a reply naming the record — with no
repair attempted; the listener survives a Zulip stack restart. `report4.md`.

### Step 5 — docs and report

Update `cagent/README.md` (three doors, the window's start order, the new env
vars), `src/cagent_api/static/llms.txt` (the window exists and what it is for),
and the cagent entry in `devdocs/README_DEV.md`. Write `report.md`: what the
window costs per DM, what the permission denials actually looked like, and seeds
for the next episode.
**Done when**: docs match reality and `report.md` exists.

## Notes for the implementer

- The window is deliberately weaker than cagent's authenticated entrances. If a
  DM asks for a cluster change, the right behaviour is to say so and point at
  the authenticated door — not to widen the permission set.
- Keep the listener dumb, like agforge's: no queue persistence, no delivery
  guarantees. A DM missed during a restart is acceptable; the sender resends.
- Every DM costs a real turn. Note in `report.md` what a chat turn actually
  costs so a later episode can decide whether trivial messages deserve a
  cheaper path.
- Three entrances is a transitional state, recorded in `devdocs/todo_done.md`:
  the window is where everything is heading. Do not design around the other two.
- Deus Ex Machina: if the Omni Agent does this work rather than an in-system
  agent, leave the one-line handoff note in `report.md`.
