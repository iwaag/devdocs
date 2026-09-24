# Step 2 report — a legitimate wait ends nothing; a verdict is about the evidence it read

Plan: [plan.md](plan.md) step 2. 2026-09-24 (JST), by the Omni Agent. Observer stays on its local
backend (`qwen3.8:27b-mxfp8` through agcode). Nothing about the model, the guide or the thresholds
changed.

## What changed

| Area | Before | Now | Where |
|---|---|---|---|
| What ends tracking | A request left the index when nothing below it was unfinished, and a ✔ conversation whose `resolved_live` incident was *dismissed* counted as finished. "No recovery request is needed" and "this work is over" were one thing, so a human wait left tracking in the look that judged it legitimate (W1) | A request stays until every conversation under it is `done` or `cancelled` **by record** (`TERMINAL`). A ✔ does not end it, a `legit` verdict does not, and silence does not. Completion and cancellation keep their own evidence: the owner's `[state]`, or the requester's `accepted`/`done` | agobserver `Monitor.retain` |
| A dismissed incident | Never looked at again unless its candidate came back, and never closed | Closed when its conversation records an outcome: `done` → **`finished`** (new closed state: the wait ended with the work's own record and nobody was asked, so it is not a rescue), a cancellation → `cancelled`. Anything else leaves it dismissed. A mechanical fact on the same work still re-opens it (p2's A2 rule). A recovering `resolved_live` whose ✔ conversation becomes `done` also closes as `finished` instead of waiting for an un-✔ that should not come | `Monitor.settle`, `verify` |
| Asynchronous judgment | The worker's verdict was filed under the incident key, and the next look used it whatever had happened since (J1). A `stall` kept in the record drove the second request and the report without being asked again | Every job carries a **snapshot identity**: kind, traced state, ✔, the newest relevant post in the stalled conversation, and the same in the request's own conversation. "Relevant" means speech by anybody but Observer, or a `state`/`served`/`start`/`owed`/identity note, so bookkeeping notes do not count. A verdict whose snapshot differs is discarded, counted and logged, and the current state is queued. A pending job is replaced rather than joined, so there is at most one per incident. A kept `stall` is re-judged when its snapshot moves. Neither path touches the request allowance | `Monitor.snapshot`, `judged`, `handle` |
| Health | Showed the running judgment and the pending count | Adds `oldest_pending_seconds`, `invalidated` (total) and `churning` (incidents whose evidence moved under `CHURN_LIMIT` = 3 judgments in a row). The relay's watchdog reads a backlog older than 3 × the judgment timeout, or any churning incident, as **`degraded`**, and says so on `/ops`, `/healthz` and by DM like its other failing states | `Monitor.health`; agdevworld `agentroom.watchdog` |

The triage guide is unchanged. It already says `legit` means "the wait is intended", and the defect
was in how the verdict was consumed, not in the model's answer.

## Evidence

**Fixtures** (agobserver `tests/test_p3_reproductions.py`). W1 and J1 lost their strict-xfail marks.
Added:

| Test | Shows |
|---|---|
| `test_w1_…` | A ✔ on a conversation waiting on the human is dismissed with no nudge, and the request is still tracked after 13 h and a restart (controlled clock) |
| `test_j1_…` | A `stall` about the state before the human's answer is discarded, and the new state is queued. No request |
| `test_a_recorded_outcome_ends_the_wait_and_the_tracking` [cancelled / accepted] | The owner's `cancelled` closes the dismissed incident as `cancelled`, and the requester's `accepted` closes it as `finished`. The request leaves the index on that look, with no request posted |
| `test_a_dismissal_does_not_hide_a_later_blockage_of_the_same_work` | A new unserved answer on the ✔ conversation becomes `undelivered` and is asked about once, with its owed note |
| `test_a_stall_verdict_on_record_is_judged_again_when_its_evidence_moves` | `stall` → one request. The human then answers at home, so the kept verdict is re-judged (`legit`) before the retry and no second request goes out. The first request still counts |
| `test_evidence_that_keeps_moving_under_a_judgment_is_visible_in_health` | Three invalidations in a row put the incident in `churning` and are counted. One job stays pending and nothing is asked |
| agentroom `test_a_judge_falling_behind_or_churning_evidence_is_degraded` | The watchdog's two new `degraded` reasons |

Suites: agobserver 115 passed (p2's 108, including R1–R9, plus 7); agentroom 331 passed.

**Dry run before rollout.** One look of the new monitor over a fresh copy of Observer's mirror and
of its incident store, with a recording client and a judge that aborts if called. It took 0.4 s,
kept 15 tracked, and made one change: C3's dismissed `resolved_live` on task 1
(`incident-resolved_live-9741`) closed as **`finished`**, since that task records `completed`. That
is two posts, both in Observer's own channel. Nothing was asked and no judgment ran.

**Live.** Observer (`com.agdev.agobserver-zulip`) and the relay (`com.agdev.agentroom`) were
restarted on this code at 02:29Z. The first look did exactly what the dry run showed: cycle 1 in
0.58 s, 18 looked at, 15 tracked, the one `finished` closure, and the watchdog `ok`. The discovery
window and a human wait across a real restart are exercised live in step 5. The 13-hour boundary
here is a controlled clock.

## Consequences to watch

- A ✔ conversation that somebody meant to abandon, with no `cancelled` recorded, now keeps its
  request tracked until a record exists. That is intended (a ✔ stops nothing), and each such request
  costs about 20 ms per look off the mirror. p3's `✔ workplan-locations-2` (a planning twin closed
  by hand) is one: it keeps `o8286` tracked until somebody records what it was. Step 3 records
  outcomes only where the evidence exists.
- Re-judging a kept `stall` when the requester's side speaks means a Front reply to Observer's
  request is itself evidence and gets judged again. That is at most one extra local judgment per
  retry interval, visible in `invalidated`.
