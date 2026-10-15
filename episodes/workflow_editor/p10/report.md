# workflow_editor p10 — Delegation and inspectable execution (final report)

## Result

A workflow can now entrust a node's work to another managed project's
workflow. It answers the child's questions, receives its artifacts, judges
them and continues. People can follow both the overview and the detailed
execution from Zulip and the run view.

It was verified on the deployed system (agdevworld `394def3`) with a real
cross-project round trip, agops `delegate-check/run-001` → forge
`provide/run-007`:

- forge asked for plan agreement and then a result confirmation;
- the parent agent answered both;
- it received `tagline.txt` and checked its hash;
- it judged the result `satisfied` and completed.

The round trip took 16 minutes, 7 attempts in the one slot, $2.58, and
opened no Observer incident.

## Definition of done

| Item | Status |
| --- | --- |
| A delegate node completes a real cross-project request/question/result cycle | done (report6) |
| Durable identity and receipts survive retries and restart without duplicate work | done: message-id binding, receipts per fact, crash-point tests (report2, report6), executor restarts (report3) |
| Waiting releases capacity; child events resume only eligible parents | done: no polling, one job per waking fact, holds respected, `interrupted` for an ignored fact (report3) |
| Decision ownership and integration authority are explicit and evidenced | done: `accepts`, `delegate-scope` on both sides, separate records (report1, report2, report6) |
| The existing mechanical receipt leads to progress, detailed IO and the run view | done: forge receipt #16202 (report6), follow-up when IO is not ready yet (report4) |
| A person can follow delegated work down to the child's detailed output | done: parent → child links in messages and views, the child's IO channel (#301) |
| Observer understands delegation waits and detects broken delivery/resumption | done: `awaiting-delegation`, `relay-stalled`, `unbound-dispatch`, unowned continuation; no duplicate child recovery (report5) |
| Deployed, documented, ready for immediate inspection | done: deployed, `README_DEV.md`; inspection links in ignored local environment notes |

## Design in brief

- `delegate` stays its own node type, with `project`, `workflow` and
  `accepts`. The child is an ordinary run created through the target
  workflow's Zulip intake.
- The parent's `run.json` holds the relationship (`delegations`, seven
  operations). The notifier's journaled deliveries carry the request and
  answers. The intake binds posts only by delivered message id. A relay (in
  the intake pass and reconcile) turns the child's history into numbered
  facts for the parent.
- Agents decide: what to request, how to answer, whether the result serves
  the request. The system owns identity, delivery, binding, relay and
  continuation.
- Detailed IO is one Zulip channel per run, fed from the attempt logs:
  public output only, redacted, bounded, with gaps said as gaps.

## Remaining limitations

- Parent and child must run on the same execution host.
- Files go to the child as named references in the request text, not as
  attachments.
- A killed executor whose agent outlives it still ends that attempt as
  `unknown`, even when its records are complete (p4 semantics). Recovery
  stays Observer's or the person's.
- `checks/simplerUi.ts` and `checks/runInteraction.ts` each fail one check;
  this predates p10 (it is the same on `e1e6be3`).
- forge's `text` capability got its first real delivery here
  (`provide/run-007`); its catalog entry is still `unverified` until forge
  records the proof.
- An `agobserver` service aggregate in Nautobot shows `service_missing`
  (pre-existing). On the review recheck, the old agforge service is inactive
  with no active placement; its retirement is no longer pending the Developer.

## Review correction

The IO reader previously stopped advancing when one JSON log line exceeded
its 2 MiB batch budget. It now reads that line through its newline, preserves
the complete entry for the full-output view and continues with later events.
Incomplete lines remain pending; UTF-8 is decoded after chunks are joined.
The batch budget is not a memory cap: one oversized line is assembled in memory.
Regression coverage includes a large tool result, later output, the attempt's
end boundary, full-entry retrieval and no duplicate delivery after restart.
Deployment and final validation are recorded in `report-review.md`.

## For the person to inspect

- Open the editor using the links in `pj-agdev/.local/devenv.md`, section
  *workflow_editor p10 review correction*. Machine-specific addresses belong
  in those ignored local notes.
- Parent: agops `delegate-check/run-001`; select "Get the asset from forge",
  or open Detail → Delegations.
- Child: forge `provide/run-007`.
- The forge request topic: `wf-forge.provide ›
  request-agops.delegate-check.run-001.d1` (the request, the receipt with
  its links, both questions, the parent agent's answers).
- Detailed IO: `wf-agops.delegate-check.run-001.io` and
  `wf-forge.provide.run-007.io`.
- Delegate inspector: the agops `delegate-check` workflow.
