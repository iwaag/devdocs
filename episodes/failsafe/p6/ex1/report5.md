# failsafe p6 ex1 — step 5: validation, deployment

## The required-results table, on the final implementation (pyagag `a42c711`)

| scenario | required result | tests | live |
|---|---|---|---|
| Same citation below/at/above 200 posts, or truncated history | No accidental adoption; missing evidence is explicit | `test_a_citation_adopts_nothing_below_at_or_above…` (199, 200, 201, 450); `…legacy_note_nothing_classifies_is_unknown…` (199–201: unknown, listed, with its command); `…truncated_read_neither_adopts_a_citation_nor_drops_a_real_task` | ✓ the migrated realm reads **0 unknown** relations over 136 requests; the p6 citation #15357 reads `reference`, and m8519's acceptance holders are its real requester again (report4) |
| Delegation, citation with a reply, deliberate adoption | Correct work owner, callback destination and completion scope for each | `test_delegation_citation_and_adoption_each_have_their_owner_callback_and_scope`, `…receipt_decision_climbs_only_work_relations`, `…task_its_owner_started_names_its_real_requester…` | ✓ **L1** delegation (`rel=work` notes, mission under the desk, acceptance recorded `by 9 (Omni Agent)`); ✓ **L2** comment: `send` recorded `rel=reference` by itself (#15549), the citing desk lists "cites …", the notes conversation stays its own tree; the reply #15553 was served **into the citing desk** (#15555) with receipt #15556. Adoption: tests only (p6's live `agrun adopt` path unchanged) |
| Rename, resolve, archive, reused topic name | Relation still identifies the intended request | `test_rename_resolve_and_a_reused_name_keep_the_relation_on_its_request` | ✓ L1's task and plan were ✔'d and stayed under the request. ✓ **L3**: autolab archived `work-m15587` (the account permitted to), and the task in the archived channel is still traced under its mission and request |
| Monitoring suppressed on unfinished work | Panel explains; work open; Observer follows its scope | `test_suppressed_monitoring_keeps_the_work_open…`, `…suppression_on_one_unit_leaves_the_rest…`; agobserver `test_suppressed_monitoring_leaves_tracking_until_the_request_moves_again` | ✓ L1: Front recorded `suppressed` (#15513) while the result waited. The card stayed **active** ("waiting for Front's agreement; monitoring suppressed by Omni Agent … nobody chases it while it waits"), every unit carried the suppression, and `tracked.json` stayed `{}` |
| Completed, cancelled or withdrawn request | All readers agree on the outcome and the remaining child obligations | `test_an_ended_request_reads_its_actual_outcome…` (all three kinds); agobserver `test_an_ended_request_leaves_tracking_and_raises_nothing…` | ✓ completed: o11450 (remaining a11459 and r11466 listed) and o11522 (step 4). ✓ withdrawn: o14251's stray unit and **L2** (#15559). ✓ cancelled: **L3** (#15606, by the Developer on their own #15580). Trace, card and Observer agreed in every case, and withdrawn/cancelled were never shown completed |
| New substantive activity after a disposition | New obligations visible and monitored | `test_new_substantive_activity_is_not_covered_but_bookkeeping_changes_nothing`, `…new_result_under_a_suppression_is_monitored_again`, `…recorder_s_own_reply…is_not_new_activity` | ✓ L1: the Omni Agent's review #15518 uncovered the request conversation only; the untouched units stayed covered until Front and autolab posted into them. ✓ L2: a new question after the withdrawal made the request active again, and its new obligation showed ("#15564 asks Omni Agent"). ✓ Front's own reports after recording (#15514, #15560, #15607) did not undo anything |
| Repeated operation or restart during recording | Same final disposition, no duplicate, no lost decision | `test_repeats_reversals_and_restarts_converge`, `…recorder_s_own_reply…` (a repeat converges) | ✓ **Restart during L1's suppression** (04:54:55Z): the trace was identical line for line, the card identical apart from the requester fix deployed in the same restart, and `tracked.json` stayed `{}`. ✓ **Final restart** (05:37:25Z) over the seven cases of this phase (o11450, o11522, o14251, o8512, L1, L2, L3): 88 lines of trace, cards and Observer state, identical after 150 s |

## Live trials

All were posted by the Omni Agent standing in for the Developer, except
where noted. The conversations are left open as the record.

| trial | request | what happened | cost |
|---|---|---|---|
| settlement (step 4) | #15455 | three legacy dispositions and one `agrun finish` | Front $0.57 |
| **L1** delegation → suppressed → restart → new activity | `front-desk-20260928-p6ex1-live` #15473 | Front asked for confirmation (#15475), and the stand-in gave it (#15478). autolab planned m15485 and Front started it. The result #15507 arrived; Front recorded `suppressed` (#15513) and reported (#15514). **Restart.** Review #15518 → Front agreed (#15521) → integrated `72bbd3b` → accepted (#15531) → done | — |
| **L2** comment with reply → withdrawn → new activity | `front-desk-20260928-p6ex1-comment` #15542 | Confirmation #15544/#15547. Comment #15550 in the Omni Agent's own `pj-robustp1 › p6ex1-notes` (`rel=reference`). Reply #15553 served home. Withdrawn #15559. New question #15562 → active → closed #15566 | — |
| **L3** archived source → cancelled | `front-desk-20260928-p6ex1-archive` #15569 | Front refused the stand-in's confirmation twice (#15572, #15576). **The Developer confirmed in person (#15580).** autolab planned m15587 without starting it, then retired it on request: the task was cancelled and `work-m15587` archived. Front recorded `cancelled` by the Developer (#15606) | — |

L1 to L3 together cost Front $3.03 over 26 runs, memo renders included, and
autolab $0.78. Observer made no stall report during any trial.

## Regressions

| suite | result |
|---|---|
| pyagag | 1108 passed |
| agobserver | 177 passed |
| agentroom (relay) | 363 passed |
| agfront | 193 passed |
| agautolab | 331 passed (five exact-string expectations now `rel=work`) |
| agforge | 265 passed (two exact-string expectations now `rel=work`) |
| archsage | 39 passed |
| cagent | 204 passed |

p6's checks are retained unchanged:

- unreceived results;
- result-bound acceptance;
- receipt recovery;
- holds;
- the listener's archived-source timing test.

## Fixes found by the live trials

- **The actor owed an agreement** (pyagag `559f206`).
  - L1's card said "waiting for **autolab-agstudio1**'s agreement". autolab
    had started the task itself, so the task's only root note was its own.
  - Trace nodes now carry the requesters the trace checks receipts against,
    including the parent hop (`Node.requesters`). The card said "Front's"
    after the restart.
- **A closed plain exchange read `resolved_live`** (pyagag `a42c711`).
  - At 02:50:37Z, between the deployment and the settlement, Observer opened
    `resolved_live` on o14251's archsage refresh topic.
  - That topic had been answered, taken up and ✔'d, and retirement had
    hidden it until then. Observer's own judgment dismissed it two minutes
    later (#15472).
  - `stall_candidates` now treats such an exchange as closed, as Observer's
    `satisfied()` already did.

## Deployment

- pyagag `a42c711` is locked, synced and committed in agfront, agautolab,
  agforge, agobserver, the relay, archsage and cagent.
- Everything was kickstarted at 05:51:31Z:
  - every listener;
  - autolab's gateway;
  - forge's service;
  - cagent-api and cagent-zulip;
  - the relay.

  Startup recovery queued nothing new. `nctl drift` converged=46.
- **Deliberate exclusions**:
  - comfynotify keeps its old pin. It writes no root notes and reads no
    relations or dispositions.
  - **agautolab1 (VM) was checked explicitly** and not redeployed. It runs
    only `autolab-gateway` (agautolab `b38a6af`, pyagag `0d26145`), an HTTP
    window with no Zulip listener. It has no serving home, so it writes no
    root notes and reads none of the new records.
  - If a listener is ever placed on agautolab1, it must be redeployed
    first. Otherwise its notes would read unknown, which is visible, not
    silent.
- **Faults**: none armed on any instance (`faults/` empty). p6's
  `exit-before-receipt` hook remains, and only a person creates it.
- **Obsolete paths removed**:
  - `retired.json`;
  - `agobserver.hold --retire/--unretire`, which now exit and name the
    replacement;
  - the relay's `retired` keys;
  - `trace._reference`.

## Found, not changed

- **Front's harness memory** `feedback_confirm_before_contacting_agents`
  (2026-08-20, in Front's Claude Code project memory, outside the repos).
  - It requires the Developer's own go-ahead before Front contacts another
    agent.
  - Front applied it inconsistently: it took the stand-in's confirmation in
    L1 and L2, and refused it in L3.
  - The Developer's in-person confirmation resolved L3. Whether to keep the
    rule is the Developer's call, so it was left as it is.
- **#15576 addressed its response request to Front itself** (`to=15`), not
  to the Developer. It is harmless here, because the Developer read the
  conversation. The earlier #15572 addressed the Omni Agent. This is the
  model's slip, and no tool checks `to=` against the person being asked.
- **One Front reply had to be recovered.** #15475 came from the listener's
  repair run ("reply unusable … no `<ag-reply>` block"), p3's mechanism
  working as designed.
