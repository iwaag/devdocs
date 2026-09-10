# p2 step 1 — unfinished work stays reachable

## The defect

`refactor` p1's report recorded it as a standing limitation:

> **A related topic is still resolved when its mission is blocked.** The
> conversation closes and the work is not accepted — the distinction working
> — but archiving the channel afterwards takes the unfinished task's topic
> with it.

The operation room's completion had exactly one dependency rule: *anything*
blocked or failed keeps the **selected request's** conversation open
(`Closer._apply`'s `trouble` flag, which guards only `kind == "conversation"`).
Every other target was independent of every other. So a mission blocked on an
unfinished task was correctly refused acceptance — and its plan topic was
resolved, its unfinished task's topic was resolved, and `work-m<id>` was
archived on top of both. The one thing that must not happen when work is
unfinished is that the place to finish it disappears.

## What changed

`agdevworld/agentroom/src/agentroom/close.py`, two functions and one rule.

**`dependents(action)`** — the action keys a work record decides: its own
conversation, each of its children's, and the dedicated `work-` channel named
after it (by anchor id for a record kept in the chat, by label for one kept in
Plane). Keys rather than coordinates, because a key is what a result row is
matched to and what `_apply` walks.

**`_hold_dependents(actions)`**, run at the end of `plan_actions` — every
`READY` topic or channel a **blocked** work record names becomes `KEPT`, with
the record's label in the reason. Deliberately narrow in three ways:

- only `READY` moves: an already-✔ topic stays `DONE`, because there is
  nothing there to hold;
- only `topic` and `channel`: the selected request's own conversation is held
  by the existing rule, which is broader and should stay so;
- only what a blocked record names: an independent finished branch of the same
  request — another mission, forge's Plane Work and its conversations — still
  closes.

**The same rule during execution.** `_apply` now collects `dependents()` of any
work action whose write *fails*, and skips those targets with
`… could not be accepted, so the work it is about is not finished; retry once
that is resolved`. The preview applies the rule to what it already knew was
blocked; a failed write is the same fact learned a moment later. The plan's
fixed order — work first, then topics, then channels, then the request — is
what makes one pass enough.

Nothing about cancellation, replacement or disposal changed: a cancelled
mission is still `KEPT` untouched, and a `replaced` one is still another
request that is never closed alongside its replacement.

## Verification

`agdevworld/agentroom/tests/test_close.py`, six new tests over the existing
Front-Desk chain fixture, with forge's half left open so there is an
independent branch to watch:

| test | what it pins |
|---|---|
| `…holds_back_its_own_conversations_and_channel` | an unfinished task 2: exactly the plan topic, the task-2 topic and `work-m10` are `KEPT`, each naming `m10` |
| `…independent_branch_still_closes_while_a_mission_is_blocked` | forge's two topics resolve; nothing of the mission does; nothing is archived; the request stays open |
| `…unfinished_conversation_is_still_usable_afterwards` | the topic is not ✔ (so a post there is not a twin) and its channel still exists |
| `…acceptance_that_fails_during_apply_holds_the_same_targets` | acceptance refused by the realm → the same targets `SKIPPED`, with the retry reason |
| `…retry_after_a_failed_acceptance_finishes_the_request` | second click writes `done`, resolves, archives, closes the request; `partial` is false |
| `…retry_after_the_unfinished_task_is_done_closes_everything` | task 2 completed between clicks → everything closes |

All six fail against the pre-change module (checked by reverting the two call
sites and re-running: `6 failed, 31 passed`) and pass with it.

```
$ uv run pytest -q        # agdevworld/agentroom
242 passed in 8.44s
```

## Limitations left standing

- **The hold is per-record, not per-branch inside a record.** A mission with
  five finished tasks and one unfinished one keeps all six task topics open,
  not just the unfinished one. The plan permits this ("a simple broader hold
  is also acceptable") and it is the honest reading: the tasks share one
  channel, and archiving it is a single act.
- **A channel is bound by name.** `work-<label>` is derived from the record,
  so a channel some other convention named is not held by it — and was not a
  candidate for archiving in the first place (`closing._channels` refuses any
  channel holding a topic this request did not reach).
- Nothing here touches forge's storage yet; a forge Work still comes from
  Plane and is judged by the shared parent/child rule. That is step 2.
