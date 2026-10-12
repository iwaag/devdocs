# Workflow editor p7 — present results for human review

## Goal

Implement [braindump.md](braindump.md) so that a person can open a delivered result with one click, inspect it with the explanation of what changed and what to check, and give feedback through the existing workflow conversation. Complete a real RTS improvement run through presentation, human acceptance, and shutdown of its presentation server.

The person should concentrate on expressing wishes and judging results. Repeated instructions about how to serve or locate artifacts belong in reusable workflows and project guides.

## Agreed direction and scope

- Keep `presentation` and `cleanup` as uses of `do`, with configurable visual appearance. Preserve the execution semantics of existing types, including result-confirmation talks and delegation.
- Record presentations separately from ordinary artifact paths, binding each presentation and human decision to the exact result revision.
- Manage the local presentation server independently of the agent attempt, so it survives human waits.
- Preserve the original run definition and record run-specific changes as **adjustments**, each with an immutable definition revision and history evidence.
- Limit adjustments to future work at node boundaries. Preserve completed work and existing questions; corrections become additional work rather than rewritten history.
- For RTS, use: plan and agreement → implement → verify → present → human check → cleanup → final report. Cleanup precedes the generated final report so the report includes its outcome.
- Keep result acceptance, cleanup progress, and execution completion distinct.

Deferred: cloud storage integrations, resource discovery, authentication distribution, general service orchestration, arbitrary mid-node graph mutation, automatic rollback of completed work, and migration of unrelated agents. PPT delivery is motivation, not a second project implementation in p7.

This is experimental development. Backward compatibility is not required. Prefer a clear contract over compatibility adapters or a general migration framework. Before deploying breaking changes, identify affected definitions, records, active runs, and processes; preserve useful evidence and document any explicit conversion or reset. Do not modify unrelated user changes.

## Step 1 — define the presentation and adjustment contracts

Inspect the current source and installed release before implementation. The system implementation lives in `pj-agdev/agdevworld/workflow_editor`; the executor is `pj-agdev/agautolab`. The former experimental editor is an archive. Consult Nautobot or `nctl` for service state, as required by the development instructions, and inspect the relevant execution records before deployment.

Use the existing model, compiler, reducer, execution queue, document evidence, publication, intake, and notification mechanisms. Write the contracts in the editor's documentation and establish the required schema/compiler versions.

Specify:

1. Appearance fields and presets, independent of execution type, permissions, prompts, and state.
2. Presentation identity, result revision, review content, URL, lifecycle observations, and links to result-confirmation evidence.
3. Local server ownership and lifetime across agent exit, human waits, restart, acceptance, failure, and cancellation.
4. Original definition, adjustment revisions, active revision, adjustment provenance, and replay semantics.
5. The exact boundary at which a revision becomes active, including concurrent input, stale clients, and generated documentation steps.
6. How accepted results with pending or failed cleanup appear in the run view and notifications.

Host-specific URLs, ports, process details, and credentials belong in ignored runtime configuration and state. Versioned project files describe portable behavior and logical references.

Deliverable: documented contracts and a concrete schema/cutover decision for the following steps.

## Step 2 — separate node appearance from execution behavior

Add optional appearance fields, initially an icon and color, to workflow nodes. Provide presentation and cleanup presets that create ordinary `do` nodes with editable names, descriptions, and appearance.

Carry appearance through YAML parsing and round-trip saving, validation, the editor inspector and canvas, run snapshots, and historical views. Define its digest behavior explicitly: appearance alone must not change executable meaning or invalidate an agreement about work; semantic changes retain their existing approval behavior.

Preserve recognizable execution types and state indicators. Custom colors must not obscure running, waiting, completed, or failed states; retain text and glyphs independently of color. Existing nodes without appearance use their normal defaults.

Deliverable: people can customize node appearance without changing execution behavior, and implementation, presentation, and cleanup are visually distinguishable.

## Step 3 — record and expose reviewable presentations

Introduce a presentation record associated with a run and originating node. It identifies a title, immutable artifact revision or manifest, review content, and a logical serving reference. Resolve machine-specific serving details through the local runtime state rather than embedding them in tracked project files.

Provide an operation for registering a presentation and show a prominent review link in the run view. Use the existing notifier to include the same usable link in the relevant conversation; preserve its delivery and retry guarantees. A local file path alone is not delivery.

The review content identifies what changed, how to use the result, and what the person should check. For RTS, bind a frozen copy of the game and its explanation to one presentation revision. Later workspace changes must not silently alter it. Re-presentation creates a new revision and a new confirmation subject.

Reuse the result-confirmation document evidence mechanism to bind the person's answer to a manifest identifying the actual presented files. A URL alone is insufficient evidence because its content or availability can change. Keep presentation records and decision evidence after shutdown; show that a stopped presentation is no longer live instead of offering a misleading active link.

Verify the URL from the intended user's browser route. Do not advertise loopback access as remote access. Validate URL schemes and render review text safely; serve only the intended presentation content, not the project or runtime configuration directory.

Deliverable: one click opens the correct result and its explanation, and the recorded decision identifies exactly what was reviewed.

## Step 4 — manage the local presentation server lifetime

Add a small local presentation lifecycle facility, with start, status, and stop operations usable by the executing agent. Keep it scoped to serving local presentation bundles rather than general resource management.

The executor currently terminates remaining processes in an attempt's process group when the agent exits. Therefore, the presentation server must have an independently managed lifetime and durable ownership by run and presentation, not be an incidental background child of the attempt.

Implement these behaviors:

- Starting the same presentation twice returns the existing instance or a clear recoverable state rather than allocating duplicate servers.
- Mark a presentation available only after checking its serving endpoint.
- Human waiting and normal agent exit leave the server available.
- Status and recovery after a controller restart distinguish observed liveness from a stored claim that the server is running.
- Stop targets only the owned instance, using identity stronger than a bare PID or port; repeated stop is safe.
- Cleanup completes only after shutdown is verified. Failures remain visible and retryable without asking the person to accept the result again.
- Cancellation has an explicit cleanup path independent of ordinary successor readiness, since a cancelled graph cannot execute a normal cleanup successor. Preserve cancellation status and show cleanup failures separately.

Distinguish pausing an attempt from cancelling a run: a paused review should not lose its presentation merely because the agent is not running. Record the chosen policy and expose recovery operations.

Deliverable: a presentation survives an agent attempt, can be recovered or diagnosed after restart, and can be stopped reliably by its owning run.

## Step 5 — support recorded adjustments to future run work

Retain the byte-for-byte initial definition snapshot. Store each adjustment as a new immutable authored definition and compiled execution graph, with a revision identity, parent revision, reason, source input or talk evidence, and application event. Store complete revisions; do not rely on mutable source YAML or recompilation by a later compiler to reconstruct history.

Implement a shared operation used by CLI/API and the run UI to inspect a proposed change and apply it. Show the original definition, current adjusted definition, and differences. Do not automatically update the reusable workflow definition.

Apply adjustments under the existing run lock and stale-state checks at a node boundary. Existing additional-instruction acknowledgment still applies. The agent's acknowledgment of an instruction and the application of a graph revision are separate recorded facts.

For p7:

1. Add, remove, or change only unstarted future work, including presentation and cleanup nodes. Do not broaden repository access or change execution identity through this facility.
2. Preserve started/completed node identity, meaning, state, and recorded dependencies. Do not remove or rewrite a node with a pending question.
3. Keep a graph acyclic with valid dependencies, and preserve the mapping between authored work and generated reports. Completed reports and decisions remain unchanged.
4. Make the original history replayable across revision events; preserve state for unchanged nodes and initialize only new work.
5. Refuse stale or invalid adjustments atomically. Incomplete revision writes must not leave the run pointing to an unavailable definition.
6. Bind material changes of agreed scope to renewed agreement through a talk and document evidence. Do not add a blanket approval prompt for routine implementation choices already authorized by the person's request.

A requested correction after review adds repair, verification, re-presentation, and a fresh result-confirmation talk. Preserve the earlier question, answer, and presentation. Reconnect only future work so cleanup follows the latest review. Never convert a revision request into acceptance or erase the earlier review by resetting completed nodes.

Deliverable: an ongoing run can change its future presentation and cleanup work, with reproducible history and no alteration of its original definition or completed evidence.

## Step 6 — adapt the RTS workflow and review screen

Update the active RTS `improve-game` workflow to add presentation and cleanup, and designate `human-check` with `confirms: result`. Preserve generated planning, per-node reports, and final reporting under the default documentation policy.

Build the review screen around the frozen game bundle. Display the playable game and a visible explanation panel in one screen, with the request-specific changes, controls, and verification checklist. Link back to the existing talk answer location. Keep feedback and acceptance in that established conversation rather than adding another independent decision endpoint.

The presentation node creates the bundle and review content, starts its server, verifies reachability, and records the review link. The human-check node asks about that presentation revision. On acceptance, record the decision and proceed to cleanup; on a correction request, use an adjustment for repair and re-presentation.

Cleanup stops the run-owned presentation instance and records the verified outcome. The final report includes the delivered revision, verification, human decision, and shutdown result. Display “accepted; cleanup pending” when applicable rather than announcing execution completion early.

Put reusable presentation instructions in the workflow/project guide so a new braindump need not repeat serving or delivery instructions. Check the status of other RTS definitions before changing them; do not revive an obsolete workflow merely to make every file look current.

Deliverable: a reusable RTS workflow that takes a new improvement request through accessible review and cleanup.

## Step 7 — verify, deploy, and record the release

Use isolated registries, repositories, process instances, and fake notification transport for deterministic tests. Run the affected editor tests, type/build checks, relevant browser checks, and executor tests when that integration changes. Add tests for behavioral contracts rather than visual constants.

Required evidence:

1. Appearance round-trips correctly and does not change execution behavior or obscure state.
2. A review link opens the game and request-specific explanation together, including through the intended user route.
3. An agent exit during a human wait does not stop the presentation server.
4. Workspace edits do not mutate a presented revision; stale answers cannot accept a newer presentation.
5. Repeated start/stop, restart recovery, startup failure, and shutdown failure are handled without duplicate or unrelated processes being affected.
6. An adjustment preserves past state and replays correctly; stale, cyclic, mid-node, question-destroying, and access-expanding changes are rejected.
7. A correction can lead to new work and a new review without resetting completed nodes or reusing the prior decision.
8. Acceptance proceeds to cleanup without a second acceptance request. Cleanup failure prevents normal execution completion and remains retryable.
9. Cancellation stops owned resources through its separate cleanup path and accurately reports any remaining failure.
10. Pending additional instructions cannot be bypassed by applying an adjustment or finishing the run.

Inspect active attempts and presentations before release. Deploy through the established agdevworld release mechanism, updating the executor only if required. Record the tested commits, schema changes, affected records, rollback constraints, and live smoke evidence. Consult Nautobot or `nctl` for the relevant service state; do not infer it solely from old development notes.

Deliverable: a verified release with deployment evidence and explicit limitations.

## Step 8 — complete a real RTS review cycle

Use a human-authored improvement request. Let the in-system executor implement and verify it, prepare the presentation, and ask for review. Confirm that the person can open the game and its explanation without searching for files or supplying serving instructions.

Exercise a correction and adjustment in a real run if the person requests one; otherwise retain isolated end-to-end evidence for that branch. Never manufacture feedback or human acceptance to complete the trial.

After the person's actual acceptance, verify that the presentation server stops and the final report records the accepted revision and shutdown outcome. Preserve the run, presentation manifest, adjustment history, decision evidence, and report references. Record any work performed by the outside development agent on behalf of the in-system executor as a handoff candidate under the development policy.

P7 is complete when a real RTS result has been opened for review, accepted by the person, and its presentation server has been shut down; the supporting lifecycle and adjustment failure paths have been verified; and the reusable workflow carries the delivery behavior for subsequent requests.

## Reporting

Write step reports with changes, verification evidence, limitations, and the next entry point, followed by a final `report.md`. Explain material changes from this plan when implementation evidence warrants them. Keep local infrastructure facts in ignored documentation and runtime state.

Commit and push the intended changes according to the local rules, without including unrelated user edits.
