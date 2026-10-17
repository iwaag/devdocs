# p11 Step 3 — cagentctl and nctl

Implemented and pushed: nctl `5eb1724`, cluster backend `935fd7f`, managed
cagent `7e62bee`. The managed backend pin and its nested nctl checkout now
match the live backend source. Existing run-copy migration/deployment remains
Step 5; infrastructure acceptance remains Step 6.

## Removed enforcement

cagentctl discovers ordinary workflow records or accepts `--context DIR` for
a directly supplied record directory. Neither path requires an open attempt,
a change/scan workflow, an approver or an agreed plan. Approver/playbook settings
were removed from the configuration model and example. Operations no longer
bind authority to plan questions, answer take-up, plan hashes, batch hashes,
preview totals, source commits, sources_update or host/playbook allow-lists.

Batch hashes, preview results, source revisions, selectors and resolved hosts
remain effect evidence. An explicit reconcile host still selects the planner's
scope; removing authorization scope does not select the whole cluster. Ansible
limits retain their ordinary selector meaning, including patterns. Actual
resolved hosts and recap outcomes are recorded; empty/failed executions do not
become success. Dirty sources are described rather than vetoed.

Destroy actions follow the ordinary recorded execution path. Compulsory person
prepare/confirm commands were removed; historical handoff entries remain readable
and do not hold new operations. General nctl dispatch has no read-command
allow-list. Source update takes an explicit Git revision, records actual outcomes
and permits Git to merge it; conflicts and unavailable revisions are concrete
errors. Unknown effects remain investigable without a person-only release step.

nctl removed allow_destroy and imposed allowed_hosts from CLI, execution,
per-action dispatch and result types. Default reconcile/batch previews remain
read-only; --yes selects execution rather than identifying an approver. Direct
Braindump completion/review deletion similarly default to an explicit preview
without an interactive human prompt. No former dry invocation became a mutation.
Downstream supported action handlers retain target identities and actual
transport/argument diagnostics. No nintent/Ansible approver check was found in
these dispatch paths; no changes to their domain schemas were needed.

## Evidence

- nctl ordinary suite: **1,358 passed**.
- cagentctl suite: **25 passed**, including ordinary workflows without plan,
  approver configuration or WFE_ATTEMPT; direct contexts without a WFE record;
  changed plans/batch bytes; destroy dispatch; unregistered playbooks and changed
  variables with dirty sources; explicit source updates; unrestricted nctl
  arguments; durable unknown/partial outcomes and recovery.
- Ansible boundary conformance: **4 passed**, using temporary inventories,
  playbooks and markers. No production hosts were changed by these tests.

Removed tests asserted obsolete refusals; success regressions cover their real
command and ledger boundaries. Real failures, partial results, lost responses,
missing records, interrupted writes and per-host outcomes remain covered. Raw
logs and test fixtures are in ignored local evidence. These are source/boundary
results, not a claim that a real guest lifecycle or fresh installed attempt has
already passed.
