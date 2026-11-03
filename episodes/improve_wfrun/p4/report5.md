# improve_wfrun p4 step 5: mechanisms verified, real agent decisions evaluated

Date: 2026-11-03. Plan: [plan.md](plan.md). Host-specific logs: `pj-agdev/.local/improve-wfrun-p4/trial/` (`log.md`,
`samples.jsonl`, `usage.py`), screenshots in `shots/`.

Every "agent" decision below was made by `coordination-agent` on the deployed system, on the `sonnet` profile, from the
guides in the warden project. Fixtures are in the disposable project `p4trial`. A stub appears only in the executor test
and proves nothing about judgment.

## Trials against the plan's cases

| Case | Result | Evidence |
| --- | --- | --- |
| Rendering plus research/recovery | **Met with a stand-in.** A 180 s CPU fixture stood in for a render; no real render was used | `busy/run-001` 05:45:43–05:49:04, `quick/run-001` 05:46:03–05:46:18 on studio; the short task ended 2 min 46 s before the long one |
| Godot plus Godot | **Partly met.** Decided on the actual work with no category rejection, and the attempts overlapped. Each Godot process lasted under 1 s, so overlap of the Godot processes themselves is not shown | `godot-lite/run-001`, `run-002` on gpupc, attempts 05:49:56–05:50:15 and 05:50:00–05:50:18; 8 frames each |
| Shared-state conflict | **Met.** The agent saw the hazard in the request and separated the runs by node, with a reason | `hazard/run-001` on studio, `run-002` on gpupc, both 30/30 own (decisions 30, 32) |
| Changing work | **Met.** The agent looked again and chose to change nothing, which it recorded | `changing/run-001` reconsider event 05:58:50 → note 43, 23 s later: "32 CPUs at load ~8, no change needed" |
| Alternative worker | **Met for choice, publication and completion.** Session replacement and worker-held files were not exercised: these runs used fresh sessions and had no local-only dependency | `bench/run-002` and `freework/run-001` moved studio → gpupc by the agent; recorded and published |
| Busy system | **Met.** Coordination stayed responsive and recovery did not wait for the long attempt on its node | `recover/run-112` 06:59:41–07:00:14 beside `busy/run-013` (to 07:01:06) on studio |
| Incorrect decision | **Partly met.** The wrong decision was traceable, and warden protected later work. The collision itself was not stopped | Below |
| Deferral | **Met**, by event (agent's own deferral) and by time (my hand deferral, agent's reconsideration) | `bench/run-002` decisions 57 → 59; `quick/run-004` decisions 71 → wake → 73 |
| Multiple attempts and restart | **Met** on both worker types | studio restart 05:58:58: both attempts adopted. gpupc unit restart 06:11:15: both adopted |
| Racing requests | **Met in fixtures, with live stale decisions** | `test/coordination.test.ts`; 6 live stale decisions applied nothing |
| Judgment unavailable | **Met** for a killed turn. A model outage across several turns is covered by fixture only | Turn 12 killed 06:08:41, turn 13 decided both jobs by 06:08:58 |

### The incorrect decision

I admitted `hazard/run-003` and `run-004` on studio by hand (decisions 33, 34, actor `p4-check`). Attempts had already
opened when warden's turn came, so:

- Warden tried to move `run-004` to gpupc and was refused, correctly, because its attempt was open (decision 35).
- Its note 37 told `run-004` it had been moved, before it read the refusal. Note 38 corrected that 4 s later.
- `run-003` ended with "1/30 own, 29 from hazard/run-004". The interference is traceable from the two outcomes and the decisions.
- `run-005`, requested afterwards, went to gpupc because "two hazard runs already share it on studio".

Warden did not stop either colliding run. For fixtures lasting 90 s that is defensible, but **a stop as a revision has not
been seen from the agent in any trial.**

## Measured

| Measure | Value |
| --- | --- |
| Coordination turns | 28: 27 ended, 1 `lost` (turn 14, the defect fixed in `1053a73`) |
| Turn duration | median 20.6 s, longest 47.2 s |
| Model cost | US$3.68 over 28 attempts, about US$0.13 each (`sonnet`) |
| Decisions | 63 applied, 6 stale, 3 superseded, 1 refused, including my hand decisions |
| Queued → the agent's applied decision | median 21.1 s, longest 228.7 s (33 decisions, deliberate waits included) |
| Queue delay of `p4trial` attempts | median 24.7 s, shortest 3.1 s (a standing decision), longest 232.2 s (the serial baseline) |
| Most ordinary attempts at once | studio 4, gpupc 2 |
| Time with 2 or more ordinary attempts | studio 648 s, gpupc 492 s |
| Disk | about 16 MB per coordination run copy |

### Against a serial baseline

The same pair, `busy` then `quick`, on studio:

| | `quick` queued → started | `busy` run time |
| --- | --- | --- |
| Concurrent (`quick/run-001`) | 14 s | 180 s |
| Serial by hand decision (`quick/run-003`) | 232 s | 180 s |

Limits: one pair each, a CPU fixture, and a host otherwise at load 8 to 10 of 32 CPUs. It shows the waiting removed for
this shape of work. It says nothing about heavy work sharing a node.

### Recovery, the case that motivated this phase

| Step | `busy/run-014` (T10) | `conduct/run-074` (unplanned, the same kind as the incident) |
| --- | --- | --- |
| Interrupted | 06:58:25 | 07:10:52 |
| Run continued | 07:00:04 | 07:12:29 |
| Elapsed | 1 min 39 s | 1 min 37 s |

Most of this is Observer's look interval and intake, not coordination. On one remote case under a release build, `busy/run-006`
took 5 min 28 s (load about 49 on studio).

## What the agent did well and badly

- Reasons cite actual facts: load, free memory, the path in the request, which node is idle.
- It placed a timing measurement on an idle node when one existed (`bench/run-001`) and deferred only when none did (`bench/run-002`).
- It kept concurrency when the hazard was absent, without inventing a category rule.
- It deferred all 21 stale `evidence-fixture` jobs in its first turn and said why.
- **A wrong statement:** note 37, as above.
- **A missing fact shaped decisions:** "studio lists no Godot". Studio has Godot, but its worker reports no host facts, so all
  Godot work went to gpupc. I left this unfixed so that it is a visible finding; it is an environment fact to add.
- **Friction:** one start in a turn changes the basis of the next, so 6 decisions went stale and the agent retried, sometimes
  without the basis, and once with the reason "x" (decision 65, which applied nothing).
- **Untested judgment:** no trial had real interference that forced a hard choice on heavy GPU or display work.

## Defects found by the trials and by reading afterwards

| Defect | Found by | Fixed in |
| --- | --- | --- |
| A deferral's `--on` never fired: it compared ids from two tables | reading, after no trial had used it | `1053a73` |
| A turn claimed before it was linked was closed `lost`, its decisions uncounted | live, turn 14 | `1053a73` |
| A move's basis was computed over the target node, so it was always stale | live, T3, T4 | `38adb4b` |
| An adopted process gone without an exit file was held for ever (older than p4) | live, T8 | agautolab `246e6f5` |
| The keeper's reconcile could call a slow claim's attempt lost | reading | agautolab `96b5a32` |
| macOS memory not measured under launchd | live, first view | agautolab `d63a91f` |

Two of these came from reading code after fixture tests had passed, so **other defects of this kind may remain.**

## Checks run

| Check | Result |
| --- | --- |
| workflow_editor `npm test` at the release `1053a73`, in the build | 742 of 742 |
| workflow_editor `npm test` at the final test commit `c2cc951` | 743 of 743 |
| agobserver `uv run pytest -q` | 127 passed |
| agautolab `uv run pytest -q` at `96b5a32` | 52 passed, 1 skipped, 1 failed; the failure was a stale expectation of the new `placed` field. Fixed in `1f20225`, and that file re-run: 4 passed. The whole suite was not re-run |

New regression tests cover two attempts on one worker, simultaneous remote forwarding, heartbeats, ends and publication,
stopping only one attempt, and concurrent enqueue and decision application. I did not confirm that the new remote test
fails without the state lock. Replay check on the live records: `wfe observe --terminal` under the release read all 665 runs, every one `ok`, with no problems.

## Mistakes in my own trial runs

- T7 first pass: the kill came after the turn had decided, so it showed nothing. Redone.
- T6/T8 first pass: a quoting mistake skipped the restart and the kill. Redone.
- T8 second pass: I killed the launch gate, not the harness. This is what exposed the adoption defect. Redone.
- T9 and T10: two script mistakes (short receipts, then an unsplit `--node` argument) created `busy/run-008`–`010` without the
  intended node, so warden placed them. Run-010 was on gpupc, and my local kill hit nothing. Redone as T10.
- `busy` tells the agent to use `wfe run progress --text`, but the option is `--note`, so some agents noted a failed call.
