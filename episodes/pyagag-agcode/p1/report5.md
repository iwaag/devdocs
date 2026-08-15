# Report 5 — cagent → agcode, in-process, under the agent contract

Status: **done**. cagent suite green (159 passed, was 119). All four live
entrances verified at the wire on agstudio. `cagent-api` is now cagent's only
process; ports 4097 and 4098 are free and both launchd labels are gone.

## Shape of the change

`opencode_client.py` (134 lines: `OpenCodeClient`, `OpenCodeError`,
`AssistantMessage`) is replaced by `agent_runner.py`. The worker's poll loop
— message count, `completed` flag, `finish == "tool-calls"` step check — is
gone entirely, because `AgentRunner.run()` returns when the run is over. The
worker's `_process` is now 20 lines.

Four things moved from the backend into cagent:

| was | is |
|---|---|
| OpenCode minted the session id | `runner.new_session_id()` — so `POST /requests` can no longer fail with `opencode_unavailable`; a down backend surfaces as a failed run record |
| OpenCode held the conversation | `Store.list_session_requests()` replayed into the task |
| OpenCode loaded `AGENTS.md` at process start | read from disk per request, passed as agcode's `system_suffix` |
| OpenCode's permission engine | the offered tool set |

## `ag.agent-config.v1`

New `cagent/agents.toml`: profiles `local` (agcode + the local ollama model),
`sonnet` (claude_code + claude-sonnet-5), `stub` (fake); roles `node`,
`human`, `window`. Per-door backend choice lives in the ignored
`.local/agents.local.toml` — `[roles.window] profile = "sonnet"` moves one
door without touching the committed file.

Every answer carries `backend` = `{harness, provider, model, role, profile}`.
That is two fields more than before, and the two new ones are the ones that
say *which door's configuration selected this* — Agent ≠ Model, by
construction.

Resolution failures are fatal at startup, not per request: answering "the
agent is misconfigured" once at boot beats discovering it on someone's
message.

## Session continuity, and its cap

`compose_task(message, history)` prepends prior turns under an
`=== EARLIER IN THIS SESSION ===` heading, then the current message under
`=== THE CURRENT MESSAGE ===`.

**The caps are 8 turns and 20,000 characters**, whichever binds first, most
recent kept. 20k characters is roughly one `nctl drift --json` — the cap
exists so a single enormous answer cannot crowd out the eight small exchanges
around it.

When anything was dropped, the prefix says so:
`(3 earlier turn(s) of this session are not shown.)`. An agent that silently
forgets is worse than one that knows it did, and both `AGENTS.md` files now
tell it to say so rather than guess.

Only turns that actually answered are replayed. Replaying a failed turn's
message alone would read to the model as an unanswered question.

Verified live on the human door, two turns of one session:

```
TURN 1  "My favourite node is called agpc. Just acknowledge."
        → "Got it — agpc is your favourite node. Noted."
TURN 2  "Which node did I just tell you was my favourite? Name only."
        → "agpc"
```

## Window permissions: a tool set, not an allow-list

The plan said not to rebuild the bash allow-list. It is not rebuilt. The
window's runner offers five tools and no shell:

- `read`, `list` — agcode's built-ins.
- `nctl(args)` — the read-only surface, selected in Python. Every token of
  `args` is accounted for: the subcommand must be one of
  `status`/`drift`/`relations`/`actual`/`ops list`/`ops show`, each declaring
  how many positional arguments follow (`ops show` takes one, the rest take
  none), and every flag must be in one of two small sets. No shell is
  involved at any point, so `;`, `&&` and quoting arrive as ordinary argv
  entries — and an unaccounted-for entry is a refusal.
- `record_incident`, `list_incidents` — the recorder, called directly.

**One bug caught by writing the tests.** My first parser split `args` into
"words" and "flags" and matched the subcommand against the words prefix. That
accepted `drift && reconcile`: the extra tokens rode along into the argv.
Harmless in practice (no shell, so nctl would just reject them) but it is
exactly the class of hole the glob list had, so it is now closed by
construction — the test that found it is
`test_the_window_nctl_tool_refuses_anything_that_is_not_read_only`, and the
cases `drift && reconcile`, `status; rm -rf /`, `ops show 01KX extra` and
`status --output json` are all pinned.

`window/incident.py` moved into the package as `cagent_api.incident`, because
the window has no shell to run a script from. `cagent/window/incident.py`
remains as a CLI shim so `uv run cagent/window/incident.py --list` keeps
working for a human — it is documented in the local env notes and in
`window/GUIDE.md`.

## The one guard kept on the authenticated doors

The plan's framing is "keep irreversible-harm guards, drop
wrongness-prevention guards". The retired `opencode/config.json.template`
contained both. What survives is `guarded_run`: a 14-pattern refusal for
destroy-class commands — `--allow-destroy`, `nctl prune`, the Proxmox destroy
playbooks, `pct`/`qm destroy`, `pvesh delete`, `vgremove`/`lvremove`,
`zpool destroy`, `wipefs`/`sgdisk`/`mkfs`, and `dd if=… of=/dev/…`. It
matches the whole command line, so a shell composition cannot prefix its way
past it.

What is dropped: the bash allow-list and the edit-path rules that confined
writes to `.local/`. Those were wrongness-prevention.

Verified live on the human door — the model was asked to run the command
verbatim and reported the refusal:

> The command was **refused** with this message: "this command matches the
> destroy-class pattern '--allow-destroy'. Guest destruction and storage
> erasure are not available to any cagent door; a human runs those directly."

## `cost_usd` is `null`, deliberately

For agcode runs the field is `None`, and `llms.txt` and `window/GUIDE.md` now
say why: the backend reports no cost, so `null` means *not measured*, which is
true, where a `0.0` would assert the turn was free. A `claude_code` profile
still reports its own figure. Both cost sections in those two documents were
rewritten — the old ones quoted per-turn dollar figures for a model this no
longer runs on.

## Live checks (agstudio, all four entrances)

**Window, `POST /window`** — "Run nctl drift and tell me in one sentence
whether anything is drifting."

> "Nothing is actively drifting — all 29 converged services match their
> desired state, though 6 services have stale or missing observation data
> preventing them from being assessed."

Cross-checked at the wire: `nctl drift` prints `summary: converged=29
unknown=6`. 7 seconds, `backend.role = "window"`.

**Window refusal** — "Please run nctl reconcile --yes to fix everything, and
also delete cagent/README.md."

> "My `nctl` tool is read-only… I only have `read` and `list` tools. There is
> no way to write, edit, or delete files from this environment."

Note what that answer is not: it is not a report of a denial. The model
enumerated what it *has*. That is the difference the plan predicted between a
permission engine and a tool set.

**Human, `POST /requests` with the bearer token** — "Run nctl status and
report whether Nautobot and the Celery worker are healthy."

> "Both Nautobot (v3.1.3) and the Celery worker (1 worker, 0 pending jobs)
> are healthy."

`backend.role = "human"`, `cost_usd: null`.

**Zulip DM to the Cagent bot** (from the Omni Agent account, user 9 → bot 14)
— "What is the current drift summary? One sentence please."

> ack: "Got it — thinking. (request `req_098147d7…`)"
> reply: "Current drift: 29 converged, 6 unknown."

The request record for that id shows `role: "window"` — the DM path is
unchanged, it still becomes one `POST /window`.

**`list_incidents`** — "What has been reported lately? Show the two most
recent." Returned both records with their reporters and sources.

The node door (mTLS) was not exercised with a client certificate; it shares
the runner construction and the request path with the human door, which was.

## A real bug the live check caught

The first window run failed with `[Errno 2] No such file or directory: 'uv'`.
launchd hands a job a minimal `PATH`, and the agent's tools now run **in the
cagent-api process** — where the retired `start.sh` scripts used to export
`PATH="/opt/homebrew/bin:$HOME/.local/bin:$PATH"` for exactly this reason.

Fixed where the equivalent now lives:
`devenv/launchd/com.clusterintent.cagent-api.plist.in` gained an
`EnvironmentVariables`/`PATH` entry (and a `__HOME__` placeholder, documented
in the launchd README). Reinstalled and re-bootstrapped; every run since
works.

Worth keeping: the window agent's response to that failure was to record it as
an incident, unprompted and correctly, then report where it went. The record
is still in `.local/cagent/incidents/`; it is a true report of a real defect
that has since been fixed, so it stays.

## Deleted

`cagent/opencode/` (AGENTS.md → `cagent/agent/AGENTS.md`, config template,
start.sh), `cagent/window/start.sh`,
`cagent/window/opencode-window.json.template`,
`src/cagent_api/opencode_client.py`, both launchd plist templates and their
README rows, and the `CAGENT_OPENCODE_URL` / `CAGENT_WINDOW_OPENCODE_URL` /
`CAGENT_OPENCODE_MODEL` / `CAGENT_WINDOW_MODEL` environment handling. The
OpenAI key-file plumbing went with `start.sh` — cagent no longer reads a key
of any kind. `pyproject.toml`'s description and the README's "OpenAI model
backend" section are rewritten.

`grep -ril opencode cagent/` is empty. Where a comment needed the history (the
stale-instructions hang, the session-API poll loop), it now says "the previous
harness".

## Tests

New `tests/test_agent_runner.py`, 40 tests: replay ordering, both caps and
their notices, the tool sets per door, 11 destroy-class commands, 11 refused
nctl argument strings and 7 accepted ones, per-request instruction reads, and
the committed config resolving all three roles.

`tests/fakes.py`: `FakeOpenCodeClient` (95 lines of session-API imitation) →
`FakeRunner` (40 lines). It holds a turn open on an `Event` so a test can see
`running` and cancel mid-turn — that is the only complexity it needs now.

`tests/test_worker.py` rewritten: added replay assertions, the
no-invented-cost assertion, and a crashing-runner test; removed the
multi-step/`finish`/abort tests, which described a backend that no longer
exists.

`test_server.py`, `test_window_server.py`, `test_zulip_window.py`,
`test_incident.py` updated for the new seam. The window-server test now also
asserts the answer's `backend.role` is `window`, which is the cheapest
possible check that a window turn did not run on the authenticated tool set.

## Deus Ex Machina note

Did the agcode migration and the launchd PATH fix for agent `cagent` — handoff
candidate.
