# zulip_cagent_receive — Step 3 report: the unauthenticated window

Date: 2026-08-12. Status: **complete**. cagent has a third listener: no
authentication, `POST /window`, its own OpenCode instance with a
deny-by-default permission set, its own guide. A curl gets an answer, a
reconcile comes back denied, and every answer leaves a run record naming its
backend.

## What was built

| Piece | What it is |
|---|---|
| `cagent/window/opencode-window.json.template` | the window's OpenCode config: `bash` deny-by-default with a read-only allowlist, `edit`/`write`/`patch`/`webfetch`/`external_directory` denied |
| `cagent/window/AGENTS.md` | how the window behaves; fixed at OpenCode start |
| `cagent/window/GUIDE.md` | the capability card, served raw at `GET /guide`, re-read per request |
| `cagent/window/start.sh` | a second `opencode serve` on `:4098`, own `XDG_*` under `.local/cagent-window/` |
| `devenv/launchd/com.clusterintent.cagent-window-opencode.plist.in` | supervision for it, matching the two existing cagent jobs |
| third listener in `cagent-api` | `POST /window`, `GET /guide`, `GET /healthz`, `GET /requests/{id}` on `CAGENT_WINDOW_PORT` (default 8790) |

Deviation from the plan's naming: the config is `opencode-window.json.template`,
not `opencode-window.json`, because it is rendered by `start.sh` (`__MODEL__`,
`__AGENTS_PATH__`) exactly like the neighbouring `opencode/config.json.template`
— the absolute instructions path cannot be committed.

### Wiring choices

- **Own OpenCode client and own worker thread, shared store and evidence.** A
  shared worker would have run window turns on the *authenticated* instance's
  permissions, which is the whole thing this step exists to avoid. Sharing the
  store means a window answer leaves the same durable record as any other
  request, and `GET /requests/{id}` polling works unchanged.
- **A third `identity_class`, `window`.** Unauthenticated, so all window
  requests share one owner key. That is enough to make window requests
  readable from the window and `403` from a node, and vice versa — verified in
  `tests/test_window_server.py`.
- **No TLS on this listener.** There is no credential to protect in transit,
  and the guide is meant to be fetchable with a bare `curl`. What makes an
  anonymous door acceptable is the permission set, not transport.
- **Single entrance.** `POST /requests`, `/sessions`, `GET /` and `/llms.txt`
  are *not* mounted on the window port; they answer 404 there.
- **`backend` on every request.** `GET /requests/{id}` now returns
  `{"harness": "opencode", "provider": ..., "model": ...}`, read from the
  assistant message rather than from configuration. This satisfies the
  episode's Agent ≠ Model constraint, and it applies to all three entrances,
  not just the window.

## Evidence

A capability question through `POST /window`, unauthenticated:

```json
{"state": "completed",
 "identity": {"class": "window", "name": "window"},
 "response": "I can read-only cluster status, drift, relations, actual state, past operations,\nrepository files, and record defect reports. I cannot make cluster changes. …",
 "cost_usd": 0.00213761,
 "backend": {"harness": "opencode", "provider": "openai", "model": "gpt-5.6-luna"}}
```

`nctl status` through the window answered "Nautobot connectivity is healthy
and authenticated" in 9 s for $0.0019, so the allowlist admits what it should.

**The denial, from the permission layer, verbatim** — asking the window to run
`uv run --project nctl nctl status && uv run --project nctl nctl reconcile --yes`:

```text
The user has specified a rule which prevents you from using this specific tool
call. Here are some of the relevant rules [{"permission":"*","action":"allow",
"pattern":"*"},{"permission":"bash","pattern":"*","action":"deny"},
{"permission":"bash","pattern":"uv run --project nctl nctl status*","action":"allow"},
… {"permission":"bash","pattern":"*reconcile*","action":"deny"} …]
```

and in the OpenCode log:

```text
evaluated permission=bash pattern="uv run --project nctl nctl reconcile --yes"
  action.permission=bash action.pattern=*reconcile* action.action=deny
```

Asked plainly for a reconcile or for a file write, the window answers without
attempting: "This unauthenticated window cannot run `reconcile` or make
cluster changes. Use the authenticated chat UI or node entrance with a client
certificate." So the *edit/write* denials were not exercised at the layer —
the instructions stop it first. Their syntax is the plain `"deny"` string, not
a glob, and it is the same syntax the neighbouring cagent config already
relies on; the glob-shaped rules are the ones verified empirically below.

## The finding: broad deny patterns are worse than none

The first permission set also denied shell metacharacters and risky words
(`*;*`, `*|*`, `*>*`, `` *`* ``, `*$(*`, `*ssh*`, `*curl*`, `*ansible*`,
`*upload*`, `*desired*`, `*session*`, …) as a guard against a chained command
riding an allowed prefix. A real defect report exercised it:

> "you told me earlier that agpc was reachable; it is not, ssh refuses the
> connection"

The window tried to record it verbatim. `*ssh*` denied it. It re-quoted and
tried again — `*;*` denied it. It tried a third form — denied. On the fourth
attempt it emitted `$'\x3b'` and `$'\x73\x73\x68'` and the command was
**allowed**. Three denials, one bypass, ~25 s, three times the cost, and the
record was written in the end anyway.

Two things follow, and both changed the design:

1. **A deny list of substrings is not a boundary.** An agent that wants
   through re-encodes the string; the layer only sees bytes. What is actually
   load-bearing is `"*": "deny"` plus a short allowlist — a deny can only
   subtract from what an allow already admits.
2. **Broad denies collide with the window's own job.** The one command that
   takes free-form human text is the incident recorder, so every prose
   character in a deny list is a self-inflicted failure on the exact path this
   episode exists to build.

The shipped set therefore drops every character deny and every word deny that
guards a command the allowlist never admits, tightens the `nctl` allows to
exact strings wherever no argument is needed (so there is no trailing `*` for
a chained command to ride), and keeps only the four denies that overlap the
remaining wildcard tails: `*reconcile*`, `*apply*`, `*prune*`, `*lifecycle*`,
`*--allow-destroy*`. `AGENTS.md` now also tells the window not to work around
a denial but to report it.

Re-tested afterwards: the same defect report, with both `;` and `ssh` in it,
recorded verbatim on the **first** attempt for $0.0021 in 9 s.

Known and accepted limits, stated rather than patched over:

- `ops show *` and `incident.py*` still end in a wildcard, so a chained
  command could in principle ride them. The word denies catch the obvious
  destructive ones; a determined agent with escapes could get past those, as
  shown above. This is the same "narrow list, not a capability boundary"
  property already recorded for the authenticated entrance in `llms.txt`.
- An incident whose text contains `reconcile`, `apply`, `prune` or
  `lifecycle` will be denied, and the window is told to report the denial
  rather than mangle the wording.

## Costs measured through the window

| Turn | USD | Seconds |
|---|---|---|
| capability question ("what can you do, what does it cost") | 0.0021 | 7 |
| `nctl status` read | 0.0019 | 9 |
| refusal (reconcile / write request) | 0.0017 | 6 |
| recorded incident | 0.0021–0.0058 | 9–27 |

All on `openai/gpt-5.6-luna`. `GUIDE.md` carries these numbers.

## Notes

- `cagent-api` and its OpenCode instance turned out to already be under
  launchd (`com.clusterintent.cagent-api`, `com.clusterintent.cagent-opencode`);
  killing the old process was enough to have the new code picked up. The
  window's OpenCode is installed the same way.
- Tests: `tests/test_window_server.py` (7) covers the unauthenticated create,
  the body shape, the absent doors, the re-read guide, `healthz`, and both
  ownership directions; `tests/test_worker.py` gained the backend assertion.
  Full cagent suite: 119 passed.
