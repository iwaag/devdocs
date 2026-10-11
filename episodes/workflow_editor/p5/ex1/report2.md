# p5/ex1 step 2 — a stable delivery persisted before sending

Plan section: [plan.md](plan.md) §2.

Commits: agdevworld `501d0fa` (editor store, CLI, docs), agautolab `47853e1` (transport, tests). Both pushed. They also contain the step 3 reconciliation and migration code, reported in [report3](report3.md). Not deployed yet (step 7).

## What changed

### Editor: `server/notify.ts`

The outbox still holds one row per event, keyed by its stable id, and is still rebuilt from run history. The new delivery tables sit beside it.

| Table | Holds |
| --- | --- |
| `deliveries` | One row per message: ordered event ids, content, marker, documents called unpublished, destination at freeze, `tried` destinations, state, claim and owner, attempts and failures, `next_try`, `uncertain`, `first_began`, the evidence that established absence (`checked`), Zulip message id |
| `delivery_events` | `event` is the primary key, so an event belongs to one delivery, ever |
| `deliveries_active` | Partial unique index on `run_key` for `ready`, `sending`, `failed` and `unresolved` deliveries: at most one delivery in progress per run, enforced by the database |
| `senders` | The single sender: owner, pid, host, since, last claim |
| `relocations`, `notes` | Topic history (step 3) and migration diagnostics |

The lifecycle is `ready` → `sending` → `sent`, or `failed`, which is retried with backoff using the same payload. `unresolved` waits for a person (`wfe notify resolve`). `resolved` closes a legacy reconciliation.

- **Freezing.** `claim()` hands out a run's delivery in progress. If the run has none, it selects the next events with the existing rules: adjacent routine events go together, at most six; critical events, publication notices and published follow-ups go alone; documents wait briefly for publication. It renders that selection while no transaction is open, because rendering reads `run.json`, documents and Git. Then one `BEGIN IMMEDIATE` transaction checks that the run still has no active delivery, that the same event ids are still its next unassigned events, and that the destination is unchanged. Only then does it insert the delivery and its memberships. If any check fails, nothing is written and the next pass tries again. `render()` no longer writes `unpublished` to the outbox as a side effect. The frozen delivery keeps it, and the follow-up is noted beside the delivery in `followed_up`.
- **Begin before the network.** `begin` sets `sending`, counts the attempt, records the destination in `tried`, and sets `first_began`. It refuses an uncertain delivery unless it receives `--checked-absent` evidence. When a claim finds a delivery still `sending`, which means an earlier attempt never recorded a result, it turns the delivery into `failed` + `uncertain` before handing it out. That covers a crash after posting, a crash after `begin` and before posting, and an acknowledgment that could not be written.
- **Results are bound to the claim.** `sent`, `failed` and `begin` take the delivery id, its current claim, and optionally its events. The store refuses unknown deliveries, stale claims, a sender that is no longer the owner, and event lists that differ from the frozen membership. `sent` atomically marks the delivery and all its outbox events as sent, and updates the run's binding. That binding takes the found location and never lowers `last_message_id`. Repeating the same acknowledgment returns "already recorded". A different message id for a sent delivery is refused.
- **One sender.** `claim --sender <owner> --pid <pid>` records the owner. Another owner is refused while the recorded pid is alive on this host. When the owner is on another host, it is refused outright, with `wfe notify sender --release`. There is no lease expiry: an old claim is never taken as proof that its request stopped. When the owner's process is gone, the next sender takes over, and every `sending` delivery becomes uncertain before anything is sent.
- **The store is authoritative for uncertainty.** The transport keeps no state file. The old `inflight.json` is imported once (`wfe notify import-inflight`) into the run's legacy reconciliation and retired by renaming (step 3).
- **Publication follow-ups.** The `published` follow-up marker now names its event (`ref <run> cp<n>:published <commit>`), so no two follow-ups share a marker. Reconciliation compares the whole last line, not a suffix. The old `endswith` check would have matched `… #5:x #6:y` for the marker `… #6:y`.

### CLI and transport contract (protocol 2)

New `wfe notify` subcommands: `protocol`, `claim`, `begin`, `sent <delivery>`, `failed <delivery>`, `legacy`, `resolve`, `import-inflight`, `sender`, `deliveries`. `due` now returns provisioning tasks and problems only. It no longer returns a `deliveries` key, so the old transport (`due.get("deliveries", [])`) finds nothing to send. `sent --id` and `failed --id` are refused with "update agautolab.wfnotify". `agautolab.wfnotify` checks `wfe notify protocol == 2` at start and refuses to run otherwise. Deployment order and rollback are in step 7.

### Transport: `agautolab/wfnotify.py`

The pass is: `due` (provision), then `claim`, then for each delivery:

1. Follow a moved topic.
2. Search if reconciliation is required.
3. Record `begin` (with the absence evidence when it searched).
4. POST.
5. Record `sent`.

A refused POST records `failed` without uncertainty. A lost answer records `failed --uncertain`. If `sent` cannot be recorded after a successful POST, the transport logs the message id, tries `failed --uncertain`, and does not post again. The delivery stays `sending`, so the next claim reconciles it. The sender identity is `<host>:<pid>` of the process. A standing pass failure, such as a protocol mismatch, is logged once rather than every 5 s.

The run view's coordination panel says when a message may already have been posted. `wfe notify status` lists the sender, problems and migration notes.

## Evidence

All tests use temporary areas, the real `wfe` CLI, and the fake Zulip. Nothing is posted to Zulip.

| Scenario (plan §6) | Test | Result |
| --- | --- | --- |
| POST succeeds, response lost, event B arrives after A | `test_growing_batch_is_not_posted_twice`: the step 1 reproduction. `#6:answered` is posted, its answer lost; attempt 2 runs to completion before the retry | `#6:answered` posted once. The existing message is acknowledged with the same id and content. `#13:reported` and `#15:completed` follow in order. Number of POSTs equals the number of messages |
| Restart between POST and acknowledgment | `test_restart_between_post_and_acknowledgment` | A new notifier finds the marker; the delivery is `sent` with the original message id; no repost |
| Restart after claim, before POST | `test_restart_after_claim_before_post` (claimed only) and `test_restart_after_begin_before_post` (begun) | Claimed only: the same delivery is sent once (`attempts = 1`). Begun: it is uncertain, the search establishes absence, and the exact frozen payload goes out under the same delivery (`attempts = 2`, `checked = "absent from #…"`) |
| Acknowledgment cannot get the database lock | `test_acknowledgment_without_the_database_lock`: a second connection holds `BEGIN IMMEDIATE` on `notify.sqlite` from the moment of the POST until after the pass | Both `sent` and `failed --uncertain` time out after 5 s. One POST; the delivery stays `sending`. The next pass finds the message and acknowledges it; no repost |
| Two notifier processes overlap | `test_two_notifiers_only_the_owner_sends`: notifier A's identity is a live child process; A claims and begins, then stalls | B is refused (names A's pid) and posts nothing. Once A's process is killed, B takes over, searches, establishes absence, and sends once. A's late `sent` with its old claim is refused |
| Store-level rules | editor `test/notify.test.ts` | The same delivery and content on retry. `begin` refused without absence evidence. A stale claim's acknowledgment refused. A repeated acknowledgment harmless. The follow-up marker names its own checkpoint. The frozen record keeps its unpublished documents |
| Existing behavior | the rest of `test_wfnotify.py` and `notify.test.ts` | Hierarchy, human wait, restart, lost response, long outage, taken name, renamed topic, links, identity, policies: all pass |

Full runs after the change: editor `npm run check` passes (type check, 129/129 tests, build). autolab `pytest tests/test_wfnotify.py tests/test_wfexec.py` 16/16.

## Limits

- Owner liveness is a pid check on this host. A reused pid makes a dead owner look alive: the failure mode is a refusal, not a duplicate post, and `wfe notify sender --release` is the explicit way out.
- The 5 s lock wait in the acknowledgment test is the store's busy timeout. Step 4 covers the store's own contention behavior.
