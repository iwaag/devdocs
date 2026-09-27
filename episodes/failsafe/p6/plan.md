# Failsafe p6 — reconcile completion, receipts, and human holds

## Goal and scope

Make finished requests stop appearing as `waiting` or `need you`, while still
detecting genuinely unreceived results and unanswered questions. Front Room,
Observer, and `agentchat trace` should explain the same obligations from the
same evidence. Agents should be able to repair missing receipts and settle
holds through ordinary tools.

Read `../p5/report.md` and `../../progress_panel/p1/report.md`. This is a private
experimental, breaking-change phase: backward compatibility is unnecessary.
Implementers choose schemas, commands, and refactoring scope. Replace obsolete
paths as needed; prefer existing journals and conversation records over a new
ledger. Add only the checks needed to associate decisions with the right work.

## step1 — reproduce and locate the inconsistencies

- Check the running deployment through Nautobot or `nctl status` / `nctl drift`
  and the ignored environment notes; see `pj-clusterintent/nctl/README.md`.
  Capture a small replay fixture before repairing the affected records.
- Use m8519/o8512: both tasks are accepted and the mission is done, but task 1
  answer #8557 lacks a recognized Front receipt. Tracing the mission says done;
  tracing its Front request says awaiting delivery. Resolving the topics did
  not clear the progress card.
- Use m11741/o11711: research was accepted and integrated, and sage refresh
  #15337 includes the accepted revision, yet an older manual hold survived
  until explicitly released. Later cleanup from its desk linked m8519 beneath
  it, propagating that request's wait. Recheck current relations and distinguish
  work dependencies from references made while discussing cleanup.
- Inspect the other resolved legacy waits, including o8816, o9010, o8721, and
  o9227. The last includes cancelled m9349; preserve that disposition.
- Follow result publication, acceptance, serving input, reply delivery,
  receipt writing, resolution, and archiving. Establish where evidence is lost
  or misinterpreted. The historical resolve/archive race is a hypothesis;
  Front's unsuccessful repair attempt does not prove its cause.

Useful starting points:

- `pyagag/src/agag/trace.py`: `classify` checks for an unserved answer even
  inside `DONE_WORDS`, which includes `accepted`. This deliberately catches a
  producer marking an undelivered result `delivered`; retain that capability.
- `agag/{progress,acceptance}.py`: stage completion and overall request state
  can currently disagree. Acceptance already identifies the decision holder
  and the shown result.
- `agag/listen.py`: `_receipts_pending`, `_mark_inputs`, `_mark_owed`, and
  `_after_delivery` already support journaled receipt recovery. Find the gap
  before adding another retry mechanism.
- `agag/{zulip,selfnote}.py`: `note_served`, `mark_served`, and `served_note`
  write into the requester's home conversation. An archived source channel
  alone does not make repair impossible. Front's hand-written marker attempt
  used a different shape; inspect actual parser acceptance.
- `pj-agdev/agobserver/src/agobserver/{hold,monitor}.py`, agfront's
  `zulip_listener.py`, and agentroom's `progress.py` own the other affected paths.

## step2 — share completion and outstanding-obligation semantics

- Distinguish execution completion, requester acceptance, result receipt, and
  open questions in the shared interpretation. Derive the actionable state
  from these facts rather than treating every missing receipt as unfinished
  work. Choose the representation; avoid a UI-only exception.
- A valid acceptance covering the relevant result settles that result's
  review obligation. A missing transport receipt may remain diagnostic, but
  should not demand redundant acceptance or execution. A producer's completion
  claim alone still leaves an unreceived result outstanding.
- Bind settlement to the result and decision it actually covers. Later results,
  changed work, new questions, and another request's replies remain independent.
  A resolved topic by itself is not evidence that those obligations were met.
- Apply the same interpretation to trace, Observer, progress stages, and card
  aggregation. Explain any remaining wait with its specific outstanding item.
  Review how cross-conversation links propagate obligations: citing a request
  during cleanup should not manufacture ownership of its work.

## step3 — complete receipt recovery and give agents a repair operation

- Make ordinary serving and restart recovery finish missing receipts from the
  input actually processed and the reply confirmed delivered. Cover owner and
  mention routes, renamed/resolved homes, and archived source channels.
- Expose a small receipt inspection/repair operation through existing agent
  tooling. It should identify the source answer, receiving agent, destination,
  and evidence that the answer was handled. Reuse journal evidence where
  available; provide an explicit recorded reconciliation for historical cases
  where it is absent. Do not claim an old serving happened without evidence.
- Use shared note writers/parsers, stable message references, and bounded
  receipt coverage. Repeating or interrupting the repair must converge without
  another worker run, duplicate report, or consuming an answer that arrived
  after the processed input.
- Update Front's tool help and guidance so it can diagnose and repair this
  condition without guessing selfnote syntax or asking for repeated approval.

## step4 — give human holds a complete lifecycle

- Identify what decision or recovery action each hold protects, who owns it,
  and what evidence settles it. Keep the reason and release history inspectable.
- Provide ordinary hold inspection/release tools and connect release to the
  authorized resume, acceptance, cancellation, or completion path when that
  action settles the hold's stated purpose. Unrelated activity is insufficient.
- Distinguish an intentional indefinite hold from a decision already settled.
  If settlement cannot be established, explain the remaining decision rather
  than silently releasing it or leaving an unexplained `need you` indefinitely.
- Make retries/restarts preserve the outcome. Front, Observer, and the panel
  should agree without direct JSON edits by the developer.

## step5 — reconcile the existing requests

- Re-evaluate the step1 cases using the new interpretation and tools. Repair
  missing receipts and obsolete holds from their actual evidence, then close
  completed conversations through the normal completion path.
- Confirm m8519 remains accepted, m11741 retains its integrated research and
  bound sage refresh, and cancelled trial missions remain cancelled. Identify
  any genuinely unfinished repository work before calling cleanup complete.
- Verify the original requests and requests linked during cleanup. Existing
  false waits should disappear from active work or appear as explicit settled
  history; real outstanding items should retain their owner and next action.
- Record each correction and its evidence in the phase report. A one-time
  repair should use the same operation available for future occurrences.

## step6 — validate recurrence handling and deploy

Use transcript replay and focused interruption tests for iteration, followed by
short live trials on the final deployed code. Long research runs are unnecessary.

| Scenario | Required result |
|---|---|
| Producer finishes while requester is unavailable | Result remains awaiting receipt; completion alone does not hide it. |
| Relevant result is validly accepted but receipt is missing | Work reads finished consistently; missing bookkeeping can be repaired. |
| Crash after reply delivery, before receipt | Restart completes the receipt without repeating work or the report. |
| Duplicate repair, rename/resolve, or archived source | Repair reaches the same answer and home and converges. |
| New answer arrives during repair; unrelated acceptance exists | Neither is incorrectly consumed by an older decision or receipt. |
| Human hold is settled, retained intentionally, or followed by a new request | Only the covered decision is released; intentional holds and new obligations remain visible. |
| Cleanup references another request | Readers preserve actual ownership and do not manufacture new unfinished work. |

- Run affected shared-library and consumer regressions. Deploy the required
  dependency pins and restart affected readers/listeners; a library commit
  alone does not update their environments.
- Demonstrate one ordinary completion and one interrupted-receipt recovery
  through Front, including resolution/archiving. Compare trace, Observer, and
  fresh Front Room data before and after a restart. Confirm the legacy cases
  stay settled without developer nudges or false recovery requests.
- Remove trial faults and obsolete guidance. Write `report.md` with adopted
  semantics, proven causes, repaired records, validation, interventions, and
  limitations; update relevant development docs and commit/push owning repos.

Completion requires the table to pass on the final implementation, the legacy
cases to have evidence-backed dispositions, and live readers to agree. Record
developer repairs as interventions, not autonomous agent successes.
