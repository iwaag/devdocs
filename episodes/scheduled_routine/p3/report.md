# scheduled_routine p3 — Phase report

Executed 2026-08-25 UTC. Step evidence is in report1–4. Step 3's two-hour
windows were accelerated with the dispatcher's `--now` test clock after the
Developer asked for a short test; logical and real times are both recorded in
report3.

## Result

The schedule is now one visible, editable flat-event repository:

- `autodev/rtschedule` holds `schedule.json` and the static GUI.
- one deterministic dispatcher runs every five minutes and is the only
  scheduler job;
- Front can inspect and mutate only `schedule.json`, through the narrowly
  granted `rtschedule` CLI;
- `fire` uses the unchanged routine trigger; `decide` enters
  `#front / front-schedule` as the Developer;
- `http://localhost:8093/` shows the past seven days and next 24 hours.

The old imgprompt and rtnotes plists are deleted and unloaded. The standing
rtnotes cadence is eleven concrete fires over 24 hours plus one decide near
the horizon to extend it. This daily decide was chosen over an unbounded
recurrence rule so every extension remains a visible Front judgement.

## Front runs per schedule request

This table counts `front-schedule` servings, not the downstream routine,
autolab, or forge conversations.

| input / schedule request | purpose | Front runs |
|---|---|---:|
| r1 | rtnotes once in 10 min | 1 |
| r2 | every 2 h today | 1 |
| “cancel the last one” | remove r2's ten events and close its guard | 1 |
| r3 | migrate the standing two-hour cadence + renewal decide | 1 |
| r4 | successful conditional rtnotes → imgprompt | 3 |
| r5 | first failing fixture, initially misjudged | 3 |
| r5 correction | remove the wrongly-added fires | 1 |
| r6 | actual broken greeting, strict no-add result | 3 |
| **total** | | **14** |

Each conditional request used one creation run and two runs for its decide:

| decide | runs | result |
|---|---:|---|
| e15 (r4) | 2 | yes; three imgprompt events added |
| e20 (r5) | 2 | yes initially; later corrected and events removed |
| e22 (r6) | 2 | stopped on provenance concern, then no after Developer evidence |

The r4 downstream work added 2 Front runs for rtnotes and 5 for imgprompt.
r5 added 2 rtnotes runs. r6 added 3 rtnotes runs. Those callbacks are the
cost of obtaining project/asset evidence; they are not schedule-edit runs.

## Judgement sources and role division

The division held:

- Front used the bound autolab workrun topics to judge rtnotes. It did not
  open the rtnotes repository or Plane. The successful run cited clean check
  output and autolab's commit; the final failure cited exit 1, exact FAIL
  lines, and two failing tests from autolab.
- Front used forge's assetplan/assetrun reports to coalesce and finish the
  imgprompt overlap. It did not inspect forge storage.
- No cluster condition occurred, so Front did not ask cagent. nctl was used
  only by the Omni Agent for the required local-service baseline/final health
  check, not as a capability given to Front.

r6 exposed one provenance boundary: the temporary Developer definition was
in `routine-rtnotes`, outside `front-schedule`'s chatlog. Front first called
its own workplan post a spoof, then verified Developer message 2159 directly
after being pointed to it. The concern was cautious but wrong; one Developer
reply was required.

## Guidance that grew

The Front guide gained the schedule lines once, before the live requests:

- use `tools/schedule.md`, report ids/exact UTC times, and derive routine names
  from `routine-<name>` topics;
- expand recurrence no farther than 24 hours;
- put conditions in a later decide, after evidence is expected;
- read resolved run topics with `--since`/`wait`;
- ask autolab for project reality and cagent for cluster reality; never open
  repos, Plane, or nctl.

No line was added after a live occurrence. The r5 correction persisted in
the conversation and Front applied it when it authored e22's stricter ask;
the r6 provenance false alarm is evidence for a possible future improvement,
not a reason to grow the guide inside this phase.

## Trap comparison with p2

| trap | p2 | p3 | evidence |
|---|---:|---:|---|
| rename-blind Front read | 1 loop | 0 | all resolved work topics were read successfully via following paths |
| orphaned ack after restart | 1 stall | 0 | no listener restart in p3; every ack got a later answer |
| decide earlier than A completion | n/a | 3 | e15/e20/e22 all fired while rtnotes still ran; none was moved |
| cross-topic provenance false alarm | 0 | 1 | r6 temporary standing definition |
| wrong conditional judgement | 0 | 1 | r5 fires added, then removed before firing |
| in-flight routine overlap | not exercised | 2 extra fires | e17/e18 coalesced into the one F2-26 forge Work |

The p2 defects were not fixed, as required. p3 did not reproduce them under
restart. The installed agfront `agentchat` still lacks the documented `wait`
command, so absence of a p3 incident is not closure.

## Was the flat-events model enough?

It was enough to express every requested **future action**:

- one-shot fire;
- ten-event recurrence expansion and cancellation;
- 24-hour rolling cadence;
- conditional A→decide→B without executable conditions in the dispatcher;
- expiry through the request's `until` guard.

Its limit appeared in decisions and observation. A fired decide has no result
field and cannot be moved. When A was still running, Front kept a long serving
alive, posted into the workrun, and waited for a callback instead of adding a
new later decide — so the expected “decide moved” self-correction did not
happen. The GUI consequently cannot distinguish yes/no/misjudged decisions or
show which overlapping fire started/joined a Work. Front never asked for an
executable recurrence or condition rule; it wanted evidence and provenance
that the current event schema does not retain.

## Easier Next Time candidates — evidence only, not built

1. **Rename-following reads.** p2 has the loop; p3 has zero recurrences but
   still relies on callers remembering `--since`, and the installed CLI lacks
   `wait`. Make plain reads follow `✔ ` consistently and deploy the same CLI
   version everywhere.
2. **Orphaned acknowledgements.** p2 has one restart stall; p3 has no restart
   trial and therefore no counter-evidence. Recovery should regard an ack
   without a later substantive answer as awaiting.
3. **Overlap policy.** p3's two compressed extra fires were safely coalesced
   by Front, but only because all three meant the same standing imgprompt run.
   Record whether a trigger is joined, deferred, or intentionally starts a
   second run before denser real schedules make that judgement ambiguous.

Additional evidence for a later schedule episode: persist decide outcome,
reason, and evidence topic (or model a follow-up decide event) so the static
page can show what happened without becoming a write GUI. Do not add decision
logic to the dispatcher.

## Commits and running state

- pj-agdev: `1899a67` (dispatcher), `796c7a7` (Front editing and plist
  retirement), `133ae63` (GUI service).
- agfront: `4a56bc3` (schedule grant, guide, workspace usage doc).
- rtschedule: `3056360` initial schedule through `58c2c73` GUI/local-link
  configuration; every live edit/fire is individually committed between.
- devdocs: step reports `669334b`, `936171d`, `88d6060`, `59162f5`, followed
  by this phase report.

Loaded jobs at close: `com.agdev.routine-dispatch` and
`com.agdev.routine-gui`; old per-routine jobs absent. Front, autolab, forge,
Nautobot, and nctl health were checked during the phase. The temporary broken
rtnotes code and temporary standing definition were both restored; the final
check and all 9 direct rtnotes tests pass.
