# observer p3 step 4 — the recovery path (2026-10-14)

agdevworld `b9a950b`, agautolab `63e8fa4`, pj-agdev `83fca55`. Source only; deployment is step 7.

## What was built

**WFE — `run.recover`** (`shared/run.ts`, `server/exec.ts` `ExecStore.recover`, `wfe run recover`, `docs/observation.md`). A recovering agent continues an `interrupted` or `unknown` hold for an incident; it does not impersonate a person's `run resume`.

- Refused: a `stopped` hold (`recover-human-hold`), a cancelled or unheld run, a hold that follows another attempt (`recover-stale`), an outdated `--expect-seq`, an attempt of the run still open or its process alive on this host (`recover-process-alive`), a third recovery since the last person-originated control — creation, resume, answer or instruction (`recover-budget`).
- One per incident and attempt (receipt `recover:<incident>:<attempt>`); a reworded incident of the same exhausted episode is refused by the budget too.
- Actor: inside a recovery run's attempt (`WFE_ATTEMPT`), the recorder is that run's agent, verified against the executor's open attempt, and the entry names `agent: {workspace, workflow, run, attempt}`; a run cannot recover itself; outside an attempt `--by` names the recorder.
- Effect: `recoveries[]` in the record (additive), the continuation queued (`recover:<ws>/<run>/<incident>/<attempt>`), the next attempt's brief says it was continued by recovery, by whom, for which incident and why, and that this is the recovering agent's finding, not a fact about side effects. The coordination event says "continued by recovery". The HTTP ops route does not offer it.
- The run view and `docs/runs.md` now say an interrupted/unknown hold may be continued by recovery, and show each recovery; a stop stays only the person's.

**WFE — agent requesters at intake** (`intake.agents: [{id, name, workflows}]`). An agent's account may open a request topic only in the workflows named for it; the post becomes a run whose input is a `request` in the agent's name (`request.md`, `entrustedBy: note`, the post as `source`); its other posts are ignored. The same post never makes a second run (its receipt).

**The recovery recipient** — `agops/recover` (`agobserver/docs/agops/`: `recover.yaml`, one `do` node, manual documentation, agent `recovery-agent`, profile `sonnet`, Zulip requests; `agent/guide.md`, its contract: what it may look at, the two acts it may take, what it never does, why Git cannot prove an external effect did not happen, the report it writes). It runs in the ordinary slot and ends after deciding, so it never holds the slot while the continuation it queued waits.

**Observer — `wfrecover.py`** (`Recoverer`, enabled by `[recovery]` in `workflow.toml`):

- An assignment is journaled with its outbox post in one transaction; the post (`request-<incident>` in `#wf-agops.recover`) goes out through the outbox with read-back by marker.
- Acknowledged = a recovery run whose input is that post (late acknowledgements recorded from the same observation). Performed = `run.recover` naming the incident in the target's history, with its actor. Restored = the target's agent reporting work after it (or completion). `interrupted-again` closes the incident and the new episode opens its own.
- Bounds: a second assignment only after the post was *delivered* and not taken up within 30 min, or after a recovery run took it up and did not act within 1 h; then the owners. A recovery run that ends without continuing is `handed-off` (owners told; its report says what a person must decide). A recovery run is never itself sent to recovery. Nothing is assigned while the executor is down.
- While assignments are open the recovery workspace is observed whole (`--workspace agops --terminal`): a recovery run ends in seconds and is then settled, which the first rehearsal look showed.

## Evidence

- WFE suite 300/300 (`test/recover.test.ts` 5 tests: recovery recorded, queued, briefed and announced, duplicate is a no-op; stale attempt, outdated view, missing actor, live process, a person's stop and a cancellation refused; the budget and its reset by a person's resume; the recovering run's attempt as actor, self-recovery refused, the continuation taking the slot after the recovery run; the HTTP route refusing `run.recover`, the CLI recording it and refusing a person's hold. `test/intakestore.test.ts`: an agent's request becomes a `request` run in its name, its later posts and posts in other workflows ignored). Type check and build pass.
- agobserver suite 205 (`tests/test_wfrecover.py` 7 tests: request → acknowledged → performed → a new PID is not restoration → restored, one post across a restart; a lost post answer found by marker, a Zulip outage of over an hour never doubles the request; unacknowledged requests bounded to 2, then one owner message; a recovery run that ends without continuing → `handed-off`; a recovery run interrupted itself → owners, not recovery; the continuation interrupted again → a new incident, then a person's stop during investigation → `held`; nothing assigned while the executor is down).
- agautolab `tests/test_wfexec.py` 7 passed after the guide change.

### Rehearsal through the real code paths

A private area (scratchpad; evidence kept in the ignored `pj-agdev/.local/observer-p3/step4-rehearsal/`): projects `demo` (`ship`: two `do` nodes) and `agops` (`recover`, the files above) with local remotes; the source `wfe`; autolab's real `wfexec` loop with a scripted agent (`agent_command`); the real Observer monitor and Recoverer; Zulip replaced by a loopback that hands a post in `#wf-agops.recover` to the area's intake (`wfe intake receive`/`process`), as the transport does. The scripted target agent's first attempt does what jevlab `study/run-001` did: starts `work`, backgrounds `sleep 60 && …`, ends its turn.

- `ship/run-002` (clean pass): attempt 1 ended `interrupted` at 12:15:45; Observer opened incident `i436cb2e1f9` at 12:15:48 (fact: "…the session ended (end_turn) with 1 background task(s) stopped by the harness") and journaled assignment 1; the post became `recover/run-004` through intake; the recovery attempt recorded `run.recover` (seq 5, `recovery-agent`, agent `agops/recover/run-004`, attempt `amv7jf9eh-ac40ef`) at 12:15:51 and completed; the continuation attempt `amv7jfddu-b5a1b6` read "continued by recovery" in its brief, completed both nodes; Observer closed the incident `restored` at 12:15:57 on "node.complete (seq 10) in attempt amv7jfddu-b5a1b6 after the recovery (seq 5)". One incident, one assignment, no manual restart. Detection 3 s, assignment 3 s, recovery recorded 6 s after detection, verified 12 s after detection (stub agents; real agents take minutes).
- `ship/run-001` (bounds, by accident): my stub's continuation used a wrong flag (`progress --text`) and exited 3, twice. Observer closed each incident `interrupted-again` and opened the next episode; WFE refused the third recovery (`recover-budget`: "recovered 2 times since the last person's instruction"); the recovery agent left the run held with that refusal in its report; Observer closed the incident `handed-off` and told the owners once. The run stays held for a person.
- Defects found and fixed during the rehearsal: a settled recovery run was invisible to the acknowledgement check (now the recovery workspace is observed whole while assignments are open); posts enqueued during a look waited for the next look (flushed at the current time); acknowledgements of a recovery run that finished between looks are recorded from the same observation.

## Notes

- The `wfrun` guide change is live at once (wfexec reads it at each claim), while `run.recover` and Observer's new process ship in step 7: until then the guide mentions a recovery that cannot yet happen. Nothing breaks; a person's resume still works.
- did the recovery agent's work for a scripted stub in a private rehearsal only — no in-system agent's work was done by the Omni Agent.
