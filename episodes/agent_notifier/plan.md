# agent_notifier — Plan

Braindump: `braindump.md`. The desire: an agent that queues a minutes-long
ComfyUI job (a MiniMax clip is ~420–480 s) should **end its run** and be
**called back in Zulip** when the job finishes, instead of sitting in a
`sleep` loop inside a paid run.

Decision from the discussion: **option 2, deterministic**, shaped as *a tool
the agent calls once* plus *a small always-on notifier that turns a ComfyUI
terminal state into one Zulip post*. No LLM anywhere in the notifier; the
judgment about what a result means stays with the agent that receives the
post.

Experimental, non-public environment; destructive phase, no backward
compatibility. **MUST NOT** lines are the only prohibitions; everything else
is advice the implementer may override with a stated reason.

## Why this shape works with zero new listener code

- The skeleton listener is event-driven (Zulip long-poll, `sweep_serve` in
  pyagag `agag/zulip.py`) with two triggers: **the owner of a topic is served
  by anybody else's real post in it**; a participant is served when it is
  **mentioned**. Its own docstring calls this "what replaces waiting inside a
  run". A post by the notifier into the agent's home topic is therefore a
  callback, with no mention needed.
- `devenv/routine/trigger.sh` (pj-agdev) is the precedent: one `agentchat
  send` from a cron-like process buys Front a run.
- autolab's `handle_workrun` re-serves a `workrun-` topic with the same
  workspace and `chatlog.md`; the notifier's post arrives in the chatlog like
  any other message. agforge's `assetrun-` topics behave the same way.
- `agforge/src/agforge/comfy_video.py` already has the polling half
  (`wait_for_outputs`, `output_references`, `free_memory`); p5/p6 of
  `advance_mediagen_study` measured the ComfyUI behaviours listed below.

## Facts the implementer should not have to rediscover

- **ComfyUI has no webhook.** `GET /history/<prompt_id>` answers `{}` until the
  job is done, then an entry with `status.status_str` (`success`/`error`),
  `status.completed`, and `outputs.<node>.<key>[] = {filename, subfolder,
  type}` (the key name varies by node — match anything with a `filename`).
  `GET /queue` gives `queue_running`/`queue_pending` (each item's second
  element is the prompt_id). `GET /system_stats` gives `devices[0].vram_free`.
  `/ws?clientId=…` pushes `executing`/`execution_success`/`execution_error`;
  optional, lighter than polling, but it does not replace `/history`.
- **History lives in ComfyUI's memory.** A ComfyUI restart loses every entry;
  a prompt_id that was running is then neither in `/queue` nor in `/history`.
  That is a terminal state ("lost") and must be posted, not waited on.
- **An identical graph is served from cache in ~0.4 s** and returns the same
  file; still a normal `success` in `/history`. A notifier must cope with the
  job being finished before the ticket is even written.
- Wall times to calibrate a default timeout: SDXL still 10–26 s, one 124-frame
  MiniMax clip 420–480 s, `POST /free` settles in ~5 s. Nothing legitimate
  takes over 20 min today; make the timeout a per-ticket parameter anyway.
- The ComfyUI base URL is `AGFORGE_COMFYUI_URL` in `agforge/.local/.env`
  (tracked files carry no host literal). The host's `.local` mDNS name stalls
  ~5 s on an unanswered AAAA lookup — read the URL from that env, and if you
  see 5 s per request, force IPv4.
- `agentchat send <channel> <topic> <text>` posts as whoever
  `AGENTCHAT_ZULIP_ENV` names and prints the message id; a topic that does not
  exist is created. With `AGENTCHAT_HOME` unset it writes no `[selfnote]`
  (trigger.sh does exactly this). Zulip mention syntax is `@**Full Name**`.
- Agent runs already receive `AGENTCHAT_ZULIP_ENV`, `AGENTCHAT_HOME`
  (`<channel>/<topic>`) and PATH from `agautolab/src/agautolab/role_run.py`;
  agforge's runs get the same from its listener. That is where a new tool is
  handed over.
- pyagag `ZulipClient.create_bot(full_name, short_name)` creates a generic bot
  and returns usable credentials; `agag/provision.py::_write_bot_env` shows the
  env-file shape (`ZULIP_URL`, `ZULIP_EMAIL`, `ZULIP_API_KEY`). Run it with
  `.local/zulip/developer.env`. Check for an existing bot first — regenerating
  a key kills the running credential.
- Serving is serial per listener: a post that arrives while autolab is busy
  waits its turn. Fine; nothing is lost (startup/expiry sweeps re-read topics).
- Precedent for state and logs: keep them under an ignored `.local/` tree,
  launchd templates in `pj-agdev/devenv/launchd/*.plist.in`, reload with
  `launchctl kickstart -k gui/$(id -u)/<label>`.

## Prohibitions

- **MUST NOT post as the Developer account.** p6's preflight reads "a
  Developer post I did not write" as "a second Omni session is driving"; a
  machine posting as the Developer breaks that and every audit that asks
  whether the human spoke. Use the dedicated bot from step 1.
- **MUST NOT let a ticket end without a post.** Success, error, timeout,
  ComfyUI unreachable, history lost — every terminal state is a post. A
  silent watcher is the p9 failure mode (26 minutes, nothing in any log).
- **MUST NOT put the ComfyUI host, IP or model filenames in tracked files.**

Everything else — language, package location, polling vs websocket, ticket
format — is the implementer's call.

## Steps

### Step 1 — the notifier identity

Create a Zulip bot for the notifier (suggested name `comfy-notifier`; it is a
tool, not an agent: no own channel, no `#agents` introduction, no entrance).
Write its env file to `pj-agdev/.local/zulip/comfy-notifier.env` (mode 0600).
Verify with one `agentchat send` into a scratch topic of `#general` that the
post appears under the bot's name and that the message id prints.

Done when: the env file exists, one test post is visible in Zulip from the bot,
and the developer account was not used for the post.

### Step 2 — the notifier daemon

A small always-on process on agstudio (suggested: `pj-agdev/comfynotify/`, a
Python package with its own `uv` venv depending on pyagag for the Zulip
client, or shelling out to `agentchat`; launchd label
`com.agdev.comfy-notifier`; template beside the others in
`devenv/launchd/`).

Contract:

- Input: **tickets** — one JSON file each in an ignored directory (suggested
  `pj-agdev/comfynotify/.local/tickets/`), written by the step-3 tool:
  `prompt_id`, `comfyui_url`, `channel`, `topic`, optional `mention`, optional
  `note` (free text the agent wants echoed back to itself), `timeout_s`,
  `created_at`.
- Loop: poll `/history/<prompt_id>` every few seconds per open ticket (also
  glance at `/queue` to tell "still pending" from "lost"). On a terminal
  state, post **once** into `channel`/`topic`, then move the ticket to
  `done/` (or delete it) so a restart never posts twice. Recover open tickets
  from disk on start.
- Terminal states and what the post must carry:
  - `success`: prompt_id, wall time since ticket creation, every output
    `{filename, subfolder, type}` and its `/view?…` URL, `vram_free` after,
    the ticket's `note`.
  - `error`: same header plus the `status.messages` / exception text from the
    history entry, truncated to something readable.
  - `timeout`: elapsed, whether the job is still in `/queue` (then say so — the
    agent may want to re-ticket rather than treat it as dead).
  - `lost`: not in `/queue`, not in `/history` for N consecutive polls after
    having been seen (or never seen) — say ComfyUI probably restarted.
  - `unreachable`: ComfyUI did not answer for M minutes.
- Post shape: first line human-readable (`comfy <state> <prompt_id[:8]> in
  <wall>s`), then a fenced JSON block with the full record, so the next run
  reads it without re-polling. Prefix with `@**<mention>**` only when the
  ticket carries one; a post into the agent's own home topic needs no
  mention.
- Log to `pj-agdev/comfynotify/.local/out/notifier.log`; one line per ticket
  state change is enough.

Advice: keep the ComfyUI client code in one module and lift it from
`comfy_video.py` / p5's `onecell.py` rather than rewriting the history
parsing; those already survived `SaveVideo` changing its output key. If you
use `/ws`, still confirm with `/history` before posting.

Done when: a ticket written by hand for a finished prompt_id, a running one,
a nonsense one, and one against a stopped ComfyUI each produce exactly one
post, and a `launchctl kickstart -k` in the middle of a wait posts once, not
zero or two times.

### Step 3 — the tool agents call

A CLI (suggested `comfynotify`) with usage text good enough that an agent
reading `--help` uses it correctly the first time (Tool Giving, not an
Unexplained Chainsaw):

```
comfynotify watch <prompt_id> [--to <channel>/<topic>] [--mention <name>]
                              [--note <text>] [--timeout <seconds>]
```

- `--to` defaults to `AGENTCHAT_HOME`; `--timeout` defaults to something
  around 20 min; ComfyUI URL from `AGFORGE_COMFYUI_URL` or `--comfyui`.
- It only writes the ticket and prints where the callback will land. It does
  not poll, does not fork a watcher, does not need the daemon to be up at
  that instant (the daemon picks tickets up from disk).
- Optional and worth it if cheap: `comfynotify submit <workflow.json> [same
  flags]` — POST `/prompt` and ticket the returned prompt_id in one step, so
  the id can never be lost between two commands.
- Hand it over where `agentchat` is handed over: on PATH for autolab's
  `workrun-` runs (`role_run.py` handover) and for agforge's `assetrun-` runs.
  A symlink into the venv `bin` that already supplies `agentchat` is enough.

Done when: from inside a shell that mimics a run's environment
(`AGENTCHAT_HOME` set, `AGENTCHAT_ZULIP_ENV` set), `comfynotify watch` of a
just-submitted 6-node SDXL text2img (≈10–26 s) lands one post in that topic
within ~10 s of the job finishing.

### Step 4 — the receiving side

Mechanism needs nothing new; the guides do. Add to
`agautolab/agent/guides/workrun_supercoder/guide.md` (and agforge's
`assetrun_generator` guide) a short section next to "When the task is to ask
another agent", in the same voice:

> When a generation takes minutes, do not wait for it. Submit, run
> `comfynotify watch <prompt_id>` (`--help` explains it), write down in your
> report what you are waiting for and what to do with the result, and finish.
> You will be called again when the result is posted into this topic.

Then prove it end to end on a `workrun-` topic: a throwaway mission task whose
job is "generate one 640×640 SDXL still through ComfyUI, ticket it, finish;
when called back, download the file and report its size". Read the second
serving's `transcript.jsonl` to confirm the run found the notifier's post in
`chatlog.md` and did not re-poll. Record: latency from job end to post, from
post to the run starting, and the cost of the two runs against one run that
waited.

Watch for: the run's own `ack`/reply making the bot the last real poster
(good — that is what silences the topic until the notifier speaks); the
notifier's bot never being the *owner* of anything (it must not appear in any
`topic_filter`); and whether `serve_run`'s report/Plane bookkeeping is happy
with a task that ends "waiting" and finishes on the second serving. If it is
not, that is a finding — fix or report, do not hide it with a `sleep`.

Done when: the throwaway task closes over two servings with no human post in
between, and the numbers above are in `report4.md`.

### Step 5 — use it in anger, then write it down

- Fire one real clip through the same path (the p6 pipeline or a plain
  `onecell.py` run, 640×640, length 124): submit → ticket → end; callback →
  frames/sheet/analysis. One task, two servings, ~8 min of GPU in between with
  no agent process alive.
- Record in `report5.md`: the post as it appeared, both servings' wall times
  and cost (`.local/agent/<role>/run-NNNN.json`), and anything the agent did
  wrong on the callback (that is the Failure Farming yield of this episode).
- Update `pj-agdev/.local/devenv.md` (launchd label, log path, env file, how
  to reload) and `devdocs/README_DEV.md` with two lines: what `comfynotify`
  is and that it is a tool, not an agent. Re-post no introductions — nothing
  about any agent's entrance changed.
- If the Omni Agent did any step that belonged to an in-system agent, leave
  the one-line Deus Ex Machina note here.

Done when: one real clip has round-tripped, the docs above are updated, and
`report.md` for the episode summarises what was measured.

## Out of scope

- Freeing VRAM, choosing resolutions, retry policy: the agent's job on
  callback, not the notifier's.
- Notifying about anything but ComfyUI. Keep the ticket format generic enough
  that a second backend could reuse it, but do not build for one.
- SwarmUI: leave it running, do not talk to it.
