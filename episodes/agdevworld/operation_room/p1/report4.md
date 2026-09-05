# Step D — the state model, in two layers

Investigated item **D** of `plan.md`: redefine the braindump's
`[planned / running / awaiting reply / done]` against what steps A–C proved
observable, and say how the two layers are joined. The plan asks not to force
them into one state machine. After A–C that is not a preference, it is the
only honest option — the two layers disagree about what an *entity* even is.
The conversation layer's entity is a `(channel, topic)`; the process layer's
is a host process; and the only durable bridge between them is a file one
agent happens to write.

## The braindump's four states, judged

| braindump state | verdict |
|---|---|
| 計画のみ (planned only) | **collapse.** Nothing distinguishes it. "A post is what starts a task" (README_DEV, p9) — before the post there is a topic, and the topic *is* the state. A plan topic with no run topic is just an open conversation, which is `awaiting`. Keeping a separate "planned" would mean asserting a thing has not started, from evidence that only says nobody has spoken. |
| 実行中 (running) | **keep, but demote to an inference with a named source.** Step B: no lockfile, no pidfile, no reserved record number, no start time. Every "running" is read off a trace left for another purpose. Best case names the topic and catches ~99% of topic-driven runs; worst case (`pgrep`) is unusable on this machine. |
| 返答投稿待ち (awaiting reply) | **keep, and promote.** It is the cheapest and most reliable state in the system, computable for every agent from Zulip alone, exactly the way the listener computes it for itself (step A). |
| 完了 (done) | **keep.** Zulip's `✔ ` rename, and nothing else. Already the system's own definition everywhere. |
| **stalled** (new) | **add.** `awaiting` plus age. No new read. This is what the screen is for. |

## Layer 1 — conversation / task, derived from Zulip

Entity: `(agent, channel, topic)`. Every state below is computed by an
observer with a read-only Zulip credential and no access to any node.

- **awaiting** — the agent owes a reply here. Owner route (unresolved topic
  in its own channel or under its prefixes, last real speaker is somebody
  else) or mention route (named, and no `[selfnote][served]` note covers that
  post). The two are exclusive; see report 1 for why mixing them reports every
  answered call as pending forever.
- **acked** — the topic's newest real post is the agent's own `SWEEP_ACK`.
  The run has started and has not answered. This is `running` as seen from
  chat: late, but attributed and free.
- **stalled** — `awaiting`, and older than a threshold. p9's incident was 26
  minutes; 15 is a defensible start and the number is a setting, not a
  finding.
- **done** — the topic carries `✔ `.

Everything on this layer keys on the **bare** topic name. 36% of Front's
served notes and 90% of autolab's name a topic that today exists only under
its `✔ ` name; verbatim matching turns those into false stalls (report 1).

**Cost governs the refresh rate.** A full sweep is 183 calls and an immediate
repeat returns HTTP 429 — from the same realm quota the agents' own listeners
spend. This layer must be event-driven or cursor-incremental, not polled. It
is the one place where the operation room can actively harm the system it
watches.

## Layer 2 — process / backend, observed on the host

Entity: a process, a directory, or an endpoint. Cheap, local, no quota.

- **listener** — `launchctl list` pid × `agag-status.json` age gives three
  states: *idle* (pid, fresh), *busy in a handler* (pid, stale > ~90 s),
  *down* (no pid). Idle staleness never exceeded 51 s over two 5-minute
  samples of five listeners. Blind to runs under ~60 s, which is 61–89% of
  them — a long-run detector, and the screen should label it as one.
- **harness run** — a role workspace directory with no run record written
  after it. Names channel, topic, generation and role; ~99% of
  topic-driven runs, 94% unambiguously (report 2). Needs a per-agent list of
  the non-topic workspace roots (`agentws/`, the gateway roles) to reach the
  rest.
- **ComfyUI** — `/queue` depth and `/system_stats` VRAM. Live, honest,
  and mostly not ours: 599 of the last 600 history entries are SwarmUI's
  interrupted jobs.
- **ollama** — `/api/ps` resident models and their `size_vram`. Not sessions,
  not concurrency, not callers.
- **routine dispatcher** — `com.agdev.routine-dispatch` is periodic, so "no
  pid" is its normal state; its liveness is `schedule.json`'s newest
  `fired_at`, not a process.

## The joins, and there are only three

This is the part the design has to be honest about.

1. **prompt_id.** ComfyUI job ↔ `agentws/<work id>/generator/{pending,watching}.json`
   (a Plane Work id) and ↔ the `@**Comfy Notifier** watch <id>` post (a
   channel/topic). Live only: 0 of 12 historical prompt_ids still resolve, and
   ComfyUI's history spans about 36 hours.
2. **the workspace path.** `.local/topics/<channel>/<topic>/<N>/<role>/` is
   literally named after the conversation, which is why the in-flight
   directory attributes and the run record does not. **The record itself
   carries no topic, no work id, no channel** — `request_id` is the record's
   own file number.
3. **the routine name.** `#front` holds 8 `routine-<name>` definition topics
   and 8 `front-routine-<name>` fire topics, one per routine, reused. A fire
   is a post into the fire topic, so *"is routine X running"* is layer 1's
   state of `front/front-routine-<name>` — no new mechanism. `schedule.json`
   supplies what is *due*: `events[{routine, at, fired_at}]` against
   `requests[{until}]`.

Nothing else joins. There is no path from an ollama model, a `pgrep` hit, a
run record, or a ComfyUI image job back to a task.

## The routine GUI: reuse or replace

`com.agdev.routine-gui` is `python3 -m http.server 8093` over
`.local/rtschedule/`, a Gitea clone, serving a static `index.html` that reads
`schedule.json` in the browser. It is alive (pid 8496).

**Recommendation: leave it running and do not fold it in.** It is the
*editor's* view of the schedule file — it belongs to whoever maintains
routines, its data is the file itself, and it needs no credential. The
operation room needs the same file for one strip (what is due, what fired,
what has expired), which is one `fetch` of a JSON document it can read
directly. Replacing a working zero-dependency page with a tile inside a
credentialed service trades a real thing for a small saving. The overlap the
braindump allows for is exactly this case.

What the operation room *adds*, and 8093 cannot, is the second half of a
routine's life: `schedule.json` says a fire happened at 09:01:42; only layer 1
says whether `front/front-routine-papers` was ever answered.

## The shape this implies

Not one state machine. Two independent views, and a small explicit join table
between them:

- **rows are conversations** (layer 1), because that is what a person means by
  "what is the system doing" and it is the layer that is complete, cheap to
  reason about, and expensive to refresh;
- **process and backend facts decorate those rows where a join exists**, and
  otherwise stand alone as machine-level tiles that claim no ownership;
- **a row's colour comes from layer 1 only.** Green because a process exists
  would be a lie the moment the join is missing, and the join is missing for
  every image generation, every ollama call, and every non-topic run.

And one rule that falls out of every step: **the screen must distinguish "no"
from "unknown".** A backend that will not answer, an agent whose prefixes we
were never told, a run older than ComfyUI's 36-hour history — each of those is
*unknown*, and this system's observability is built almost entirely from
absence of evidence. A status board that renders unknown as "idle" is worse
than no board, because the one incident it exists to prevent — p9's 26 silent
minutes — looked exactly like idle.
