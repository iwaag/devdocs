# Step B — how close the host gets to "a run is happening right now"

Investigated item **B** of `plan.md`. The plan named three candidates and
expected a coarse polling approximation. What the probes found is better than
that in one place and worse in two others, and one of the three candidates is
dead on the code.

Probes: `opsprobe.stepb` (live signals) and `opsprobe.stepb_retro`
(detection rate against 1,150 historical run records). No Zulip calls, no
posts.

## 1. Transcript growth — dead

The plan's first candidate assumes `run_harness` streams the transcript to
disk. It does not. `harness.py:805-808` writes it with a single `write_text`
**after** the subprocess returns, and the only other write is the identical
one on the timeout path (`:776-778`). A transcript file appears at the moment
the run *ends*, at full size. There is nothing to watch grow.

Worse, most runs write none at all: `transcript_path` is optional and only
callers that ask for it get one. autolab's `supercoder` — its longest role —
does not, and there is no `transcript.jsonl` anywhere under its 407 role
workspaces.

**Verdict: not a signal.** Its arrival is a *completion* event, and a weaker
one than the run record written beside it.

## 2. The workspace directory — the signal the plan did not list

`serve_topic` builds `.local/topics/<channel>/<topic>/<N>/<role>/` and writes
`chatlog.md` into it *before* the harness starts (`topics.py:95`,
`entrance.py:130`); `write_run_record` files `.local/agent/<role>/run-NNNN.json`
only after it returns. **A role directory with no run record after it is a run
in flight, and unlike every other signal here it names the channel, the topic,
the generation and the role.**

Measured against every historical record (`stepb_retro`), asking whether a
poller would have seen a directory inside each run's `[start, end]` window:

| agent | records | window covered | unambiguous (exactly one dir) |
|---|---|---|---|
| agfront | 545 | 539 (99%) | 515 (94%) |
| agautolab | 480 | 379 (79%) | 379 (79%) |
| agforge | 114 | 88 (77%) | 88 (77%) |
| arxivsage | 8 | 7 (88%) | 7 (88%) |

The window has to open ~5 seconds early, because the directory and its
chatlog are built just before the harness starts. Widening that slack to 120 s
raises coverage by a few points and destroys attribution — agfront's
unambiguous rate falls from 94% to 38%, because at 24 runs an hour a wider
window catches the neighbours. **5 seconds is the setting.**

The misses are not noise, they are a category: runs that do not happen in a
topic workspace at all.

| role | runs | unmatched |
|---|---|---|
| agautolab `coding` | 33 | 33 (100%) |
| agautolab `director` | 12 | 12 (100%) |
| agautolab `front` | 63 | 29 (46%) |
| agforge `generator` | 65 | 26 (40%) |
| agfront `front` | 545 | 6 (1%) |

forge's generator runs in `.local/agentws/<work id>/generator/`, and autolab's
gateway roles in their own directories. Both are enumerable the same way; the
probe simply does not read them yet. **The signal is sound; its coverage is a
list of directories somebody has to write down.** And `<work id>` is a Plane
Sub-Work id, so that directory attributes better than the topic one does.

Two structural gotchas found on the way:

- **The workspace's role name and the record's role name are different
  vocabularies.** An entrance serving builds `…/<N>/front/` and files its
  record under `.local/agent/entrance_front/`. Per-role pairing is therefore
  wrong; the probe compares against the newest record of *any* role in that
  agent, which is what "nothing has finished since" actually means.
- **No run record names the workspace it came from.** `request_id` is the
  record's own file number (`"request_id": "run-0217"`), and `project` was
  null on the newest one. See the "cannot" list in the final report.

## 3. `pgrep` — unusable on this machine, and not for a subtle reason

`ps` finds 8 processes named `claude` or `codex` on agstudio right now.
**None of them is an agent run.** They are the developer's own IDE harnesses —
VS Code and the remote server extension, several `--resume=` sessions among
them. The agents were all idle at the time.

That is not a tuning problem. agstudio is the developer's own Mac, and the
harness binary an agent runs is *the same binary the developer runs all day*
— `agents.local.toml` even points `claude_code` at the VS Code extension's
bundled binary. A name match cannot separate them.

Filtering by working directory (`lsof -a -d cwd -p <pid>`) does separate them:
an agent's harness runs with `cwd` set to its role workspace. But that is the
directory from signal 2, arriving later and with less information, so the
process check earns its place only as a *confirmation* that the directory
found there has a live process behind it — worth having for exactly the case
where a run died without writing its record.

## 4. `agag-status.json` staleness — measured, and the ambiguity is resolvable

Sampled every 5 s for 5 minutes across all five listeners while all were idle:

| agent | min age | max age | mean |
|---|---|---|---|
| agfront | 0.1 s | 38.9 s | 14.6 s |
| agforge | 0.8 s | 51.4 s | 23.1 s |
| agautolab | 0.2 s | 44.2 s | 16.4 s |
| arxivsage | 0.8 s | 43.9 s | 19.4 s |
| agecho | 0.3 s | 31.4 s | 12.2 s |

Idle staleness never exceeded **51.4 s**, which is Zulip's long-poll heartbeat
(`POLL_TIMEOUT_SECONDS = 90` is the client's ceiling, not the server's
cadence). A threshold of ~90-120 s separates "polling" from "not polling"
cleanly.

The plan calls the busy/dead ambiguity a trap for the observer, and it is —
but it is **resolvable**, and cheaply. `launchctl list` gives the listener's
pid and last exit status for nothing:

```
com.agdev.agfront-zulip     pid 83788   last exit 143
com.agdev.agautolab-zulip   pid 83779   last exit 143
com.agdev.agforge-zulip     pid 43597   last exit 143
com.agdev.arxivsage-zulip   pid 71579   last exit 143
com.agdev.comfy-notifier    pid 76983   last exit 143
com.agdev.routine-dispatch  -           last exit 0     (periodic, not resident)
```

So: **pid alive + status fresh = idle; pid alive + status stale = busy in a
handler; no pid = down.** Three states from two cheap local reads.

The real limit is resolution, not ambiguity. Run durations from the records:

| agent | median | p90 | max | under 60 s |
|---|---|---|---|---|
| agfront | 28.4 s | 61.8 s | 3600 s | 484 / 545 (89%) |
| agforge | 20.0 s | 156.2 s | 529 s | 95 / 114 (83%) |
| agautolab | 45.6 s | 222.4 s | 1140 s | 291 / 480 (61%) |
| arxivsage | 15.5 s | 24.3 s | 24.3 s | 8 / 8 |

**Most runs are shorter than the idle jitter of the file that is supposed to
reveal them.** Status staleness can only ever see the long tail — which is,
to be fair, the tail an operation room cares about. It is a *long-run* detector,
not a run detector, and the design should call it that.

## 5. What Zulip adds, from step A's reconstruction

Two things the filesystem cannot give a remote viewer:

- **Ack posted, reply not posted.** `serve_topic` posts `SWEEP_ACK`
  ("Message received. Please wait for the reply.") before the run and the
  answer after it. Between those two posts the topic is demonstrably being
  served. It is late — everything before the ack is invisible — but it is the
  only in-flight signal that needs no access to the node.
- **autolab posts progress while it runs.** `RunProgress`
  (`agautolab/zulip_listener.py:793`) flushes accumulated harness events into
  the `workrun-` topic every `PROGRESS_INTERVAL_SECONDS = 120`. That is a
  live, task-attributed, human-readable in-flight signal — and it exists for
  exactly one agent and one role. It is the shape everything else lacks, and
  worth naming as the precedent when p2 decides whether to ask for more.

## What "running" can honestly mean

Ranked by how much they are worth:

1. **role workspace with no record after it** — names the topic, ~99% of
   topic-driven runs, needs a directory list per agent to reach the rest
2. **launchd pid + `agag-status.json` age** — three-state listener health,
   two local reads, blind to runs under ~60 s
3. **ack-without-reply in Zulip** — remote, late, but attributed and free
4. **autolab's 2-minute progress posts** — the best signal in the system, and
   it exists for one role of one agent
5. **`pgrep`** — confirmation only, never identification, on this machine

Not available at any price: a run's own start time, its pid, its identity, or
the fact that it exists at all, from the agent itself. There is no lockfile,
no pidfile, no reserved record number, and `write_run_record` is a single
write at the end. **Every "running" state in this system is inferred from a
trace somebody left for another purpose**, and the operation room should say
so on the screen rather than draw a green light it cannot back.
