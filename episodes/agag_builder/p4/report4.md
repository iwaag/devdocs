# Step 4 report — desired placement declaration

Completed on 2026-08-22 against the local scratch Nautobot service.

`nctl status --json` reported Nautobot reachable/authenticated, intent GraphQL
available, the worker healthy, and zero pending Jobs. A partial desired-state
batch then declared:

- active desired service `agecho`;
- active, `nctl_managed` placement `agecho-agautolab1` on `agautolab1`;
- deployment profile `agag_agent`, schema `1`;
- GitHub repository `https://github.com/iwaag/agecho.git`, branch `main`;
- Python agent/package `agecho`;
- instance name `agecho-agautolab1`;
- a dedicated workspace under the target user's home directory.

No credential source or secret was stored in Nautobot. Those remain
controller-side role inputs.

The preview planned exactly two creates and no conflicts. The committed batch
created both records atomically. Re-previewing the same batch reported two
unchanged operations, proving idempotence.

`nctl render production --out ansible_agdev/inventories/generated` completed
with all six eligible nodes included and all 22 active placements applied. The
rendered inventory placed only `agautolab1` in `agag_agents` and mapped the five
expected values: agent, instance name, repository URL, repository version, and
repository destination.
