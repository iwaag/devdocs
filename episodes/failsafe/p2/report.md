# failsafe p2 — report: diagnose early and learn from recoveries

## Outcome

P2's completion conditions are met:

| condition | result |
|---|---|
| the silent-exit target is demonstrated | ✓ a real harness exit reached Front in **129 s** and **123 s** under the operational configuration (target 300 s); both resumed and completed without Omni Agent rescue |
| healthy quiet work survives the extra checks | ✓ a 5-minute quiet foreground command was probed 3 times with no suspicion, request or competing run; a no-fault control asked nothing |
| ambiguous cases have bounded follow-up | ✓ a frozen live process and a failing probe each reached Front and then the developer within the configured bounds, with an Observer restart during diagnosis |
| recovered incidents reach developer review | ✓ 5 incidents went to 2 review topics: first occurrences, unrecovered ones and a third recurrence named the owners; no duplicate across a process exit mid-handoff |
| resumption cannot substitute for acceptance | ✓ the close-out needs the requester's agreement after a shown result; live, each resumed serving asked for confirmation instead of closing |

Details: `report1.md` (timing and facts), `report2.md` (the health
interface), `report3.md` (the recovery loop), `report4.md` (developer
reviews), `report5.md` (trials).

## The health contract

An owner may expose one command, `python -m agag.health` (pyagag
`agag.health.v1`; `docs/health-v1.md`). The monitor calls it with a
timeout of its own.

- **What it reports.** Asked about a serving by the ack that opened it,
  the command reports four facts, each with its observation time and
  source:
  - **process**: `alive`, `exited` or `unknown`;
  - **progress**: the last harness event;
  - **wait**: a tool call not yet returned, child processes, or none;
  - **serving**: the listener journal's stage, and whether the
    conversation is queued again.

  It also lists what could not be established.
- **The verdict** is `running`, `waiting`, `stopped` (confirmed), `ended`
  or `unknown`.
  - A live process alone is not health: alive with no event and no named
    wait is `unknown`.
  - A record from another serving is never applied.
- **Where the facts come from**: `agag.execution`, a live record per
  harness run written by `run_harness`, and the process table. Only
  `claude_code` and `agcode` streams say when a tool call returns.
- **Participation does not require it.** An owner that is not listed in
  Observer's `health.toml` keeps the conversation-only rules.

## Operational timings

| condition | action | value |
|---|---|---|
| every request | looked at | every 60 s (p1: 120) |
| confirmed ended execution with unfinished work | recovery request | `unheld` 60 s (p1: 300); a probe's `stopped` in the same look |
| no confirmed progress (post or harness event) | health check, then every look | 120 s |
| a serving ended asking nobody, nothing moving | health check | 300 s (p1: `quiet` 1800 s + a judgment) |
| continued uncertainty | Front asked to investigate | 180 s after the first suspicion |
| still unresolved | developer told, with facts and unknowns | 600 s after the first suspicion |
| a judgment on kinds still judged | bounded | 240 s (p1: 900) |

- For probed owners, `silent` (2700 s) and `quiet` (1800 s) are no longer
  used.
- The first suspicion moves only on progress or a confirmed live wait. A
  repeated claim or verdict does not move it, and no model sits on the
  timed path.

## Trial evidence (report5)

| trial | measured |
|---|---|
| A: silent exit, operational | exit → Front **129 s**; resumed after 11 s; resume left the task open; agreement closed it; rescued; review opened |
| A2: recurrence + Observer exit during review delivery | exit → Front **123 s**; occurrence 2 posted once across the process exit |
| B: 5-min quiet tool call, operational | 3 checks, no suspicion, normal completion |
| C: frozen live process, accelerated, Observer restart | first suspicion kept across the restart; Front +132 s, developer +312 s (bounds 120/300 + one look) |
| D: probe fails, accelerated | Front +121 s, developer +302 s; the other 20 requests stayed monitored |
| D2: D after its fix | rescued by the doubted serving's own result before escalation; 3rd occurrence named the owners |
| E: control after the last fix | no probe, no suspicion |

## Measures

- **False interventions: 0.** Nobody was asked about healthy work. There
  were two brief false suspicions: a mission waiting for acceptance was
  checked in the seconds after the requester answered. They asked nobody
  and are fixed (P2-2).
- **Duplicate actions: 0.** Each incident made one request, and each
  review occurrence was posted once.
- **Human interventions.** The Omni Agent stood in for the Developer at
  the requester's points: the requests, one task agreement Front asked it
  for, and the acceptances. It also injected faults and restarted
  services. In C, after the developer was told, it released the injected
  freeze. **No Omni Agent rescue.**
- **Probe cost**: 0.07–0.11 s per probe, with no model. Observer ran
  0 judgments during the trials.
- **Model cost of the trials** (01:39–02:55Z): autolab $2.49 (27 runs),
  Front $4.49 (45 runs), **$6.98** in total.
- **Developer handoffs**: `review-autolab-stopped` (2 occurrences, both
  recovered) and `review-autolab-uncertain` (3: two unrecovered, one
  recurring notice). Both are left open for the Developer's ✔.

## Where it lives

- **pyagag**:
  - `bea4583`: `agag.execution` and `agag.health`; `run_harness(live=)`;
    `run_role(live=)`; trace `ending_intent`/`ending_to`/`ack_at`; `unheld`
    60 s.
  - `f1e5973`: a delivered serving is `ended`.
  - `3e9729e`: `docs/health-v1.md`.
- **agautolab**:
  - `8f155f1`: resumption is not acceptance, plus the supercoder guide.
  - `3cbd378`: live records and `silent-exit`.
  - `d66aaf9`… (pj-agdev): `freeze-after-tool`.
  - `eb2c032`: the introduction.
- **agobserver** (pj-agdev):
  - `60a8d0e`: `health.py`, the monitor's health path, 60 s looks, the
    240 s judgment deadline.
  - `95f6d84`: `review.py`.
  - `d66aaf9`: `timing.json` and `review-exit`.
  - `5004bf3`, `e5b3dd4`: the trial fixes.
  - `b0c33e6`: the introduction.
  - The config shape is in `health.example.toml`.
- **agfront** (`bcba734`): the `front`, `desk` and `routine_run` guides.
  - A run whose process is gone is resumed even though its serving reads
    open.
  - A live process is never run twice.
  - A resume is not an agreement.
- **Dependency pins**: pyagag `f1e5973` in agautolab, agfront and
  agobserver. agforge, archsage, cagent, the relay and comfynotify stay on
  `f5c4359`: nothing they use changed.
- **Documentation**: `README_DEV.md` (the observer, trace and autolab
  sections; p1's timings updated) and both introductions, re-posted. Host
  details are in `pj-agdev/.local/devenv.md`.

## Tests

| suite | passed |
|---|---|
| pyagag | 952 (`test_health.py` 17) |
| agobserver | 150 (`test_health_path.py` 16, `test_review.py` 5) |
| agautolab | 313 |
| agfront | 180 |

## Superseded behaviour removed on the covered path

- `silent` (2700 s + a judgment) and `quiet` (1800 s + a judgment) for a
  probed owner.
- `unheld`'s 300 s grace (now 60 s).
- The 120 s look interval (now 60 s).
- The 900 s judgment deadline (now 240 s).
- For a probed owner: two requests ten minutes apart. It is now one
  request and a fixed escalation time, stated in the request.

## Remaining limitations

- **Unsupported environments.**
  - Only autolab exposes the interface. Front, forge, archsage and cagent
    are not probed and keep `silent`/`quiet`.
  - `agy`, `codex` and `gemini` streams do not report tool results, so for
    them a quiet run is `waiting` only through child processes.
  - The probe is host-local: files and the process table. An owner on
    another host would need another transport. Cross-host execution stays
    out of scope.
- **`waiting` was never the live verdict.** Claude Code emits `system`
  events while a long Bash call runs, so trial B read as `running` every
  time. The `waiting` path is proven by tests only.
- **Not fixed, seen in the trials** (report5):
  - autolab replies lose everything after a code fence nested in their
    `ag-reply` block (four extra round trips);
  - Front once relayed a task agreement into the workplan topic, and
    autolab's planner claimed the task closed without a close-out (the
    acceptance refusal corrected it);
  - Front once produced no reply.
- **The review's hypotheses and improvement candidates are fixed per kind**:
  prompts for the developer, not analysis. A reported incident that later
  moves says so in its incident topic, not in its review.
- **Reviews wait for the developer.** The two review topics are open for
  the Developer. o11711 (m11741) is still held from p1.
