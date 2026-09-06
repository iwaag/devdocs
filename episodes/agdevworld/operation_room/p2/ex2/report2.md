# ex2 step B — the relay is a launchd agent

`com.agdev.agentroom` is bootstrapped, running, and holding a live board. The
relay is no longer something a developer has to remember to start.

## What was installed

`pj-agdev/devenv/launchd/com.agdev.agentroom.plist.in`, in the same flavour as
its eight siblings: `__PROJECTS_ROOT__` as the only placeholder, the generated
copy under the ignored `.local/launchd/`, the installed copy in
`~/Library/LaunchAgents/`. It runs `agdevworld/agentroom/service/serve.sh`,
sets `PATH` explicitly (`serve.sh` execs `uv`, which launchd's own `PATH` does
not carry — the cagent-api precedent), and passes
`AGENTROOM_ZULIP_ENV`, `OPSROOM_ZULIP_ENV` and `AGENTROOM_STALLED_SECONDS=900`
as **paths and a number, never credential values**.

One deliberate difference from the siblings: **`ThrottleInterval` is 30 s**,
not launchd's default 10. Every respawn re-sweeps the realm, so a crash loop
here is not a noisy log, it is the agents' Zulip quota being spent as fast as
launchd can spend it.

## The two traps the plan named, checked before the daemon was trusted

**Local Network permission is per binary, and a launchd agent is its own
responsible process** — so a read that works from a terminal proves nothing
about the same code under launchd. The check was therefore run *as a launchd
job*: a one-shot plist calling `service/serve.sh check` with only
`AGENTROOM_ZULIP_ENV` set, so it cost the ~50-call read rather than the
241-call sweep. It printed

```
agents: 7
unresolved topics: 94 in 47 channels
ops: not configured (OPSROOM_ZULIP_ENV unset)
```

and exited 0. No prompt, no denial, `PATH` found `uv`, TLS to the realm
succeeded. The one-shot job was booted out afterwards. **This is the dry run
the plan wanted before p3 touches ComfyUI and ollama** — and it says the
permission is already granted for this interpreter path, not that it will be
for theirs.

**The mDNS AAAA stall is not present for this host.** The realm is
`https://agstudio.local:8543`, and `agstudio.local` is this Mac's own name:
three probes measured `time_namelookup` at 4.9–5.8 ms and `time_total` ≈ 21 ms.
No IPv4 forcing and no IP literal were needed. The trap is real for *other*
`.local` names on this network; it is not real for a host resolving itself, and
guessing either way would have been worth less than the three curls.

## What was verified after bootstrap

| check | result |
|---|---|
| `launchctl list` | `43166 0 com.agdev.agentroom` — a pid, exit status 0 |
| `GET /healthz` | `{"ok": true}` within 5 s of bootstrap |
| `GET /ops` reaches `live` | 25 s: `sweeps 1, sweep_calls 241, topics 120, channels 55, queue true, error null` — the hand-started numbers exactly |
| `/agents`, `/work` unaffected | 7 agents; 94 topics in 47 channels |
| `KeepAlive` | `kill -9` → new pid within 5 s, `live` again at 35 s |
| the log | `agdevworld/agentroom/.local/out/agentroom.log`, one line: `agentroom listening on http://127.0.0.1:8094 (cache 30s, ops on, stalled at 900s)` |
| the 43 relay tests | still pass |

The `kill -9` also produced a number worth keeping: **two full sweeps two
minutes apart did not draw a 429.** p1's measurement was an *immediate* repeat.
That does not make sweeping on a timer a good idea — the quota belongs to the
agents' listeners — but it does mean a restart is not a self-inflicted outage,
and it is why `KeepAlive` is plain `true` rather than something conditional.

## The one thing not driven live, said plainly

The plan lists "event queue expiry → re-registration runs unattended" as a
check. It is in the code and it is the only way back (`_poll_forever` catches
`QueueExpired`, fails the board to `unknown`, returns, and `_run`'s loop
re-registers and re-sweeps), and the `kill -9` above exercises that entire
recovery path — register, sweep, poll — under launchd. What was *not* done is
forcing a real expiry, because that needs the `queue_id`, which the relay
holds in memory and does not expose on any route. Manufacturing an
`/ops/debug/queue` for the sake of one test seemed a worse trade than saying
this sentence.

Likewise, **"still alive after a Mac re-login" was not tested**, because a
re-login is the developer's action, not mine. `RunAtLoad` plus residence in
`~/Library/LaunchAgents/` is the same mechanism the nine sibling jobs use and
they do survive logins; that is inherited evidence, not a fresh measurement.
Worth one glance at `launchctl list | grep agentroom` after the next login.

## Constraints

Untouched. Nothing was written to Zulip by this step (the subscription the
observer makes on startup is p2's, and it re-made it on each restart as
designed). No credential value and no absolute local path entered a tracked
file: the template carries `__PROJECTS_ROOT__`, and the generated plist lives
in the ignored `.local/launchd/`.
