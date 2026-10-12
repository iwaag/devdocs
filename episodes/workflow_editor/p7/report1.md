# Workflow editor p7 — step 1: presentation and adjustment contracts

## What was inspected

- **Source and release.** The editor is `pj-agdev/agdevworld/workflow_editor` (agdevworld `1d87d56`, one documentation commit after the installed release). The installed release is `0c2c072` (`wfe-release.sh status`: current `0c2c072`, previous `78caec2`, no open attempt). The executor is `pj-agdev/agautolab` at `9a9aa98`, `com.agdev.agautolab-wfexec` running with `[notify]` and `[intake]` enabled.
- **Service state.** `nctl status` (pj-clusterintent) reports Nautobot, the worker and the node dumps healthy. `nctl drift --host agstudio.home.arpa` has no targets: the agdev services on this host (wfe service, wfexec) are not modelled in Nautobot, so their state was read from launchd and `wfe` itself.
- **Execution records.** Every run in the catalog is completed: `rts-vs-bot` `improve-game/run-001`, `run-002`; `ppt` `study/run-001…003`, `create/run-001` (accepted). The exec store has no queued job and no open attempt (19 jobs done, 1 dropped, 19 attempts ended). `notify.sqlite` is at `user_version` 3 with 43 deliveries, all sent.
- **What the code does today.** One reducer (`shared/run.ts`) replays a run's history against a single graph. The definition snapshot is one byte copy plus one frozen effective graph. Result confirmation binds a decision to loose deliverable files by hash. The executor (`agautolab.wfexec`) launches the agent in a process group of its own (`start_new_session=True`) and, when the agent's leader process exits, sends `SIGKILL` to the whole group. A server started as a background child of the agent therefore dies with the attempt. That is the constraint step 4 designs around.
- **RTS today.** `improve-game` (v2, default policy) is `implement → verify → human-check`, without execution settings, intake, presentation or cleanup. `run-002`'s reports say twice that the browser check was not done ("環境にブラウザがなく未実施"). The person answered "オッケー！" from the agdevworld run view, which is loopback-only (`zulip.json` has no `viewBase`). `build-game` is still `ag.workflow.v1` and unsupported since p5.

## Contracts written

In the editor's documentation (agdevworld):

- [`docs/presentations.md`](../../../../pj-agdev/agdevworld/workflow_editor/docs/presentations.md) — presentation identity and revision (manifest digest), the frozen bundle in the run folder, review content and its safe rendering, the review screen and what it serves, the lifecycle states and their operations, local server ownership and lifetime (session of its own, owner token, start signature, verified stop), pause/cancel/complete policy, confirmation bound to a presentation, notifications, and host configuration (`present.json`).
- [`docs/adjustments.md`](../../../../pj-agdev/agdevworld/workflow_editor/docs/adjustments.md) — original definition and immutable revisions (`r1`, `r2`, …) with complete authored files and compiled graphs, replay across revisions, what may change (only `pending` work; no access, execution or policy change), the boundary (no node running, no pending instruction, sequence and parent revision checked under the lock), authorization by source (answer, instruction, agreement, person), and the correction-after-review sequence.
- `docs/contract.md` — *Appearance* (icon and color tokens, presets, digest exclusion, state legibility).
- `docs/runs.md` — pointer section and the `presentation-open` completion rule.

The six points the plan asked for map as follows:

| Plan item | Where |
| --- | --- |
| 1. Appearance fields and presets | contract.md *Appearance* |
| 2. Presentation identity, revision, review, URL, observations, evidence | presentations.md *Identity*, *Lifecycle*, *Confirmation* |
| 3. Server ownership and lifetime | presentations.md *Serving instances*, *Pausing, cancelling, completing* |
| 4. Original, revisions, active revision, provenance, replay | adjustments.md *Revisions*, *Who authorizes it* |
| 5. When a revision becomes active | adjustments.md *When an adjustment may be applied* |
| 6. Accepted with cleanup pending | presentations.md *Pausing, cancelling, completing*, *Notifications* |

## Decisions

- **URL is host state.** `run.json` keeps the instance id as a logical serving reference. The URL is resolved from `<area>/.local/present/present.sqlite` and `<area>/present.json` when a view or message is rendered. No port, host name, token or path enters a tracked file.
- **The bundle lives in the run folder** (`presentations/<p>/`), copied by the file layer. It is published with the run, so the evidence outlives the server. The server serves only manifest files and refuses to start on a hash mismatch.
- **Serving origin.** Each instance has its own origin (its own port), never the editor's. A presented program therefore cannot use the editor's same-origin write rights. The explanation panel is rendered on the server and contains no script.
- **Default bind is loopback.** Links are worded "opens on this host". The intended route is the person's browser on agstudio, which is where they already use agdevworld (`localhost:8093`). A reachable bind needs `publicBase` in `present.json`.
- **Lifetime.** The instance starts in a session of its own, outside the attempt's process group, and is owned by run + presentation in the host store. A stop is verified by the process being gone and the port no longer answering with the instance's identity. Nothing restarts by itself.
- **Pause keeps, cancel stops.** `run.stop` (a held run) leaves the presentation served. `run.cancel` stops it through a cleanup path that does not depend on the graph's successors.
- **Completion waits for cleanup.** The completion that would complete the execution is refused while a presentation is open. This is generic, so it does not depend on recognizing a "cleanup" node, which stays an ordinary `do`.
- **Adjustment authority comes from the person's records.** An answer taken up, an acknowledged instruction, an agreement to the exact proposal, or a person applying it. Nothing adds a blanket approval prompt.
- **One result confirmation per run, counted among unfinished nodes.** A correction adds a new confirming talk, and the decision follows the latest confirmation question.

## Schema and cutover decision

| Artifact | Change | Version |
| --- | --- | --- |
| Workflow files | optional `nodes.<id>.appearance` | stays `ag.workflow.v2`. Earlier releases ignore the field (unknown node fields are preserved) |
| Definition digest | appearance excluded | projection kind unchanged, so no approval becomes stale |
| Compiler | unchanged expansion and instructions | stays `ag.wfe-compile.v2` |
| `run.json` | additive: `presentations`, `definition.active`, `definition.revisions`, `questions.<q>.presentation`, `decisions[].confirmation.presentation`; ops `presentation.register`, `presentation.observe`, `run.adjust` | stays `ag.workflow-run.v2`. The new fields are absent until first used, so every existing record replays to its stored state unchanged |
| `present.sqlite` | new host store | `user_version` 1 |
| `exec.sqlite`, `notify.sqlite`, `intake.sqlite` | none planned | 1, 3, 1 |
| Notify protocol | new event types only; transport unchanged | stays 2 |
| autolab | common contract guide (facts on presentations and adjustments) | no protocol change |

**Cutover.** Existing completed runs need no conversion. A run that uses a new operation cannot be read by `0c2c072` (`op-unknown`). Rolling back after the first presentation or adjustment therefore makes those runs unreadable to the old release, though they are not damaged. Step 7 records this as the rollback constraint. No compatibility adapter is added.

## Next

Step 2: the `appearance` field through model, YAML round trip, validation, inspector, canvas and run views, with the two presets.
