# p6 ex1 — small follow-up fixes

Completed 2026-10-17. This follow-up addresses the bounded planner defect
and missing operating guidance identified in [report.md](report.md).

## Changes

- **Mixed deployment profiles (remaining limitation 3): fixed.** nctl groups
  a service's managed placements by deployment profile and plans each group
  separately. Each action retains its profile's hosts, host scope, production
  blockers and existing dependency ordering on overlapping hosts. Placement
  diffs naming a profile do not trigger another profile's deployment or
  unsupported fallback. Service-wide diffs still apply across profiles.
- **Mac local access (limitation 5): documented.** The Ansible NFS guide now
  explains that processes on the serving Mac use the backing directory,
  with ownership matching the execution identity; other hosts use NFS client
  placements. A self-mount is not offered as a normal client configuration.
  This is documentation of the observed platform behavior, not a change to
  macOS or a new self-mount validation rule.
- **Configuration changes (limitation 2): operating guidance added.** The
  guide explicitly says to reapply the setup playbook through the agreed
  change path after changing export options, clients or ownership. Existence
  checks do not prove those settings were applied. Content drift detection
  itself remains follow-up work.

The deferred Mac fstab/symlink cleanup was left as agreed. A general
ledgered privileged-command facility, NFS configuration-content observation
and an additional reverse-direction large-file trial remain outside this
small correction. No installed export, mount or desired placement changed.

## Validation and availability

- nctl ordinary suite: **1361 passed**. Regression cases cover separate
  profile actions, host-scoped planning, placement-specific diffs,
  unsupported profiles, manual-placement exclusion and same-host dependency
  ordering. The existing real drift-to-planner mixed-node test was updated
  to expect the healthy host's own unsupported profile rather than the old
  mixed-profile refusal, and continues to check isolation.
- Live read-only planning for the server host and Linux client both returned
  `ok: true`, `state: planned`, exit 0. These checks used the current cluster
  state; the mixed-profile cases are exercised by the regression tests.
- Whitespace checks passed. No Ansible execution implementation changed;
  the Ansible change is documentation only.

| Repository | Published revision |
| --- | --- |
| nctl | `3042863` |
| ansible_agdev | `675ddab` |
| pj-clusterintent pins | `7a0aa76` |

All three revisions are pushed. The deployment checkout uses these sources
directly, so the planner fix needs no service restart. A future cagent change
plan must bind its sources to the new backend revision; an existing plan's
older source agreement is not silently updated. Managed cagent's backend
gitlink remains its recorded development base until its next explicit update.
