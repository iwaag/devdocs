# project_migration p6 ex3 — report: snake interactions on the remade model, after the rule removal

Status: **complete** (2026-10-18 02:15 JST). Request: [braidump.md](braidump.md). The purpose, as the
Developer clarified during the run, was an **operation check**: does a playprey run go through end to end
now that workflow_editor p11 has removed the process rules? Content quality was not the goal.

## Result

| Item | Result |
| --- | --- |
| Run | playprey `onboard/run-004`, predecessor `onboard/run-002`, requested by the Omni Agent with the Developer's words quoted |
| Outcome | **Completed, accepted, integrated** into playprey main (`5b857403`) without a manual integration step |
| Delivery | `shared://agshare/runs/playprey/onboard/run-004/o1/` (1547 files, 39.1 MiB, `delivery.py verify` passes) |
| On main now | profile `anaconda` (refit to the remade scan), the tongue-absent capability and `knowledge/cases/anaconda.md` from run-002 (merged through this run), case lab `anaconda/p2`. No file over 5 MB was added (the 50 MiB guard no longer exists) |
| Content | **Construction only, no interactions.** The remade scan closes cleanly by only about 2° (the old one about 6–7°). The agent judged that the region-cut technique, not the scan, is the limit, and recorded it in the case file |

## How the workflow behaved without the removed rules

- **Scope decided by the agent.** It asked one plan question (q1), then reduced `fit` to nothing by its own
  decision and recorded why. No agreement or acceptance blocked any node.
- **Graph changed freely.** After a send-back, the agent added a `cutfix` node and started it alongside the
  open `confirm`; after the correction it cancelled `cutfix` and closed `confirm`.
- **Self-recovery.** Two attempts ended with `register` still running (below). Both times the agops
  recovery agent (`recover/run-011` and the next one) diagnosed the log and queued a resume, with no person.
- **Integration.** It integrated right after `cleanup`, with no renewed acceptance (p6 S3 no longer
  applies).

## What stalled it

| # | What | Cost | Disposition |
| --- | --- | --- | --- |
| E1 | Claude Code auto-mode classifier (Omni harness): refused queuing the request twice and answering q1 once ("Instruction Poisoning", unexplained, "Unrequested Commit in a Connected App") | about 15 min of waiting for the Developer | Passed once the Developer stated the delegation in the session, and localrule.md line 6 was added. Same external limit as p6 S4 |
| E2 | The playprey agent backgrounded a long command (headless Godot material build, then a regression probe) and ended its turn "to wait for the notification". The session ended and the command died | 2 interrupted attempts, 2 recovery runs, about $11 of attempt cost spent partly on redoing work | The fact was already in the wfrun guide (jevlab precedent); the agent followed the harness's own "you will be notified" message instead. Guide amended to say that message is false here, with this trial as a reference (agautolab `4338231`, pj-agdev `55cd154`). The next attempt completed `register` |
| E3 | Omni's own review: sent q2 back to build a new region cut, contrary to the run's purpose | one wasted attempt start (`cutfix`), a stop and a resume | The Developer corrected it. Withdrawn through instruction i1 and a resume with the reason; the agent cancelled `cutfix` and accepted o1 |

## Interventions (deus ex machina)

- Amended the shared wfrun guide (E2): handoff candidate. The guide fact alone did not prevent the
  failure twice, so whether the amended text works is the next attempt's evidence, not yet proven by a
  backgrounding situation.
- Answered q1 and q2, gave instruction i1, stopped and resumed the run: all as "Omni Agent (for
  Developer)", on the Developer's delegation stated in this session.

## Costs (executor attempts)

| | Attempts | Cost | Wall time |
| --- | ---: | ---: | ---: |
| playprey-agent | 6 | **$15.21** plus one stopped attempt with no cost recorded | about 65 min |
| agops recovery agent | 2 | $1.45 | about 7 min |

Omni's session time is not counted.

## Leftovers

- `onboard/run-002` (held since p6 by the Developer's decision) is now superseded: its work reached main
  through run-004. It can be cancelled.
- `compare/run-001` is still held (p6 T3).
- The interactions need the region-cut technique change named in `knowledge/cases/anaconda.md` (p2
  section). Not attempted, by the Developer's decision.
- Evidence defect noted at review: the `sweep_*_collar_close.jpg` stills in `anaconda/p2/shots/` render
  empty.
