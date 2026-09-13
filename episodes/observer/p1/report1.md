# observer p1 — step 1: the agent and the request contract

## What exists now

`agobserver`, an agag instance whose whole entrance is one Zulip channel and
whose unit of work is one topic in it. A watch is stated in ordinary words,
accepted by a local-model run, and recorded in the topic it was asked in.

- **Project**: `pj-agdev/agobserver` — a plain directory in `pj-agdev`, like
  `comfynotify`, not a submodule. Nothing about the design needs its own
  repository yet, and a submodule would need a GitHub repository created by
  hand before the first commit could be made; that decision can be reversed
  later without touching a line of the code.
- **Instance**: `agobserver-agstudio1`; bot user 23, its own channel filed in
  the `agents` folder with the Developer watching it. Distinct from the
  `Opsroom Observer` bot (user 22), which is the operation room's read
  credential and not an agent.
- **Introduction**: posted to `#agents` under `intro-agobserver-agstudio1`,
  carrying the roster block and an `ag.exec-options.v1` block.

## The environment it was fitted to

Read through `nctl` and the two projects' ignored notes before anything was
written. `nctl drift` is `converged=43`, `nctl agents observe` lists six
registrations and now has a seventh instance to learn about. `ollama` is a
declared, converged service placement on this host, and it is the backend
`cagent` already runs its unauthenticated door on — so the local model is not
a new dependency, only a new consumer of one.

**Model chosen: `qwen3.8:27b-mxfp8`, through the `agcode` harness.** Measured
from the actual runtime (`python -m agag.agcode`, the same entry point
`run_harness` uses), not from a configuration file:

| probe | result |
|---|---|
| read a file and judge its contents | correct, 3.7 s, 2 turns |
| `run` a shell command, then judge an unanswered request in a transcript | correct (`MET`), 10.1 s, 3 turns |
| same transcript with the request answered | correct (`NOT_MET`), 7.4 s |
| `glm-4.7-flash` on the same negative case | correct, 25.9 s |

Tool execution works and the natural-language judgment the whole episode
depends on is already right on both polarities. `qwen3.8:27b-mxfp8` was taken
over the `mlx-bf16` build of the same model (smaller, and this is a task that
runs every interval) and over `glm-4.7-flash` (3.5× slower here).

`agcode` is not in `agag.agent_config.HARNESS_PROVIDER`, so its usage pool
derives as `unknown`. That is the right answer rather than a gap — a local
model spends no shared account, so there is no window a "until N % used"
condition could be judged against — and `default` is therefore *declared*
with `pool: unknown` so the derivation agrees with the declaration and
`pool_diagnostics()` is empty.

## The contract

**Every topic in the instance channel is a watch.** There is no second door
and no magic topic name; `watch-<something short>` is the recommended name
and the prefix swept in any other subscribed channel, which is where a watch
requested from elsewhere would arrive in a later phase. The channel
description and the introduction say the same sentence, so a requester never
has to guess whether the name was the important part.

**A watch is a message id.** The record is the topic, and what a program must
resolve without guessing is `[selfnote]` lines in it:

```
[selfnote][watch] <slug>        the anchor — its own id IS the watch (w6676)
[selfnote][accepted] {json}     condition, target, destination, requester
[selfnote][state] <word>        active | needs-input | met | undeliverable
```

The same rule the rest of the realm arrived at the hard way: a topic name is
reusable and a resolve renames it, so anything that remembers a name
eventually reads somebody else's conversation. Only notes written by this bot
are read, and reading follows the `✔ ` rename.

Beside them is one **visible** post saying the same thing in words. It is not
duplication: the notes are for the process, the post is for the human
scrolling the channel, and a reader holding only one of them cannot derive
the other.

**The acknowledgement names nobody.** `serve_topic` normally prefixes a reply
with a mention of the last other speaker — the realm's turn-taking rule, and
exactly wrong here, because the requester is waiting for the *notification*
and naming them would buy them a paid run to read "understood".
`handoff=False` is that decision, and it is the first thing step 4 should
check on a live in-system requester.

**The visible reply is written by the code, not by the model.** The run
produces `decision.json` and nothing else; the acceptance post is assembled
from its fields. A local model is good enough to judge a condition and not
good enough to be this agent's contract voice.

**A destination is resolved, never trusted.** A message link (`/near/<id>`)
is the recommended form and the only one that survives a rename, because an
id is the identifier a rename does not touch; `<channel>/<topic>` is accepted
and followed across the `✔ ` rename. Resolution happens again at send time,
and a deleted anchor is *absent* rather than an invitation to guess at
whatever holds the freed name.

**Progress is local and losable.** `.local/watches/<watch>.json` holds what a
poll accumulates — evaluations, the last observation, delivery — atomically
written. Losing it loses memory, never work: every active watch is found
again by reading the channel. Proven by reading all watches back out of Zulip
with the store ignored entirely.

## Live verification

The listener was run in the foreground against the real realm.

1. **A valid request becomes active.** `watch-release-zip`, asked by the Omni
   Agent, condition in prose, target a path, destination a message link.
   Accepted as **`w6676`**, state `active`, store record written.
2. **Its destination is identifiable.** The link resolved to
   `ops-testbed/observer-p1-dest`, and resolving it again afterwards from the
   stored form returned the same conversation.
3. **The acknowledgement named nobody** — the acceptance post carries no
   mention.
4. **A request needing clarification is not a watch.**
   `watch-missing-destination` ("Tell me when /tmp/observer-p1/report.txt
   appears.") got **one** concrete question and `[selfnote][state]
   needs-input`. It has no anchor, so it is not an active watch and nothing
   will evaluate it.
5. **Recovery from Zulip alone** listed exactly one scheduled watch (`w6676`,
   `active`) and one unscheduled topic, with the store not consulted.
6. **The introduction explains how to request and how to cancel** — a
   `watch-` topic with condition, target and a message link; a ✔ on the topic
   cancels.

Intake cost: three runs, `local`/`agcode`/`ollama qwen3.8:27b-mxfp8`,
20–28 s each, ~3.5k input and ~0.9k output tokens. Zero paid tokens.

## What went wrong, and what it taught

- **`store.update(directory, watch, **fields)` collided with `watch=` as a
  stored field** and raised `TypeError` *after* the three notes were already
  in the topic. The topic was therefore a correct, complete record of an
  active watch that no store knew about — which is exactly the split the
  design claims to survive, and it did: the retry re-read the topic, found
  the existing anchor, reused **the same id `w6676`**, and wrote the store.
  Fixed by making `directory` and `key` positional-only, so every stored
  field arrives as a keyword and none of them can take a parameter's word.
- **The local model does not refuse for a missing destination.** Given "Tell
  me when … appears" it answered `accepted: true` with
  `"destination": "Tell me"` — it filled the field from the nearest phrase
  rather than reporting the gap. The deterministic parser caught it and the
  requester got the right question, so the contract held; the guide now says
  in one line that a phrase is not a destination, but the *code* is what
  protects this, and that ordering is deliberate.

## Left for the following steps

The listener's triggers cannot wait. `sweep_serve` reacts to posts and
`on_sweep` fires on startup and queue re-registration; nothing posts when a
file finishes downloading, so a watch served only by them is evaluated once
and never again. The due-watch timer is step 2, and it is a separate periodic
worker rather than a plist change.

*Deus Ex Machina note: the Omni Agent built agobserver end to end — an agent
that an in-system agent could in principle have been asked to build. Handoff
candidate.*
