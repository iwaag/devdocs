# Workflow editor p5/ex1 — notification reliability and SQLite contention

## Goal

Close the two reliability findings in the [p5 final report](../report.md): notification duplication when an uncertain delivery grows into a new batch, and intermittent SQLite locking during store initialization. Also correct the run header's backend display using existing attempt evidence.

The completed `improve-game/run-002` remains valid normal-path evidence. This follow-up requires isolated failure-path verification and a system rollout; it does not require another game-development run or another human approval of the game.

## Scope and ownership

- `pj-agdev/agdevworld/workflow_editor`: durable notification delivery state, CLI contract, database initialization, run-header display, and corresponding tests.
- `pj-agdev/agautolab`: notifier transport, restart recovery, and integration tests.
- System deployment through agdevworld's existing release lifecycle.

Devdocs remains the work-record authority. Notification delivery and database coordination remain local operational state. Preserve the separation between agdevworld system source and managed project repositories.

Result-acceptance UX needs a separate design: human-check answers, execution completion, and explicit result decisions currently mean different things. This follow-up must not infer acceptance or change recorded answers or decisions. Migrating `build-game`, repairing unrelated old browser checks, game browser verification, and archiving the rehearsal destination are deferred, with their status retained in the final report.

## 1. Establish the baseline

Inspect repository changes, applicable instructions, installed service/CLI versions, notifier configuration, and current execution activity before implementation. Preserve unrelated work.

Relevant starting points:

- `workflow_editor/server/notify.ts`: `due()` currently reconstructs adjacent pending batches; `Delivery` carries event IDs, rendered content, and a marker.
- `agautolab/src/agautolab/wfnotify.py`: reconciliation compares the complete marker; restart recovery also uses a local `inflight.json` file.
- Both `NotifyStore` and `ExecStore` initialize WAL before setting `busy_timeout`.
- `workflow_editor/src/runView.ts`: the header reads the recorded executor backend while attempt rows can contain the actual backend.

Reproduce the changing-batch defect in a temporary area. Capture the SQLite failure's exact operation and error code when reproducible; distinguish observed behavior from suspected causes. A single successful rerun does not close the locking finding.

## 2. Persist a stable delivery before sending

Keep stable event IDs in the outbox. Add a durable delivery record that groups the events selected for one message. Once exposed to a sender, its membership, order, marker, rendered content, and document-link evidence are immutable. New events wait for a subsequent delivery.

Persist at least:

- A delivery ID and run identity.
- Ordered event membership, with an event belonging to at most one active delivery.
- Exact content and reconciliation marker, including the documents described as unpublished.
- Destination identity and any explicitly recorded relocation.
- Delivery state, attempt/claim identity, retry timing, uncertainty, and acknowledged Zulip message ID.

Create and claim a delivery atomically using short SQLite transactions. Do not hold a database transaction during document reads, Git operations, or network calls; prepare content first and revalidate selection when committing it. At most one active delivery may advance a given run. Preserve existing batching limits, critical-event separation, publication waiting, and ordering for deliveries not yet frozen.

Expose an explicit claim/begin-send transition before the network request, so a crash after posting cannot leave a delivery looking safely unattempted. On restart, an interrupted send requires reconciliation. A second notifier must not send the same claim concurrently; enforce exclusive sender ownership with crash recovery, or an equivalent tested claim mechanism. An expired claim alone is not proof that an earlier network request has stopped.

Update the CLI and Python transport together. Acknowledgment and failure commands identify the delivery and its claim, validate membership, and reject stale or unrelated updates. Acknowledgment atomically marks the delivery and all its events sent and updates the run binding. Repeating the same acknowledgment is harmless. Failure and retry retain the same delivery identity and payload.

Make the durable store authoritative for send uncertainty. Retire or reconcile `inflight.json` explicitly so a single transport file cannot silently lose another run's pending uncertainty.

## 3. Reconcile before retrying

For an uncertain delivery, search for its original marker using the recorded sender and destination. A matching post acknowledges the existing delivery without another POST. If the earlier request is confirmed absent, retry the exact frozen payload; later events are sent afterward.

Preserve topic rename/move handling. Record relocation explicitly and retain enough destination history to reconcile an earlier send at its original location. A reset or inaccessible destination must not silently turn an uncertain delivery into a fresh send.

The current last-100-message search is insufficient to prove absence. Search or paginate across the relevant history; if the available API, permissions, or history cannot establish whether a post exists, retain uncertainty and expose an actionable problem. Network/search failures must not be interpreted as absence. Document the limits: deletion of a posted message and external service behavior prevent an unconditional exactly-once guarantee.

Frozen content may predate later publication or run progress. Keep the existing follow-up publication mechanism so newly available documents are announced separately; do not rewrite an already attempted message to include newer links or summaries.

### Existing state

Use a versioned, transactional migration and test it against a pre-change database fixture. Preserve bindings, sent rows, message IDs, event ordering, and pending work; do not rescan old sent events into new messages.

Unattempted pending rows can enter the new batching path. For legacy uncertain rows, recover exact membership from available inflight evidence and reconcile the old marker before creating new deliveries. If the original batch cannot be established, mark it for explicit reconciliation instead of guessing or silently resending. Account for partial acknowledgments or inconsistent legacy state with diagnostics.

## 4. Make SQLite initialization and contention recoverable

Apply the same initialization discipline to execution and notification stores:

1. Set the connection's busy timeout before lock-taking initialization, including WAL and schema setup.
2. Keep schema creation/migration transactional where supported and safe under concurrent process startup.
3. Close connections after initialization failure and preserve actionable errors identifying the store, operation, and SQLite code without credentials or run content.
4. Where necessary, retry only known transient contention with a bounded deadline and backoff. Propagate non-lock errors immediately.

Keep database transactions short. Retry a database operation or a transaction only when its rollback and idempotency are established. Never wrap agent execution, publication, or a Zulip POST in a generic database retry loop. A failure to persist a successful POST acknowledgment must lead to reconciliation, not immediate reposting.

Verify the notifier remains able to retry after a bounded contention failure and the executor retains its job/attempt fencing. Do not claim that changing PRAGMA order alone proves the problem solved.

## 5. Display the observed backend accurately

Use execution-attempt evidence for the run header: show the active attempt's backend, or the latest ended attempt's backend when idle, with wording that identifies it as the latest attempt. If that attempt lacks a backend, say it is unknown rather than borrowing a different attempt's value. Keep any configured executor backend distinct from the observed backend.

Reuse existing execution API data where possible. Do not rewrite `run.json` or infer the backend from an agent name. Verify completed, active, multiple-backend, and missing-attempt cases in a focused view check.

## 6. Verification

Use temporary registries and databases, a fake Zulip transport, and deterministic fault injection. Tests must not post synthetic messages to ordinary project topics.

| Scenario | Required evidence |
| --- | --- |
| POST succeeds, response is lost, event B arrives after event A | Reconciliation finds A's original marker; exactly one post contains A, and B is delivered afterward |
| Restart between POST and local acknowledgment | Frozen delivery survives; restart acknowledges the existing message without reposting |
| Restart after claim but before POST | Reconciliation and retry complete safely using the same delivery |
| Acknowledgment cannot acquire the database lock | Delivery remains recoverable; no direct repeat of POST |
| New events/publication arrive during retry | Original membership/content stay fixed; later events and publication follow-ups remain ordered |
| Two notifier processes overlap | Only the owner sends; crash recovery does not authorize concurrent POSTs |
| Marker is older than the latest 100 topic messages | Reconciliation still finds it, or reports unresolved uncertainty without reposting |
| Topic moves, search fails, or access disappears | No false absence; destination and uncertainty remain visible and recoverable |
| Legacy database migration | Sent evidence and bindings survive; pending work resumes; ambiguous old uncertainty is surfaced |
| Concurrent first-open and repeated-open of both stores | Independent processes initialize/read/write without corruption or lost rows |
| Controlled short and prolonged write locks | Short contention recovers; deadline expiry is bounded and diagnostic; a later attempt succeeds |
| Backend display | Active/latest-attempt value and unknown fallback match recorded evidence |

Use barriers and deliberate lock holders for concurrency tests rather than timing-only sleeps. Include multiple independent processes; same-process calls do not establish startup safety. Retain the original long-outage scenario and assert no lost events, duplicate posts, or unintended repeated work. Record the stress repetition count and results without presenting finite tests as proof against all races.

Run editor `npm run check`, the relevant notification/execution/browser checks, and autolab `uv run pytest -q tests/test_wfnotify.py` plus affected executor tests. Run the updated focused contention tests repeatedly as a deliberate reliability check. Report unrelated baseline failures separately; new failures in affected behavior block completion.

## 7. Rollout and completion

1. Record tested commits for both repositories and the migration/transport compatibility contract. Document the required deployment order so mismatched CLI and notifier versions cannot send.
2. Inspect live execution activity and pending/uncertain notifications. Use an idle execution window, quiesce notification senders, and back up operational databases consistently, including SQLite WAL state, together with legacy inflight evidence.
3. Deploy through the existing agdevworld release script and update/restart the notifier in the documented order. Do not bypass the release script's open-attempt protection.
4. Verify service and CLI versions agree, migrations preserve existing records, and the notifier resumes without replaying sent events. Inspect the completed RTS run and backend header read-only.
5. Verify rollback against the migrated schema before rollout. Prefer code rollback compatible with retained delivery state; restoring an old database after new posts can replay messages and is not an automatic recovery procedure.

Failure-path acceptance comes from isolated deterministic tests. No new real RTS run or live Zulip test post is required. Observe normal delivery when naturally available; if no events occur during rollout, state that limitation rather than inventing traffic.

Write `report.md` here with implemented changes, commit/release versions, migration and rollback evidence, the defect reproductions and passing regressions, bounded-contention behavior, and remaining limitations. Link this follow-up from the parent final report, preserving its original observations.

This follow-up is complete when the growing-batch duplicate case and crash/acknowledgment variants pass, both stores pass controlled contention and recovery checks, backend display matches attempt evidence, and the coordinated system rollout preserves existing run and notification records. Result-acceptance UX and other deferred product work remain explicit follow-ups.
