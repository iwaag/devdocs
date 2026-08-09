# Step 4 report — static web app deployment profile

Status: complete (2026-08-09).

`static_web_app` is now a production deployment profile for dependency-free
browser projects hosted by command-node Gitea. It supports macOS and Linux,
coexisting per-instance checkout/state paths, an explicit port, repository
revision, and `run_state: started|stopped`. Its action clones or updates the
repository and controls a Python static HTTP server; the reconciliation
contract probes the declared HTTP endpoint.

## Implementation

- Production composition gained `placement_list_variable`, allowing several
  same-profile placements on one host without conflicting flat host vars.
- Profile reconciliation can derive desired run state from placement config,
  use checks on actionable profiles, and classify
  `service_should_be_stopped` as an automatic repair.
- Drift accepts both an explicit missing observation and an absent service
  entry as convergence for `run_state: stopped`; a running entry correctly
  requests a stop.
- The Ansible profile maps to OS-specific playbooks and keys checkout, PID,
  and log paths by `instance_name`. Repository URLs are restricted to the
  public `autodev` namespace on command-node Gitea.
- Desired service `snake-web`, placement `snake-web-agstudio`, and endpoint
  `http://agstudio.home.arpa:8123` were atomically declared. The endpoint
  makes the selected port visible, but no automated port-collision comparator
  exists yet.

## Live execution and evidence

The first action attempt (`01KZJ149YRYW8W8915R4WW91C8`) was interrupted
during the then-private repository clone while access was being diagnosed;
it performed no service deployment. After the repositories were made public,
dry operation `01KZJ27T9JB4SEGAHRKSWT15GG` planned only the expected profile
action.

- Start operation `01KZJ2821RRFNST2MPH0WPKH7W`: converged in one round;
  Ansible reported 7 ok, 2 changed, 0 failed/unreachable. Curl returned the
  Snake document with `<title>Snake</title>`.
- Stop dry operation `01KZJ296NSNFWZSTWDE6NX4B5W` claimed
  `service_should_be_stopped`. The action stopped the process and curl failed
  to connect to port 8123. That run exposed an absent-entry drift edge case;
  after the fix, verification operation `01KZJ2BN0MDMMF3R7Z55K1YWYY`
  contained zero actions and four converged agstudio targets.
- Restart operation `01KZJ2C2E12M210077BR8XZHMC`: converged in one round;
  Ansible reported 7 ok, 1 changed, and curl again returned the Snake page.

Private final captures are retained at
`pj-clusterintent/.local/autolab-meets-cagent-step4-final-drift.json` and
`pj-clusterintent/.local/autolab-meets-cagent-step4-stopped-drift.json`;
durable operation artifacts are under `~/.local/state/nctl/events/<operation-id>/`.

Validation:

- nctl full suite: 1,292 passed.
- Ansible Linux and macOS playbook syntax checks passed.
- Final drift: placement applied/converged; cluster summary 16 converged and
  7 unrelated unknown/stale-observation targets.
