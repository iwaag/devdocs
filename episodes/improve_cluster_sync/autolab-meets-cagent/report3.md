# Step 3 report — autolab nodes can call cagent

Status: complete (2026-08-09).

The autolab mediator now has a zero-enrollment cagent client path. The
`autolab_node` role installs the human-entrance bearer token from controller
local state with mode 0600 and installs an executable `autolab-cagent`
wrapper. The wrapper supports `submit`, `get`, `wait`, and combined `ask`,
uses the async request contract, and defaults to the self-signed experimental
TLS posture at `https://agstudio.local:8789`.

`agent/CHARTER.md` tells the mediator when to use cagent and, for a completed
resident-service mission, which non-secret facts to report: service slug,
Gitea repo URL, target, port, and run state. cagent remains responsible for
the desired-state batch.

The role's checkout source was also corrected to the actual command-node
Gitea (`autodev/agautolab.git`). The CHARTER commit was published there before
deployment, and agautolab1 pulled it successfully.

## Validation and live evidence

- agautolab unit suite: 61 passed.
- Ansible syntax check passed.
- Deployment recap: 16 ok, 5 changed, 0 failed/unreachable; gateway health
  probe and restart succeeded.
- Remote mode probe: `/home/eiji/.local/state/cagent/human_token` is `600`.
- From agautolab1, `autolab-cagent ask ...` completed a real request/poll
  cycle:
  - request `req_a78e92295fe6438aa772e4944704eddc`
  - session `ses_01bf45d90ffek3dAxsonJmiFY8`
  - terminal state `completed`
  - identity `human/operator`, as deliberately accepted by the episode
  - response was grounded in fresh `nctl drift`: agautolab1 node and compute
    converged; production `included` with no reasons.

Durable request evidence is under
`~/.local/state/cagent/evidence/req_a78e92295fe6438aa772e4944704eddc/`.
