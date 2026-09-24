# Step 4 report — a receipt for what a serving was given, on every route

Plan: [plan.md](plan.md) step 4. 2026-09-24 (JST), by the Omni Agent.

## What changed (pyagag `023224a`)

| Area | Before | Now |
|---|---|---|
| What a serving was given | Not recorded. The journal knew the served conversation's input boundary and the mention that triggered it, nothing about the threads beside it | **`agag.serving.note_input`**: each renderer that hands a thread to a run records its span in the serving journal (first and last id, and whether the read reached the conversation's beginning). `topics.write_threads` (Front's ordinary servings, autolab's task runs) and `agfront.evidence.write_evidence_threads` (Front Desk, routine runs) both do. An unreadable thread records nothing |
| The parent hop in the input | Front's threads were only the conversations it had posted in. A task autolab started itself (p1) was not one, so its report reached a serving only if the run fetched it with a tool | `remotes_for_home` adds **the conversations opened under one of home's remotes and started by their owner for this bot** (a root note by the opener naming the parent, by anchor when it has one, plus a `[start] … for <this bot>`). It's the same hop callbacks (`parent_rootchat`) and the trace follow. The task's answer is now a thread the serving is handed |
| Receipts after delivery | Owner route: only answers named by Observer's `[owed]` notes. Mention route: only the triggering post | **`Listener._mark_inputs`**, on both routes after a *confirmed* delivery. Per thread, the newest post naming this bot inside the span and above its receipt is marked in home. A post arriving after the thread was read lies outside the span and stays owed. A thread read in part is marked only when no post naming this bot lies between the last receipt and the start of the span, so a receipt never covers an answer nobody was given. Just-written marks count at once (p2's `_wrote_mark`) |
| Interruption before receipts | A restart between delivery and the mention route's mark wrote the mark. Anything else was lost | `_receipts_pending`: a delivered record whose trigger mark, owed marks or input marks are not all written gets them on the next pass, **without a rerun**, before the entry is judged again. The prepared-reply redelivery path writes them too. Before execution and before delivery nothing is marked, so the answers stay owed and the next serving takes them |
| `_mark_owed` home | Read by its open name. A home ✔'d between the reply and the receipt got none (p3 step 1's code fact) | Home by the serving's anchor, under ✔ included |
| Startup recovery | Open topics only, so a callback in a ✔ topic that arrived while the listener was down (autolab ✔s a task in the second after its report) came back only if Observer asked (S2) | **Callbacks under ✔**: a ✔ topic this bot does not own, holding a post naming it above its receipt and newer than the listener's **horizon**, is queued like any mention. It's served at home, never into the ✔ topic, so nothing is reopened and no twin is made. The horizon is the newest message the mirror held at the first start of this rule, kept in the queue file, so the realm's older ✔ history (p1's unmarked callbacks) is not replayed |

The trace, the listener's dispatch (`owed`) and recovery all read the same receipts by id (p2's
`served_key`), so "taken up" means one thing everywhere. Queue identity is untouched: entries are
still keyed by name, while receipts are resolved by the message ids they carry.

## Evidence

**Fixtures** (pyagag 774 passed, no xfail left):

| Test | Shows |
|---|---|
| `test_s1_…` (the step 1 reproduction, mark removed) | An owner-route serving handed a delegated answer writes its receipt after the reply, and the mention route does **not** serve it again |
| `test_s2_…` (rewritten as the real sequence) | The listener has run, then goes down long enough for its event queue to expire. A callback lands and is ✔'d. At restart it is queued from the index and served once |
| `test_the_realm_s_older_callbacks_under_a_check_mark_are_not_replayed` | ✔ history from before the horizon is adopted, not served |
| `test_an_answer_arriving_after_the_thread_was_read_stays_owed` | The first receipt stops below the answer posted mid-run. That answer is served once, by the mention route, and marked |
| `test_a_restart_between_delivery_and_receipt_writes_the_receipt_without_a_rerun` | A crash after delivery, before the receipt: the next process writes the receipt, and the run is not repeated |
| `test_identity::…task_its_owner_started_for_this_bot…` | A task started for Front is one of Front's threads. A task started for somebody else is not |

Consumers on `023224a`: agfront 182, agautolab 262, agforge 265, agobserver 115, agentroom 331,
archsage 28, cagent 202 passed.

**Deployed.** All six listeners and the relay restarted on `023224a` at 02:54Z, each checked
first for a harness child (none). Every listener logged `callbacks under ✔ are recovered from
#9983 on` and queued nothing new at startup. forge's 1 and Observer's 11 queued entries are the
standing ones (mentions their mention routes ignore: "not an argue"), the same count as before
the restart.

**Live behaviour** (an owner-route serving handed an answer, a mid-serving answer, a callback
under ✔ across a restart) is step 5's table, on the frozen revision.

## Limits

- A thread longer than the history read (`HISTORY_MESSAGES`) and holding a post naming this bot
  before the span gets no receipt from that serving, by design. The mention route then serves it,
  which is the old behaviour.
- The parent hop is one hop, like the callback. The note search it uses reads the newest 1,000
  root notes (302 in the realm today).
- A run that fetches a conversation itself with a tool, outside its threads, is not recorded as
  having been given it, so that answer stays owed and is served once more.

Also found and fixed while committing: the parallel pushes of steps 2 and 3 had not reached
`pj-agdev`'s remote (origin was still at step 1). Everything was pushed in sequence and every
repository checked against its remote.
