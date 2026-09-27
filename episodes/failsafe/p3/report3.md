# failsafe p3 — step 3: trustworthy health evidence and waits

## 1. Work evidence is not harness activity (R4)

**Owner: pyagag `agag.execution`, `agag.health`** (`4fc54b2`).

The live execution record now keeps two clocks:

| field | what moves it |
|---|---|
| `last_work_at`, `last_work` | a tool call starting or returning; the model's text or partial output (`generating`); a subagent's own events (`subagent: …`) |
| `housekeeping` (`count`, `last_at`, `last`) | `system` events (what Claude Code emitted all through p2's five-minute Bash call), stream `ping`s, event kinds nobody named |
| `last_event_at`, `last_event` | any event, for the record |

Each open tool call records its own **bound**, where the tool declares one:

- Claude Code's Bash: its `timeout`, 120 s by default and 600 s at most;
- a background Bash call: 5 s;
- a subagent or a fetch: none.

The probe's verdicts now read:

- **`running`** needs *work* within the window. Housekeeping is reported
  with its age ("no work for 400 s (only housekeeping since: system
  task_started, 5 s ago)") and never makes a run `running`.
- **`waiting`** on a tool call holds only inside that call's own bound
  plus 60 s (`TOOL_GRACE`), or for a call that declares none. A Bash call
  still open past its bound is `unknown`: the harness kills a call at its
  bound, so one still open is not doing its work.
- **Every wait reports `cpu_seconds`**, the CPU time of the processes
  under the run. A probe is one look; whether a wait advances is a
  comparison between looks, and that is the monitor's job.
- A record written before this change has no `last_work_at`, and its
  events are used as before. A probe by hand of a p2 serving (#12307)
  answers `ended` as it did.

## 2. The real `waiting` verdict, and a stuck wait

p2 never saw `waiting` live, because trial B's `system` events made it
`running`. Under the new rule, a quiet Bash call inside its timeout is
`waiting`: "alive; waiting 380 s on Bash (sleep 300), within its bound of
400 s, with a process under it (0.02 s CPU)". That is pinned by
`test_a_bash_call_inside_its_bound_is_an_explained_wait_and_past_it_is_not`,
and observed live in step 6.

**The bounded reassessment policy** (Observer, `Monitor._idle_wait`):

| wait | how it is reassessed | when it stops being explained |
|---|---|---|
| a tool call with a declared bound (Bash) | the probe, every look | past its bound + 60 s: the probe says `unknown` |
| no declared bound (a subagent, a fetch, background processes) | the monitor compares `cpu_seconds` look to look. It advances when the tree adds ≥ 0.5 s CPU; a new wait starts its own clock | no advance for **`wait_idle` = 900 s** |
| either, once unexplained | `uncertain` from that **first suspicion**: Front asked after 180 s, the developer told after 600 s, as p2 | a repeated identical look does not move the first suspicion; an advance or new work clears it |

- **Why these intervals.** The only long waits in trial evidence were
  Bash calls, and they are covered by their own bound: p1's T2 had
  6-minute silences, and p2's B had a 5-minute quiet command. 900 s is
  well past either, and above Claude Code's 600 s Bash ceiling, so a
  healthy foreground call can never reach it.
- **The cost of the limit.** A deliberate unbounded wait that uses no CPU
  for over 15 minutes is asked about. What is asked for is
  investigation, never a second run. It is overridable per trial
  (`timing.json` key `wait_idle`) and shown in the health record's
  `timing`.
- **A healthy long task continues without duplicate execution.** An
  advancing wait, or a subagent's own events (work), keeps it healthy for
  as long as it lasts. An unhealthy one reaches Front within
  900 + 180 s + a look, and the developer within 900 + 600 s + a look.
  So the uncertainty is not reset forever by a wait that is merely named.

Tests:

- agobserver `test_p3_failsafe.py`:
  - a named wait that stops advancing reaches Front within
    `WAIT_IDLE + ASK_AFTER + 2 looks`, with "nothing under it has
    advanced … declares no bound of its own";
  - a wait that keeps advancing is left alone for 40 min;
  - a bounded wait is left to the probe.
- `test_health_path`'s 30-minute healthy wait now advances, as a real
  test run's process tree does.

## 3. Several probes timing out (R5)

**Reproduced first** (`test_slow_probes_…[as-p2-ran-them]`):

- seven requests, each with an open probed task;
- six probes each take their whole (scaled) timeout of 0.6 s, and the
  seventh answers `stopped` at once;
- with p2's scheduling (one after another, no budget) the look took
  **3.64 s, the sum of the slow probes**.

At the operational timeout of 10 s, k failing probes add 10k s to every
request's next look. The `stopped` target is 240 s (look, 120 s, look),
so seven such probes (70 s) would push a stopped task past it.

**The fix** (agobserver `a47f6ac`, `Monitor.prefetch`):

- The look now traces every request first, then runs all due probes side
  by side (`PROBE_WORKERS` = 4), waiting at most **`PROBE_BUDGET` = 20 s**.
- A probe still running at the budget is recorded as not answering
  ("did not answer within this look's probe budget"), which is ordinary
  uncertainty. Its thread ends at the probe's own timeout.
- The per-request pass then uses the results.
- Scaled trial `[with-the-budget]`: the look took **1.01 s** (budget
  1 s), and Front was asked about the stopped task in that same look.
- A look now lasts at most about 20 s more than its tracing, however many
  probes fail.

## 4. Unsupported harnesses stay explicit

Nothing was manufactured for them:

- `agy`, `codex` and `gemini` streams still say nothing about tool
  results. Their records have `tool_results: false`, and the probe keeps
  saying so in `unknowns`.
- Their events are classified by kind like any other, so a flat
  `tool_use` counts as work.
- Only autolab is probed. The other owners keep the conversation-only
  kinds.

## Tests

| suite | passed |
|---|---|
| pyagag | 968 (`test_health.py` 22: +5) |
| agobserver | 156 (`test_p3_failsafe.py` 7) |
| agautolab | 317 |

## Rollout

- pyagag `4fc54b2` in agautolab and agobserver: the writer and the reader
  of live records. The other consumers stay on `b2cdf75`, because none of
  them keeps a live record.
- Kickstarted at 04:18:08Z: autolab's listener and gateway, and
  Observer. No run was in flight.
- The health record shows `wait_idle: 900` among the operational timings.
