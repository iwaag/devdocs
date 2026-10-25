# p17 follow-up — a real session and the room's review list

Investigated on 2026-10-25 at the Developer's request. The implementation
and deployment verification below concern review discovery and display.

## Session inspected

Conversation `20261025-013759-jz8ebz`: waterfall animation with mushroom
motion, ponderer/conduct/run-041 and run-042, delegating to
gimmickimg/freework/run-013.

- The initial feasibility-only request received a plan, with generation
  explicitly described as not yet attempted. The next person message
  specified waterfall animation and MiniMax H3; run-042 delegated the work.
- The child prepared its own human review rv1. The record shows browser
  start by the configured person at 10:39:03 JST, followed by approval at
  10:39:47, with feedback asking to complete the work. This is a recorded
  human decision, not ponderer's own approval presented as the person's.
- Approval completed the review node, but reused the child's already-live
  presentation and left stopping it to the child's separate cleanup node.
  That ready node did not receive an ordinary continuation. Recovery-agent
  resumed the child at 10:44:21, and the page was verified stopped at
  10:44:42. Thus final cleanup succeeded, but automatic completion depended
  on recovery. This continuation/lifecycle gap was addressed afterwards in
  [ex1](ex1/report.md): cleanup by actual ownership and an executor-owned
  continuation of the work an approval makes ready.
- The child completed and integrated; the parent received its result,
  disposed it as satisfied and posted its summary at 10:45:47. Both runs
  are completed, with no current held or failed work in the room status.
- The room overview still describes the earlier feasibility stage and
  flags later messages; it is not an updated final-results summary.

Read-only evidence: conversation API, parent and child run histories,
review decision and cleanup facts, delegation result and integration.
Nautobot/nctl state was checked before service inspection.

## Missing review discovery and the fix

The room's `reviews` was empty, although the child had rv1. The conversation
reader collected only directly linked runs (ponderer's turns). A child's
review can wait before a result exists to relay, so checking only the parent
or completion manifest misses it.

WFE now traverses bound delegation descendants for a room, including already
satisfied delegations, using the recorded child identity. It deduplicates
runs and reports unreadable/mismatched children without substituting another
run. The review list retains completed rounds as well as pending ones.
Other callers of the run reader keep their previous direct-only behavior.

aghangout displays a distinct Human review list above the chat composer in
both reading and playback modes. Pending rounds sort first, with their
subject, status and owning run. A Start review button opens the review page
and submits the existing start operation. Completed, changes-requested and
cancelled rounds retain result links; they are not restarted as undecided
rounds. The configured person remains the actor of actual clicks.

## Validation and deployment

- Regression test covers a child waiting for review before completion,
  deduplication when also directly linked, and retained review history after
  approval. The existing full correction/review flow also passes.
- aghangout build and 31 tests pass. Browser checks cover pending-first
  order, completed history, one click opening the page and sending one start
  request, and a 390px mobile viewport. Synthetic pending review data and
  intercepted writes were used for that interaction test; no human decision
  or chat message was created on the Developer's behalf.
- Releases: WFE `3c8b438`; aghangout `b154fcc`. No record/store schema or
  executor contract changed. Deployment details and verification logs are
  in ignored `pj-agdev/.local/p17/` notes.
- WFE's full check and release check pass: 587/587 tests, TypeScript and
  both builds. The deployed conversation API now returns the child's rv1
  as complete; its review is also reachable through the API from agpc.

The Developer's existing approved waterfall review becomes discoverable from
its chat as a completed result. The next pending child review can be started
there without finding a separate run link.
