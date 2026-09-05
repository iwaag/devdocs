# operation_room p1 — investigation report

The question was how much of "what is the system doing right now" is
observable, so the operation room can be designed against evidence rather than
hope. Steps are reported in `report1.md`–`report4.md`; this is the summary the
plan asks for. Probes were throwaway and live in the ignored
`pj-agdev/agdevworld/.local/opsprobe/`. Nothing was posted to Zulip.

**The headline.** Task state is *fully* reconstructable from Zulip by an
observer that is not the agent, including the stalled detection this screen
exists for — and it is the only layer that is complete. Execution state is
not observable at all; it is inferred from traces left for other purposes, and
the best of those traces is a directory nobody created for this. Backend
activity is attributable to a task through exactly one key, only while the job
is live.

## 1. Capability matrix

| signal source | what it tells you | freshness / latency | attributable to a task? | work needed to use it |
|---|---|---|---|---|
| **Zulip topics + `[selfnote][served]`** | who owes a reply, since when; acked-not-answered; done (`✔ `) | live via event queue; a full sweep is 183 calls and an immediate repeat is HTTP 429 | **yes** — `(agent, channel, topic)` | incremental/event-driven refresh; a roster of instance names and sweep prefixes, which no API exposes; bare-topic keying across the `✔ ` rename |
| **`launchctl list`** | which listeners and services exist, pid, last exit | instant, free | no (per agent, not per task) | none |
| **`.local/agag-status.json`** | when the poll loop last succeeded → idle / busy / down when combined with the pid | idle age ≤ 51 s measured; blind to runs < ~60 s | no | a ~90–120 s staleness threshold; must be paired with the pid |
| **role workspace dir with no later run record** | a harness run is in flight, and in which topic/generation/role | seconds; 5 s slack before the run's own start | **yes** — channel/topic/role | enumerate non-topic workspace roots (`agentws/`, gateway roles); compare against the newest record of *any* role |
| **`run-NNNN.json`** | a run finished: harness, provider, model, duration, cost, tokens, outcome | on completion only | **no** — carries no topic, channel or work id | nothing (excellent for cost/backend accounting, useless for "now") |
| **`pgrep` for harness CLIs** | that *some* claude/codex process exists | instant | no — 8 matches on agstudio, none an agent | cwd filtering, which only re-derives the workspace signal |
| **autolab `RunProgress` posts** | what a run is doing, live, in its own topic | every 120 s | **yes** | none — but it exists for one role of one agent |
| **forge `pending.json` / `watching.json`** | a ComfyUI job belongs to this Plane Work | live, file-based | **yes** — Plane Sub-Work id | none |
| **`@**Comfy Notifier** watch <prompt_id>` post** | the same join, readable from chat | live | **yes** — channel/topic | none |
| **ComfyUI `/queue`, `/history`** | jobs queued/running/finished | live; history ~36 h / 600 entries, evicting; 599 of 600 are SwarmUI's | via `prompt_id`, **live only** | mDNS `.local` reachability (see below) |
| **ComfyUI `/system_stats`** | device, `vram_free` / `vram_total` | live | no | none |
| **ollama `/api/ps`** | resident models and their VRAM | live, lags by `keep_alive` | **no, in principle** | none |
| **`.local/rtschedule/schedule.json`** | routines requested, fires due, `fired_at` | file, updated by the dispatcher | routine name → `front/front-routine-<name>` | read the file; already served on `:8093` |
| **`#front` `routine-*` topics** | the routine roster and each definition | live | — | none |

## 2. Proposed state definitions

Two layers, deliberately not unified (report 4).

**Conversation / task layer** — entity `(agent, channel, topic)`, from Zulip
alone:

- `awaiting` — the agent owes a reply, by the owner route or the mention
  route. The routes are exclusive.
- `acked` — the newest real post is the agent's own ack: a run has started and
  not answered. This is "running" as chat can see it.
- `stalled` — `awaiting`, older than a threshold (15 min proposed; p9's
  incident was 26).
- `done` — the topic carries `✔ `.

The braindump's *planned only* is **collapsed into `awaiting`**: a post is
what starts a task, so before the post there is only an open conversation, and
nothing observable separates "not started" from "not answered".

**Process / backend layer** — entity a process, a directory, or an endpoint:

- listener: `idle` / `busy` / `down` (pid × status age)
- harness run: `in flight` (role workspace with no record after it), with
  channel/topic/role
- ComfyUI: queue depth, VRAM, and per-job status for live jobs
- ollama: resident models and their VRAM
- routine dispatcher: `due` / `fired at` / `expired`, from `schedule.json`

**Joins between the layers — there are three, and no others:** `prompt_id`
(live only), the workspace path, and the routine name. A row's status colour
must come from layer 1 only.

## 3. What cannot be done

Established, not suspected. Each was tested or read off the code.

1. **A run cannot be seen starting.** No lockfile, no pidfile, no reserved
   record number, no start timestamp. `write_run_record` is a single write
   after the harness returns, and `next_record_path` scans for a free number
   at that moment.
2. **A run record cannot be attributed to a task.** `request_id` is the
   record's own file number; no topic, channel, or work id is stored.
3. **Transcripts cannot be watched growing.** They are written once, at full
   size, after the run — and most runs write none at all.
4. **`pgrep` cannot identify an agent run on agstudio.** The developer's own
   IDE runs the same binaries; 8 matches, 0 of them agents.
5. **`agag-status.json` cannot see a short run.** Idle staleness reaches 51 s
   and 61–89% of runs finish inside 60 s.
6. **ollama cannot report sessions.** `/api/ps` reports *resident models*:
   name, size, `size_vram`, `expires_at`, `context_length`. No count of
   requests, no concurrency, no caller. A model resident on `keep_alive` is
   indistinguishable from one mid-generation. The braindump's "number of
   running ollama sessions" is not obtainable, by any route.
7. **Image generations cannot be attributed.** They go through SwarmUI, which
   returns no `prompt_id`.
8. **Past backend jobs cannot be reconstructed from ComfyUI.** 0 of 12
   historical ticketed prompt_ids still resolve; history holds 600 entries
   over ~36 h and 599 of them are somebody else's interrupted jobs.
9. **An agent's own identity cannot be read from outside.** Its instance name
   lives in that node's ignored `.local/instance.toml` and its sweep prefixes
   are compiled into its `AgentSpec`. Guessing either produced this
   investigation's two worst errors: 7 phantom stalls for Front and 59 for a
   second autolab instance.
10. **The task layer cannot be polled hard.** 183 calls per full sweep, HTTP
    429 on an immediate repeat, from the quota the agents themselves need.
11. **A `pj-`-prefixed topic cannot say *which instance* owns it.** Prefixes
    identify a kind of agent, never an instance.

## 4. Two operational traps for whoever builds p2

- **macOS Local Network permission is per binary.** Every ComfyUI call from
  the probe failed `EHOSTUNREACH` while the host was up and answering the same
  URL from another interpreter. A poller running under an unapproved
  interpreter reports every backend as down. Related: `.local` mDNS lookups on
  this network also stall on unanswered AAAA.
- **Unknown is not idle.** This system's observability is built almost
  entirely on absence of evidence — no record yet, no post yet, no answer from
  a host. A board that renders unknown as idle cannot show the one failure it
  exists to catch: p9's 26 silent minutes looked exactly like idle.

## 5. What this says about the design

Build the conversation layer first and completely — it is the only layer that
is both complete and honest, it needs no access to any node, and `stalled` is
the whole value proposition. Decorate it with process facts where a join
exists. Show machine-level backend tiles that claim no ownership. Keep the
routine GUI on `:8093` as it is, and add only the half it cannot show: whether
a fire was ever answered.

And write the provenance on the screen. Every green light in this system is an
inference from somebody else's leftovers, and the screen that says so is the
one that can be trusted.
