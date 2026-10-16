# project_migration p5 — Step 8: legacy retirement

Completed 2026-10-16 after the scan artifact consumer and browser path were
proven. Raw desired-state backups, narrow rollback batch, applied results,
archived plists and final observations remain in ignored local notes.

## Retired paths

- Booted out the legacy cagent Zulip listener and HTTP API; archived both
  installed LaunchAgent plists rather than leaving them enabled at login.
  Both labels are absent and the three old listener ports are closed.
- Published retirement introductions with no live roster channel/prefixes:
  transition post `17298`, final post `17360`. They identify the five
  managed workflow entrances and preserve the older conversations.
- Removed `executor/`, the obsolete launchd templates, Ansible's
  `autolab_gateway` and `cagent_client` roles, the client setup playbook and
  the unused power webhook. `autolab_node` no longer includes the removed
  gateway role. Shared agent/observation/reconcile roles remain.
- Preserved old cagent source, request evidence, incidents, conversations
  and local CA/auth history as inactive history. The removed node-wrapper
  deployment source is kept at `cagent/history/node-client` solely for
  historical conformance; no role deploys it.

Sources: `ansible_agdev` `e74195d`; `pj-clusterintent` removal/pin `685d0cc`
and final retirement introduction `bb64e6a`. The historical TLS test points
to the preserved client source and passes (**23 cases**); Ansible
conformance passes (**4 cases**).

## Registered and actual state

The retirement batch was backed up and previewed: three updates, zero
conflicts. Omni Agent applied it under the Developer's instruction to
execute the plan; no historical classifier refusal was assumed to apply
to this action. Result: transaction committed, three updates, no errors.
The old service and agent are retired and its placement disabled.

Fresh post-stop observation of the command node succeeded. Final drift
shows the retired cagent service converged with no active placement; its
old agent is also converged. The independent observation-only LaunchAgent
remains loaded. Overall drift is **45 converged / 1 drifting**: the one
existing Observer `service_missing` condition predates this migration and
was not repaired as an incidental migration effect.

The disposable guest is absent from final drift/actual after verified
destruction and prune. No unresolved trial guest cleanup remains.

## Caller disposition and follow-ups

| Follow-up | Owner | Done when |
| --- | --- | --- |
| Replace Front's legacy cagent addressing | Front migration episode | Its guides use `wf-cagent.consult` / `wf-cagent.change` and the existing named requester configuration is demonstrated through a real request. |
| Replace legacy argue invocation | Argue host/workflow maintainer | Cluster questions open a managed consult request and return its result; the retired cagent argue role is not invoked. |
| Target locking if actual interleaving appears | Later cagent operating episode | Evidence justifies it and concurrent effects have a reviewed ownership/locking contract. |

A later-discovered legacy caller is re-pointed explicitly. No compatibility
listener or automatic forwarding was introduced. The independent refresh
continues to observe only; rollback instructions never silently restore
unattended general reconciliation.
