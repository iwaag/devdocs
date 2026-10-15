# p10 step 3 — Waits, resumption and result handling

## What was done

- **Execution through the shared state machine** (since step 2): runs of
  delegating workflows are created, and adjustments may add a delegate node
  with a target.
  - A delegate node waits on its delegation, resumes on take-up, and waits
    again after an answer or instruction.
  - It completes only on a `satisfied` disposition of the latest, taken-up
    result. Dispatch and the child's terminal state complete nothing.
- **Continuations**:
  - A waking fact (question, answer-refused, completed, failed, cancelled)
    queues one job per fact (`delegation:<ws>/<run>/<d>/<n>`), unless the
    parent is held, stopping, cancelled or executing.
  - A fact that arrives during an attempt is queued by that attempt's end
    (`finished`).
  - `reconcile` re-queues lost continuations, re-indexes delegations from
    their records and relays.
  - An attempt that ends without taking up a fact it was started for is
    `interrupted`, like ready work left behind. Pending facts are never
    dropped, and no second writer starts.
- **Controls**:
  - A parent hold keeps relayed facts and queues nothing; the resumed
    attempt's brief lists them.
  - Stopping the parent never touches the child.
  - Cancelling the parent closes its open delegations as `orphaned`. Later
    facts are not recorded, and the child stays readable.
  - `wfe delegate cancel` cancels the child as its requester: explicit, with
    the delegation and the reason in the child's cancellation. It comes back
    as a `cancelled` fact.
  - Child failures are relayed as facts and never replaced automatically.
- **Distinct meanings in the child**:
  - Plan agreement still binds the child's `plan.md` by hash; the parent's
    question event carries that hash, and `wfe delegate file` compares it.
  - Result acceptance still binds the presented bytes (`subject-changed`)
    and the child's `delegate-scope` rule.
  - Publication and integration stay the child's own, by its rules.
- **Guide**: autolab's `agent/guides/wfrun/guide.md` gains *Delegation:
  another workflow's run does the work* — facts only; `wfe delegate help`
  holds the procedure. It covers:
  - the child does the work;
  - relayed facts and take-up;
  - no waiting inside an attempt (one slot);
  - which answers are the agent's and which acceptance is the person's;
  - the four separate records;
  - no automatic replacement;
  - what a delegated child is told about its requester.

  The old line "Delegate nodes are never in a run you perform" is gone.

## Completion evidence

**Executor trial** (the real `agautolab.wfexec`, one slot, `agent_command`
= a scripted agent that uses only the `wfe` CLI, Zulip played in-process
through the stores' transport interface; private area, nothing in
production touched). Files: `pj-agdev/.local/workflow-editor-p10/step3/`
(`trial.ts`, `agent.mjs`, `trial.log`, `evidence.json`).

| Time (UTC) | What happened |
| --- | --- |
| 06:25:13–18 | parent attempt: start `asset`, `wfe delegate request`, exits `exited` (node waiting) |
| 06:25:20 | executor **stopped (SIGTERM)** with the child's start queued; executor B started |
| 06:25:21–24 | child attempt: asks q1 → relayed to the parent |
| 06:25:26–29 | parent attempt: take-up, answer q1 |
| 06:25:30 | the person **stops the parent** |
| 06:25:32–36 | child attempt: takes up the delivered answer, builds `icon.txt`, asks q2 (confirmation) → relayed while held |
| 06:25:45 | **parent jobs while held: 0** |
| 06:25:46–51 | the person resumes; parent attempt: take-up, reads `icon.txt` (hash matches), answers q2 (accepts, `accepts: parent`) |
| 06:25:53 | child attempt starts (20 s agent) |
| 06:25:55 | executor **killed (SIGKILL)**: the agent process still lives; executor C keeps the slot for it and starts nothing else |
| 06:26:57 | the agent exits while no executor owns it → `unknown` (existing p4 semantics). The child's acceptance and completion were already recorded; the hold is relayed as a non-waking fact. The child's integration ran by its own rules |
| 06:26:58–27:03 | parent attempt: takes up `completed`, checks each manifest artifact's hash, disposes `satisfied`, completes `asset` and `use` |

The result:

- **7 attempts, 0 overlapping**: never two at once, never two of one run.
- Parent `completed`: delegation `satisfied`, bound to manifest
  `sha256:90fae679…`.
- Child `completed`, `accepted from q2 by par-agent`. Its answers carry the
  delegation answer numbers 1 and 2.

**Tests**, `test/delegateflow.test.ts` (+3; 7 in all):

- A parent hold keeps facts and queues nothing, even through reconcile.
  Resuming lists the facts. Stopping the parent never cancels the child.
- Parent cancellation: the child keeps running, the relay closes the index
  row, and nothing more is recorded. The child stays readable.
- The child's plan agreement comes from the delegated answer and binds
  plan.md's hash, which the parent can compare. The deliverable then changes
  after review: the child's acceptance is refused with `subject-changed`.
  The requester cancels the child, and the parent is woken by `cancelled`.
  No acceptance is recorded.

Suites: agdevworld **326/326**, type check clean; agautolab `-k wf`
**45 passed, 1 skipped**.

## Observations

- A killed executor whose agent outlives it gives the existing `unknown`
  hold, even when the agent's records are complete. The trial resumed the
  child as its person. That is Observer's recovery path, unchanged in p10.
  The parent was unaffected: it had already been woken by the result.
- "Details published at : icon.txt (not yet published…)": the follow-up
  notice for a document outside the run folder renders an empty commit in
  a project without a web remote. This is pre-existing and was not changed.

- **The guide edit is live before the release.** `agautolab.wfexec` reads
  the wfrun guide from the checkout at every claim, so production attempts
  get the new section now, while the installed editor (`e1e6be3`) has no
  `wfe delegate` yet. It is inert: no production workflow has a delegate
  node. One side effect: the common contract's hash changes, so a run with
  session policy `run` starts a new session at its next attempt.

## Next

Step 4: one detailed-IO channel per run, fed from the attempt logs; links
from the request receipt and the run view.
