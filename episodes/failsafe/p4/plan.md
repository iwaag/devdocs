# Failsafe p4 — finish accepted work reliably and preserve long replies

## Goal and scope

Strengthen the existing Front → autolab path: close accepted work without
doing it again, deliver long replies intact, verify recovery across several
small trials, and correct known inaccurate task records.

Read `../p3/report.md` and `../p3/report6.md`. P3 detected a silent exit in
144 s and Front resumed it in 177 s, but another trial needed developer
intervention, an agreement caused duplicate execution, and long replies
were still cut. Preserve the five-minute detection target while addressing
these observed gaps.

This is a private experimental, breaking-change phase. Backward
compatibility is unnecessary. Give implementers discretion over mechanisms
and remove obsolete code and guidance. Keep agent judgment for diagnosis
and repairs; make the supporting evidence and close-out operations reliable.

All-agent instrumentation, remote probes, cross-host recovery, and new
journal-retention work are outside scope. Multipart delivery is not the
default solution: first use Zulip's configurable message limit.

## step1 — reproduce the remaining failures and define evidence

- Inspect current code and deployments; confirm which reported gaps remain.
  Before service changes, use Nautobot or `nctl status` / `nctl drift` and
  the local environment notes (`pj-clusterintent/nctl/README.md`).
- Reproduce the agreement-triggered rerun with a short operation whose
  execution count is observable. Include a result placed in the wrong
  location, the condition seen in p3 C; no 420-second wait is necessary.
- Trace acceptance through shown result, checkpoint, agreement, integration,
  and recorded completion. Identify what can currently cause the work to
  run again or a closure claim to precede the close-out record.
- Establish separate measures for detection, correct recovery decision,
  actual resumption, completion, and developer escalation. Escalation proves
  handoff, not autonomous recovery or task completion.

Hints: inspect autolab's `zulip_listener.py`, `missionspace.py`, and
`workrun_supercoder` guide; pyagag's serving and acceptance records; and
Observer's recovery requests and verification. Reuse existing evidence and
operations rather than building a second task ledger.

## step2 — close the accepted result without repeating its work

- Bind acceptance to the result/checkpoint the requester actually reviewed.
  Make validation, integration, and recording resumable from that evidence.
- Distinguish closing an accepted result from starting another work serving.
  Choose the smallest implementation change that makes this distinction
  effective; another guide-only fix is insufficient without verification.
- When the result is missing or incorrect, repair the specific deficiency
  and show changes requiring renewed acceptance. Do not repeat completed
  generation, commands, or waits merely to reconstruct a report.
- An interrupted or repeated close-out must recognize completed operations
  and continue from them. Keep existing overlap/conflict review behavior;
  an old acceptance does not authorize a changed combined result.
- Report task completion from the resulting record, not from the fact that
  an agreement was received. Preserve mission-level acceptance separately.

## step3 — raise Zulip's limit and remove the client-side fixed cap

- Verify the deployment's effective `MAX_MESSAGE_LENGTH` and persistent
  configuration source. At planning time the server default was 10,000
  characters with no override; `pyagag/src/agag/post.py` independently had
  `MAX_CONTENT = 10000` and cut the body before its metadata line.
- Start by setting a 100,000-character server limit through the deployment's
  persistent configuration. Verify it survives container recreation; do not
  rely on editing a generated file inside the running container.
- Make posting obtain and cache the server-advertised `max_message_length`
  (available through Zulip's registration API), and pass that value through
  composition. Reuse existing registration/state handling where practical.
  Define refresh and unavailable-setting behavior without a new call per post.
- Audit callers, reply-size guidance, and other hardcoded posting caps.
  Include metadata in the size budget. Do not indiscriminately remove
  deliberate context-window limits used when agents read conversation history.
- Test server storage, mirror ingestion, and the receiving agent's access
  to the complete message. Ensure a shortened context view gives a usable
  path to the omitted content and does not hide the message's state metadata.
- Define an explicit outcome for content beyond the new limit: retain the
  full output and expose the delivery problem rather than marking a cut
  result as completely delivered. A small existing file/link path or a clear
  repairable refusal is sufficient; multipart delivery is deferred unless
  concrete trial evidence makes it necessary.

References: Zulip's `zproject/default_settings.py` defines the setting;
`/etc/zulip/settings.py` overrides defaults. Confirm the Docker deployment's
configuration mechanism. API: https://zulip.com/api/register-queue .

## step4 — make recovery decisions verifiable and repeat the path

- Give Front a concise way to re-check the exact affected conversation,
  serving/ack, and execution, using existing trace and health tools where
  possible. Own acknowledgments and activity in another conversation are
  not evidence that the stopped work resumed.
- Keep recovery requests consistent with confirmed health facts; re-check
  stale evidence before acting. If work has already resumed, recognize it
  rather than starting competing work. Preserve bounded developer escalation
  when the state or the correct response remains uncertain.
- Run at least three small end-to-end trials covering a confirmed stop,
  distracting requester/Front activity, and a repeat notification or restart.
  Measure detection, decision, resumption, and completion separately.
- Report every intervention. A developer response after escalation is a
  successful escalation and an assisted recovery, not an autonomous success.
  Fix failures and rerun the affected scenario; retain the failed evidence.

## step5 — correct known inaccurate records

- Inspect m9349 and m9697 task 1 against their completion and integration
  history. Correct the later erroneous `cancelled` state without erasing
  that history or changing the missions' legitimate cancellation outcomes.
- Use a narrow correction operation or append an explicit correction record;
  select the mechanism that existing readers can interpret consistently.
  Handle archived channels without waking workers or repeating integration.
- Verify task status, plan/status views, trace, and Observer after correction
  and restart. The correction must not reopen finished work or masquerade
  as new requester acceptance.
- Leave pending human choices explicit: o11711/m11741, o8512/m8519,
  m6113/m7601/m7732 acceptances, and developer reviews. Correcting facts
  does not decide these requests on the human's behalf.

## step6 — validate, consolidate, and report

Minimize implementation-test waiting without weakening the proof. Use short
counted operations, injected clocks, transcript replay, and accelerated
intervals for repeated checks. Keep at least one real silent-exit trial on
operational timing to verify exit → Front within 300 s. Record which results
are accelerated; do not wait through a whole timeout after success is known.

| Check | Required result |
|---|---|
| Agreement and interrupted close-out | Accepted work executes once; close-out resumes without repeating it; status reflects the committed outcome. |
| Missing/misplaced result | Only the deficiency is repaired; changed results receive the required review. |
| Reply above 10,000 and below the new limit, with code and trailing metadata | Full content is stored and accessible to the recipient; no poster/server truncation. |
| New size boundary and limit discovery/refresh | Budget includes metadata; over-limit delivery has an explicit recoverable outcome; no false complete receipt. |
| Multiple recovery trials | Correct task resumes and completes without developer rescue; no competing execution or duplicate acceptance. |
| Evidence changes before recovery action | Already resumed work is recognized; stale stop evidence does not launch another run. |
| Correction of archived task records | Completed tasks read completed, mission cancellation remains valid, and no work is restarted. |

Run focused regression tests and the affected consumer suites. Roll out
needed dependency pins, update guides and development documentation, and
retire disposable trials and active fault/timing overrides. Keep host details
in ignored notes. Commit and push changes in their owning repositories.

Write `report.md` with changes, trial evidence, timings, execution counts,
delivery integrity, false interventions, human assistance, costs, corrected
records, and remaining limitations. P4 is complete when the checks above
pass, repeated autonomous recovery is demonstrated, and the five-minute
detection target remains met. Do not claim universal reliability from a
small trial set or expand the phase to every execution environment.
