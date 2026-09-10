# refactor p3 ex1 — step 4: the three reported gaps, on fresh evidence

`nctl drift` went from **converged=43 drifting=3** to **converged=43,
drifting=0, zero errors**. None of the three was fixed the way its row
suggested.

## `service_missing` on agfront — the declaration was the wrong one

p3 reported this as an observation predating the deployment. It was not:

```
desired: config: process_pattern: agfront\.zulip_listener
actual:  eiji  65011  .../agfront/.venv/bin/python3 -m agfront.listener
```

`agfront.zulip_listener` is the module the listener was *entered through*
before `agag_builder` p2 split the entry point from the serving. It is still
a real module — it is what serves a `front-*` topic — but nothing runs it as
`__main__` any more, so the pattern matched nothing and **never would have**,
no matter how fresh the observation. Refreshing it first was still the right
move: it is what turned "we think it is stale" into a running process id and
a pattern side by side.

Repaired the declaration: `process_pattern: agfront\.listener`, through
`nctl desired apply`, and in `.local/desired-state.yaml` beside it.

Two neighbouring declarations were checked rather than assumed:
`agforge` → `service/request_service\.py` and `cagent-api` → its venv
console script both match live processes.

## `agent_zulip_channel_unsubscribed` on agforge — a retired channel

Desired channels were `FreeForge, general, ops`. `FreeForge` is the
pre-`agent_standardize` p1 request channel; it has been gone for weeks.
Forge's own channel is `agforge-agstudio1` — which is what its introduction
in `#agents` advertises, what its listener sweeps whole, and what it is
actually subscribed to:

```
forge subscriptions: agents, agforge-agstudio1, general, ops, sandbox
```

So nothing was subscribed to anything. Subscribing forge to `FreeForge` was
never available — the channel does not exist — and would have been the wrong
answer if it were. The **expectation** was stale, and it is now
`agents, agforge-agstudio1, general, ops`: the four it really needs, taken
from the introduction and the live subscription list rather than invented.

## `agent_zulip_channel_unsubscribed` on agecho-agautolab1 — a fixture, retired

`agecho` is the minimal `agag_builder` p1 fixture: an agent with no work of
its own, built to prove a freshly generated agent can be found and talked to.
It proved that. Its gap named the channel `agecho-agautolab1`, which **does
not exist in the realm at all** — so "repairing" it meant opening a channel
and subscribing a bot, i.e. creating a live agent purely to turn a row green.
The plan says not to, and it is right.

What was found instead is worth keeping: the instance had **already been
retired** — `✔ intro-agecho-agautolab1` has carried the flag since
2026-08-22 — and p3's redeploy of `agautolab1` restarted its listener, which
re-posted its introduction. Zulip's resolve is a rename, so the bare name was
free, and the post opened a **twin**: two topics, one ✔ and one not, and the
un-✔ one is what the operation room reads. A retirement had been quietly
undone by a deployment.

Retired properly, in the four places it lives:

| where | what |
|---|---|
| `#agents` | the twin folded into the ✔ topic (`agentchat resolve` answers "already resolved" and leaves it; the fix is `resolve_topic` on the twin's own last message) |
| Nautobot | `desired_agent`, `desired_service_placement`, `desired_workspace` and `desired_service` deleted |
| agautolab1 | listener stopped, unit disabled and removed, user manager reloaded |
| — | the bot account, the repository and the checkout are **untouched** |

**Removing the declaration was not enough, and that is this step's finding.**
The placement leaving desired state stops `render production` carrying the
agent, so no redeploy touches it — but the systemd user unit it had already
installed stays `enabled`, and the next boot starts the listener, which posts
the introduction, which un-retires the instance. Exactly the loop that put
agecho back on the board in the first place.

There was no route off a node: `setup_agag_agent.yml` had no inverse.
`ansible_agdev/playbooks/agent/retire_agag_agent.yml` is it — stop, disable,
remove the unit, reload — and it deliberately stops there. It does not touch
the bot account, the channel, or the introduction's ✔, because retirement in
this realm *is* that flag plus the absence of a declaration; the playbook only
makes the node agree. The checkout is kept unless `-e remove_checkout=true`:
it holds the instance's ignored credentials and its run records, and a stopped
unit is reversible while a deleted directory is not.

## An observation lesson

Two refreshes, and they are not the same command:

- `nctl reconcile <host> --refresh-observation --max-rounds 1 --yes` refreshes
  the **node's** service observations.
- `nctl agents observe` refreshes **agent registration** — the Zulip account
  and channel read.

Running only the first left agforge's channel gap standing on a stale read
after the declaration had already been repaired, which for a few minutes
looked exactly like a fix that had not worked. Recorded in
`pj-clusterintent/.local/localenv_memo.md`.

## Confirmed for the right reason

```
$ nctl drift
summary: converged=43
[info] autolab-agautolab1: liveness=stale
[info] autolab-agstudio: liveness=polling
[info] cagent: liveness=polling
[info] agforge: liveness=polling
[info] agfront: liveness=polling
[info] arxivsage-agstudio1: liveness=polling
```

Registration and process health stayed separate throughout: every liveness
line is still `info`, `autolab-agautolab1` still says `stale` out loud, and
nothing was converted into an error or hidden to obtain a clean result. The
board is clean because three declarations became true, not because three rows
stopped being drawn.
