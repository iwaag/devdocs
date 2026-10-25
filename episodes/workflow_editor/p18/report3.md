# p18 step 3 — recipient-project work boundaries

Plan: [plan.md](plan.md), step 3.

## The boundary

The conductor's run holds the progression and nothing of the recipients'
work. Each commission stays a run of its own in its owning project. The
conductor's ask or delegate node holds the link: the invitation's run, the
delegation's `child` identity, and its take-up, disposition and correction
records. The step 2 mechanisms keep these apart without new records:

| Situation | Behavior | Evidence |
| --- | --- | --- |
| New independent commission | a new ask or delegate node, so a new run | `standingFlow`: two asks in two turns make two `kid` runs |
| Clarification or more conditions for work under way | delivered to that run with the message (`runs`, or the conductor's `wfe run instruct/answer --source-message`); the conductor is not woken | `standingFlow`: the instruction carries `from: Pat` and source `#m`; the conductor's wakes are unchanged |
| A result to change | the same child: `delegate revise` (fix6) or `--follow-up <rv>` after a review (p17); the corrected result continues the **same** parent run | `standingWork`: correction `c1` on the original child, a second result, one parent run, which then rests |
| Separate deliverable, same room | another run | `standingFlow`: the slogan ask hangs off its own ponder (`ponder-w1`) and is not ordered after the open logo ask |
| Recipient finishes | its own completion, whatever the room does | the logo run completes while the slogan run works and the room stays open |
| Room closes or ponderer rests | no cancellation, acceptance or integration is inferred | after `close`, the slogan run has `cancelled null` and no decision; after the parent rests, the child has no decision and only its own completion queued its integration |

Human review (p17, p17 ex1) is unchanged. A conductor with a review node
waiting reads "Work complete · Awaiting human review". A new topic wakes a
ponder node beside the review, not behind it. Approval and verified cleanup
complete the review node and the conductor returns to **rest**: idle, not
completed. The approval releases nothing (the review node has no
successor). Requested changes keep p17's continuation and the
`--follow-up` path to the same child.

## Guidance

Guides are live files (the executor and new run copies read the checked-out
trees), so the changes are on `p18` branches and merge at deployment
(step 6). Today's conductors on the old release are not told things that
are not yet true.

- pj-ponderer `p18` `9435ddb`:
  - `agent/conduct.md` drops the fact that the next message starts a new
    conductor run. It adds:
    - one run for the discussion's life, which rests between turns and is
      woken with `ponder-w<n>` (citing forusr `run-031…036`);
    - end each turn with nodes completed or waiting, with no conclusion or
      placeholder needed;
    - answer what woke you with `--re`; a silent turn is recorded
      unanswered;
    - new topics are new stretches, named with `ponder --topic`, and
      branch off their own ponder;
    - commission boundaries: new work is a new node; clarifications go to
      that run; changes go to the same child; children complete on their
      own; rest or closing infers nothing;
    - suspended and closed behavior, and held events left alone unless the
      person asks.
  - `agent/guide.md`: the graph is one run for the discussion's life.
  - `devdocs/workflows/conduct.yaml` intent: "for as long as it lasts …
    grows turn by turn … rests between turns". This applies to conductors
    created from now on; existing ones keep their pinned definition.
- agautolab `p18` `eeddf95` (wfrun guide, *A run that speaks in a
  conversation*): a led room has one standing conductor; a recipient run
  keeps its own records and completes when its work is done; a closed room
  or a resting conductor decides nothing about its result.
- Command syntax is in tool help: `wfe conversation help` (the `conductor`
  subcommand, `ponder --topic`, `state --all`, the routing paragraph).
  Nothing in the guides prescribes when to start a topic or how to divide
  commissions; those stay the agent's judgment.

## Checks

- `test/standingFlow.test.ts`: +1 boundary test (two commissions, a
  clarification, a completion while open, closing). 8/8 pass.
- `test/standingWork.test.ts` (new): a standing parent with a real
  delegation, intake and notifier. Its child completes; the result is
  returned for correction to the same child; the child takes it up with
  no change; the new result continues the same parent, which disposes and
  rests; the child is not cancelled, accepted or integrated by the
  parent's rest. 1/1 passes.
- `test/standing.test.ts`: +1 test (review beside a new topic; approval
  brings the run to rest). 9/9 pass.

## Commits

- agdevworld `a59119c` (tests).
- pj-ponderer branch `p18` `9435ddb` and agautolab branch `p18` `eeddf95`
  (unmerged until deployment).
