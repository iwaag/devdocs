# failsafe p2 — step 3: early diagnosis in the recovery loop

## What changed on the path

Observer's monitor (pj-agdev `60a8d0e`, `agobserver`) now has a **health
path** for every unfinished unit of work whose owner exposes the health
interface (step 2). The owners are listed in the ignored
`agobserver/.local/health.toml`; `health.example.toml` is the committed
shape. On this host that is `autolab-agstudio1`. For those units a silence
is **checked, not judged**:

| when | what happens |
|---|---|
| an open serving has had no confirmed progress (post or harness event) for `PROBE_AFTER` = 120 s | probe it, and again every look while that lasts |
| a serving ended asking nobody anything, on unfinished work, nothing open or asked anywhere in the request, for `QUIET_CHECK` = 300 s | probe it: p1's T3 misclassification shape, formerly 1800 s plus a judgment |
| probe says `running` / `waiting` | healthy: the suspicion clears, and the unit stays under review (re-probed every look while silent) |
| probe says `stopped` | a **`stopped`** incident in the same look; Front is asked at once |
| anything else (`unknown`, `ended` while the conversation still reads open, a failed or timed-out probe) | uncertainty, dated from the **first suspicion**; after `ASK_AFTER` = 180 s an **`uncertain`** incident asks Front to investigate |
| `ESCALATE_AFTER` = 600 s after the first suspicion without fresh evidence of work | the developer is told (the owners, by name) with the facts, the unknowns and the timeline; nothing more is asked of Front |

- **No model judgment is on this path.** A slow judge, or a `legit` verdict,
  cannot extend it.
- **The first suspicion moves only on real change**: new progress, or a
  probe confirming a live, named wait. A repeated claim, another `unknown`
  or the same evidence does not move it (tested).
- **A new serving (a new ack) is new evidence** and starts from scratch.
  The probe names the serving by its ack, so a record from another serving
  is never applied.
- **For a probed owner, `silent` (2700 s, judged) and `quiet` (1800 s,
  judged) are removed.** `unheld` stays: it is mechanical, now 60 s. Owners
  that are not probed (Front, forge, archsage, cagent) keep `silent` and
  `quiet`. That is the stated limitation, not a gap in the code.
- **Other waits on the covered path:**
  - look interval 120 → 60 s (the cycle takes about 0.6 s off the mirror);
  - judgment deadline 900 → 240 s, for the kinds that are still judged
    (`resolved_live`, and `silent`/`quiet` of unprobed owners).
- **A probe is bounded.** The owner's command runs under its own timeout
  (10 s). Failing, timing out or printing garbage makes the unit
  `unknown` with the reason; it never blocks the look. The
  `faults/probe-fail` trial fault fails every probe while it exists.

### What reaches Front

The request goes where Front's answer reaches the waiting run, as in p1. It
contains:

- the headline (`Work … has stopped` / `I cannot confirm …`);
- the health check's verdict and reason;
- the facts, each with its source: process, last harness event, the named
  wait, the listener journal;
- what is not established;
- the timeline (onset, last confirmed progress, first suspicion);
- and **the time at which the developer will be told**.

It is still an aside (`intent=report answer=none`).

Front's three guides (`front`, `desk`, `routine_run`; agfront `bcba734`)
now say:

- a run whose **process is gone** is resumed even though its conversation
  still reads "serving open", because a run that died cannot say it ended;
- a live process, or a named wait, is never run a second time beside it;
- a resume asks for the work and does not agree to a result nobody has
  seen (step 1).

### State that survives a restart

- `.local/incidents/health.json`, per unit anchor: the ack, first
  suspicion, onset, last confirmed progress, the distinct check results
  (last 10), the verdict. Units unlooked-at for 24 h are dropped.
- Incident records carry `timeline` beside the existing `requests`, so a
  restarted monitor neither asks again nor restarts the clock.
- Deduplication is p1's: one incident per (request, unit anchor). A
  `stopped` that turns `uncertain` is the same incident.
- Recovery is p1's rule for work kinds: a serving begun after detection
  that showed work. The request stays tracked until its record, so
  verification continues through acceptance.
- The health record (`monitor-health.json`) gains `probes` (owners, runs,
  failures, the last probe and its duration). `spent.probes` counts them.

## Detection targets (`monitor.DETECTION_TARGET`, from the last confirmed progress)

| kind | target |
|---|---|
| `stopped` | 240 s (interval + `PROBE_AFTER` + interval) |
| `uncertain` | 420 s (+ `ASK_AFTER`); ≤ 5 min after the first suspicion |
| `unheld` | 120 s (p1: 420 s) |
| `silent` / `quiet` (unprobed owners) | 2910 s / 2010 s |

## Tests

agobserver: 142 passed. The new ones are in `test_health_path.py` (12), on
an injected clock with a scripted probe:

- **silent exit**: Front is asked 180 s after the exit (≤ 300), with the
  facts and the escalation time, and no judgment runs. Front's resume and
  the resumed work are **rescued**, and the incident keeps its timeline.
- **healthy quiet tool wait** for 30 min: probed every look, no incident,
  no request.
- **alive without progress**: Front is asked within 5 min of the first
  suspicion, and the developer is told once, within 10 min. No second
  request follows.
- a repeated `unknown` does not move the first suspicion;
- **a failing (timed-out) real probe command** returns promptly, the unit
  is uncertain within the same bounds, and failures are counted;
- the `probe-fail` fault; an unlisted owner;
- **restart during diagnosis**: the first suspicion survives, the request
  is made once, and a further restart after it asks nothing;
- a `running` harness silent for an hour is never judged;
- the ended-asking-nobody shape is checked early;
- the real `python -m agag.health` command through `HealthProbes`.

One p3 test advanced its clock 120 s per step across a pending judgment. It
now steps 60 s, inside the 240 s deadline, which would otherwise end the
churn it measures.

agfront 180, agautolab 312: passed on pyagag `f1e5973`.

## Not deployed yet

The listeners still run the p1 code. The rollout happens after step 4,
before step 5's trials, after `nctl status` / `nctl drift`.
