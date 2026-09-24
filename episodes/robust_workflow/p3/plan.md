# Robust workflow p3 — Correct waiting, completion and receipts

Sources: [p2 report](../p2/report.md), [p2 trials](../p2/report5.md), and the follow-up review. Original desire: [braindump](../p1/braindump.md).

## Goal and working approach

Keep legitimate waits tracked, apply judgments only to the evidence they evaluated, and close accepted work with explicit records. Make ordinary answer handling leave a receipt without needing Observer to ask for one.

- Keep Observer on its current **local backend throughout p3**. Decide from the results whether a later Sonnet comparison is warranted; switching backends or running that comparison is not a prerequisite for completing this phase.
- This is a private experimental environment and a breaking phase. Backward compatibility is unnecessary. Implementers choose the design and may replace code, interfaces and disposable state. Prefer fewer shared mechanisms over additional agent rules.
- Preserve human acceptance and cancellation decisions; carry existing authorization forward. Keep secrets and machine-specific details in ignored files. No additional security framework or repeated approval for routine implementation choices is required.
- Focus on tracking, completion and receipts. Wider discovery for routines, argues and Project Room follows once these contracts work.

## step1 — Reproduce the remaining gaps and establish the baseline

- Read p2's implementation and trial evidence, and check current service state and observation freshness through Nautobot or nctl.
- Reproduce a `resolved_live` incident judged legitimate because the human has not answered, with no other unfinished child to keep the request tracked. Verify whether the request is dropped from retention.
- Hold a triage judgment while a relevant acceptance, cancellation or new answer arrives. Check whether the old verdict is applied to the new state.
- Trace a mission's final acceptance and an owner-route serving that reads a delegated answer without an Observer owed note. Identify the missing completion and receipt writes.
- Record baseline retained requests, redundant servings and unresolved receipts. Distinguish reproduced defects from hypotheses and historical data without sufficient evidence.

Done: `report1.md` contains focused reproductions, affected paths and expected outcomes.

Hints: the review reproduced `monitor.retain()` dropping a request whose only unfinished child was an `awaiting_requester` task with a dismissed `resolved_live` incident. The stale-judgment concern was found by reading `judged()`, not reproduced live. p2's missions remain open indefinitely, which can mask premature retention removal elsewhere.

## step2 — Separate legitimate waiting from terminal decisions

- Make “no recovery request is needed” independent of “this work is finished.” A human wait remains tracked; completion, cancellation and replacement require their own evidence.
- Update retention and incident handling together. A dismissal may suppress an inappropriate nudge, but must not erase unfinished work or prevent detection of a later blockage.
- Bind asynchronous judgments to the evidence snapshot they evaluated. Before using a verdict, check for relevant changes; discard or reconsider stale results without resetting recovery allowances.
- Keep re-evaluation bounded. Ignore irrelevant changes where practical, and make judgment backlog or repeated invalidation visible through existing health reporting.

Done: human waits survive the discovery window and restart, terminal decisions end tracking appropriately, and a judgment on superseded evidence cannot trigger recovery or end tracking.

Hints: `legit` currently covers both deliberate closure and waiting for a human. Fix the consuming semantics rather than merely asking the model to be more careful. Snapshot identity can use message IDs and state/version information already available; choose the smallest representation that detects meaningful changes. Preserve p2's distinction between fresh observation and a recently computed trace of stale data.

## step3 — Record mission acceptance and completion coherently

- Define when the mission itself is accepted, using the requester contract and existing review points. Accepting the last task and accepting the whole mission may coincide only when the authorization makes that clear.
- Provide an operation that records the acceptance and mission `done` state, with its evidence, and completes the associated bookkeeping. Remove planning runs and acknowledgements whose only purpose is relaying an already-made decision.
- Make retries and interruption recognize completed acceptance work. Ensure the normal requester path and existing completion UI agree on the mission's state.
- Let Observer release a request once its dependent work has genuinely finished and its incidents are settled. Reconcile p2 trial leftovers only where acceptance evidence exists; otherwise keep them visibly pending.
- Update autolab's public introduction and affected guides with the ownership change, deleting obsolete relay instructions.

Done: a newly accepted multi-task mission reaches `done`, leaves no needless tracked work, and repeated acceptance creates neither duplicate execution nor another planning exchange.

Hints: inspect autolab's mission/worklog operations and the relay's existing completion writes before adding another completion path. Topic resolve is not the acceptance record. Retention should shrink because work reached an evidenced outcome, not because a cleanup command hid it.

## step4 — Make receipts follow actual answer processing on every route

- Record which delegated answers a serving actually received as input, whether it started through an owner post, a mention, or an Observer recovery request.
- After that serving's reply is confirmed delivered, write receipts for those answers. An answer arriving later or excluded from the supplied context remains owed; unrelated home speech is not a receipt.
- Reuse the same accounting in trace, listener dispatch and restart recovery. Recognize just-written receipts before the mirror catches up, preserving p2's fix for duplicate servings.
- Make startup recovery find owed callbacks in resolved topics by anchor. Resolve alone must neither erase an outstanding answer nor reopen finished work as a twin.
- Cover interruption before execution, after execution, during delivery and before receipt persistence. Retry delivery or bookkeeping where possible without repeating completed work.

Done: a normal owner-route serving consumes the answers it actually received without an extra “nothing new” serving, while unread answers and callbacks under ✔ remain recoverable across restart.

Hints: p2's `[selfnote][owed]` handles recovery requests, but normal thread reads still lack equivalent receipts. Start with the serving journal, input construction, `_mark_owed()` and served marks. A global newest message ID is not proof that every earlier answer appeared in the prompt, especially with truncated histories. Keep request identity separate from queue names while changing these paths.

## step5 — Verify the final revision with the local backend

- Freeze the deployed version set, model configuration and thresholds for acceptance testing. Verify installed dependencies and running processes. If a fix changes the version set, repeat the affected cases and ordinary cycles on the final set.
- Run three normal cycles, including a multi-task mission, a forge-to-autolab handoff and concurrent requests. Verify mission acceptance, final delivery, receipts and release from tracking.
- Use focused fixtures plus controlled live cases for the table below. Repeat the stale-judgment and ordinary-receipt cases at least twice live. Use controlled clocks for the discovery-window boundary and label that evidence separately.

| Case | Expected outcome |
|---|---|
| Legitimate human wait after a ✔ | No recovery nudge; unfinished request remains tracked across inactivity and restart |
| Acceptance or cancellation arrives during triage | Old verdict is rejected or re-evaluated against the new evidence |
| Last task accepted, mission decision still pending | Task closes; mission remains pending until its required decision exists |
| Mission acceptance repeated or interrupted | One evidenced completion; request eventually leaves tracking |
| Answer supplied to an owner-route serving | Receipt after reply delivery; no redundant mention serving |
| Answer arrives mid-serving or outside its input | Remains owed and is subsequently handled once |
| Owed callback in a resolved topic at restart | Original request receives the answer without a twin or repeated completed work |
| Real mirror connection loss and resynchronization | Staleness becomes visible; no unsupported conclusion; tracking and receipts survive recovery |

- Exercise the real mirror failure path with an isolated fixture/service connection where practical; a flag that only declares staleness is insufficient for this case. Record whether queue expiry itself was tested.
- Keep p2's applicable detection and health targets unless baseline evidence justifies a change before testing. Measure false recovery, lost tracking, duplicate work, redundant servings, retained completed requests, elapsed time and cost.
- Count operator fault injection and ordinary human decisions separately from rescue. Record any substituted work as `did X for agent Y — handoff candidate`.

Done: the fixed local-backend version passes the agreed cases with no false recovery, lost unfinished request, duplicate completed work or Omni Agent rescue. Completion and receipt evidence is inspectable, and fixture-only limits remain explicit.

## step6 — Decide whether a Sonnet comparison is the next useful experiment

- Review local triage results after the code fixes. Preserve the exact inputs, verdicts, cited evidence, backend configuration and timing for successful, incorrect and ambiguous cases.
- Separate failures of evidence collection, truncation, execution or result handling from judgment errors given sufficient input. Inspect whether the current last-ten-message selection and 600-character-per-message truncation omit material decisions; improve demonstrated input gaps and recheck locally.
- Decide explicitly: **run a Sonnet comparison next**, or **defer it**, with evidence and a condition for reconsideration. Similar-looking cases with different inputs do not establish model inconsistency; uncertainty caused by missing facts may correctly produce `unclear`.
- If comparison is warranted, specify a small follow-up: approximately 15–20 fixed snapshots covering stalls, legitimate waits, explicit cancellation and insufficient evidence, three judgments per backend, with no live recovery effects. Establish expected outcomes independently of either model and compare missed stalls, unnecessary recovery requests, appropriate uncertainty, evidence accuracy, consistency, latency and cost.
- Limit the proposed comparison to `triage` unless findings implicate another role. Check harness arguments and output handling first: the current local/sonnet profiles change both model and harness, and triage passes `--deadline-s`. Report a backend comparison as such unless the model variable can be isolated.

Done: `report.md` contains an evidence-supported decision about a later Sonnet test. P3's acceptance remains on local; no model switch is used to conceal an unresolved implementation defect.

## step7 — Complete rollout and report

- Update affected consumer dependencies, public introductions and developer documentation; verify versions actually running. Remove superseded instructions and temporary trial configuration.
- Write `report.md` with fixes, final-revision verification, before/after retention and redundant-serving counts, remaining identity/recovery limits, and the step6 decision.
- Prioritize the next action from the findings: the bounded backend comparison if warranted, remaining recovery defects, or wider discovery once completion and receipts are sound.
- Commit and push changes and necessary submodule references.

Done: the deployed local-backend workflow has correct waiting, explicit mission completion and route-independent receipts, with its remaining limits and next experiment recorded.
