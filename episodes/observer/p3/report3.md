# observer p3 step 3 — conversation monitoring replaced by workflow monitoring (2026-10-14)

pj-agdev `2cdfa73` (agobserver). Not yet the running process: the launchd switch and the removal of the old monitor are steps 5 and 7.

## What runs now

`python -m agobserver.wfmonitor` (`agobserver-workflow`), one look every 60 s, needs no per-run watch, no Front-origin post and no conversation:

| Module | Role |
| --- | --- |
| `wfmonitor.py` | the loop: `wfe observe --json` (plus the runs of open incidents by name, so a run that settled is still verified), classification, suspicions → incidents, judgments, escalation, verification, posting, `workflow-health.json` |
| `wfclassify.py` | pure classification from the observation: respected hold, awaiting person/external, working (open call inside its declared timeout, log moved within 900 s, CPU advanced since the last look), queued behind the slot, interrupted, lost attempt, unowned work, queue stall, tool overrun, silent attempt, overdue external wait, recorded failure, publication/integration problems, unobservable sources, executor down; a recovery run is never sent to recovery |
| `wfstore.py` | `.local/workflow/observer.sqlite`: suspects (first sight survives restarts), incidents (`(uid, family, episode)` unique; id = digest of the three), judgments (with the evidence digest they read), assignments, outbox, events, CPU samples |
| `wfpost.py` | every post is an outbox row: `owed` → `sending` before the call → `sent`; a lost answer or a crash leaves it to be looked for by its marker before anything is posted again; Zulip's own refusal backs off |
| `wftriage.py`, `agent/guides/wftriage/guide.md`, role `wftriage` (local model) | one judgment of an alive, silent attempt from a compact evidence packet: `waiting` / `stalled` / `unclear` |
| `spec.py` | Observer's agent spec, moved out of the listener so the monitor does not import the legacy listener |

Escalation goes to the realm's owners by direct message, with one note per transition in Observer's channel (`incident-<id>`). Recoverable families (`attempt`, `continuation`, `publication`, `integration`) are handed to the recovery path when one is configured (step 4); without it they are told to the owners with the reason.

## Bounds (defaults, `wfclassify.TIMING`; `[timing]` overrides for trials)

| Case | Detection | Next |
| --- | --- | --- |
| `interrupted` / `unknown` hold | the first look after the record | recovery (or owners) at once |
| lost attempt (open in the record, process gone) | 180 s from first sight (the executor's reconcile ends it within 60 s) | same incident when it becomes `unknown` |
| unowned work (ready, or an answer recorded, no job, no attempt) | 180 s from the earlier of first sight and the record's time | recovery (or owners) |
| silent live attempt | suspected when the log's silence crosses 900 s (or an open call passes its declared timeout + 60 s) | judged 300 s later; owners at 900 s unless judged `waiting` on the same evidence; a `waiting` verdict postpones 30 min, doubling to 2 h, at most 6 h in all |
| queued job not claimed while the slot is free | 300 s | owners |
| executor heartbeat older than 180 s | next look | owners, one incident naming every affected run |
| unreadable source | 30 min | owners |

A repeated uncertain look never moves the first suspicion; a judgment is discarded when the evidence it read has moved (`judgment-invalidated`); escalation never fires while a requested judgment is still due. An incident closes only on a later observation: `completed`, `cancelled`, `held` (a person's hold), `manual` (someone else lifted the hold), `resumed` (moving again after the owners were told), `false-alarm` (moving again before anyone was told). A run missing from the observation or a failed observation concludes nothing.

## Evidence

- `tests/test_wfmonitor.py`, 18 tests: the four step-1 fixtures classified as expected (interrupted with the incident `(attempt, amv6sl63n-accd7c)`; the person's hold and a working run; question, long command and queued answer; missing copy, changing record and one executor incident naming the affected run); the motivating case's fact names the background task the harness stopped; an open call past its timeout (`tool-overrun`) versus one inside it; CPU progress between looks counts as work; a recovery run is not sent to recovery; a recorded failure and an overdue external wait; one incident across a restart and a reworded hold, one owner message; the intentional hold and normal waits open nothing and post nothing; unowned work waits its grace with its first sight kept across a restart, then closes `resumed` when the job is queued again; a silent attempt judged after its bound, postponed by `waiting` without escalation, judged again when the review comes due, closed `false-alarm` when the log moves; a verdict on moved evidence discarded and a `stalled` verdict told to the owners once; an executor outage with a queued run and a lost attempt is one incident naming both and closes `resumed` when the heartbeat returns; a failed observation and a run absent from it leave the incident open; a person's hold closes it as `held`; a lost post answer found by its marker, a post that never left sent once.
- Whole agobserver suite: 198 passed (the old conversation tests still run until step 5 removes them).
- A read-only look at the live registry (scratch store, no Zulip, no model): 1 run observed, `respected-hold` (jevlab `experiment/run-001`), nothing opened, 0.32 s.
- One real `wftriage` judgment on this host's local model (`qwen3.8:27b-mxfp8`, agcode) of a fixture's silent attempt: 48 s, `stalled`, naming `openTool`, `waiting`, `log.ended`, `process.alive` and the static CPU.

## Not in this step

The recovery path (`wfrecover`, the `agops/recover` workflow, `run.recover`, the intake for agent requests) is step 4; removing the old monitor and switching the process is step 5; the watchdog's relocation is step 7.
