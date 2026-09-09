# Refine routine p1 — guides and conditional execution

## Goal and scope

Organize routines as persistent process guides in Zulip. Let Front select work according to each request's conditions, delegate, reassess, and report completion.

This is a breaking change in a private experimental environment. Backward compatibility and migration of old APIs, formats, and execution history are unnecessary. Carry useful guide content into the new structure; remove or replace the old machinery freely. Authentication hardening and a general workflow engine are outside scope. Each step specifies outcomes; internal structure, APIs, record formats, and task decomposition are up to the implementer.

This phase covers immediate requests through termination. Developer interruption, forced cancellation of running tasks, and scheduled, delayed, or recurring triggers belong to later phases. Resuming after an ordinary reply wait and recovering after a process restart are part of this phase's execution foundation.

## step1 — Retire the old triggers and establish guides

- Inspect local service state through Nautobot or `pj-clusterintent/nctl`, then stop the old routine dispatcher and schedule GUI. Remove their startup configuration, schedule editing tools, and obsolete instructions for Front.
- Create a `routine` channel folder in Zulip, with one channel per routine and a fixed `guide` topic in each. Channel naming is discretionary.
- Read the existing `#front / routine-<name>` topics and carry over the content needed as guides. Historical reports are mixed in, so do not simply take the last post.
- Align how guides are updated and how their current content is read. Posting or editing a `guide` does not start execution; each execution request belongs in a separate `routinerun-<id>` topic.

Verify: the old trigger jobs are stopped, guides are readable in the new structure, and old events are not replayed.

## step2 — Let Front start runs and resume in the right conversation

- A routine request through Front's normal entrance makes Front read the guide, create a unique `routinerun-` topic in the relevant channel, and start execution.
- The initial record includes the original request, Front's interpretation of execution and termination conditions, the guide content or information identifying its revision, and the requesting channel and topic. Conditions may remain natural language; no rule language is required.
- Replies from delegated agents resume Front's judgment in the `routinerun-` that requested the work. Distinguish the final report destination from the conversation that owns ongoing execution.
- Ensure Front can start a run it created itself. The existing listener may treat self-posts differently from other authors' posts; creating a topic alone is not proof that execution started.
- Allow waiting to span short sessions and recovery to use conversation records after a restart. Avoid progress self-posts repeatedly waking Front in a loop. A waiting request should not block independent requests.

Verify with focused tests: startup, resumption on a delegate's reply, restart recovery, repeated retrieval of the same reply, and correct routing between separate runs.

## step3 — Provide observations and let Front decide whether to continue

- Give Front a way to read current utilization, the relevant account and window, reset time, observation time, and read failures, with usage instructions. The existing relay `GET /budget` is a reuse candidate. Verify access from Front's actual execution environment.
- Record at startup that “until usage of the 5h window reaches at least 50%” means the current window's utilization reaching 50%, distinct from consuming another 50 percentage points after startup. Interpret other conditions according to the request.
- Front reads results and observations, then chooses the next work, delegate, and whether to continue or finish. Allocation between source discovery and report writing, for example, remains the executing agent's choice. Do not prescribe a fixed work sequence or filler work merely to consume quota.
- If the condition is already met, Front can finish without starting additional work. If it is met during execution, stop issuing new work, let ongoing delegations reach a stopping point, and report. An exact 50% ceiling or immediate interruption is not guaranteed.
- Treat window resets, consumption by other work, stale observations, and read failures as evidence for judgment. A failed read does not establish success; record why execution continues, pauses, or ends. If paused, record what will resume it.
- Progress records capture requested work, returned results, outstanding replies, and the next decision with its reason. Send results, termination reason, remaining work, and a run link to the requesting topic, and make the run visibly finished.

Verify with controllable observation fixtures: reaching the threshold during execution, already meeting it at startup, window reset, and read failure. Run termination and deliverable achievement must remain distinguishable.

## step4 — Adapt existing views and completion to the new structure

- Build routine lists and details from the new channels, `guide`, and individual `routinerun-` topics. Make the guide, current conditions, progress, outstanding replies, results, termination reason, and requesting conversation accessible.
- Remove old schedule displays, configuration, and firing controls. Connect execution-request UI to the new start path as a request to Front. A full screen redesign is unnecessary.
- Adapt Ops/Agent Room ownership detection, execution relationship traversal, per-run usage aggregation, and shared completion to the new conversation structure.
- Clarify the roles of automatic run termination and the existing human `finish ✔` action in the UI. Handle relationships so completing the selected run does not include the persistent `guide`, routine channel, or other runs.

Verify in the browser: navigation from request to execution detail and final result. Use focused tests for completion scope.

## step5 — Deploy and prove an execution end to end

- Run relevant Front, shared-library, and relay tests, plus the frontend build. Add checks for implementation risks; exhaustive coverage for its own sake is unnecessary.
- Inspect state through Nautobot/nctl and local environment notes, then deploy and restart changed services. Update Front's introduction post to match its new entrance and execution contract.
- Request one real routine through Front's normal entrance. Exercise real delegation, reassessment after replies, progress recording, condition-based termination, reporting to the requester, and inspection in the UI.
- Choose conditions demonstrable with a short piece of useful work; consuming substantial quota is not itself a verification goal. Test utilization boundaries with step3's fixtures and observation access in the real environment, distinguishing the two in the report.
- Improve failures using conversation and execution records as evidence. If the Omni Agent performs work belonging to an in-system agent, record that fact and the handoff candidate.

Verify: read back actual work and the final report as evidence, rather than treating a startup ack as success. State which cases remain unproven.

## step6 — Documentation, cleanup, and report

- Remove remaining dependencies on the old schedule. Update startup instructions, Front's guide/tool documentation, UI/API documentation, `devdocs/README_DEV.md`, and ignored environment notes.
- Write a concise `report.md` covering changes, verification results, execution evidence, and remaining work. Keep machine details, credentials, and detailed local evidence in ignored files.
- Commit and push each changed repository and any necessary parent submodule pointers.

## Implementation hints

The following paths under `pj-agdev/` are starting points for locating changes. Reuse or replacement is discretionary.

- `devenv/routine/{dispatch.py,trigger.sh,rtschedule}` and `devenv/launchd/`: the old trigger and editing machinery. Job names are `com.agdev.routine-dispatch` and `com.agdev.routine-gui`. Environment notes say the old individual `rtnotes`/`imgprompt` jobs are already retired; inspect actual state when cleaning up.
- `agfront/src/agfront/{instance.py,listener.py,zulip_listener.py}`: currently uses the `front-` prefix and returns from mentions to the home conversation. Both the `front` and `character_talk` guides contain old routine instructions.
- Existing `[selfnote][rootchat]` records delegation origin, `[served]` records handled replies, and `[work]` records Work relationships. When Front creates its own run, distinguish the requesting conversation's relationship from ownership of delegated work. The existing conventions “another author posts to start work” and “self-posts do not start work” are startup pitfalls.
- `agdevworld/agentroom/src/agentroom/budget.py` and the `/budget` section of its README: provides `ok`, `windows[].percent`, `resets_at`, `read_at`, and `stale`. Dollar equivalents from `/cost` are not window utilization. The existing implementation handles caching and distinguishes read failures.
- `routines.py`, `ops.py`, `closing.py`, `close.py`, and `server.py` in the same directory, plus frontend `src/routineState.ts` and `src/operationDashboard.ts`: currently assume `#front`, `routine-<name>`, `front-routine-<name>-<stamp>`, and a schedule. `cost.py` also has places that assume a conversation's channel is `front`.
- Reports and test-related notes in the cross-project `devdocs/episodes/agdevworld/front_desk/p4/`: practical findings from reused delegation destinations, resolved topics, and archived channels. Compatibility code for old formats is unnecessary, but these findings help investigate recurring failures.
- Most changes belong in Front and agdevworld within `pj-agdev`. If the shared listener needs changes, update the shared library and dependency pins too. Use `pj-clusterintent` for state inspection and any deployment configuration changes that become necessary.
