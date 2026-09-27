# failsafe p4 — step 4: recovery decisions that can be checked, repeated

## The change

**`agentchat recheck <anchor> --after <ack>`** (pyagag `673e70d`,
`agag.trace.recheck`) re-reads the one conversation that holds the stalled
work, now. It says whether its **owner** resumed the work after the serving
reported stopped. The verdicts:

- `FINISHED`: its record says `completed`, `done`, `accepted` or
  `cancelled`;
- `RESUMED`: a later serving worked or ended;
- `RESUMING`: a later serving was acknowledged, no work yet;
- `ASKED`: a post there waits for the owner's ack;
- `STOPPED`: nothing since;
- `UNREADABLE`.

Each verdict comes with the next move ("do not post there", "post there
once", …). By construction, Front's own acknowledgement and activity in any
other conversation are not evidence. `--json` gives `agag.recheck.v1`.

**Observer** (pj-agdev `agobserver`) names the exact re-check in every
request about stalled work, using the anchor and the stopped serving's
`ack_at_detection`:

    - Re-check right before acting: `agentchat recheck 13062 --after 13070` — it says whether this work has resumed since.

The request stays consistent with the confirmed health facts (p3's
`920b651`). Its evidence is a snapshot, and the request now says how to
refresh it.

**Front's three guides** (front, desk, routine_run): the post is evidence
as of its writing, so run the named re-check right before acting and quote
the verdict.

- RESUMED / RESUMING / ASKED → do not post there.
- FINISHED → say so.
- STOPPED → resume once, in that conversation.
- UNREADABLE or undecidable → ask the developer, a bounded escalation.

Observer's own escalation is unchanged: the developer is told 600 s after
the first suspicion if nothing shows the work moving.

Tests: pyagag 981 (6 recheck verdicts); agobserver 169 (the request names
the re-check). The pin `673e70d` is in all seven consumers, rolled out at
07:31:30Z with no run in flight.

## Trials (operational timing: 60 s look, `probe_after` 120 s, `ask_after` 180 s, `escalate_after` 600 s; no `timing.json`)

Each trial:

- a `#front › front-failsafe-p4-t<N>` request from the Omni Agent standing
  in for the Developer;
- Front opens the mission with autolab;
- `faults/silent-exit` kills the task's harness at its first tool call,
  before the counted command;
- Observer detects the stop and asks Front;
- Front decides;
- the stand-in agrees in the task topic, and Front records the mission's
  acceptance on the stand-in's post.

| | T1 confirmed stop (m13059) | T2 distraction (m13123) | T3 evidence moves first, Observer restarted (m13196) |
|---|---|---|---|
| exit (harness killed) | 07:33:05 | 07:47:00 | 07:52:41 |
| what else happened | — | an unrelated question to Front 2 s after the kill (#13137); a second one posted **the same second** as Observer's request (#13152), so Front served both in one serving | the stand-in resumed the task directly in its topic the second Observer's request landed (#13218, 07:55:34); Observer was **restarted** (kickstart) at 07:55:38 |
| **detection**: Observer's request | 07:35:31 — **146 s** | 07:49:33 — **153 s** | 07:55:33 — **172 s** |
| **decision** (Front's re-check → action) | 07:36:10: `recheck 13062 --after 13070` → STOPPED → resume posted in the task topic (#13080, 07:36:03) — **correct** | 07:50:05: answered the question **and** `recheck 13126 --after 13135` → STOPPED → resume #13154 (07:49:53) — **correct** | 07:55:50: `recheck 13199 --after 13208` → **RESUMING** (ack #13219) → **posted nothing** into the task — **correct** |
| exit → Front's decision | 178 s (resume post) | 173 s | 189 s (the resume was the requester's, 173 s) |
| **resumption**: the new serving's first work | 07:36:22 (#13085) | 07:50:05 (#13156) | 07:55:46 (#13220) |
| result shown | #13087 07:36:22 | #13161 07:50:06 | #13222 07:55:46 |
| Observer | rescued 07:36:31, 1 request | rescued 07:50:33, 1 request | rescued 07:56:38, 1 request; **no second request** after the restart |
| **completion**: `[state] completed` | 07:44:32 (agreement 07:44:16) | 07:51:09 (agreement 07:50:50) | 07:57:42 (agreement 07:57:12) |
| mission `done` (Front's `agentchat accept`) | 07:45:14 | 07:51:26 | 07:58:09 (first try named the wrong id, refused with the reason; retried correctly) |
| executions of the counted command | **1** | **1** | **1** |
| developer escalation | none | none | none |

The time between the result and the agreement is the stand-in's reaction,
not the system's.

Measures across the three trials:

| measure | value |
|---|---|
| detection (exit → Observer's request) | 146, 153, 172 s (target 240 s; goal 300 s) |
| correct recovery decision | 3 of 3. Two resumes on STOPPED, one abstention on RESUMING |
| resumption without developer help | 3 of 3. T3's resumption was the requester's own post; the system's part was not to compete with it |
| completion | 3 of 3 `completed`, 3 of 3 missions `done` by record |
| competing executions | 0 (the count is 1 in every trial) |
| duplicate recovery requests / acceptances | 0 / 0 (T3's refused `accept` wrote nothing) |
| false interventions | 0. Observer asked only about the three killed servings; no other incident opened in the window |
| developer escalations | 0 |
| human interventions | stand-in requests, agreements and acceptance posts; T2's two distraction posts and T3's direct resume (both part of the scenario); fault injection; Observer's restart in T3. **No rescue** |

## Findings in the trials (not fixed in this step)

- **Front promised a wake-up that nothing would give** (T1 #13076): its
  first, supervising serving ended "will reassess at the scheduled wakeup
  at 16:40". Claude Code's own `ScheduleWakeup` is offered to the run, and
  in a headless run nothing brings it back. Observer held the work, and
  the serving was closed (`intent=progress end=`), so nothing was lost.
  Still, it is a promise nothing keeps. Carried to step 6.
- Front's replies sometimes carry the requester's mention twice (the
  listener's handoff plus Front's own). Cosmetic.
- T3: Front's first `agentchat accept` named the conversation's post
  instead of the mission and was refused with the reason. The retry was
  correct: the p3 C pattern, handled by the refusal message.

## Cost

07:31–07:58Z: autolab superdirector 3 runs $0.38, supercoder 9 runs $0.57;
Front 16 runs $3.51. **$4.46.** Observer ran no model judgment.
