# Step 1 — the backend that reads Zulip live

`agdevworld/agentroom/`, a stdlib-only HTTP relay. agdevworld has been a pure
frontend since `modernize_agdevworld` p1; this puts a service back beside it in
the same repository, where `assistant/` used to live, rather than in a new one.

## Shape

cagent's **window** listener was the model, as the plan asked
(`cagent/src/cagent_api/server.py`): `ThreadingHTTPServer`, no framework, no
auth, `GET` only. cagent needs three doors because two of them can change the
cluster; this one only reads a chat realm, on loopback, in a private lab, so
the window's shape is the whole of what it needs. CORS is answered `*` — the
page is served from `:5173` or `:8090` depending on how it was started, and
there is nothing behind this port to protect from a page that can already
reach it.

Routes:

| route | answer |
|---|---|
| `GET /healthz` | `{"ok": true}` |
| `GET /` | the service name and its route list |
| `GET /agents` | `#agents` `intro-<instance>` topics, latest post + history |
| `GET /work` | every unresolved topic of every project channel, flat |
| anything else | `404` with the route list |

A Zulip failure is a `502` carrying the error text, so the view can say the
room is unreadable instead of rendering an empty room — the two are not the
same thing, and the p9 accident recorded in `README_DEV.md` is what that
distinction is for.

## Zulip reading is borrowed, not rewritten

`pyagag` is the dependency (`agag.zulip.ZulipClient`, git `main`, pinned by
`uv.lock`). The two things the plan warned against re-implementing are
imported rather than copied:

- `agag.zulip.RESOLVED_TOPIC_PREFIX` — the `✔ ` rename. A second definition
  here would be free to drift from the one every agent uses.
- `agag.selfnote.is_selfnote` — what counts as machine-to-machine. This relay
  hides exactly what `agentchat read` hides.

Rate-limit handling (`RateLimited`, the budget headers) comes with the client.

## Cache

`Room` holds a process-lifetime in-memory cache, 30 s by default,
`AGENTROOM_CACHE_SECONDS=0` to disable. No file is written anywhere: the
snapshot shape the cluster views use (`public/cluster/*.json`) is what this
episode is explicitly not doing.

## Credentials

`AGENTROOM_ZULIP_ENV` is a path to an ignored `KEY=value` file — the format
`ZulipClient.from_env` reads. No new bot was provisioned; an existing
credential is used, which is what the plan permits (Zulip has no read-only API
key, so a dedicated bot would not have been more restricted anyway). Nothing
about the local host, the realm URL or the account is written into a tracked
file. `.gitignore` gained `.venv/`, `__pycache__/`, `*.pyc`; `.dockerignore`
gained `agentroom`, so the frontend image does not carry the relay.

## Evidence

```
$ AGENTROOM_ZULIP_ENV=<ignored path> service/serve.sh check
agents: 6
unresolved topics: 58 in 41 channels

$ curl -s http://127.0.0.1:8094/healthz
{"ok": true}
$ curl -s http://127.0.0.1:8094/nope
{"error": "no route /nope", "routes": ["/healthz", "/agents", "/work"]}
```

`service/serve.sh check` does one read and prints counts instead of listening —
the fastest way to tell a credentials problem from a UI one.

Steps 2 and 3 report on what the two data routes actually return.
