# improve_wfrun p4: final report — agent-directed execution coordination by warden

Date: 2026-11-03. Plan: [plan.md](plan.md). Steps: [report1.md](report1.md) … [report6.md](report6.md).

## Outcome

The one-attempt-per-lane limit is gone from the deployed system. A queued job now starts when warden decides, on the node warden
chooses, and a node runs as many attempts as were admitted. This works on both the coordinator's own worker and the remote one.

The incident that motivated the phase no longer blocks. A recovery run executed beside a long attempt on the same node, and
the interrupted run continued in 1 min 39 s, before that attempt ended.

**Real-agent judgment has been shown only on light, disposable fixtures.** Nothing here tested heavy GPU or display work
sharing a node, or a stop chosen by the agent. Completion in the plan's sense is reached for "the agent can make and revise
useful decisions and the machinery carries them out"; how good those decisions are under real load is still open.

## Mechanism and demonstration, separately

| Implemented and tested by fixtures | Also shown by a real agent on the deployed system |
| --- | --- |
| Admission by decision, standing decisions, no lane capacity | Yes: concurrent admission on both nodes, up to 4 ordinary attempts on studio and 2 on gpupc |
| Coordination lane, turns, durable coalesced events | Yes: 28 turns while work ran |
| `wfe coordinate` view and operations | Yes |
| Deferral by event and by time | Yes: one agent-chosen deferral by event; a hand deferral whose wake the agent handled |
| Placement between workers, with provenance and constraints | Yes for placement. The constraint refusal was shown in fixtures; live, only "attempt open" was refused |
| Stale basis, racing decisions | Stale: yes, 6 times. Other races: fixtures |
| Several attempts per executor, adoption after restart | Yes, both worker types |
| Remote worker's locked bookkeeping | Yes, plus a new regression test |
| Pause after three turns without a decision | **Fixture only.** Live, one killed turn was followed by a working one |
| Observer's coordination findings (overdue deferral, stall) | **Fixture only.** The conditions did not occur live |
| Session replacement and worker-held files on a move | **Not exercised.** Trial runs used fresh sessions and had no local-only files |

Per-case results and limits are in [report5.md](report5.md).

## Measured

| | |
| --- | --- |
| Short task queued → started, beside a 180 s task | 14 s, against 232 s in a serial baseline (one pair, CPU fixture) |
| Interruption → continued, same node | 1 min 39 s; a second, unplanned case 1 min 37 s |
| Coordination turn | median 20.6 s, longest 47.2 s, about US$0.13 |
| Conversation message → reply, first turn of a run | 35 s |
| Conversation message → attempt, later turn (standing decision) | 3 s |

## What this costs you

- **The first job of each run now waits for a turn**, about 20 s plus a model call. Conversations feel this on their first message only.
- **Every turn is a Sonnet call and a run copy** of warden: about US$0.13 and 16 MB. Trials made 28 in about 90 minutes.
- **The 21 stale `evidence-fixture` jobs now cost a turn** each time their deferral comes due, next at 2026-11-04 05:44 UTC.

## Revisions

| Repository | Revisions |
| --- | --- |
| agdevworld | `96404c2`, `38adb4b`, `1053a73` (released); `c2cc951` test and docs only |
| agautolab | `f1b41a7`, `d63a91f`, `246e6f5`, `96b5a32` (gpupc); `1f20225` test only (studio) |
| pj-agdev | `706e1d1` agobserver; `6f2aba5` pins |
| warden (Gitea) | `8619d68` |
| p4trial (Gitea, new, disposable) | fixtures at `8fcf256` |
| nctl | `5c6dcf1` recipe text; worker pin batch in ignored records |

## Checks

| Check | Result |
| --- | --- |
| workflow_editor `npm test` | 743 of 743 at `c2cc951`; 742 of 742 in the build of the release |
| agobserver `pytest` | 127 passed |
| agautolab `pytest` | 52 passed, 1 skipped, 1 failed at `96b5a32`. The failure was a stale expected value, fixed in `1f20225`; I re-ran that file only (4 passed) |
| Live records | all 665 read `ok` under the release |
| Browser | Home, run view, Run details, room, on the deployed service |

## Problems and open items

1. **Six defects were found after the first deployment**, all fixed ([report5.md](report5.md)). Two came from reading code after the tests had
   passed, so more of this kind may remain.
2. **I set the commit author to "Claude" by a command-line override on 8 commits** (agdevworld `96404c2`, `38adb4b`; agautolab
   `f1b41a7`, `d63a91f`, `246e6f5`; pj-agdev `706e1d1`; warden `8619d68`; p4trial `9194f8f`, since rebased), instead of your configured identity. You did not ask for that. Most are
   pushed, so I left them as they are. Later commits use your configuration.
3. **#617 in `p4-check-room` reads as `iwaag` but I sent it** through the UI. #618 corrects it.
4. **Owner DMs were sent during this work**: one for a turn I killed, and one executor-down during heavy builds.
5. **The executor's heartbeat still lags under heavy local builds.** The keeper thread did not change that.
6. **Studio reports no host facts**, so warden believes it has no Godot and sends all Godot work to gpupc.
7. **Worker run folder round trip:** on gpupc, a forwarded `wfe` call can bring back an older copy of a file the agent is
   appending to (`progress.txt` lost lines in `busy/run-005` and `run-010`). Seen in fix p2 too. Concurrency is not its cause. Not changed.
8. **A launch in one lane's tick delays that lane's other attempts' stops and claims** until it returns.
9. **Basis friction:** starting one job makes the next job's token stale within the same turn. Harmless so far, but it costs retries.
10. **No browser page for the queue and decisions as a whole**; they are CLI only, with per-run facts in the run view.
11. **The conductor still sometimes ends with its node unrecorded**, the separate issue from `opinion.md`. Recovery now handles it quickly.
12. The `queue-stalled` condition documented since observer p3 has never been implemented.
13. A stray untracked `report.md` of `recover/run-082` is in `pj-warden/.local/p4-aside/`.

## For you to decide

- **Drop the 21 `evidence-fixture` jobs?** Their runs are completed, and p3 left them as not mine to drop. Warden deferred them with
  the same view. Dropping them ends the recurring turn.
- **Is the first-message delay and the per-turn cost acceptable?** If not, the evidence-based option is letting conversation runs start
  with a standing decision. I did not add that, because it would be a category rule the plan asked to avoid.
- **A real trial** with heavy GPU or display work is the next useful evidence, and it needs work you actually want done.
