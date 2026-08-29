# sage p1 — step 7 report: declare the sage

Declared arXiv sage in Nautobot desired state and refreshed its observation.

## Desired records

The ignored operator input now declares:

- service `arxivsage`;
- placement `arxivsage-agstudio` with profile `agag_agent`, GitHub source,
  and **manual** management (this placement is the macOS launchd deployment);
- workspace `arxivsage-agstudio1` at the local checkout;
- agent `arxivsage-agstudio1` with Zulip user id `20` and its `agents` and
  own-channel subscriptions.

nintent's batch reference resolver cannot create a placement and reference it
from a new agent in the same batch. Applied the service/placement/workspace
batch first, then the agent batch; each transaction was atomic. `plane_user_id`
is the model-supported empty value for an intentionally undeclared Plane
identity, rather than an unrelated user's id.

## Verification

- `nctl agents observe --json` found the active Zulip user and both required
  channel subscriptions. Plane is intentionally absent.
- A reconcile dry-run showed that `nctl_managed` would run the Linux/systemd
  `agag_agent` playbook, which conflicts with this phase's launchd placement.
  Changed the placement to `management_mode: manual`; the subsequent reconcile
  plan contained only `observe_node`.
- `nctl reconcile agstudio --refresh-observation --yes` collected and ingested
  the local node observation successfully (operation `01M170G7S2E1DTFHK50TR2S1K8`).
- `nctl drift --json` now reports
  `arxivsage-agstudio1: liveness=polling`; its agent and workspace targets are
  converged. The only agent warning is the deliberate
  `agent_plane_identity_undeclared` until a future phase decides to create a
  Plane identity.
