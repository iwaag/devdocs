# Step 5 report — agautolab becomes a desired service

Status: complete (2026-08-09).

`agautolab` is now an active desired service rather than invisible
infrastructure. It has explicit HTTP port-8791 endpoints and two active
placements:

- `agautolab-agstudio`: `manual`, matching the already-running macOS gateway
  and its existing checkout.
- `agautolab-agautolab1`: `nctl_managed`, using the existing Linux
  `autolab_node` role and systemd user service.

## Implementation

The new `autolab_node` profile exposes repository URL/revision/destination
and gateway port as placement config, invokes
`playbooks/agent/setup_autolab_node.yml`, proves checkout presence, and probes
`/healthz` for managed placements. Reconciliation action planning now omits
manual placement hosts, preventing the Linux/systemd role from touching the
manually supervised agstudio gateway.

`nctl relations` also gained an active-service projection so its JSON output
shows service state and each placement's node, management mode, state, and
gap codes even when the service has no binding edges.

## Live evidence

- The five-operation batch (service, two endpoints, two placements) previewed
  with five creates and no conflicts before atomic commit. A canonical
  pre-change export remains private at
  `pj-clusterintent/.local/autolab-meets-cagent-step5-before.yaml`.
- Dry reconcile `01KZJ2NZP9HPN91PN4RBKD81F7` targeted only agautolab1.
  Apply `01KZJ2P08MMA03TWAJSZXTN6EH` converged in one round; the retained role
  was already correct and reported 15 ok, 0 changed, 0 failed/unreachable.
- Final observation operations were `01KZJ2TW22TEESZK0GJTY5Q9F4` (agstudio)
  and `01KZJ2VBEJ7S38ZHRJX1YFH1NV` (agautolab1). Direct `/healthz` requests to
  both port-8791 endpoints returned `{ "ok": true }`.
- `nctl relations --service agautolab --json` reports service state
  `converged`, with the agautolab1 `nctl_managed` and agstudio `manual`
  placements both `satisfied` and no gap codes.
- Independent repeat dry reconciles for both hosts produced zero actions.

Private captures are retained at
`pj-clusterintent/.local/autolab-meets-cagent-step5-relations.json`,
`pj-clusterintent/.local/autolab-meets-cagent-step5-drift.json`, and the two
`step5-repeat-*` files. Full nctl suite: 1,294 passed; Ansible syntax check
passed.
