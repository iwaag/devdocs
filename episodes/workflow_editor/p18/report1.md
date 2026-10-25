# p18 step 1 — conductor ownership and lifetime: the state representation

Plan: [plan.md](plan.md), step 1.

## What was found

In the current code, a facilitated conversation ends up with one ponderer
run per response. This is not because each reply starts a run directly:

1. ponderer's `conduct` graph starts with one `ponder` node. The agent
   adds activities with `ponder --next`, replies, and completes its last
   node.
2. The reducer (`shared/run.ts` `summarize`) makes the run `completed`
   when every node is completed. `run-044` shows it: `node.complete
   ponder [ponder running→completed, run in-progress→completed]`.
3. Routing rule 4 (`openConductor`) delivers only to a conductor that is
   not completed or cancelled. The next message therefore issues a
   `conduct` request, and `dispatch` starts a new run. `afterResolution`
   also replaces a completed conductor with a new turn when awaited
   outcomes arrive.

Live data (store v12, 74 rooms) shows the effect. `20261024-160217-forusr`
used 6 conductor runs (`run-031`…`run-036`), one per person message.
`20261025-131942-nobaqg` used 2. No conductor ever recorded
`instruction.receive` or `instruction.ack`, because every message started
a run of its own.

The audit of what a long-lived run needs found these facts. Step 2 acts
on them.

- The executor integrates a run's copy only when the run completes or is
  accepted (`finished`). A run that never completes would never publish
  its changes to `main`.
- Guides are read from the run's own copy (`server/context.ts`: "A run's
  defaults are read from its own copy"). A copy made when the
  conversation began would keep the guide from that day for good.
- The bootstrap brief lists every delegation, presentation and review
  round, and `wfe conversation state` lists every node. All of these grow
  with the conversation.
- `run.json` keeps each revision's full compiled graph. `run-038` is
  already 115 KB after 69 entries.
- `conduct` uses the `fresh` session policy, so no model session lasts
  across turns. That stays as it is.

## The representation

### The run: `standing`

A conductor run carries `standing` in `run.json` (`shared/run.ts`; the
contract is in agdevworld `workflow_editor/docs/runs.md`, *Standing runs*):

| Field | |
| --- | --- |
| `role`, `conversation` | `conductor` of that conversation |
| `state` | `open` (room open and facilitated), `suspended` (facilitation ended), `closed` (room closed) |
| `since`, `seq`, `by`, `reason` | the transition that set the state |
| `wakes[]` | each continuation from rest: `{n, node, seq, at, by, events}` |
| `held[]` | events a transition found waiting (`i4`, `d1#3`): they start nothing until released |
| `replaces` | the cancelled conductor this one explicitly replaces |

The execution state `idle` was added. When every node is completed and
the run stands `open` or `suspended`, the run is **resting**: it is not
completed, and it holds no attempt, slot or polling turn. A run without
`standing` completes exactly as before. Each state reads as follows:

| Situation | When | Status text |
| --- | --- | --- |
| `awaiting-input` | idle, `open` | Idle · Ready for the conversation's next message |
| `suspended` | idle, `suspended` | Suspended · Facilitation ended; nothing unsolicited comes to it |
| `due` | idle, with an instruction no attempt was briefed with | Continuing / Queued |
| `completed` | `closed` at rest (finalized) | Closed · Conversation closed; reopening continues this run |

A named answer wait, a delegated-result wait, a human review, a failure
and a hold keep their existing situations. An idle conductor never reads
as `due` or `stalled`, and never as working.

Operations, all recorded by WFE's conversation service:

| Operation | Transition |
| --- | --- |
| `run.create … standing` | a new conductor, `open` |
| `standing.establish` | a run that did not stand: a conductor that created its own room, or the transition of an existing room (step 5). A completed run becomes idle, and everything unacknowledged is held |
| `standing.facilitate` / `standing.suspend` | facilitation starts again / ends |
| `standing.close` / `standing.reopen {state}` | room closed / reopened; reopening holds what was waiting |
| `standing.release` | the person's resume releases held events |
| `run.adjust` with `source {kind: wake, events}` | the **wake**: one new `ponder` node, allowed only while the run is open, no attempt is open, nothing is ready or running, and the events are instructions not yet acknowledged, briefed or held |

Instruction consumption: on a standing run, `attempt.begin` marks every
unacknowledged, unheld instruction as `seen` by that attempt. Once that
attempt ends, the instruction wakes nothing again, whether or not the
agent acknowledged it. An instruction that arrives during an attempt is
not seen by it. If that attempt ends with nothing ready, the instruction
wakes the run. A delegation fact the room holds is neither reported
`due` nor counted as a missed event at an attempt's end.

### The conversation's side (implemented in step 2)

- `conversations.conductor` stays the conductor's identity. A request to
  the conductor is delivered to that run (an instruction, or an answer to
  its question) whenever the run is not cancelled, including while it is
  idle or closed. A cancelled conductor is never replaced silently: the
  request fails with the reason, and replacement is an explicit act that
  records `replaces`.
- The room's transitions are recorded on both sides. Close and reopen
  become activity events (`room`) as well as `standing.close` / `reopen`.
  Facilitation end/start drive `standing.suspend` / `facilitate`. Reconcile
  repairs any run whose standing state differs from its room's.
- A request delivered to the conductor that its next turn ended without
  answering is settled `unanswered` at that point. Before, this happened
  when the run ended. Nothing stays "working" because a turn chose
  silence.
- Topic labels (`ponder --topic`) are activity metadata. They name a
  stretch and never a run.

### How the plan's transitions map

| Event | Representation |
| --- | --- |
| Ponderer sends a reply | the attempt ends normally; with nothing left the run is `idle` (`awaiting-input`) |
| No immediate work remains | `idle`: no job, no slot, no polling turn |
| Human answer or delegated outcome | an answer continues the waiting talk node; a delegation fact continues the delegate node; an instruction with nothing ready gets one wake node. Each continues the run once, by receipt |
| A new topic | a wake node (or the agent's own `--next`) appends a stretch in the same run |
| Facilitation ends | `standing.suspend`: nothing unsolicited; its asks and delegations still reach it |
| Facilitation starts again | `standing.facilitate`; the brief is delivered to the same run |
| Conversation closes | `standing.close`: at rest it is finalized (`completed`); open work keeps waiting; children are not cancelled and reviews are not approved |
| Conversation reopens | `standing.reopen`: the same run; waiting events are held; the new request is delivered |
| Conductor cancelled | nothing revives it; replacement is explicit and recorded in `standing.replaces` |

## Checks

- `test/standing.test.ts` (7 new tests) covers: idle vs. completed and
  the status words; a wake and briefing/consumption, including a message
  that arrives during an attempt; refused wakes; suspension and
  facilitation; close finalizing and reopen holding with release; close
  with open work; cancellation; establishing a completed run with held
  instructions; replay equality.
- WFE `node --test`: 607/607 passed, so no ordinary run's reading
  changed. `tsc` is clean.

## Commits

- agdevworld: the reducer, statuses, the *Standing runs* contract and the
  tests (step 1).
