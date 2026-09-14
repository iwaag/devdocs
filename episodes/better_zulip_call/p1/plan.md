# Better Zulip calls p1 — shared reads and continuous event intake

## Goal and approach

Reduce Zulip calls structurally: maintain shared, event-updated conversation indexes instead of reconstructing the same state for each view, operation, and agent. Zulip remains the authoritative conversation and work record; indexes and execution queues are recoverable local state. Reuse the operation-room engine as a starting point, then extend the shared mechanism to listeners and Observer.

This is a breaking redesign in a private experimental environment. Backward compatibility, legacy-state migration, production hardening, and a general-purpose message broker are unnecessary. Replace APIs, modules, and disposable experimental state where useful. Choose storage, process boundaries, naming, and implementation order within each step freely. Keep credentials and machine-specific facts in ignored files. Preserve meaningful conversation behavior, not incidental implementation rules.

## Step 1 — Establish the call baseline and shared boundary

- Inspect running services through Nautobot or `pj-clusterintent/nctl`; use the ignored environment notes to locate deployed packages and credentials.
- Measure calls by credential identity, endpoint, and operation: idle listening, board reload, completion preview/apply, restart, and an active Observer watch. Count actual HTTP requests, including retries, separately from local reads. A lightweight counter/log is sufficient.
- Identify overlapping reads and choose a reusable read-model interface for messages, channel/topic metadata, agent introductions, and conversation relationships. Start from `agentroom/ops.py`; place reusable mechanics where `pyagag` consumers can use them without importing frontend code.
- Prefer one collector for shared public conversations. Keep account-specific intake where visibility or behavior differs; a new authorization platform is not part of this phase.

Verify: record a reproducible baseline and the chosen ownership boundary. Existing figures (~50 calls per board sweep, ~240 per ops startup) are historical observations, not current measurements. Deliberately exhausting the account quota is unnecessary.

## Step 2 — Build an event-updated, recoverable read model

- Maintain messages by ID, their current channel/topic, channel metadata, introductions, and parsed work/reference notes. Derive parent/child and callback indexes from existing conversation records. Names are display/lookup data; reused names must not redirect an anchored request.
- Handle new posts, relevant edits, moves, resolution/reopening, deletion, and channel/subscription changes. Fetch missing history on demand and combine simultaneous requests for the same missing data. Record history coverage so a partial read is not interpreted as a complete conversation.
- Keep ingestion running independently of consumers. Persist enough state and checkpoints to resume ordinary restarts; commit applied events and their checkpoint together or use an equivalent recoverable scheme.
- Bootstrap with queue registration before history collection, then reconcile queued events with the fetched state. Replayed events must not duplicate or regress data. If the queue expires, explicitly resynchronize affected coverage; recent-message fetching alone cannot recover missed edits/deletions.
- Expose freshness and completeness. On disconnect, retain the last useful state and mark it stale. Store format and rebuild strategy are implementation choices; no permanent second work ledger is needed.

Verify with focused fixtures: bootstrap overlap, event replay, edit/delete, move/rename with name reuse, and queue-expiry recovery. Check that a normal consumer restart does not trigger a full realm scan.

## Step 3 — Serve the existing views from shared state

- Move `/agents`, `/work`, `/ops`, and their related detail/routine views onto the common reader. Preserve useful UI behavior while removing independent sweep caches and duplicate state reconstruction.
- Replace completion-triggered `room.forget()` with updates limited to successful actions and affected records. Reconcile write results with arriving events; pending/failed actions should remain distinguishable from confirmed changes.
- Serve warm board reloads locally. Older detail/history requests may hydrate missing coverage once. Display synchronization trouble alongside the last useful data rather than replacing the whole board with channel errors.
- Delete superseded read paths instead of retaining compatibility adapters.

Verify: repeated warm board reloads cause zero additional Zulip requests attributable to those reloads. Background event polling is counted separately. Closing one request does not force a realm-wide refresh, and another browser sees the updated state.

## Step 4 — Use indexed relationships for completion

- Build completion previews from the shared parent/child, state, and location indexes. Hydrate only missing evidence. Share fetched evidence across preview and apply where it is still current.
- Track which records and relationship scopes a plan depends on. Revalidate changed, incomplete, or uncertain dependencies before applying, including the possibility of newly added child work. Initially retaining targeted direct reads before writes is reasonable.
- Keep a preview decision token, but do not treat the existing fingerprint as a freshness check. Event lag remains possible; neither an unchanged local revision nor an unchanged hash proves Zulip has not changed.
- Use successful write results and targeted verification to produce the post-apply result instead of walking the entire graph again. Retrying a partial completion should account for already successful actions.

Verify: preview/apply reuse, a child added after preview, a changed work state, and partial failure/retry. Unrelated channel growth should not increase warm completion reads; calls should follow missing or changed relevant evidence plus necessary writes/checks. Full transactional isolation from concurrent Zulip users is not required.

## Step 5 — Separate listener intake from agent execution

- Refactor `pyagag` listening so long model/tool runs do not stop event polling. Feed a durable pending-work queue from the shared event/index mechanism; retain account-specific DM intake where appropriate.
- Coalesce repeated events for one conversation, evaluate current state when execution starts, and preserve events arriving during execution. Keep owner/mention routing, selfnote handling, served acknowledgements, and callback recovery coherent.
- Recover unfinished dispatch after restart using durable state and existing conversation evidence. Support at-least-once recovery with duplicate suppression where needed; do not promise exactly-once external effects.
- Adopt the mechanism in pj-agdev listeners and pj-clusterintent's cagent. Update dependency pins, generated listener templates, and deployment configuration together. Remove obsolete full-sweep recovery paths once the new recovery covers their responsibilities.

Verify: intake continues during a long handler, a burst coalesces, a later post still receives service, and restart resumes pending work without routinely rerunning completed work. Cover Front's unstarted runs and delivered-report continuation; last-speaker checks alone cannot find these obligations.

## Step 6 — Remove repetitive Observer reads and coordinate call budgets

- Drive Observer's accepted-watch schedule, location, cancellation, and condition changes from the shared model. Keep periodic evaluation of the actual watched resource; it is a different concern from repeatedly reading the Zulip request.
- Replace per-evaluation Zulip lookups with indexed state when current. Use targeted checks before notification where needed for cancellation and destination accuracy. Rebuild lost scheduling state from the conversation index.
- Put call accounting, concurrent-read coalescing, and rate-limit coordination at the shared transport boundary. Coordinate by credential identity across participating consumers, honor server retry guidance, and avoid a wave of retries after failures. A simple scheduler is sufficient.
- Treat separate reader credentials as contention isolation, not call reduction. Distinguish polling, hydration, verification, writes, and retries in the measurements.

Verify: unchanged watches do not repeatedly fetch their Zulip records; cancellation/move before notification still works. Inject a 429 to verify that readers share the pause and views retain useful state. External tools using the same credential may remain outside local budget coordination; document that limitation.

## Step 7 — Deploy, demonstrate, and report

- Check current state through Nautobot/nctl, deploy affected packages, and refresh services and guides. Rebuild disposable indexes if simpler than migrating them. Update launch configuration and repository/submodule pointers as necessary.
- Run relevant backend tests and the frontend build. Use controlled fixtures for failure timing and a small live conversation flow for cross-service behavior: request, delegate, receive a callback, inspect, and complete. Include an Observer cancellation and a restart. Posting the test messages needed for this bounded demonstration is part of executing this plan.
- Repeat the Step 1 scenarios at comparable scale. Report total and per-credential calls, foreground/background breakdown, steady-state/recovery costs, and visible update latency. Verify correctness as well as fewer 429s.
- Remove obsolete code/configuration, update developer and agent guidance plus ignored environment notes, and write `report.md` with evidence and remaining limitations. Commit and push changed repositories and required submodule pointers.

## Implementation hints

- `pj-agdev/agdevworld/agentroom/src/agentroom/room.py`: `_cached()` builds outside its lock, so concurrent misses can duplicate a sweep; `forget()` clears every view cache. `server.py` calls it after completion.
- In the same package, `ops.py` already registers before sweeping and updates state from events. It is not yet a complete shared replica: it truncates/skips history, handles `update_message` primarily as rename, and ignores subscription events. Audit its data coverage and new-channel discovery before reusing it for decisions.
- `closing.py` caches reads only within one `Reader`/discovery and probes both bare and resolved topic names. `close.py` invokes discovery for preview, before apply, and after apply. Its fingerprint hashes action keys/states, not the underlying evidence. Anchor/location indexes can remove repeated name probes while retaining real open/resolved twins.
- `pyagag/src/agag/zulip.py`: `sweep_serve()` already coalesces event hints but rereads topic history before dispatch and runs handlers synchronously. Full sweeps happen on queue registration, not on a periodic timer. `on_sweep`, served marks, rootchat recovery, and Front's continuation hooks identify behavior the replacement must cover.
- `pj-agdev/agobserver/src/agobserver/worker.py`: current code reconciles every ten ticks and uses anchor-message lookups for watch location. Some environment notes describe an older per-tick listing; measure the deployed version. Inspect `destination.py` and `notify.py` for rename, cancellation, and delivery checks.
- `pj-clusterintent/cagent/src/cagent_api/zulip_window.py` has separate DM and topic paths; its `/window` request-status polling is not Zulip traffic.
- Zulip's [queue registration](https://docs.zulip.com/api/register-queue) supports event selection and `all_public_streams`; evaluate it against the deployed server before retaining automatic public-channel subscriptions. Verify the event coverage actually received, especially edits/deletions. See [event formats](https://docs.zulip.com/api/get-events). Queue IDs/checkpoints are not an indefinitely replayable event log.
