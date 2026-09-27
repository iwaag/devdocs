# failsafe p2 — step 5: validation, including real recovery

All live trials ran on agstudio on 2026-09-27 (UTC), with the real
in-system agents:

- Front;
- autolab (Claude Code supercoder);
- Observer's monitor with the health path. Observer ran no model on this
  path: 0 judgments.

The test bed was `pj-robustp1` (`wordcount.py`).

The Omni Agent's role:

- It stood in for the Developer at the requester's points: the requests,
  one task agreement Front asked it for, and the mission acceptances.
- It injected the faults and restarted services.
- It rescued no trial.

Before changing any service it ran `nctl status` (ok) and
`nctl drift --host agstudio` (converged). Rollout: autolab, Front and
Observer were kickstarted at 01:35Z with no run in flight, again at 01:39Z
for the trial aids, and Observer again after each fix.

## Settings

| | operational | accelerated (trials C, D, D2 only) |
|---|---|---|
| look interval | 60 s | 30 s |
| `probe_after` | 120 s | 60 s |
| `ask_after` (uncertainty → Front) | 180 s | 120 s |
| `escalate_after` (→ developer) | 600 s | 300 s |
| `quiet_check` | 300 s | 300 s |

- Accelerated values come from `agobserver/.local/timing.json`, which is
  read at every look (monitor `d66aaf9`). The health record reports the
  values in force.
- It was written at 02:05Z and deleted at 02:42Z. After that the health
  record showed the operational values again.
- Trials A, A2, B and E ran with the operational values. The five-minute
  target was measured only on those.

Trial faults, one-shot, created only by a person:

| fault | where | effect |
|---|---|---|
| `silent-exit` | autolab | SIGKILL at the first tool call, then the serving posts nothing |
| `freeze-after-tool` | autolab | SIGSTOP after the first tool result |
| `probe-fail` | Observer | while present, every probe answers `unknown` |
| `review-exit` | Observer | the process exits after posting an occurrence, before recording it |

## Results

| trial | what was done | evidence | outcome |
|---|---|---|---|
| **A** harness exits without a closing post (operational) | `silent-exit`; mission m12078, two tasks | exit 01:40:08 (pid 74556, SIGKILL; nothing posted after ack #12094 but the progress flush #12095). Probe at 01:42:17: `stopped`, "the run ended (failed, exit -9), its serving delivered no reply (journal: delivered, nothing posted) and nothing is queued". Request #12101 at 01:42:17 | **129 s** from the exit to Front (target 300 s). Front resumed at 01:42:28 in the task topic: "This asks for the work; it agrees to no result yet". The resumed serving showed its result with a confirmation request (#12111) and **did not close**. Front agreed (#12113); the task closed with `accepted … #12113`. Task 2 ran and closed (the fault's re-injection was mistimed, see below). Rescued 01:43:18; `review-autolab-stopped` opened, naming the Developer. Mission `done` on the stand-in's acceptance (#12160) |
| **A2** recurrence, and Observer exits during review delivery (operational) | `silent-exit` + `review-exit`; m12175 | exit 01:51:15; request #12194 at 01:53:18 | **123 s**. Front resumed at 01:53:28; result #12204; agreement #12206; closed 01:54:17. Rescued 01:54:19, then the fault ended Observer right after posting occurrence 2 (#12221, #12222). launchd restarted it, and the restarted monitor recorded the handoff with **no second post** ("Handed to the developer…" #12223, once). Occurrence 2 was appended quietly, per policy. `done` |
| **B** healthy quiet tool call beyond the probe threshold (operational) | m12242: one 5-minute foreground command | the task topic was silent 01:55:20 → 02:00:29 (5 min 9 s); 3 health checks, all `running` | No suspicion, no incident, no request, no competing run; normal completion; `done`. Claude Code emits `system task_started` events while a long Bash call runs, so the probe saw a live stream, not a quiet wait |
| **C** alive, but progress and wait cannot be established (accelerated) | `freeze-after-tool`; m12295; Observer restarted during diagnosis | frozen 02:06:29. First suspicion 02:07:50, "alive, but no event for 81 s and no tool call or child process explains the wait". **Observer restarted 02:08:01**. Front asked 02:10:02 (#12313); developer told 02:13:02 | The first suspicion survived the restart. Front was asked **once**, +132 s after the suspicion (bound 120 + one look). The developer was told at +312 s (bound 300 + one look). Front investigated and started nothing beside the live process (#12316). `review-autolab-uncertain` got occurrence 1, **not recovered**, naming the Developer. The stand-in developer then released the injected freeze (SIGCONT 02:13:18); the run finished, and the incident read "Moving again" 02:14:33. `done` |
| **D** health probe fails (accelerated) | `probe-fail`; m12379, a 7-minute quiet command | suspicion 02:17:02; Front 02:19:03 (#12397); developer 02:22:04 | Within bounds: +121 s and +302 s. Front investigated without a second run (#12400). **Other work stayed monitored**: 20 requests looked at every cycle, 11 probe failures counted. Defect **P2-1** found here (below) |
| **D2** D repeated after the fix (accelerated) | `probe-fail`; m12474, 4-minute command | Front 02:41:25 (#12492); the same serving's result 02:42:26 | **Rescued** on that serving's own work, before escalation. Occurrence 3 of `review-autolab-uncertain` named the owners: "recurring: 3 occurrences", so the recurrence policy was exercised live. `done` |
| **E** control after the last fix (operational) | m12539, no fault; the acceptance withheld until 6 min after the last work (past `quiet_check`) | `health.json` holds nothing for the request | No probe, no suspicion, no incident; `done` |
| model judgment stalls | not live: nothing on the health path calls a model | — | Covered by the p2/p3 tests (`triage-stall`, the judgment deadline now 240 s) and by `test_health_path`: a timed-out probe command. No judgment ran in any trial |
| resume, then acceptance | inside A and A2 | #12105 → #12111 → #12113; #12198 → #12204 → #12206 | Resume alone left each task open. The requester's agreement to the shown result closed it |

## Defects found, fixed and re-tried

- **P2-1: an `uncertain` incident was not recovered by the doubted
  serving's own work.**
  - What happened (D): the rule required a new serving, as a stop does. D's
    serving was alive and posted its result at 02:23:02. The incident
    stayed reported until a later serving.
  - Fix (monitor `5004bf3`): an `uncertain` incident is recovered by work
    after detection from any serving. A `stopped` one still needs a new
    serving.
  - Re-tried in D2: rescued at 02:42:26.
- **P2-2: a mission waiting for its acceptance was suspected at the moment
  the requester answered.** No request resulted.
  - What happened (A 01:50:18, D 02:35:35): the answer ended the explicit
    wait, while Front's serving of the request's own conversation was
    open. The check looked only below that conversation.
  - Fix (monitor `e5b3dd4`): an open serving or a queued post anywhere in
    the request, its own conversation included, holds the move.
  - Re-tried in E: no probe at all.

## Other observations (not changed in p2)

- **Nested code fences cut autolab's replies.** A test output or file
  shown in a triple-backtick block inside its `ag-reply` block ended the
  reply at that fence (C: #12328, #12338; D: #12410, #12419).
  - Front noticed each time and asked for the output inline. The agents
    recovered, at the cost of about four extra serving pairs.
  - The D2 and E requests asked for inline output and had none.
  - This is a reply-contract defect in its own right, left for its own
    episode.
- **In D, Front relayed the task agreement into the workplan topic.**
  - autolab's planning serving answered "Task 1 … is closed" (#12439)
    without any close-out.
  - The system corrected it without help: `agentchat accept` refused
    ("m12379 still has unfinished task(s): 1 (awaiting requester)",
    `[opfail]` #12446), Front then posted the agreement into the task
    topic (#12447), and the task closed.
  - A planning serving claiming a task closure is a false claim worth its
    own look.
- **Front produced no reply once** (D2 #12509, "(this run produced no
  reply…)"). Its next serving answered normally.
- **My own trial driving**: a wait loop in A failed on `wc`'s padding.
  The fault meant for task 2 was placed after task 2 had finished.
  - The recurrence was run as A2 instead, and the unused fault file was
    consumed there.
  - Task 2 of A therefore ran without a fault.

## Totals

| measure | value |
|---|---|
| silent exit → Front (operational) | **129 s, 123 s** (target 300 s) |
| exit → work moving again (the resumed serving's result) | A 2 min 44 s; A2 2 min 42 s |
| false interventions (requests about healthy work) | **0** (B, E); two brief false suspicions that asked nobody, fixed (P2-2) |
| duplicate recovery actions or reviews | **0**: one request per incident; each occurrence posted once, including across the A2 exit and the C restart |
| human interventions | the stand-in's requests, acceptances and one task agreement (D, asked for by Front); fault injection; SIGCONT after C's escalation (releasing the injected fault). **No Omni Agent rescue** |
| developer handoffs | `review-autolab-stopped` (occurrences 1–2), `review-autolab-uncertain` (occurrences 1–3: two unrecovered, one recurring notice) |
| probe cost | 0.07–0.11 s per probe; no model. A 1, A2 1, B 3, C 12, D 1 (+11 injected failures), D2 6 probes |
| model cost | Observer 0 judgments. Agent runs 01:39–02:55Z: autolab supercoder 20 runs $1.81, superdirector 7 runs $0.68, Front 45 runs $4.49; **$6.98** in total |

## Tests after the fixes

agobserver 150 passed (`test_health_path` 16, `test_review` 5), agautolab
313, pyagag 952.
