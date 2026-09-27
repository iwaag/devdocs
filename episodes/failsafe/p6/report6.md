# failsafe p6 — step 6: recurrence handling validated and deployed

## The required-results table, on the final implementation (pyagag `7690f13`)

| scenario | required result | tests | live |
|---|---|---|---|
| Producer finishes while requester unavailable | result remains awaiting receipt; completion alone does not hide it | `test_a_producer_s_completion_alone_leaves_its_unreceived_result_owed` (trace, card "answer #… is not yet taken up by Front", `undelivered` candidate); agobserver `test_an_unreceived_answer_is_owed_until_a_decision_settles_it` | — |
| Relevant result validly accepted but receipt missing | work reads finished consistently; missing bookkeeping repairable | `…valid_acceptance_after_the_result_settles_it…`, `…mission_s_acceptance_alone_settles…`, `…same_request_reads_the_same_from_the_mission_and_from_the_request`, `…reconciled_on_its_decision_and_repair_converges` | ✓ the nine legacy answers: every reader said completed after step 2's rule; Front repaired all nine with `agentchat receipt --repair` in one serving (report5) |
| Crash after reply delivery, before receipt | restart completes the receipt without repeating work or the report | `test_the_exit_before_receipt_fault_is_one_shot_and_the_restart_finishes_the_receipt`, `test_a_restart_between_the_home_reply_and_the_served_mark…` (existing), `test_journal_evidence_writes_the_mark_the_listener_would_have_written` | ✓ trial below: fault at 20:32:50Z, receipt #15438 at 20:32:54Z, no rerun, one report |
| Duplicate repair, rename/resolve, or archived source | repair reaches the same answer and home and converges | `…repair_converges` (repeat writes nothing), `test_a_renamed_or_resolved_home_and_an_answer_not_naming_you`, listener `…source_channel_was_archived_meanwhile`, `…follows_a_home_resolved_meanwhile` | ✓ #8557's source `work-m8519` is archived: its receipt went to the ✔ home (#15379) |
| New answer during repair; unrelated acceptance exists | neither is incorrectly consumed by an older decision or receipt | `test_an_answer_arriving_after_the_repaired_one_stays_owed`, `…a_result_after_the_acceptance_it_did_not_see_stays_owed`, `…another_request_s_acceptance_settles_nothing_here`, `…a_mark_is_never_written_over_an_earlier_answer_nobody_was_given`, `…reconciled_receipt_covers_exactly_the_answer_it_names` | — |
| Human hold is settled, retained intentionally, or followed by a new request | only the covered decision is released; intentional holds and new obligations remain visible | `…hold_on_acceptance…settles_with_the_acceptance`, `…hold_on_a_resume_settles_when_the_work_is_served_again_not_on_unrelated_activity`, `…intentional_hold_stays_until_its_holder_releases_it…`, `…hold_covers_only_its_work_and_new_obligations_stay_visible`; agobserver `test_a_request_a_person_holds_is_traced_but_never_acted_on` (record, then explicit release) | ✓ o11711's hold record #15378 reads SETTLED by #15283 on every reader |
| Cleanup references another request | readers preserve actual ownership and do not manufacture new unfinished work | `test_citing_another_request_during_cleanup_adopts_nothing`, `…opened_for_the_work_is_still_adopted`, `…deliberate_move_adopts…`; realm replay (133 requests) | ✓ o11711 no longer holds m8519 (#15357 is a citation) |

The "followed by a new request" case is structural. Holds live in the
request's own conversation, so another request is another origin and no
hold covers it. The test pins the harder same-request version: a new plan's
owed answer stays an `undelivered` candidate beside a hold.

## Live trial on the final code

The request was `#front › front-desk-20260928-p6-live` (#15392), as the
Omni Agent, standing in for the Developer: one task in pj-robustp1. Front
asked for confirmation first (#15394), and the stand-in gave it (#15397).

| time (Z) | event |
|---|---|
| 20:31:01 | Front opens `workplan-p6-live-trial` |
| 20:31:25–31 | autolab plans m15405, opens and starts task 1 |
| 20:31:53 | Front serves the plan answer; receipt `[served] … 15415` (ordinary) |
| 20:31:53 | Front serves the task's shown result #15420 (mention route) |
| 20:32:12 | **fault armed**: `agfront/.local/faults/exit-before-receipt` |
| 20:32:29 | Front agrees in the task (#15426) |
| 20:32:47–48 | autolab records `[change] accepted … +shown=15420`, integrates `fe69bdd` (pushed), `completed`, close-out #15434, ✔ |
| 20:32:49 | Front's reply #15436 delivered home |
| 20:32:50 | **fault**: "exiting after the delivery of mention work-m15405/workrun-task1-m15405, before its receipt"; launchd restarts Front in the same second |
| 20:32:52 | startup recovery: 1 pending entry |
| 20:32:54 | `marked work-m15405/workrun-task1-m15405 served up to 15420` (#15438). No rerun, no second report |
| 20:32:54–33:21 | the close-out #15434 is served normally; Front records the mission's acceptance (`[acceptance] #15426 by 15 (Front) after=#15420`, `done`, ✔) and reports (#15445); receipt #15447 |

Afterwards:

- Front's own trace, the panel and Observer all say **completed**, with no
  missing receipt.
- Observer tracked nothing and posted nothing during either trial.
- Front used two refusals as designed:
  - `accept` before the task closed (#15428);
  - the initial request as evidence older than the mission (#15440).

  It then recorded its own entrusted agreement.

**Restart comparison.** Front, Observer, autolab and the relay were
kickstarted at 20:36:08Z, and 150 s later:

- `agentchat trace 15392` was identical line for line;
- the card was `completed` with the same unit states;
- `tracked.json` was still `{}`;
- startup recovery queued 0 conversations;
- the legacy requests stayed settled: 0 AWAITING_DELIVERY / "no receipt"
  lines on o11711, o8512, o8816, o9010, o8721 and o9227, with no nudge.

**Archiving.** Autolab archives a work channel only when the mission is
cancelled. m8519's channel was archived by the Developer. The Omni Agent's
account may not archive (`HTTP 400: You do not have permission`), so the
trial's `work-m15405` stays unarchived. The archived-source path rests on:

- the listener test;
- step 5's live repair of #8557 in the archived `work-m8519`.

Cost of the trial: Front $1.06 (five desk servings, two memo
presentations), autolab $0.31.

## Regressions

| suite | result |
|---|---|
| pyagag | 1079 passed |
| agobserver | 176 passed (on the new source) |
| agentroom (relay) | 363 passed |
| agfront | 193 passed |

The others (agautolab, agforge, archsage, cagent) have no code change.
They use the listener and trace through pyagag, and pyagag's listener and
lifecycle tests run the real listener.

**Realm replay** (all 133 `#front` requests off the step-1 mirror copy):

- the 10 intended changes;
- three old requests now read as the panel always had, with their real
  unaccepted answers (6566, 3161, 7596);
- two citations dropped (2458, 1550);
- the "every answer above the mark" change altered nothing.

## Deployment

- pyagag `7690f13` pinned and synced in agfront, agautolab, agforge,
  agobserver, archsage, cagent and the relay.
- Locks committed and pushed in every repo, with the submodule pointers in
  pj-agdev (`ab0d251`).
- Every listener, the gateway, forge's service, cagent-api/-zulip and the
  relay were kickstarted at 20:19:55Z on the final pin, with no serving in
  flight.
- `nctl drift` converged=46.
- comfynotify keeps its old pin (it parses no receipts or holds).
- agautolab1 (VM) was not redeployed.

## Faults and guidance removed

- The only trial fault (`exit-before-receipt`) was consumed by its one
  use. Nothing is armed.
- The fault's hook stays in the listener, like autolab's `silent-exit`.
  Only a person creates the file.
- Obsolete guidance was replaced:
  - README_DEV's `agobserver.hold` lines (hold/release of `held.json`)
    point at the new section;
  - `--release` of a retirement is now `--unretire`;
  - the host notes no longer describe `receipts_from`;
  - `held.json` is documented as unread.
