# p10 step 2 — Durable dispatch and two-way communication

## What was done (agdevworld)

- **The relationship in the parent's record** (`shared/run.ts`, `shared/delegation.ts`):
  - `delegations.<d>` and seven operations:
    - the parent agent's (fenced by its attempt): `delegation.dispatch`,
      `take-up`, `answer`, `instruct`, `dispose`;
    - the system's: `delegation.bind` and `delegation.event`.
  - A delegate node waits on `{delegation: d}` from the dispatch, resumes
    when a fact is taken up, and waits again after an answer or instruction.
  - It completes only after a `satisfied` disposition of its latest,
    taken-up result.
  - `run.cancel` closes open delegations and leaves the child alone.
  - The child's `run.decide` from a confirmation is refused (`delegate-scope`)
    when its requester agent answered alone and `accepts` is `person`.
  - Answers carry `delegation: {uid, answer}` provenance, checked against the
    child's `entrustedBy`.
- **File layer and relay** (`server/delegate.ts`, new):
  - target resolution: the one registered workspace, or the `zulip.json`
    selection; refused when ambiguous;
  - checks that the target validates and accepts requests;
  - cycle checks over the registry's definitions and the active ancestor
    chain;
  - writes `delegations/<d>/request.md` and `original-input.md` (the parent's
    braindump, or the original it kept);
  - the relay maps the child's history to facts (`childFacts`) under the
    receipt `delegation:<uid>:<child seq>:<type>`;
  - a result manifest (outcome, final report and artifacts by sha256 and the
    commit holding them, decision, branch; its own digest);
  - continuations by fact receipt;
  - reads the child's files with a hash comparison;
  - a rebuildable index, `<area>/.local/delegate/delegate.sqlite`.
- **Delivery** (`server/notify.ts`, `shared/notify.ts`):
  - one notifier conversation per delegation (`runs.kind = delegation`) into
    the target workflow's channel, topic `request-<parent>.<d>`;
  - the request, answers and instructions as critical events sent alone, in
    record order;
  - text rendered from the parent's record (request.md checked by hash),
    mentions neutralized;
  - the parent's run topic gets `delegated` / `delegation` news.
- **Intake** (`server/intake.ts`, `shared/intake.ts`):
  - A bot post is a delegation post only when a sent delegation delivery
    recorded its message id (`NotifyStore.delegationPost`). A post whose
    delivery may have been posted but is unacknowledged waits.
  - The request becomes the child (an ordinary request run, `entrustedBy:
    delegation`), then `delegation.bind` on the parent.
  - Answers and instructions are applied with the parent's recorded words and
    actor. A post into a request whose run is not that delegation's child is
    a problem.
  - What the child refuses wakes the parent (`answer-refused`).
  - The intake's pass also runs the relay.
- **Executor** (`server/exec.ts`):
  - `finished()` queues the continuation for facts that arrived during an
    attempt;
  - `reconcile()` re-indexes, relays and re-queues lost continuations;
  - the brief gains a delegations section: state and every untaken fact in
    full. A child is told who asked it and the acceptance scope.
- **CLI and API**:
  - `wfe delegate request|list|show|take-up|answer|instruct|dispose|file|sync`,
    each with `--help` (`cli/delegate.ts`; procedure lives there, not in the
    guides);
  - `GET …/runs/<wf>/<run>/delegations` (each delegation with its child's
    situation now; for a child, its parent).
- **Docs**: `delegation.md` *Operations*, `runs.md`, `intake.md`, `contract.md`.

## Completion evidence

`test/delegateflow.test.ts` covers two projects, the real notifier and
intake stores, and an in-process transport:

1. **Duplicate dispatch**: a retransmitted `request` gives one delegation. A
   second dispatch from the same node while one is open is refused. The
   request and the person's original are kept apart. The node waits, and the
   attempt ends `exited`.
2. **Interrupted send, then a failed bind**:
   - The request post goes out, then the process "crashes" before the
     acknowledgment. The intake sees the bot's post and **waits**.
   - The transport reconciles by marker: the post is found, not posted again.
   - The bind is made to fail once after the child is created. The inbox
     records `failed`, and a retry finds the child by its receipt.
   - Result: **exactly one child**, bound once, with the dispatched
     `request.md` byte for byte and the original beside it.
3. **Multiple questions and answers**:
   - q1 and q2 keep their ids through the relay (events 1 and 2), the
     parent's answers 1 and 2, and the child's answer provenance.
   - An answer before take-up, or to a question the child closed, is refused.
   - **Unrelated senders**: a bot post copying a delegation marker that no
     delivery recorded, and a stranger, change nothing.
   - An answer whose application is repeated is recorded once.
4. **Result round trip**:
   - The confirmation presents `icon.txt` by hash. The parent reads it back
     with a matching hash and accepts (`accepts: parent`).
   - The child decides `accepted` and completes. The parent gets `decided`
     and `completed` with a manifest naming `icon.txt`.
   - Completing the node before a disposition is refused. `satisfied` is
     bound to the manifest digest, then the node completes and `use` is
     ready.

`test/delegation.test.ts` adds reducer rules:

- scope: a confirmation with `accepts: person` needs `--because` from the
  person's taken-up answer;
- latest-result and take-up checks for `dispose`;
- a replacement needs its predecessor;
- an attempt that ends without taking up a waking fact recorded before it is
  `interrupted`; a fact that arrives during the attempt is not;
- parent cancellation orphans the delegation, and late facts are refused;
- `delegate-relation` and the child-side `delegate-scope` refusals.

Full suite: **323/323**, type check clean, build OK.

## Deviations and decisions

- **The delegation uid is `<parent uid>/<d>`**, not `~<d>` as report1
  wrote. Receipts carrying the uid allow only `A-Za-z0-9:._/#-`.
  `docs/delegation.md` is updated.
- **Creation guards lifted in this step**, not step 3. The plan's condition,
  "once dispatch and relationship recording are available", is met here.
  Running the round trip through creation needs it, and nothing was deployed
  in between. Removed: `delegate-unsupported` in preflight and the reducer,
  the adjustment refusal of delegate nodes (an adjustment may now add one
  with a target), and the `delegate-not-executable` warning. Stale UI texts
  go too ("execution not supported", the disabled request option).
- **Parts of step 3 are already in the code**: `finished()`/`reconcile()`
  continuations and the interrupted rule. Step 3 verifies them with the
  real executor loop and adds the remaining controls.
- **People may still post** in a delegated child's request topic. A
  configured requester's reply is recorded as their own words. Only the
  parent's agent answers as the parent, through delivery.

## Next

Step 3: the executor round trip (one slot, restart, parent hold) through
autolab's `wfexec`; hold and cancellation controls; the wfrun guide's facts.
